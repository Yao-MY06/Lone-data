---
类型: 笔记
归属: 辅助线
状态: 完结
---
# 黑马点评学习日 7 · 达人探店 + 好友关注（Feed）

> 对应实战篇 8 + 9 章。本篇既讲设计，也记 2026-10-20 的实跑证据。

---

## 一、这一章在讲什么

**同一个业务，数据结构选错就要返工。**

- 点赞：一开始只要"别重复点" → **Set** 够用
- 加一个"显示最早点赞 TOP5" → **Set 废了，必须换 ZSet**
- 关注：要算"共同关注" → **Set 的 `SINTER`**
- Feed：要按时间倒序 + 无限滚动 → **ZSet（score=时间戳）+ 滚动分页**

---

## 二、达人探店

### 2.1 发布与查看

两张表：`tb_blog`（笔记）+ `tb_blog_comments`（评论）。图片先走 `/upload/blog` 拿文件名，再存笔记。

> 🕳️ `SystemConstants.IMAGE_UPLOAD_DIR` 硬编码成 `D:\lesson\nginx-1.18.0\...`，不改成自己的目录，上传必失败。

### 2.2 点赞：为什么必须用 Set（讲义 8.3）

初始实现只有 `liked = liked + 1` → 一个人能点无限次。要"一人一次、再点取消"，本质是**判重**：

```java
String key = BLOG_LIKED_KEY + id;                     // blog:liked:{blogId}
Boolean isMember = stringRedisTemplate.opsForSet()
        .isMember(key, userId.toString());
if (BooleanUtil.isFalse(isMember)) {
    update().setSql("liked = liked + 1").eq("id", id).update();
    opsForSet().add(key, userId.toString());
} else {
    update().setSql("liked = liked - 1").eq("id", id).update();
    opsForSet().remove(key, userId.toString());
}
```

`SISMEMBER` 一条命令 O(1) 判断，Set 天然去重。

### 2.3 点赞排行榜：Set → ZSet（讲义 8.4）

需求变成"按点赞时间取 TOP5"。Set 无序，做不到 → 换 ZSet，**score 存点赞时间戳**：

| 操作 | Set 版 | ZSet 版 |
|---|---|---|
| 判断点过没 | `SISMEMBER` | `ZSCORE` 返回 null 即没点过 |
| 写入 | `SADD` | `ZADD key userId 时间戳` |
| 取 TOP5 | 做不到 | `ZRANGE key 0 4`（分数升序 = 最早点赞） |

**注意**：member 依然唯一，所以"一人一次"这个约束在换结构后**自动保留**——这是选 ZSet 而不是"Set + 另一个排序结构"的关键理由。

### 2.4 `IN` 查询不保序（本日最实用的坑）

```java
List<UserDTO> users = userService.query()
        .in("id", ids)                              // WHERE id IN (5,1,3)
        .last("ORDER BY FIELD(id," + idStr + ")")   // 强制按 Redis 给的顺序返回
        .list();
```

`IN` 的返回顺序**由数据库决定**，与 Redis 里的点赞/距离顺序无关。凡"先从 Redis 拿有序 ID，再按 ID 查库"的场景，**都要 `ORDER BY FIELD` 还原顺序**。Feed 分页、GEO 附近查询都用了这招。

---

## 三、关注与共同关注

`tb_follow(user_id, follow_user_id)`。关键改造：关注/取关时**同步维护 Redis 的 Set**：

```java
String key = "follows:" + userId;
isFollow ? opsForSet().add(key, followUserId) : opsForSet().remove(key, followUserId);
```

因为"共同关注"要算交集：

```java
Set<String> intersect = opsForSet().intersect("follows:" + userId, "follows:" + id);
```

一条 `SINTER`，对应 MySQL 里要写两个子查询 + 去重的活。

---

## 四、Feed 流

### 4.1 两种产品形态

| 形态 | 特点 | 例子 |
|---|---|---|
| **Timeline** | 不筛选，纯按时间倒序 | 微信朋友圈、关注流 |
| **智能排序** | 算法加权 | 抖音、小红书 |

本项目是关注流 → Timeline。

### 4.2 拉 / 推 / 推拉结合

| 模式 | 做法 | 读 | 写 | 问题 |
|---|---|---|---|---|
| **拉（读扩散）** | 读的时候查所有关注的人，合并排序 | 慢 | 快 | 关注多时读放大 |
| **推（写扩散）** | 发笔记时写进每个粉丝的收件箱 | 快 | 慢 | 粉丝多时写放大（大 V 问题） |
| **推拉结合** | 普通人推、大 V 拉 | 中 | 中 | 实现复杂 |

本项目用**推模式**（讲义 9.4）：

```java
List<Follow> follows = followService.query().eq("follow_user_id", user.getId()).list();
for (Follow follow : follows) {
    String key = FEED_KEY + follow.getUserId();        // feed:{粉丝id}
    opsForZSet().add(key, blog.getId().toString(), System.currentTimeMillis());
}
```

**key = `feed:{粉丝id}`，member = blogId，score = 发布时间戳**。一个粉丝一个 ZSet，天然按时间排好序。

### 4.3 滚动分页（本日最难）

**为什么不能用 `page/size`**：Feed 在不停插入，位置整体后移 → 第二页会重复读到第一页的尾部。

**滚动分页三参数**：

```
ZREVRANGEBYSCORE key max min LIMIT offset count
```

| 参数 | 含义 |
|---|---|
| `max`（lastId） | 上次返回的**最小时间戳**，本次从它往前取 |
| `offset` | 同一时间戳内**已取过几条**，跳过 |
| `count` | 本次取几条 |

**为什么需要 `offset`**：score 是毫秒时间戳，同一毫秒可能有多条（并发发帖）。只传 `max` 会从那一毫秒的第一条重来 → 重复。

```java
long minTime = 0; int os = 1;
for (tuple : typedTuples) {
    long time = tuple.getScore().longValue();
    if (time == minTime) os++;          // 同分，累加
    else { minTime = time; os = 1; }    // 换分，重置
}
os = minTime == max ? os : os + offset; // 没跨边界就接着上次的 offset 累加
```

最后一行的含义：本页最小时间戳**等于**上次的 `max` → 还在同一"同分块"里，`offset` 要累加；**小于** `max` → 已跨过边界，`os` 就是本页数出来的次数。

---

## 五、实跑证据（2026-10-20）

**准备**：三个用户 —— A(1012)、B(1013)、C(1014)。

| 步骤 | 请求 | 结果 |
|---|---|---|
| B 关注 A | `PUT /follow/1012/true` | `{"success":true}` |
| 库 / Redis 双向核对 | `tb_follow` = `(1013,1012)`；`SMEMBERS follows:1013` = `1012` | ✅ 双写一致 |
| A 发笔记 | `POST /blog` | `{"success":true,"data":23}` |
| 推送落点 | `ZRANGE feed:1013 0 -1 WITHSCORES` | `23` → `1792485452702` |
| B 读收件箱 | `GET /blog/of/follow` | `{"list":[{"id":23,...,"title":"Feed 实跑测试笔记"}],"minTime":1792485452702,"offset":1}` |
| 共同关注 | A、B 都关注 C(1014) | `SINTER follows:1012 follows:1013` → `1014`；接口返回 `[{"id":1014,...}]` |
| 反例 | A、B 互相关注后再算交集 | 返回 `[]` —— 交集为空是正确的，说明不是"随便返回点什么" |

---

## 六、三个必须记住的坑

1. `IN` 查询不保序 → 必须 `ORDER BY FIELD(id, ...)`
2. Feed 不能用 `page/size`（数据在插入，位置会移）→ 用时间戳滚动
3. 同一毫秒多条 → 分页参数必须带 `offset`

---

## 七、边界（阶段门 ⑧ 要求说清的）

- **推模式写放大**：粉丝 10 万就是 10 万次 `ZADD`。真实系统要"普通用户推、大 V 拉"。
- **收件箱是无限增长的 ZSet**：需要一个截断策略（只留最近 N 条），否则内存一路涨。
- **取关不清理收件箱**：取关后旧笔记仍留在 `feed:{userId}` 里，属于"历史遗留"，可接受但要知情。
- **本项目未做的**：点赞排行榜（`likeBlog` 仍是 `liked+1`）、共同关注的前端页面。

---

## 来源

- 讲义 `F:\study\heimadianping\02-实战篇\讲义\Redis实战篇.md` 第 8、9 章
- 代码：`service/impl/BlogServiceImpl.java`、`service/impl/FollowServiceImpl.java`、`controller/FollowController.java`
