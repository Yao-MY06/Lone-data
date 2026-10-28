---
类型: 笔记
归属: 辅助线
状态: 完结
---
# 黑马点评学习日 8 · GEO + 签到 + UV

> 对应实战篇 10–12 章 + 阶段门自检。本篇是阶段四的最后一天。

---

## 一、三样东西的共同点

**用极小的空间表示极大的数据**，代价是精度或表达能力。

| 能力 | 底层结构 | 省在哪 | 代价 |
|---|---|---|---|
| GEO（附近商铺） | ZSet（geohash 分数） | 不用存经纬度索引表 | 只有"距离排序"，没有"按类型筛" |
| BitMap（签到） | String 的位 | 一人一月 4 字节 | 只能存 0/1，且必须连续编号 |
| HyperLogLog（UV） | 特殊 String | 再大也 <16KB | 概率统计，误差 <0.81% |

---

## 二、GEO（附近商铺）

### 2.1 命令族

| 命令 | 作用 | 版本 |
|---|---|---|
| `GEOADD key 经度 纬度 member` | 加坐标点 | 3.2 |
| `GEODIST` | 两点距离 | 3.2 |
| `GEOPOS` / `GEOHASH` | 取坐标 / hash | 3.2 |
| `GEORADIUS` | 圆形范围查询 | 3.2（6.2 起废弃但可用） |
| `GEOSEARCH` | 圆/矩形范围查询 | **6.2 新增** |

### 2.2 两个设计决策

**① member 只存 shopId，不存整个对象。** Redis 是内存库，14 家店能存，140 万家就爆了。存 id，回表查详情。

**② 按 typeId 分组，一个类型一个集合。** GEO 里**没有"类型"字段**，没法按类型筛，所以 key 设计成 `shop:geo:{typeId}`：

```java
String key = RedisConstants.SHOP_GEO_KEY + typeId;   // shop:geo:1
```

### 2.3 查询实现

```java
GeoResults<RedisGeoCommands.GeoLocation<String>> results = stringRedisTemplate.opsForGeo()
        .radius(key,
                new Circle(new Point(x, y), new Distance(5000)),   // 圆心 + 半径 5km
                RedisGeoCommands.GeoRadiusCommandArgs.newGeoRadiusArgs()
                        .includeDistance().limit(end));            // 带距离、取到 end 条
```

然后 `subList(from, end)` 截出本页 → 拿 shopId → `IN` 查库 → **`ORDER BY FIELD` 还原距离顺序**。

> 🕳️ **版本坑（本机实测）**：`GeoOperations.search()` 在 **spring-data-redis 2.3.9 里不存在**（`javap` 只有 `radius`），讲义 10.3 的写法在本工程编译不过。讲义的做法是升级到 2.6.2 + lettuce 6.1.6；本项目改用 `radius`（= `GEORADIUS`），Redis 6.2 仍支持，功能等价。

### 2.4 数据导入

14 家店的坐标从 `tb_shop` 取，按 `type_id` 分组成两条 `GEOADD` 流：

```bash
mysql -uroot -p'@Love2099' -N -B -e "use hmdp; select type_id,id,x,y from tb_shop;" \
  | tr -d '\r'    # ← 关键：Windows 输出 CRLF，不带 \r 会报 "value is not a valid float"
| while IFS=$'\t' read -r t id x y; do
    docker exec redis redis-cli -a 123321 geoadd "shop:geo:$t" "$x" "$y" "$id"
  done
```

结果：`shop:geo:1` 9 个成员、`shop:geo:2` 5 个成员。

---

## 三、签到（BitMap）

### 3.1 先算账

MySQL 方案：1000 万用户 × 每人每年 10 次 = **一年 1 亿条**，一条 22 字节。

BitMap 方案：**一人一月一个键，每天一个 bit** → 一个月 4 字节。

**Key 设计**：`sign:{userId}:{yyyyMM}`，例如 `sign:1013:202610`。按月分割，跨月天然隔离，查本月只读一个 key。

### 3.2 签到

```java
int dayOfMonth = now.getDayOfMonth();
stringRedisTemplate.opsForValue().setBit(key, dayOfMonth - 1, true);   // 下标从 0 开始
```

### 3.3 连续签到：位运算（本章最难）

```java
// 1. 取出「本月到今天」的所有位，按无符号整数返回
Long num = stringRedisTemplate.opsForValue().bitField(key,
                BitFieldSubCommands.create()
                        .get(BitFieldSubCommands.BitFieldType.unsigned(dayOfMonth))
                        .valueAt(0))
        .get(0);

// 2. 从最低位（今天）往回数，遇到 0 就停
int count = 0;
while (true) {
    if ((num & 1) == 0) break;
    count++;
    num >>>= 1;
}
```

**两个关键点**：

- **为什么最低位是"今天"**：`BITFIELD GET u{dayOfMonth} 0` 从 bit 0（1 号）开始读，1 号落在**最高位**，今天落在**最低位**。
- **为什么用 `>>>` 而不是 `>>`**：`>>` 是算术右移，负数补 1，永远数不完。签到数当然非负，但 `>>>` 更稳。

**实跑数据**（2026-10-20，用户 1013）：

| 状态 | `GET sign:1013:202610` | `BITFIELD GET u20 0` | `count` |
|---|---|---|---|
| 未签到 | `(nil)` | — | 0 |
| 伪造 10-18、10-19 已签（bit 17、18） | `"\x00\x00p"` | 7 | 0（今天没签 → 正确） |
| `PUT /user/sign` 签今天（bit 19） | `"\x00\x00p"` | 7 | **3** |

`0x70` = `0111 0000` → bit 17/18/19 为 1 → 18/19/20 号连续 3 天。`BITFIELD` 返回 7 = `0b111`，与手算一致。

### 3.4 加餐：BitMap 做防穿透

把所有已存在 id 哈希到一张大位图，查询前先看那一位。**这就是布隆过滤器的雏形**——学习日 4 埋的伏笔在这里收。

---

## 四、UV 统计（HyperLogLog）

| 概念 | 含义 |
|---|---|
| **UV** | 独立访客，同一人一天多次只算一次 |
| **PV** | 页面访问量，每次访问都算 |

**难点**：要判重就得把每个访客 id 都存下来 → 上亿用户内存炸掉。

**HLL 的答案**：不存"是谁"，只估算"有多少个不同的"。

| 指标 | 数值 |
|---|---|
| 单个 HLL 内存 | **永远 < 16KB** |
| 误差 | **< 0.81%** |
| 命令 | `PFADD` / `PFCOUNT` / `PFMERGE` |

UV 统计本来就不需要精确到个位，0.81% 完全可以接受——**这是"用精度换空间"的典型取舍**。

---

## 五、阶段四收口自检

| 阶段门 | 状态 | 证据位置 |
|---|---|---|
| ① 源版本 + 源码位置 | ✅ | `hm-dianping/README.md` 第一、二节 |
| ② 登录 + 受保护接口 | ✅ | token 登录；`/user/me` 200 vs 无 token 401 |
| ③ miss / hit 可区分 | ✅ | `selectById` 计数 3→6→6 |
| ④ 秒杀可核对 | ✅ | `orderId=650766366583816193`，库存 100→99 |
| ⑤ 重复下单被拒 | ✅ | `用户已经购买过一次！` |
| ⑥ 锁 / Lua / Stream | ✅ | 锁日志 + `seckill.lua` + `XADD`/`XACK`，pending=0 |
| ⑦ Feed / GEO / 签到 | ✅ | `feed:1013`、距离 314.57m、连续 3 天 |
| ⑧ 机制与边界说明 | ✅ | README 第五节 |
| ⑨ 版本缺失证据 | ✅ | README 第三节（javap / COMMAND INFO） |
| ⑩ 架构图 + 已知问题 | ✅ | README 第四、七节 |

---

## 来源

- 讲义 `F:\study\heimadianping\02-实战篇\讲义\Redis实战篇.md` 第 10、11、12 章
- 代码：`service/impl/ShopServiceImpl.java` `queryShopByType`、`service/impl/UserServiceImpl.java` `sign` / `signCount`
