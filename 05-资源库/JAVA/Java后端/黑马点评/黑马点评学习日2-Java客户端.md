# 黑马点评 学习日 2 —— Java 客户端

> 日期：2026-10-14（**AI 走读推演，用户未实际运行**）
> 对应讲义：入门篇 **15–22**（客户端对比 / Jedis / 连接池 / SpringDataRedis / RedisTemplate / RedisSerializer / StringRedisTemplate / Hash）
> 工程：`F:\study\黑马点评\redis-demo\`（由 `01-入门篇\代码\redis-demo.zip` 解压）
> 前置笔记：[[黑马点评学习日1-Redis基础与环境|学习日 1]] · 卡片 [[../../../../04-知识笔记/Redis五种数据类型与原子能力|Redis 五种数据类型与原子能力]]

---

## 0. 一句话总结

学习日 2 是**一个故事**：Java 连 Redis 的姿势越来越高级，每一步都在收拾前一步的麻烦。
**原生 Jedis**（连接贵、线程不安全）→ **连接池** → **SpringDataRedis + RedisTemplate**（免管理连接，但引入**序列化**新麻烦）→ **StringRedisTemplate / 手动 JSON**（换来可读与跨语言）。
**全部认知的落点只有一句：序列化层决定了你在命令行里看到什么。**

---

## 1. 依赖与配置

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-redis</artifactId>   <!-- 默认客户端是 Lettuce，不是 Jedis -->
</dependency>
<dependency>
  <groupId>org.apache.commons</groupId>
  <artifactId>commons-pool2</artifactId>                     <!-- 连接池，缺了 lettuce.pool 不生效 -->
</dependency>
<dependency>
  <groupId>com.fasterxml.jackson.core</groupId>
  <artifactId>jackson-databind</artifactId>                  <!-- 手动 JSON 序列化要用 -->
</dependency>
```

```yaml
spring:
  redis:
    host: localhost          # 讲义写 192.168.150.101，本机要改
    port: 6379
    password: 123321
    lettuce:
      pool:
        max-active: 8        # 最多 8 个连接
        max-idle: 8
        min-idle: 0
        max-wait: 100ms      # 借不到连接最多等 100ms —— 很紧，高并发易抛异常
```

> **关键事实：这个 demo 里没有任何 Jedis 代码。** `spring-boot-starter-data-redis` 默认带的是 **Lettuce**。所以讲义 15–17 的 Jedis 在本工程中**没有对应实现**，需另加依赖或另建工程。

---

## 2. 块 A · Jedis（demo 无此代码，补最小片段）

```java
// 直连
Jedis jedis = new Jedis("localhost", 6379);
jedis.auth("123321");
jedis.set("name", "虎哥");
System.out.println("name = " + jedis.get("name"));   // name = 虎哥
jedis.close();

// 连接池
JedisPool pool = new JedisPool("localhost", 6379);
try (Jedis jedis = pool.getResource()) { /* ... */ }  // 借出 / try-with-resources 自动归还
```

输出：`name = 虎哥`；Redis 里 `GET name` → `虎哥`（**裸字符串，无引号**——Jedis 直接操作字节，不做序列化包装）。

**为什么需要连接池，两个原因**：

1. **建 TCP 连接很贵**——每次 `new Jedis()` 都要三次握手，用完丢弃。
2. **Jedis 实例线程不安全**——内部只有一条连接，多线程共享会串包（典型报错 `Unexpected end of stream`）。

> 对照：**Lettuce 是线程安全的**（基于 Netty，一条连接多路复用），所以 SpringDataRedis 默认选它，不配池也能跑。

---

## 3. 块 B · RedisTemplate（`RedisDemoApplicationTests`）

### 3.1 前提：`RedisConfig` 换了序列化器

```java
GenericJackson2JsonRedisSerializer jsonRedisSerializer = new GenericJackson2JsonRedisSerializer();
template.setKeySerializer(RedisSerializer.string());
template.setHashKeySerializer(RedisSerializer.string());
template.setValueSerializer(jsonRedisSerializer);
template.setHashValueSerializer(jsonRedisSerializer);
```

**SpringBoot 自动配置的默认 `RedisTemplate`，key 和 value 都用 JDK 序列化** → Redis 里长这样：

```
127.0.0.1:6379> KEYS *
1) "\xac\xed\x00\x05t\x00\x04name"     ← 连 key 都是乱码
```

**后果**：命令行查不了、`KEYS shop:*` 前缀匹配失效、别的语言读不了。**默认的 RedisTemplate 永远别直接用。**
本 `RedisConfig` 的 bean 名与自动配置同名（`redisTemplate`）→ 自动配置退让，注入的是这个定制版。

### 3.2 `testString()` —— 记住那个引号 ⭐

```java
redisTemplate.opsForValue().set("name", "虎哥");
System.out.println("name = " + redisTemplate.opsForValue().get("name"));  // name = 虎哥
```

```redis
KEYS *     → name
GET name   → "虎哥"        ← 带引号！
```

**为什么带引号**：
- **key** 走 `RedisSerializer.string()` → 存成 `name`，干净。
- **value** 走 `GenericJackson2JsonRedisSerializer` → 编成 **JSON 字符串**，而 JSON 字符串语法**自带双引号**，所以 Redis 里躺的是 `"虎哥"` 这几个字符。
- Java 侧 `get()` 自动反序列化 → 拿回 `虎哥`，所以打印不带引号。

**同一个值：Java 眼里是 `虎哥`，Redis 眼里是 `"虎哥"`。** 这是"序列化层存在"的直接证据。

### 3.3 `testSaveUser()` —— `@class` 是什么

```java
redisTemplate.opsForValue().set("user:100", new User("虎哥", 21));
System.out.println("o = " + redisTemplate.opsForValue().get("user:100"));  // o = User(name=虎哥, age=21)
```

```redis
GET user:100
→ {"@class":"com.heima.redis.pojo.User","name":"虎哥","age":21}
```

`GenericJackson2JsonRedisSerializer` 会把**全类名写进 JSON**，这样取的时候不用你告诉它类型，它自己就能 new 出 `User`。

- 好处：存对象零成本，读回来直接是对象。
- 代价：① JSON 变胖；② 这份 JSON **只有 Java 认得**；③ 类名/包名一改，旧数据读不出来。

---

## 4. 块 C · StringRedisTemplate（`RedisStringTests`）

### 4.1 `testString()` —— 方法里有 bug，但正好是教材

```java
stringRedisTemplate.opsForValue().set("verify:phone:13600527634", "124143");  // 写手机号 key
Object name = stringRedisTemplate.opsForValue().get("name");                   // 却读 name
```

| 情况 | 输出 | 为什么 |
|---|---|---|
| 单独跑 | `name = null` | 写的和读的**不是同一个 key** |
| 块 B 的 `testString()` 先跑过 | `name = "虎哥"` | `name` 已被**块 B 的 JSON 序列化器**写过 |

**第二种才是重点**：输出**带引号**。因为 Redis 里 `name` 的值是 `"虎哥"`（含引号），而 `StringRedisTemplate` 读的是**原始字符串**，不做反序列化 → 引号一起读出来。

> **这就是"两个 template 序列化方式不一致"的现场事故**：块 B 写、块 C 读，读出畸形值。**结论：项目里统一用一套，绝不混用。**

### 4.2 `testSaveUser()` —— 手动序列化

```java
String json = mapper.writeValueAsString(user);        // 手动
stringRedisTemplate.opsForValue().set("user:200", json);
User user1 = mapper.readValue(jsonUser, User.class);  // 手动，要传类型
```

```redis
GET user:200 → {"name":"虎哥","age":21}      ← 没有 @class！
```

| | 块 B（GenericJackson2Json） | 块 C（手动 ObjectMapper） |
|---|---|---|
| JSON 里有 `@class` | **有** | **没有** |
| 存 / 取 | 全自动 | 都要自己写 |
| 可读性 | 胖、含类名 | **干净、跨语言可读** |
| 适合 | 纯 Java 内部 | 要人肉查 / 要长期存 |

**自动的代价是绑死 Java，手动的代价是多写两行代码。** 黑马点评项目后面选了手动（配 `StringRedisTemplate`），因为要能在命令行 review 数据。

### 4.3 `testHash()`

```java
stringRedisTemplate.opsForHash().put("user:400", "name", "虎哥");
stringRedisTemplate.opsForHash().put("user:400", "age", "21");   // ← 注意是字符串 "21"
System.out.println(stringRedisTemplate.opsForHash().entries("user:400"));
// entries = {name=虎哥, age=21}
```

```redis
HGETALL user:400 → name 虎哥 age 21
```

**`"21"` 必须是字符串**：`StringRedisTemplate` 的序列化器是 String 的，只能存字符串，数字要自己转。

（`name()` 是空方法，占位，不用管。）

---

## 5. 四种姿势对照表

| | Jedis | RedisTemplate（**默认**） | RedisTemplate + JSON（本 demo） | StringRedisTemplate |
|---|---|---|---|---|
| key 在命令行 | 裸字符串 | **乱码** | 裸字符串 | 裸字符串 |
| value 在命令行 | 裸字符串 | **乱码** | JSON（带 `@class`） | 裸字符串 |
| 线程安全 | ✗（要池） | ✓ | ✓ | ✓ |
| 直接存对象 | ✗ | ✓ 自动 | ✓ 自动 | ✗ 手动转 JSON |

**选择规则**：

- 要人能看懂 / 跨语言 / 项目里要 review 数据 → **`StringRedisTemplate`**（黑马点评项目用的就是它）
- 纯 Java 内部、想省事存对象 → `RedisTemplate` + `GenericJackson2JsonRedisSerializer`
- **默认 `RedisTemplate` → 永远别直接用**

---

## 6. 本日证据状态（诚实标注）

| 项 | 状态 |
|---|---|
| 环境（Redis 6.2 容器可连） | ✅ 学习日 1 用户亲手确认 |
| Java demo 导入 / 编译 / 运行 | ⏳ **未做** |
| `testString` 写入后命令行看到 `"虎哥"` 引号 | ⏳ **未做**（本日全部为 AI 走读推演） |
| Jedis / 连接池实跑 | ⏳ 未做 |

> **最小闭环建议（只需一次）**：跑块 B 的 `testString()`，然后命令行 `GET name`，**亲眼看到那个引号**。看到引号，序列化这件事就真懂了——这是本日唯一必须亲手验证的一步。

---

## 7. 下一步

学习日 3 = **导入黑马点评项目 + 短信登录**（实战篇 1 章 1.1–1.9）。
**这是本阶段第一次需要"真跑"的地方**——阶段门第 ② 项（登录真实成功 + 受保护接口调用记录）从这里出证据。
