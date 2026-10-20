# pi 最小 Agent 认知基线（项目二 2.1 产出物一）

> 目的：在啃 51 万行 Claude Code 反混淆源码之前，用 pi 建立"Agent 骨架长什么样"的最小认知基线。**只回答三个问题**：消息怎么组装、工具调用怎么分发、状态怎么流转。每个结论必须标源码位置。
> 上游计划：[[../../02-项目管理/智能故障诊断与自动评测平台实施与验收|项目二实施与验收]] §2.1

## 〇、事实登记（2026-10-20 抓取，写死在页顶，避免后续凭记忆）

| 项 | 值 |
| --- | --- |
| 仓库 | `github.com/earendil-works/pi`（原 `badlogic/pi-mono`） |
| 包名 | CLI `@earendil-works/pi-coding-agent` **1.3.0**；运行时 `@earendil-works/pi-agent-core` **1.3.0**；模型层 `@earendil-works/pi-ai` **1.3.0** |
| 本页读的源码 | `@earendil-works/pi-agent-core` **1.3.0 发布包**内 `dist/agent-loop.js.map` 的 `sourcesContent`（还原出 `src/agent-loop.ts`，**949 行 / 29,509 字符**，`wc -l`）；与仓库 `main` 分支同名文件（949 行 / 29,513 字符）**逐行 diff 无差异**（4 字符差来自行尾空白） |
| 源码落地位置 | 三份还原源码已迁到 `F:\study\agent\reference\pi-1.3.0\`（`agent-loop.ts` 949 / `agent.ts` 613 / `types.ts` 539 行）；pi 运行时在 `F:\study\agent\tools\pi\`（CLI `--version` 实测输出 1.3.0） |
| 口径警告（已核销） | 原本担心"读 main 会与发布版偏移"。实际比对证明两者一致，**本页行号对 1.3.0 发布版同样有效** |
| 模型接入 | pi 内置 `providers/deepseek.ts`，`env-api-keys.ts:92` 把 provider `deepseek` 映射到环境变量 `DEEPSEEK_API_KEY` —— 接入只需环境变量，不必写配置文件 |

### 顺带得到一个方法论发现（写给 2.2 报告用）

`@earendil-works/pi-agent-core` 这个包**自己就随附 `dist/*.js.map`，且 map 内 `sourcesContent` 字段包含完整原始 TypeScript**。我这次还原 pi 的 loop 源码，用的就是和社区还原 Claude Code 完全一样的手法：读 source map 的 `sourcesContent`。

→ 这说明"从 npm 包 source map 还原源码"**不是 Claude Code 或某个仓的异常事件，而是 TypeScript 项目的标准打包行为**。2.2 报告的合规声明章节可以直接引这个同源先例：连 pi 这种开源项目都默认发布 `sourcesContent`，只是没人去读；Claude Code 的区别仅在于它没删 `cli.js.map`，而包体积大、读者多。

## 一、loop 骨架：只有两层循环

入口三个，共用同一个 `runLoop()`：

- `agentLoop()`（`agent-loop.ts:38`）：带一条新 prompt 启动；
- `agentLoopContinue()`（`:71`）：从现有上下文续跑，不新增消息；
- `runLoop()`（`:163`）：唯一的主循环实现。

```
runLoop (L163)
├─ pendingMessages = getSteeringMessages()        L176  进来先取一次"用户插话"
└─ while (true)                                    L179  ← 外层
   ├─ hasMoreToolCalls = true                      L180
   ├─ while (hasMoreToolCalls || pendingMessages.length > 0)   L183 ← 内层
   │   ├─ prepareNextTurn(lastCompletedTurn)       L186  下一轮快照：context/model/thinkingLevel
   │   ├─ declareToolChanges(...) 写入 system 消息  L211
   │   ├─ prepareRequest(...)                      L219  发请求前的最后一次改写机会
   │   ├─ streamAssistantResponse(...)             L242  流式拿 assistant message
   │   ├─ stopReason ∈ {error, aborted} → 直接 return L245
   │   ├─ toolCalls = content.filter(toolCall)     L259
   │   ├─ stopReason === "length" → 整批判失败      L268
   │   │  否则 executeToolCalls(...)               L270
   │   ├─ 工具结果写回 context.messages             L274
   │   └─ finishTurn → action === "end" → return    L289
   └─ 外层收尾：getFollowUpMessages() 非空 → 塞进 pending，continue  L302
      → explicitContinuation → continue            L311
      → break                                       L317
```

**两层循环的分工（这是 pi 最值得学的一点）**：内层管"这一轮要不要继续调工具"，外层管"agent 已经想停了，但队列里还有用户后续消息 —— 要不要再开一轮"。前者是工具驱动的续跑，后者是**人驱动的续跑**。Claude Code 的"Continue Sites"那一堆失败恢复策略，本质上是在这两层之间加了更多回到内层的入口。

## 二、消息如何组装

- **唯一真相是 `context.messages` 数组**（`AgentContext`），不是某个"完整 prompt 字符串"。每轮把 assistant message 和 tool result 依次 push 进去（`:214`、`:243`、`:275`），返回值 `newMessages` 只是本次新增的切片。
- **工具集变化用增量 system 消息声明**，不是每轮重发全量工具定义（`declareToolChanges`，`:333`；注释在 `:326-331`）。原文意思：`context.tools` 是**运行时能执行**的集合，而 transcript 里的 system 消息声明的是**模型可以调用**什么；两者的差值变成 `toolsAdded` / `toolsRemoved` 写到 system 消息上，这样"重放 transcript 永远得到恰好等于 `context.tools` 的工具集"。
  → 这是省 token 的关键设计：诊断工具白名单每轮都在变（受限工具按任务裁剪），如果用"每轮重发全量工具"，光工具描述就吃掉大量上下文。
- **发请求前有两个改写钩子**：`prepareNextTurn`（`:186`，上一轮结束后、下一轮开始前）和 `prepareRequest`（`:219`，紧贴请求）。两者都能换 context、换模型、换 thinkingLevel。
- **`systemPrompt` 不是独立字段，是 transcript 的投影**（`types.ts:382-421` 的 `AgentState` 注释原文：*"Current system prompt, replayed from the transcript's system messages"*，只读；要改提示词只能往 transcript 追加一条 system 消息）。同理 `tools` 的 setter 只是改"可执行集合"，真正声明给模型的还是 system 消息。
  → **架构结论：唯一真相 = transcript，其他状态都是它的投影。** 推论：**回放 transcript 就能恢复 Agent 状态**，不需要额外的"状态快照"结构。这条对项目二 2.2 的 Redis/MySQL 状态设计直接可用（长期记录存 transcript，短期状态只需存游标与队列）。

## 三、工具调用如何分发

三段式：**prepare → execute → finalize**，每段都可短路。

1. `executeToolCalls`（`:508`）先判定串行还是并行：只要**任一**工具 `executionMode === "sequential"`，或配置为 sequential，整批就走串行（`:516-522`）。
2. **串行**（`:530`）：逐个 prepare→execute→finalize，出错也继续下一个，除非 `signal.aborted`（`:575`）。
3. **并行**（`:586`）：**先串行地把每个调用的 prepare 做完**（prepare 可能很轻：参数校验、白名单检查），把真正的执行包成 thunk 推进数组（`:619`），最后 `Promise.all` 一次性执行（`:646`），再**按原始调用顺序**逐条生成 result message（`:650-654`）。
   → 两个细节值得抄：① prepare 阶段不并行，避免"参数校验也有副作用"；② 执行并行但结果**保序**，否则上下文里的结果顺序会和模型发出的调用顺序不一致，模型下一轮就读错对应关系。
4. **参数被截断时整批不执行**：assistant message 的 `stopReason === "length"` 说明输出被 token 上限截断，那么里面每个工具调用的参数都可能是坏的，`failToolCallsFromTruncatedMessage`（`:478`）一次性全部判失败（`:267-269`）。
   → 这与项目二 3.5「证据绑定」同一思路：**输入不可信就不执行，宁可返回错误也不产生假证据**。
5. **终止是"全批同意"而非"任一"**：`shouldTerminateToolBatch`（`:690`）= 批内**每一个**结果的 `terminate === true` 才终止整批。单个工具喊停不足以停掉一轮。
6. 钩子：`beforeToolCall` / `afterToolCall`（类型定义 `:683`），权限、审计、结果脱敏都挂这里。
7. **工具结果的"三份内容"设计**（`types.ts:424-446`）：`content` 给模型看；`details` 给日志/UI 看；`structuredContent` 给程序看，注释明确写着 **"Not sent to the model"**。
   → 这是项目二 3.5 证据绑定的现成骨架：工具把 `traceId`、日志行号、指标时间窗放进 `structuredContent`，**不占模型上下文**，绑定校验器从结构化结果里回查；模型只看到人可读的 `content`。同时 `isError` 允许"不抛异常地报告失败"（模型看到错误内容，程序仍拿到 details）。

## 四、状态如何流转与终止

终止路径只有四条，全部显式：

| 触发 | 位置 | 语义 |
| --- | --- | --- |
| `stopReason === "error"` / `"aborted"` | `:245` | 模型层失败或用户中断，立刻 `agent_end` 返回，不重试 |
| `finishTurn` 返回 `action: "end"` | `:289` | 外部决策要求结束（预算耗尽、任务完成判定挂这里） |
| 内层条件不再满足 + 外层无后续消息 | `:183` → `:317` | 自然结束：没有工具调用、没有插话、没有后续消息 |
| 整批工具 `terminate` | `:690` | 工具自己要求终止 |

另外两个状态变量容易被忽略：`explicitContinuation`（`:294`、`:311`）允许"即使没有新消息也再跑一轮空上下文"；`lastCompletedTurn`（`:173`）保存上一轮的 message/toolResults/context，是 `prepareNextTurn` 的输入 —— **压缩、记忆写入这类重活就挂在这个钩子上**。

**两个队列是分开的**（`agent.ts:191-192`、`:247-248`）：`steeringQueue`（agent 跑着时用户插话）和 `followUpQueue`（agent 想停后用户追加）。两者各自有 `QueueMode`，默认 **`one-at-a-time`**。
→ 这就是 loop 里那句注释的由来（`:201-203`）：*"otherwise one-at-a-time mode would deliver two messages in this turn"* —— 一次只取一条，避免同一轮塞进两条插话把上下文搞乱。`Agent` 类同时只允许一个 `activeRun`（`:218`），并发由外部排队，不靠 loop 自己处理。

## 五、pi 刻意没有的东西（对照项目二 3.3 / 3.4）

| 机制 | pi | 项目二要求 |
| --- | --- | --- |
| 上下文压缩 | **无内建**。注释只提到"Preparation can be long-running (for example, compaction)"（`:201-203`），压缩是使用方通过 `prepareNextTurn` 自己塞进去的 | 四级 `Compressor` 链（3.4） |
| 循环防护 | **无**。没有重复调用检测、没有 stall detection，只有 `stopReason` 和 `terminate` 两个粗粒度出口 | 最大轮次/token/墙钟 + 重复检测分级处置 + stall（3.3） |
| 上下文窗口管理 | **无**。没有 token 预算、没有截断策略 | L1–L4 预算与降级 |
| 子代理 / 权限系统 | README 明说刻意省略 | 不在最小范围 |
| 证据绑定 | **无**。工具结果原样回灌，不做引用校验 | 3.5 最强原创点 |

**这正是"pi 是学前班"的实证**：19 个文件的运行时只解决"循环能转起来"，所有可靠性机制都留给使用方。所以本项目 3.3/3.4 的每一条，都不是在抄谁，而是在补 pi/CC 之间的那段空白。

一个必须说准的细节：pi **给了 `structuredContent` 这个槽位，但不校验它**。也就是说"工具能吐出结构化证据"这件事 pi 有，但"结论必须引用已存在的证据、否则不许标为确认"这件事 pi 完全没有 —— 这正是 3.5 要自研的那一层，且槽位现成、不用改协议。

## 六、直接喂给 Java 自研引擎的四条设计决定

1. **并行执行 + 保序回传**：Java 用 `CompletableFuture.allOf` + 预分配 `List` 按索引写入，不用 `thenApply` 收集（会乱序）。
2. **prepare 阶段串行**：参数校验/白名单/越权检查一律在提交执行之前做完，避免"非法调用已经打出去"。
3. **压缩做成 loop 外的钩子**（对应 `prepareNextTurn`），不做成 loop 内分支 —— 这样消融实验才能干净地"关闭压缩"再跑同一故障集。
4. **终止取"全批同意"**：诊断批次里某个工具发现致命线索，也不该让整批停掉，应等全部工具回传后再由策略层决定。
5. （附带）**工具集变化发增量 system 消息**，为受限工具白名单动态裁剪留出 token 空间。

## 七、未完成（如实标注）

- [x] **实派任务：按用户决定暂缓（2026-10-20）**。用户持有 DeepSeek key 但明确不提供给 AI 执行，并要求先做其他事。→ 本页结论为**纯源码级**，**没有实跑 pi**。2.1 原文"安装并实际派任务使用后"这一条**未满足**；若要补，用户需自己跑一次派任务，或事后提供截图/日志。
- [x] ~~`agent.ts` 与 `types.ts` 未细读~~ → **已补读**（`agent.ts` 614 行、`types.ts` 540 行，同样由 1.3.0 包内 source map 还原）：新增结论见 §二（systemPrompt 是 transcript 投影）、§三第 7 条（content/details/structuredContent 三份内容）、§四（steering 与 followUp 两队列 + `one-at-a-time`）。
- [x] **DeepSeek 模型清单已核实**（读已安装的 `@earendil-works/pi-ai` 包内 `dist/providers/data/deepseek.json`，**不需要 key**）：只有两个 model id —— `deepseek-flash`（DeepSeek V4.1 Flash）与 `deepseek-v4-pro`（DeepSeek V4 Pro）；均 `reasoning: true`、`contextWindow: 1,000,000`、`maxTokens: 384,000`，走 `openai-completions` API，baseUrl `https://api.deepseek.com`。→ 将来实跑时不需要配 baseUrl，只设 `DEEPSEEK_API_KEY` 即可。
- [ ] 循环防护/压缩在 pi 里的"使用方实现"样例没看（`pi-coding-agent` 是使用方，它的 `prepareNextTurn` 实现可作为简版参照）。
