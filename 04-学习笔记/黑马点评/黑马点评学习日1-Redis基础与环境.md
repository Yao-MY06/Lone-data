---
类型: 笔记
归属: 辅助线
状态: 完结
---
# 黑马点评 学习日 1 —— Redis 基础与环境

> 日期：2026-10-14（首次 AI 代跑为 10-13，已作废；本日为**用户亲手重做**）
> 对应讲义：入门篇 **01–14**（`F:\study\heimadianping\01-入门篇\讲义\Redis注释版\01.快速入门.md`）
> 环境：Docker `redis:6.2` 容器
> 卡片版速览：[[Redis五种数据类型与原子能力|Redis 五种数据类型与原子能力]]

---

## 0. 一句话总结

学习日 1 只做两件事：**让 Redis 跑起来**、**把命令当成业务原语来认识**。核心认知是——五种数据类型不是"存什么"的区别，而是"**哪些操作能一条命令原子完成**"的区别；后面 7 天的每一个实战问题，都是在这五副模具里挑一副。

---

## 1. 环境（用户亲手跑通）

```bash
docker start redis
docker ps        # redis | redis:6.2 | Up | 0.0.0.0:6379->6379/tcp
docker exec -it redis redis-cli -a 123321
127.0.0.1:6379> ping
PONG
```

容器参数（首次创建时用）：

```bash
docker run -d --name redis -p 6379:6379 -v redis-data:/data redis:6.2 redis-server --requirepass 123321
```

**为什么必须是 6.2**：苍穹自带 `Redis-x64-3.2.100` **没有 Stream**，学习日 6 的消息队列过不了。课程假定 CentOS 7 + `yum/make`，Windows 不适用，故走 Docker。

### 踩坑：PowerShell 里 `docker exec -it` 挂不住

| 现象 | 根因 | 解法 |
|---|---|---|
| 命令敲下去出现 `>>`，随后 `ping` 打出的是 Windows 网络 ping 的用法帮助 | 在 **PowerShell** 里跑 `docker exec -it`，交互终端没建起来；`>>` 是 PowerShell 的续行提示符，`ping` 被它抢去执行了 `Ping.exe` | ① 先 `cmd` 切到 CMD 再执行；或 ② 用**非交互一次性**格式 `docker exec redis redis-cli -a 123321 <命令>`（不带 `-it`） |

**教训**：Windows 上给 Redis 命令，优先给非交互格式，或先声明"在 CMD 里跑"。

---

## 2. 通用命令（用户亲手，讲义 08）

只有 6 个：`KEYS / DEL / EXISTS / EXPIRE / TTL / PERSIST`。实测确认的结论：

| 命令 | 实测结论 |
|---|---|
| `EXISTS k1 k2` | 返回**存在键的个数**，不是 true/false。`EXISTS num msg` → `2`；`EXISTS num num` → `2`（同键写两遍也重复计数） |
| `TTL` | **三种返回值**：正数 = 剩余秒数；`-1` = 键存在但**永不过期**；`-2` = **键不存在**。`SET name jack` 后立刻 `TTL name` → `-1`，说明 **SET 默认不设过期** |
| `PERSIST` | 取消过期，之后 `TTL` 变回 `-1` |
| `KEYS *` | **生产禁用**——全库扫描会阻塞整个 Redis（单线程） |

---

## 3. 五大数据类型（讲义 09、11–14）

> ⚠️ 本节的**预期输出由 AI 讲解**（用户未逐条亲手跑），属于**待补证**项。用户已亲手确认的只有块 1 通用命令部分。

### String（09）

```
SET num 10 → INCR → 11 → INCRBY 5 → 16 → DECR → 15
SETNX num 100 → 0（已存在，且原值 15 未被改）
SETNX lock 1  → 1（设进去了）
MGET k1 k2 nokey → "v1" "v2" (nil)   ← 不存在的键返回 nil，不报错
```

**`SETNX` = SET if Not eXists，返回 0 表示没动原值** —— 分布式锁的基石（学习日 5）。

### 类型动态判断（本日最值钱的坑）

```
SET n 10 → INCRBYFLOAT n 0.5 → "10.5" → INCR n
→ (error) ERR value is not an integer or out of range
```

Redis 里所有值本质是字符串，`INCR` 系会**尝试解析成整数**：`"10"` 行，`"10.5"` 不行。**类型是按值内容动态判断的**，与 Java 强类型完全不同。

### Hash（11）

`HSET user:1 name jack age 21` → `2`（新增了 2 个 field）；`HINCRBY user:1 age 1` → `22`。
**自增对象是 field 而非整个 key**，所以能只改局部、不用整体读改写。

### List（12）

`RPUSH` 从**尾**进、`LPUSH` 从**头**进；`LRANGE key 0 -1` 的 `-1` = 最后一个元素；`LPOP` 删头、`RPOP` 删尾。
两头都能进出 ⇒ 既能当栈也能当队列。

### Set（13）

`SADD s1 a b c c` → **3**（末尾重复的 c 被静默丢弃，"元素不重复"的实证）。
`SINTER` 交集 / `SDIFF` 差集（前者有后者无）/ `SUNION` 并集去重。
**讲义"共同好友"练习题答案 = `SINTER`**；学习日 7 的"共同关注"是同一个命令。

### SortedSet（14）

`ZADD rank 10 a 7 b 3 c` → `ZRANGE` 得 c(3) b(7) a(10)（**默认分数从低到高**），`ZREVRANGE` 反序（排行榜用它）。
`ZRANK rank a` → `2`（**从 0 开始**）；`ZCOUNT rank 5 10` → `2`（闭区间，含 b=7 与 a=10）。

---

## 4. 讲义外发现（AI 代跑时实测，用户待复现）

用 `OBJECT ENCODING` 实测：**底层编码不是固定的**，会随数据量在压缩结构与"教科书结构"之间切换：

| 结构 | 触发条件 | 编码变化 |
|---|---|---|
| ZSet | 成员名 > 64 字节 | `ziplist` → `skiplist` |
| ZSet | 元素数 > 128 | `ziplist` → `skiplist` |
| Hash | 字段数 > 512 | `ziplist` → `hashtable` |
| Set | 加入非整数元素 | `intset` → `hashtable` |

阈值来自 `zset-max-ziplist-entries=128` / `zset-max-ziplist-value=64` / `hash-max-ziplist-entries=512`。
**小数据量用压缩结构省内存，超阈值才换成教科书结构** —— 这是原理篇（ZipList / SkipList / intset / Dict）的入口，**本阶段不展开**，归 1–2 月八股。

---

## 5. 本日证据状态（诚实标注）

| 项 | 状态 |
|---|---|
| 环境跑通（容器 Up、6379 监听、`ping → PONG`） | ✅ **用户亲手** |
| 通用命令（`EXISTS` 计数、`TTL` 三值、`PERSIST`） | ✅ **用户亲手**（实测输出已确认） |
| 五大数据类型调用的**真实终端输出** | ⏳ **待补**（本节由 AI 讲解；用户尚未逐条亲手跑） |
| 讲义两道练习题（共同好友 / 成绩榜） | ⏳ **待补** |
| 编码拐点复现 | ⏳ 待补（AI 代跑所得，用户需自己 `OBJECT ENCODING` 验证一次） |

> **记录规则**：「跑通 / 手跑 / 实测」只用于**用户亲手执行**的动作；AI 执行的写「AI 代跑」。阶段门证据不得由 AI 代跑充当。

---

## 6. 下一步

学习日 2 = **Java 客户端**（入门篇 15–22）：客户端对比 → Jedis → 连接池 → SpringDataRedis → RedisTemplate → RedisSerializer → StringRedisTemplate → 操作 Hash。
产出：一个能连上 Redis 的 Java demo，Jedis 与 RedisTemplate **各一次真实读写**。
