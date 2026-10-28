---
类型: 笔记
归属: 辅助线
状态: 完结
---
# 黑马点评 学习日 6 —— Redisson + 异步秒杀 + 消息队列

> 日期：2026-10-19（**AI 走读；后端由 AI 实现并 API 验证，浏览器端登录未走通**）
> 对应讲义：实战篇 **5 章 + 6 章 + 7 章**
> 前置：[[黑马点评学习日5-优惠券秒杀与分布式锁|学习日 5]]（埋下"锁续期"钩子）

---

## 0. 一句话总结

**Redisson 用 Hash 解决"可重入"、用看门狗解决"锁不住"、用 MultiLock 解决"主从丢锁"。**
**异步秒杀的思路是把"资格判断"压进一个 Lua（毫秒级），把"建单"推给队列（慢慢做）。**
**队列方案的选择，本质是在"丢不丢消息"和"实现复杂度"之间选。**

---

## 1. 块 A · Redisson（5 章）

### 1.1 setnx 自研锁的四个不足（1.1）

| 不足 | 含义 |
|---|---|
| **不可重入** | 同一线程再次进同一段锁代码 → 自己锁死自己 |
| **不可重试** | `tryLock` 只试一次就返回 false |
| **超时释放** | 业务超时 → 锁提前释放 → **锁不住**（学习日 5 第 ④ 步遗留） |
| **主从一致性** | 主节点写完锁未同步就从节点升主 → 锁丢失 |

### 1.2 可重入：锁存成 Hash

```
KEY   lock:order:1     （Hash）
FIELD uuid:线程id   →   VALUE 1（重入次数）
```

- 第一次加锁：`HSET` → value = 1
- 同一线程再进：`HINCRBY` → value = 2（**这就是可重入**）
- 释放一次 -1，减到 0 才真正 `DEL` → 不会自己锁死自己

### 1.3 抢锁 Lua 三分支（关键在返回值）

```lua
-- KEYS[1]=锁名  ARGV[1]=失效时间  ARGV[2]="uuid:线程id"
if (redis.call('exists', KEYS[1]) == 0) then        -- ① 锁不存在
    redis.call('hset', KEYS[1], ARGV[2], 1);
    redis.call('pexpire', KEYS[1], ARGV[1]);
    return nil;                                      -- nil = 抢锁成功
end;
if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then  -- ② 锁在且是我的
    redis.call('hincrby', KEYS[1], ARGV[2], 1);
    redis.call('pexpire', KEYS[1], ARGV[1]);
    return nil;                                      -- nil = 重入成功
end;
return redis.call('pttl', KEYS[1]);                  -- ③ 被别人占着，返回剩余毫秒
```

**为什么 `nil` 代表成功**：上层源码 `if (ttl == null) return;`。返回数字（pttl）说明没抢到，**而且知道还要等多久**，于是 `while(true)` 自旋。

### 1.4 WatchDog 看门狗（5.4）

**两个重载决定行为**：
- `tryLock(1, 10, SECONDS)` —— 传了 leaseTime，锁 10 秒，**不续期**
- `tryLock()` —— 不传，用**默认 30 秒**并**开启看门狗**

续期机制：**每「锁时长 ÷ 3」续一次**（30s 锁 → 每 10s），续完递归重设定时任务。

> 为什么是 ÷3 而不是"到期前续"：留出**两次失败重试余量**，网络抖一下也不至于锁过期。

**线程宕机 → 没人续期 → 30s 后自然释放**，所以不会永久死锁。

### 1.5 MultiLock（5.5）

解决"主从一致性"：不用主从，而是**多个独立节点各加一把锁**，全部成功才算成功。
总等待时间 = `锁个数 × 1500ms`（3 节点 = 4500ms），失败则重试。

```java
RLock lock1 = redissonClient.getLock("lock1");
RLock lock2 = redissonClient.getLock("lock2");
RLock lock = redissonClient.getMultiLock(lock1, lock2);
```

---

## 2. 块 B · 异步秒杀（6 章）

### 2.1 为什么慢

原流程六步串行：查券 → 判库存 → 查订单 → 校验一人一单 → 扣库存 → 建订单。
**一个线程串行 + 多次访问数据库。**

### 2.2 思路

> 讲义原话：**"我并不需要他一口气做完这件事，我只要确定他能做这件事，然后我后面慢慢做就可以了。"**

把**能快速判断的**（库存、一人一单）搬进 Redis，用 **Lua 保证原子**；通过就**立即返回订单号**，建单丢给后台。

### 2.3 Lua 脚本（三个返回值）

```lua
local stockKey = 'seckill:stock:' .. voucherId
local orderKey = 'seckill:order:' .. voucherId

if (tonumber(redis.call('get', stockKey)) <= 0) then return 1 end        -- 库存不足
if (redis.call('sismember', orderKey, userId) == 1) then return 2 end    -- 重复下单

redis.call('incrby', stockKey, -1)
redis.call('sadd', orderKey, userId)
redis.call('xadd', 'stream.orders', '*', 'userId', userId, 'voucherId', voucherId, 'id', orderId)
return 0
```

**用到的数据结构全是学习日 1 学的**：库存 String（`INCRBY -1`）、一人一单 **Set**（`SISMEMBER`/`SADD`）、发消息 Stream（`XADD`）。

**"扣库存 + 记用户 + 发消息"都在同一个 Lua 里** → 要么全成要么全不成。这是异步方案成立的前提：**只要返回 0，就一定欠用户一张订单。**

### 2.4 阻塞队列的两个问题（6.3 结尾）

1. **内存限制** —— `ArrayBlockingQueue` 在 JVM 堆里，塞满就丢。
2. **数据安全** —— 服务宕机，队列里的订单全没。

→ 换 **Redis Stream**：消息存 Redis，不占 JVM 内存，不怕重启。

---

## 3. 块 C · 消息队列四方案（7 章）

| 方案 | 优点 | 缺点 |
|---|---|---|
| **List**（`LPUSH`+`BRPOP`） | 不受 JVM 限制、可持久化、有序 | **无法避免消息丢失**、只支持**单消费者** |
| **PubSub** | 多生产多消费 | **不持久化**、丢消息、堆积有上限 |
| **Stream**（`XREAD`） | 可回溯、多消费者、可阻塞 | **会漏读** |
| **Stream 消费者组** | 不漏读、有 ACK、多消费者争抢 | 实现稍复杂 |

### 3.1 两个容易踩的点

**① `XREAD` 为什么漏读**：起始 ID `$` = 只读**最新**。处理一条期间来了 3 条，下次只给你最新 1 条，中间 2 条**永久丢失**。

**② 消费者组的 `>` 和 `0` 是两种语义**

| ID | 含义 |
|---|---|
| `>` | 从**下一个未被消费**的消息开始（正常消费） |
| `0` | 从 **pending-list**（已投递但未 ACK）第一条开始（异常兜底） |

所以 7.6 是两个方法：

```java
// 正常消费
read(Consumer.from("g1","c1"),
     StreamReadOptions.empty().count(1).block(Duration.ofSeconds(2)),
     StreamOffset.create("stream.orders", ReadOffset.lastConsumed()));
...
// 必须 ACK，否则永远留在 pending-list
stringRedisTemplate.opsForStream().acknowledge("stream.orders", "g1", record.getId());

// 异常兜底
... StreamOffset.create("stream.orders", ReadOffset.from("0"))
```

**"至少消费一次"就是这么实现的**：处理失败的不会丢，下次从 pending-list 捞回来。

> 🕳️ **讲义 bug**：`acknowledge("s1", ...)` 里的 `"s1"` 是 **key 名**，但队列实际叫 `stream.orders` —— 真跑会 ACK 到不存在的 key，消息永远留在 pending-list 反复重试。
> 另一个：6.2 的 `seckillVoucher` 里 `orderId` **重复声明两次**，编译不过。

---

## 4. 本日证据状态（诚实标注）

| 项 | 状态 |
|---|---|
| Redisson / 异步秒杀 / 消息队列（认知） | ✅ AI 走读完成 |
| Redis + MySQL + 后端 + nginx 全链路 | ✅ **已打通**（AI 代做，详见 [[../../.workbuddy/memory/2026-10-19|2026-10-19 日志]]） |
| 短信登录后端实现 | ✅ AI 代写，API 层验证通过（发码→登录→token→401） |
| **浏览器端登录** | ❌ **未走通** —— 登录页协议勾选框前端 bug（`label for` 未匹配 `id`）+ 浏览器缓存，`POST /api/user/login` 始终未发出。用户叫停修复。 |
| **阶段门 ②**（登录真实成功 + 受保护接口调用） | ⏳ **未取** |

**待补**：换浏览器/清缓存/硬刷新后重试登录；或直接进学习日 7（达人探店 + Feed），环境已具备。

---

## 5. 下一步

学习日 7 = **达人探店 + 好友关注（Feed 流）**（实战篇 8 + 9 章）：发布/查看探店笔记 → 点赞（Set）→ 点赞排行榜（ZSet）→ 关注取关 → 共同关注（SINTER）→ **Feed 推拉分析** → 推送到粉丝收件箱（ZSet）→ 滚动分页。
