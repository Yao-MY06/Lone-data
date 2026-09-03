# Spring Boot 技术学习手册

> 主线版本：以 Spring Boot 3.x / Spring Framework 6.x 为主，兼顾 2.x 的常见差异。3.x 起基线迁移到 Java 17+ 与 Jakarta EE 命名空间（`jakarta.*`），2.x 多为 Java 8/11 与 `javax.*`。具体版本与兼容性以官方文档为准。

---

## 0. 学习方法与总体地图

### 0.1 先建立三条主线

1. **装配主线**：Bean 如何被创建、注入、配置、条件化生效。核心词：IoC/DI、自动配置、Starter、`@ConfigurationProperties`、Profile。
2. **请求主线**：HTTP 请求如何进入应用、被校验、被处理、被包装成响应。核心词：嵌入式容器、DispatcherServlet、Controller、Validation、异常处理、拦截器、过滤器。
3. **运行主线**：应用如何打包、启动、观测、调参、排错、上线。核心词：Actuator、日志、指标、链路追踪、外部化配置、容器化、Graceful Shutdown。

### 0.2 建议学习顺序

1. Java 基础与 Maven/Gradle 依赖管理。
2. Spring 核心：IoC/DI、Bean 生命周期、AOP、事件、资源与环境抽象。
3. Spring Boot 本质：自动配置、Starter、条件装配、外部化配置。
4. Web：REST、参数绑定、校验、统一响应、异常处理、跨域、文件上传下载。
5. 数据：连接池、JDBC、JPA/Hibernate、MyBatis、事务、分页、审计、迁移。
6. 生产化：Actuator、Micrometer、日志、Docker、配置中心、健康检查、性能与排错。
7. 进阶：安全、缓存、消息、任务调度、WebFlux、GraalVM、云原生与微服务。

### 0.3 练习项目路线

- 入门：Todo REST API（CRUD + Validation + 全局异常）。
- 进阶：库存/订单服务（事务、乐观锁、分页、审计、Flyway）。
- 生产：接入 Actuator + Prometheus 指标、结构化日志、Docker Compose、健康检查、优雅停机。
- 架构：拆成两个服务，用 OpenFeign/RestClient、消息队列、Resilience4j 做超时/重试/熔断。

---

## 1. Spring Boot 到底是什么

### 1.1 涵义

Spring Boot 是建立在 Spring Framework 之上的“约定优于配置”的应用启动与装配框架。它不是替代 Spring，而是把 Spring 应用中重复、样板、易错的搭建工作产品化：依赖版本管理、自动配置、嵌入式服务器、生产监控端点、外部化配置。

### 1.2 作用与意义

- **降低搭架子成本**：不用从零拼接 XML、扫描路径、Servlet 容器、版本兼容矩阵。
- **把最佳实践固化为默认值**：合理默认可快速启动，需要时再覆盖。
- **让环境差异显式化**：通过 profile、环境变量、配置中心把“同一份产物，不同环境参数”变成标准做法。
- **让可运维性内建**：健康检查、指标、信息端点不是事后外挂，而是框架一等公民。

### 1.3 常见误解

- 误解一：Spring Boot 等于“零配置”。更准确是“少配置 + 可覆盖”；复杂系统仍需清晰配置。
- 误解二：自动配置是魔法。它本质是可读、可调试、可条件化的 Bean 装配逻辑。
- 误解三：Spring Boot 只做 Web。它同样适合批处理、CLI、消息消费、定时任务、集成服务。

---

## 2. Spring 核心概念地基

## 2.1 IoC 与 DI

### 涵义

- IoC（Inversion of Control，控制反转）：对象不再自行创建并管理依赖，而把创建、装配、生命周期交给容器。
- DI（Dependency Injection，依赖注入）：容器把依赖按构造器、Setter 或字段注入到对象中。现代 Spring 推荐构造器注入。

### 作用与意义

- **解耦**：业务类依赖接口而非具体实现，替换实现不改调用方。
- **可测试**：依赖可从外部传入替身，单测不必启动真实数据库或 HTTP 服务。
- **一致性**：对象生命周期、作用域、代理、后处理器统一由容器管理。

### 怎么用

```java
@Service
public class OrderService {
    private final OrderRepository repo;
    private final PaymentClient payment;

    public OrderService(OrderRepository repo, PaymentClient payment) {
        this.repo = repo;
        this.payment = payment;
    }
}
```

### 常见坑

- 字段注入让依赖隐藏、难以测试；构造器注入更显式，也便于 `final`。
- 构造器注入出现循环依赖时，优先审视职责边界，而不是立刻用 `@Lazy` 或 Setter 掩盖设计问题。

## 2.2 Bean、作用域与生命周期

### 涵义

Bean 是由 Spring 容器创建、装配并管理的对象。作用域决定实例数量与存活范围，常见有 singleton、prototype、request、session。生命周期包括实例化、依赖注入、初始化回调、使用、销毁回调。

### 作用与意义

- **资源共享**：singleton 适合无状态服务，减少创建开销。
- **边界清晰**：request/session 作用域把 Web 上下文隔离交给代理处理。
- **可扩展**：`BeanPostProcessor`、`InitializingBean`、`@PostConstruct` 等钩子让框架和插件能统一增强 Bean。

### 常见坑

- singleton 中保存请求级状态会串数据；应使用方法参数、ThreadLocal 谨慎封装或 request 作用域代理。
- `@PostConstruct` 中访问尚未就绪的外部资源要谨慎；重资源初始化可用 `SmartLifecycle` 或应用事件分阶段启动。

## 2.3 AOP

### 涵义

AOP（面向切面编程）把横跨多个模块的关注点，如日志、事务、安全、限流、缓存，从业务代码中抽离，在连接点周围织入增强逻辑。

### 作用与意义

- **横切逻辑集中**：避免每个方法复制 try/catch、计时、鉴权代码。
- **业务纯净**：服务方法聚焦业务规则，非功能性需求由切面统一表达。
- **一致策略**：同一套事务边界、审计字段、指标埋点可全局执行。

### 怎么用

```java
@Aspect
@Component
public class TimingAspect {
    @Around("@annotation(timed)")
    public Object around(ProceedingJoinPoint pjp, Timed timed) throws Throwable {
        long start = System.nanoTime();
        try { return pjp.proceed(); }
        finally { /* 记录耗时 */ }
    }
}
```

### 常见坑

- 同类内部自调用不触发代理；把需要增强的方法放到另一个 Bean，或注入自身代理。
- 切面顺序影响结果；用 `@Order` 明确事务、缓存、日志、重试的先后关系。

## 2.4 Environment 与外部化配置

### 涵义

`Environment` 抽象了属性源与 profile。属性可来自 `application.yml`、环境变量、JVM 参数、命令行、配置中心等，按优先级覆盖。

### 作用与意义

- **十二要素应用**：构建一次，处处运行；环境差异通过外部配置注入。
- **安全分层**：密码不进仓库，使用环境变量、密钥管理或配置中心加密。
- **行为可切换**：通过 profile 在 dev/test/prod 间切换数据源、日志级别、功能开关。

### 怎么用

```yaml
server:
  port: 8080
spring:
  profiles:
    active: ${APP_PROFILE:dev}
```

```java
@ConfigurationProperties(prefix = "app.mail")
public record MailProps(String host, int port, boolean enabled) {}
```

### 常见坑

- `@Value` 散落各处导致配置不可追踪；强类型 `@ConfigurationProperties` 可集中校验、生成元数据。
- profile 不是权限边界；生产敏感参数仍需密钥管理和最小权限。

---

## 3. Spring Boot 的核心机制

## 3.1 自动配置 Auto-configuration

### 涵义

自动配置是 Spring Boot 根据 classpath、已有 Bean、属性与环境条件，决定是否注册某组默认 Bean 的机制。核心注解是 `@ConditionalOnClass`、`@ConditionalOnMissingBean`、`@ConditionalOnProperty` 等。

### 作用与意义

- **按需装配**：有 `DataSource` 才尝试配 JPA，有 `spring-webmvc` 才注册 MVC 基础设施。
- **用户优先**：你显式定义的 Bean 通常覆盖默认 Bean，默认只补缺。
- **可解释**：`--debug` 或 Actuator 的 `conditions` 报告能说明每个配置为何生效/未生效。

### 怎么理解源码路径

启动类 `@SpringBootApplication` 组合了 `@Configuration`、`@EnableAutoConfiguration`、`@ComponentScan`。`@EnableAutoConfiguration` 通过 `AutoConfigurationImportSelector` 读取 `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`（3.x；2.x 为 `spring.factories`），再按条件过滤。

### 常见坑

- 扫描范围错误导致 Bean 找不到：启动类应放在业务根包上层。
- 引入 starter 后默认 Bean 与你手写 Bean 冲突：优先保留业务显式 Bean，删除/排除不需要的自动配置。

## 3.2 Starter

### 涵义

Starter 是一组依赖与自动配置的打包入口，例如 `spring-boot-starter-web`、`spring-boot-starter-data-jpa`。它解决“我需要这个功能，该引入哪些兼容版本”的问题。

### 作用与意义

- **依赖收敛**：父 BOM 管理版本，避免自己拼版本导致兼容性问题。
- **意图清晰**：`starter-web` 表示要构建 servlet Web 应用；`starter-webflux` 表示响应式栈。
- **减少知识门槛**：新手可从功能名入手，不必背完整技术栈清单。

### 常见坑

- 同时引入 Web 与 WebFlux 可能造成运行栈不确定；明确选择一种。
- 不需要的功能不要乱引 starter；每个 starter 都可能带来默认 Bean、端点与攻击面。

## 3.3 嵌入式容器

### 涵义

传统 WAR 部署把应用交给外部 Tomcat；Spring Boot 默认把 Tomcat/Jetty/Undertow 或 Netty 嵌入应用，产物是可执行 JAR，`main` 方法启动容器。

### 作用与意义

- **交付单元统一**：应用与服务器同版本、同生命周期，适合容器镜像。
- **环境一致性**：开发、测试、生产不再依赖外部容器安装细节。
- **云原生友好**：进程模型简单，配合健康检查、优雅停机、资源限制更直接。

### 常见坑

- 需要部署到传统应用服务器时才打 WAR，并注意初始化器；不要默认沿用老旧 WAR 思维。
- 容器线程数、连接数、超时不是越大越好；应与下游容量和隔离策略联动。

## 3.4 Actuator 与生产就绪

### 涵义

Actuator 提供生产端点：`health`、`info`、`metrics`、`env`、`beans`、`conditions`、`loggers`、`threaddump`、`heapdump` 等，配合 Micrometer 输出指标到 Prometheus 等系统。

### 作用与意义

- **可观测性内建**：健康检查给编排系统用，指标给告警与容量分析用。
- **运行期诊断**：查看生效配置、Bean 定义、自动配置报告、日志级别动态调整。
- **风险也可控**：端点是能力也是攻击面，必须认证、授权、网络隔离、按需暴露。

### 常见坑

- 生产环境不要暴露所有端点到公网；最少暴露 `health` 与必要 `metrics`。
- `heapdump`/`threaddump` 可能泄露敏感信息或造成停顿；限制访问并谨慎触发。

---

## 4. Web 开发：请求如何穿过应用

## 4.1 DispatcherServlet 与请求管线

### 涵义

在 servlet 栈中，请求先进入嵌入式容器，再交给 `DispatcherServlet`。它按 HandlerMapping 找 Controller，经 HandlerAdapter 调用方法，参数由 Resolver 绑定，返回值由 HttpMessageConverter 序列化。

### 作用与意义

- **统一入口**：过滤器、拦截器、跨域、内容协商、异常解析有固定挂点。
- **可替换**：视图、消息转换器、参数解析器都可扩展。
- **调试有抓手**：理解管线后，404、415、400、406 能快速定位到映射、媒体类型、绑定或转换。

### 常见坑

- 404 不一定是路径错：可能是包扫描、HTTP 方法、consumes/produces、静态资源映射或 context-path。
- 日期、枚举、自定义对象绑定失败，优先注册 Converter/Formatter 或使用明确 DTO。

## 4.2 Controller、DTO 与统一响应

### 涵义

Controller 负责把 HTTP 语义映射到应用用例；DTO 是对外契约；统一响应包装让前端处理错误、分页、追踪 ID 有稳定结构。

### 作用与意义

- **边界防腐**：领域模型不直接暴露给外部，避免持久化字段、内部状态泄露。
- **契约演进**：API 版本、字段废弃、兼容策略集中在 DTO/文档层。
- **前端确定性**：错误码、traceId、分页元数据稳定，降低联调成本。

### 建议

```java
@PostMapping("/orders")
public ApiResult<OrderView> create(@Valid @RequestBody CreateOrder cmd) {
    return ApiResult.ok(service.create(cmd));
}
```

### 常见坑

- 把 JPA Entity 直接 `@RequestBody`/`@ResponseBody`：容易触发懒加载、循环引用、批量更新与安全问题。
- 统一响应不要吞掉 HTTP 语义；鉴权失败、资源不存在、参数错误仍应使用正确状态码。

## 4.3 Validation

### 涵义

Bean Validation 用注解声明约束（`@NotNull`、`@Size`、`@Email`），Spring 在参数绑定后触发校验，并可用分组校验区分创建/更新场景。

### 作用与意义

- **脏数据早失败**：在进入业务与数据库前阻断非法输入。
- **规则集中可读**：约束写在 DTO 上，配合全局异常返回清晰错误。
- **不替代领域规则**：格式校验交给 Validation，业务一致性交给领域服务/数据库约束。

### 常见坑

- 更新接口全量字段非空会误伤部分更新；用分组或专用 UpdateDTO。
- 校验注解只防输入，不防并发与唯一约束冲突；唯一性仍需数据库约束兜底。

## 4.4 全局异常处理

### 涵义

`@RestControllerAdvice` 集中处理 Controller 层抛出的异常，映射为状态码与错误体；`ProblemDetail`（RFC 7807）在 Spring 6 中是一等支持。

### 作用与意义

- **一致错误模型**：前端不必解析每个接口自创格式。
- **避免泄露**：内部堆栈不直接外抛，生产只返回可诊断但不过度暴露的信息。
- **可追踪**：错误体携带 traceId，日志与响应能串起来。

### 常见坑

- 不要捕获一切返回 200；这会破坏缓存、重试、网关与监控语义。
- 业务异常与系统异常分开：业务冲突可预期并提示，系统异常要告警并保护细节。

## 4.5 Filter、Interceptor、AOP 的选择

- Filter：Servlet 层，适合编码、压缩、CORS、粗粒度安全、请求包装；拿不到 Spring MVC 的 handler 细节。
- Interceptor：MVC 层，能看到 handler，适合登录态、权限上下文、审计、API 计时。
- AOP：方法层，适合事务、缓存、领域事件旁路、服务层指标与重试。

选择原则：越靠近传输层用 Filter，越靠近 Web 用 Interceptor，越靠近业务方法用 AOP。三者不是互斥，而是职责不同。

---

## 5. 数据访问与事务

## 5.1 DataSource 与连接池

### 涵义

`DataSource` 是获取数据库连接的工厂；生产通常使用 HikariCP 连接池。池化复用连接，控制最大并发、获取超时、生命周期校验。

### 作用与意义

- **限流与稳定**：池大小是数据库并发入口的保险丝。
- **快速失败**：获取超时比线程无限等待更能保护系统。
- **可观测**：池活跃/空闲/等待指标是慢 SQL 与容量问题的重要信号。

### 常见坑

- 池大小不是越大越好；过大压垮数据库，过小造成请求排队。
- 长事务占住连接会耗尽池；把外部调用、文件 IO 移出事务边界。

## 5.2 Spring Data JPA 与 Hibernate

### 涵义

JPA 是 Java 持久化规范，Hibernate 是实现；Spring Data JPA 用 Repository 抽象减少 CRUD 样板，并通过方法名、JPQL、Specification、EntityGraph 表达查询。

### 作用与意义

- **对象-关系映射**：用领域对象表达模型，框架处理大部分 SQL 生成。
- **快速迭代**：派生查询与审计字段减少重复代码。
- **可优化但需理解**：N+1、懒加载、脏检查、一级缓存都可控，但不能无知使用。

### 常见坑

- N+1：列表遍历触发懒加载；用 EntityGraph、join fetch、DTO 投影解决。
- 开放式事务到视图（OSIV）掩盖懒加载边界；建议关闭并在服务层显式完成数据组装。
- Entity 直接暴露 API 会造成循环引用与过度写入；用 DTO/MapStruct 分层。

## 5.3 MyBatis 的选择

MyBatis 更适合 SQL 复杂、报表强、DBA 深度参与、遗留库表不可大改的场景。它把 SQL 控制权交还开发者，代价是对象映射与关联维护更手动。选型不是信仰问题：领域模型复杂且一致性高，JPA 更省力；查询复杂且性能敏感，MyBatis 更透明。

## 5.4 事务

### 涵义

`@Transactional` 通过代理在方法边界开启/提交/回滚事务，传播级别决定已有事务时如何加入或新建，隔离级别决定并发可见性。

### 作用与意义

- **原子性**：多写操作要么全成，要么全败。
- **边界即设计**：事务边界提示聚合一致性的范围。
- **可组合但受控**：REQUIRED 默认够用；REQUIRES_NEW、NESTED 要清楚代价。

### 常见坑

- 自调用、私有方法、final 类/方法不生效；默认只代理 public 外部调用。
- 在事务里调用远程接口：远程失败与本地回滚不一致，长事务还占连接；用本地消息表/Outbox 或事务后事件。
- 只读查询加写事务增加锁与日志压力；查询用 `readOnly=true`，并明确是否真的需要事务。

## 5.5 数据库迁移

Flyway/Liquibase 把结构变更版本化，随应用或流水线执行。意义在于环境可重建、变更可审计、回滚有策略。不要用 `ddl-auto=update` 管生产结构；它适合原型，不适合审计与协同。

---

## 6. 安全、缓存、消息与任务

## 6.1 Spring Security

### 涵义

Spring Security 是认证（你是谁）与授权（你能做什么）框架。现代配置以 `SecurityFilterChain` Bean 为核心，常见机制有 Session、JWT/OAuth2 Resource Server、方法级 `@PreAuthorize`。

### 作用与意义

- **默认安全头与防护**：CSRF、CORS、会话固定、点击劫持等有内建策略。
- **认证与授权分离**：身份来自网关/OIDC 或本地登录，权限决策落在服务与方法。
- **攻击面收敛**：端点公开范围、密码编码器、token 校验都可显式声明。

### 常见坑

- 把 JWT 当会话并随意吊销困难；短过期 + refresh + 关键操作二次校验更稳。
- 只在网关鉴权、服务内零信任缺失，横向穿透风险高；内网服务仍要验证调用身份与范围。

## 6.2 缓存

### 涵义

`@Cacheable`、`@CachePut`、`@CacheEvict` 把方法结果放入缓存抽象，底层可接 Caffeine、Redis 等。

### 作用与意义

- **降延迟与降压**：读多写少、计算昂贵、下游昂贵的结果适合缓存。
- **一致性是核心成本**：缓存不是免费加速，过期、失效、穿透、击穿、雪崩都要设计。

### 常见坑

- key 不含租户/版本导致串数据；把上下文维度纳入 key。
- 高并发热点 key 失效引发击穿；用逻辑过期、互斥重建或提前刷新。
- 大对象与无界缓存会挤爆内存；设置大小、TTL、序列化成本与淘汰策略。

## 6.3 消息与最终一致

### 涵义

Kafka/RabbitMQ 把跨进程调用异步化，生产者发事件，消费者按语义处理。可靠事件常见模式是 Outbox：本地事务写业务表与 outbox 表，再由中继投递消息。

### 作用与意义

- **削峰与解耦**：峰值不直接打到脆弱下游；上下游按各自节奏演进。
- **跨边界一致性**：分布式事务成本高，最终一致 + 幂等 + 对账更常用。
- **可恢复**：消息可重放、死信可处理，比同步失败更利于修复。

### 常见坑

- 消费者必须幂等；网络重试与再平衡会导致重复投递。
- 消息顺序不是全局天然成立；按 key 分区并明确业务是否真依赖顺序。
- 不要让消息体携带巨大可变状态；传事实与 ID，必要时回查权威数据源。

## 6.4 调度与异步

`@Scheduled` 适合单机定时任务；集群环境要防多实例重复执行，可用 ShedLock、Quartz 集群或外部调度。`@Async` 把方法放到线程池执行，但必须配置池、队列、拒绝策略与异常处理；默认无界或过大队列会掩盖背压。

---

## 7. 测试：从快反馈到真信心

## 7.1 测试金字塔在 Spring 中的落法

- 单元测试：纯领域规则，不启动 Spring；最快最多。
- 切片测试：`@WebMvcTest`、`@DataJpaTest`、`@JsonTest` 只加载相关层。
- 集成测试：`@SpringBootTest` + Testcontainers 起真实数据库/消息中间件；少而关键。
- 契约/端到端：验证 API 契约、关键链路与部署形态；最贵最少。

### 作用与意义

- **定位成本**：失败越靠近单元层，根因越明确。
- **重构护栏**：领域规则与事务边界有测试，升级框架更敢改。
- **环境真实**：Testcontainers 让 H2 与生产库行为差异最小化。

### 常见坑

- 全用 `@SpringBootTest` 导致套件慢、脆、互相污染。
- Mock 一切会测不到序列化、约束、SQL、事务；关键路径要有真实基础设施集成测试。
- 测试数据共享导致顺序依赖；每个测试独立构造数据或清理。

---

## 8. 生产化与运行排错

## 8.1 打包与运行

- 可执行 JAR：适合容器与进程式部署；用分层构建减少镜像体积。
- Docker：JRE 基础镜像 + 分层 JAR + 非 root 用户 + 健康检查 + JVM 容器感知参数。
- 配置：镜像不变，配置外置；机密走密钥管理。
- 停机：开启优雅停机，先拒绝新流量，再完成在途请求，最后关闭池与资源。

### 作用与意义

把“代码能跑”提升为“系统可发布、可回滚、可观测、可恢复”。生产化不是最后一周的任务，而是设计期就要考虑的约束。

## 8.2 日志、指标与追踪

- 日志：结构化输出，统一 traceId，避免打印敏感信息；级别可动态调整。
- 指标：RED（Rate/Errors/Duration）看入口，USE（Utilization/Saturation/Errors）看资源。
- 追踪：OpenTelemetry/Micrometer Tracing 把跨服务调用串成一条链。

### 常见坑

- 只记录异常不记录上下文；订单号、用户维度、请求 ID、耗时、下游状态要随事件带上。
- 指标高基数会拖垮监控系统；不要把用户 ID、完整 URL 当标签。
- 采样过低导致问题现场丢失；对错误与高延迟请求提高采样。

## 8.3 典型排错套路

- 启动失败：看 `conditions` 报告、缺失 Bean、绑定失败、端口占用、Flyway 校验失败。
- 接口慢：入口耗时 → 下游调用 → SQL/连接池 → GC/线程池 → 锁竞争。
- 内存上涨：heapdump 看支配树，怀疑缓存无界、队列堆积、监听器泄漏、ThreadLocal 未清。
- 偶发超时：连接池等待、DNS/ TLS、下游 P99、重试风暴、时钟与超时预算不匹配。

---

## 9. WebFlux 与响应式：何时值得

### 涵义

Spring WebFlux 基于 Reactor/Netty，使用非阻塞 IO 与少量线程支撑高并发连接，编程模型是 `Mono`/`Flux` 流式装配。

### 作用与意义

- **高并发 IO 密集**：网关、BFF、大量等待下游的场景更省线程。
- **背压与流式**：对慢消费者与大数据流有更明确语义。
- **不是免费性能**：阻塞 JDBC、同步 SDK、ThreadLocal 上下文都会破坏收益。

### 选型建议

团队熟悉同步模型、依赖多为阻塞驱动、业务是常规 CRUD：选 MVC 更稳。连接数巨大、下游可异步、团队能接受响应式心智负担：再考虑 WebFlux。

---

## 10. 微服务边界与 Spring Cloud 视角

Spring Boot 是单体与服务的运行时基础；微服务是架构拆分，不是必须终点。引入服务发现、配置中心、网关、熔断限流前，先问：拆分是否带来独立交付、独立扩缩容、独立故障域？如果答案是否定，模块化单体常常更优。

关键概念意义：

- 服务发现：实例动态上下线时不写死地址。
- 配置中心：集中变更与审计，但要防“配置中心一改全局雪崩”。
- 网关：统一入口做认证、限流、路由；不要把业务规则堆进网关。
- 韧性：超时、重试、熔断、舱壁、降级共同保护调用链；重试必须配合幂等与指数退避。

---

## 11. 常见面试/评审问题与答题框架

1. Spring Boot 自动配置原理？答：导入候选自动配置 → 条件过滤 → 用户 Bean 优先 → 属性绑定 → 条件报告可解释。
2. 为什么推荐构造器注入？答：依赖显式、可 final、易测试、暴露循环依赖与职责过多。
3. `@Transactional` 失效场景？答：自调用、非 public、代理未覆盖、异常类型不匹配、多数据源/事务管理器错乱。
4. JPA N+1 怎么解？答：先确认查询目标，再选 EntityGraph/join fetch/DTO 投影/批量抓取；别先怪框架。
5. 如何设计幂等？答：业务唯一键、请求 ID、去重表、状态机约束、数据库唯一约束兜底、消息消费者重放安全。
6. 生产如何暴露 Actuator？答：最小端点、认证授权、网络隔离、管理端口分离、敏感端点禁公网。

---

## 12. 30 天训练计划

- 第 1 周：核心机制。手写最小自动配置 starter；观察 `conditions`；实现强类型配置与 profile。
- 第 2 周：Web 与数据。完成 Todo API：DTO、Validation、全局异常、JPA/MyBatis 任选、Flyway、分页。
- 第 3 周：可靠性与测试。事务边界实验、N+1 复现与修复、Testcontainers 集成测试、Testcontainers + 消息幂等。
- 第 4 周：生产化。Actuator 最小暴露、Prometheus 指标、结构化日志、Docker 镜像、优雅停机、故障演练清单。

验收标准：能解释每个默认行为从哪里来，能把它关掉/替换，并能在出问题时用端点、日志、指标、追踪定位到层。

---

## 13. 速查清单

- 新功能先问：要不要 starter？默认 Bean 是否够用？要不要显式覆盖？
- 新接口先定：DTO、错误模型、状态码、幂等、超时、权限、审计字段。
- 新事务先想：边界多大？是否含远程调用？失败补偿？只读？
- 新缓存先想：key 维度、TTL、失效触发、击穿/穿透/雪崩、内存上限。
- 上线前必有：健康检查、指标、日志 traceId、配置外置、迁移脚本、回滚方案、优雅停机。
- 出问题先看：conditions/beans/env、连接池、慢 SQL、线程与 GC、下游 P99、最近变更。

---

## 14. 进一步阅读方向

- Spring Boot Reference：Features、Auto-configuration、Actuator、Externalized Configuration。
- Spring Framework Reference：Core Container、Web MVC、Data Access、Testing。
- Hibernate 文档：fetching、caching、flushing、locking。
- Micrometer/OpenTelemetry：指标与追踪语义。
- 12-Factor App、Release It!、Designing Data-Intensive Applications：把框架能力放进系统观。
