# Java Backend Roadmap and Project Switch Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Switch the personal system from the completed exam and blocked library project to a single Java backend internship roadmap, with Black Horse Review as the core project, DeerFlow-Java as the later differentiator, and a compact rolling AI collaboration context.

**Architecture:** Keep the current status panel as the only daily surface. Store the long-term sequence in one Java roadmap, project-specific scope in two separate project plans, and cross-conversation state in a bounded snapshot document. Preserve non-committed technologies in explicit candidate pools with activation gates.

**Tech Stack:** Obsidian-compatible Markdown, Mermaid, PowerShell and ripgrep for verification.

---

## File Map

- Modify `00-收件箱/00-当前状态面板.md`: active JavaSE closeout goal and today's action.
- Create `00-收件箱/AI协作上下文.md`: bounded rolling snapshot for future Codex sessions.
- Create `03-领域与长期/Java后端学习与投递路线图.md`: July-October learning, project and application stages.
- Create `02-项目管理/黑马点评生产级改造计划.md`: baseline course project, three committed production upgrades and retained candidate pool.
- Create `02-项目管理/DeerFlow-Java扩展平台计划.md`: bounded Agent MVP and retained candidate pool.
- Modify `02-项目管理/图书管理系统架构演进.md`: paused status and recovery condition.
- Modify `03-领域与长期/全栈学习路线图.md`: superseded-by-Java-roadmap boundary.
- Modify `README.md`: navigation to the new roadmap, project plans and collaboration context.
- Verify all changed files: links, single-P0 semantics, scope separation, fence balance and unsupported numbers.

The workspace is not a Git repository. Commit steps are omitted; fresh verification output is the completion evidence.

### Task 1: Switch the Daily Panel and Create Rolling Context

**Files:**
- Modify: `00-收件箱/00-当前状态面板.md`
- Create: `00-收件箱/AI协作上下文.md`

- [ ] **Step 1: Verify the stale exam state exists**

Run:

```powershell
rg -n "考试冲刺|计算机组成原理|JavaSE" "00-收件箱/00-当前状态面板.md"
```

Expected before edit: exam content matches; JavaSE does not appear as the active goal.

- [ ] **Step 2: Replace the panel with the JavaSE closeout state**

The panel must remain the only daily page and contain exactly one P0. Use these operational fields:

```markdown
## 当前模式

**Java 后端求职路线：JavaSE 收尾（目标窗口截至 2026-07-18）**

| 字段 | 内容 |
|---|---|
| 唯一目标 | 完成 JavaSE 诊断并补齐影响 JavaWeb 的关键缺口 |
| 下一步动作 | 不看教程，分别写一个集合处理、异常处理和文件读写小题；把不会或写错的点列出来 |
| 最低完成线 | 独立完成其中 1 题，并记录 1 个真实缺口 |
| 停止条件 | 3 题均已运行验证，或当日主学习块结束；不继续扩展新资料 |
```

Add one line linking `[[AI协作上下文]]` as AI-maintained context, not a daily page. Keep the existing three-line shutdown record and concrete five-minute recovery rule.

- [ ] **Step 3: Create the rolling context snapshot**

`AI协作上下文.md` must open with:

```markdown
# AI 协作上下文

> 本页由 AI 在重要决策、阶段完成或计划变化时更新。它不是日常入口，不保存完整聊天；后续协作先读取本页和 [[00-当前状态面板]]。
```

Populate the fixed sections with current facts rather than empty placeholders:

- Current P0: JavaSE closeout.
- Ability: JavaSE about 70-80%, C++ foundation, JavaWeb/Spring Boot not started.
- Availability: roughly six effective hours using the agreed time blocks.
- Target: Java backend internship, applications from September while improving projects.
- Confirmed route and date windows from the approved design.
- Black Horse committed upgrades: limited services, seckill consistency/MQ, engineering evidence.
- Black Horse candidates: the complete retained list from the design.
- DeerFlow MVP: Java MCP Server with two tools, Redis/MySQL memory service, real integration.
- DeerFlow candidates: the complete retained list from the design.
- Paused/abandoned: embedded competition and library project, with reasons.
- Next action: JavaSE diagnostic exercises.
- Open questions: actual diagnostic gaps and final MQ choice remain evidence-dependent.
- Relevant file links to the roadmap, two project plans, current panel and project SOP.
- Recent decisions: at most five entries, including the project switch and candidate-retention rule.

- [ ] **Step 4: Verify daily and context boundaries**

Run:

```powershell
rg -n "唯一目标|JavaSE|最低完成线|AI协作上下文|不是日常入口|最近.*5|黑马点评|DeerFlow" "00-收件箱/00-当前状态面板.md" "00-收件箱/AI协作上下文.md"
```

Expected: all concepts appear; the panel has one `唯一目标` row and the context says it is AI-maintained.

### Task 2: Create the Java Backend Learning and Application Roadmap

**Files:**
- Create: `03-领域与长期/Java后端学习与投递路线图.md`

- [ ] **Step 1: Create the strategy and stage table**

The page must state it is a stage/reference page, not a daily entry. Add the sequence:

```text
JavaSE 收尾 -> JavaWeb/Spring Boot -> 苍穹外卖 -> 原版黑马点评 -> 生产级改造 -> 投递 -> DeerFlow-Java MVP
```

Use these windows and outputs:

| Window | Stage | Observable output |
|---|---|---|
| 2026-07-15 to 07-18 | JavaSE closeout | diagnostic program and corrected gaps |
| 07-19 to 07-28 | JavaWeb + Spring Boot | login, validation, exception and MySQL CRUD API |
| 07-29 to 08-10 | CangQiong practice | core backend flows run and one real defect diagnosis |
| 08-11 to 08-31 | original Black Horse Review | core Redis/business flows run and can be explained |
| 09-01 to 09-15 | production upgrade | three committed upgrade directions with evidence |
| from 09-15 | applications | apply while filling evidence-based gaps |
| late September to October | DeerFlow-Java MVP | cross-language Java tool and memory loop |

State that dates are target windows; stage exit requires acceptance. Do not delay all applications merely because candidates remain unfinished.

- [ ] **Step 2: Add exact stage acceptance gates**

For each stage, copy the acceptance requirements from design sections 3.1 through 3.7. Keep CangQiong explicitly off the resume and reduce it to backend practice. Keep Black Horse baseline separate from production upgrades.

- [ ] **Step 3: Add the six-hour daily structure and auxiliary rules**

Use the approved time table: 09:00-11:30 learning/minimal exercise, 13:30-16:00 coding/debugging, 19:30-20:15 LeetCode, 20:30-21:00 acceptance/next action. State that remaining time is buffer, not hidden study debt.

Add the exact auxiliary boundaries: LeetCode 4-5 times/week and 45 minutes max; re-solve after 48 hours; interview theory starts with Black Horse and is project-linked; MySQL/Redis/MQ/Cloud follow projects; JUC/JVM follows baseline Black Horse.

- [ ] **Step 4: Add delay and application rules**

If a target window slips, remove non-core course features and fix the acceptance blocker. Never claim a stage complete because videos ended. Applications start once Black Horse is demonstrable; candidate technologies cannot block application.

- [ ] **Step 5: Verify stages and dates**

Run:

```powershell
rg -n "JavaSE|JavaWeb|苍穹外卖|黑马点评|生产级|投递|DeerFlow|09:00|48 小时|JUC|JVM|验收" "03-领域与长期/Java后端学习与投递路线图.md"
```

Expected: every stage, daily block and auxiliary boundary appears.

### Task 3: Create the Black Horse Review Production Upgrade Plan

**Files:**
- Create: `02-项目管理/黑马点评生产级改造计划.md`

- [ ] **Step 1: Add project brief and baseline gate**

Classify it as a portfolio project for Java backend internships. Define the current user/problem as demonstrating defensible backend engineering rather than accumulating middleware. State that no production upgrade begins before the original project runs and its Redis mechanisms can be explained.

Baseline acceptance must cover login, shop caching, voucher seckill, Redis lock/Lua/message handling, Feed, GEO and sign-in as applicable to the source course. Unknown source differences must be recorded rather than invented.

- [ ] **Step 2: Add the three committed upgrades**

Only these are committed:

1. Limited domain split: `user`, `shop/content`, `trade`; Gateway, Nacos, OpenFeign and Sentinel are introduced only where exercised.
2. Seckill depth: Redis Lua gate, MQ asynchronous order, idempotency, retry, final consistency and reconciliation; Redis Stream versus chosen MQ must be compared using actual requirements.
3. Engineering evidence: repeatable load test, slow-query/index evidence, tracing/metrics/diagnostics and Docker Compose.

For each, include goal, in-scope behaviors, out-of-scope shortcuts, observable acceptance and evidence location. No fixed QPS or latency claim may be inserted before measurement.

- [ ] **Step 3: Preserve all Black Horse candidates**

Create `候选技术池（默认不启用）` with every item from design section 5.1: Caffeine/Redis, Bloom filter, Elasticsearch, Canal/MQ sync, XXL-Job, WebSocket/SSE, hybrid Feed, sharding/read-write split, K8s and AI/RAG features.

For every candidate category, state its activation trigger. Examples: measured database read bottleneck before multi-level cache; real complex-search requirement before Elasticsearch; measured single-database capacity boundary before sharding.

- [ ] **Step 4: Add resume evidence rules**

Resume bullets are written only after implementation and must link to reproducible evidence. Prohibit invented concurrency, improvement percentages, cost savings and reliability claims. Add interview preparation prompts around why split, where calls fail, how consistency works and what trade-offs remain.

- [ ] **Step 5: Verify committed-versus-candidate separation**

Run:

```powershell
rg -n "只承诺|user|shop/content|trade|Redis Lua|幂等|最终一致性|压测|候选技术池|Elasticsearch|ShardingSphere|K8s|不得编造" "02-项目管理/黑马点评生产级改造计划.md"
```

Expected: the three commitments and the complete candidate pool appear in separate sections.

### Task 4: Create the DeerFlow-Java Extension Plan

**Files:**
- Create: `02-项目管理/DeerFlow-Java扩展平台计划.md`

- [ ] **Step 1: Define the cross-language boundary**

State that Python DeerFlow performs Agent orchestration and LLM calls. Java provides bounded tool and state services for a Java-backend portfolio. Avoid claiming DeerFlow internals or protocol support that has not been verified against the selected version.

- [ ] **Step 2: Define the committed MVP**

Commit only:

- One Java MCP Server with at least two restricted tools such as approved knowledge search, restricted data query or artifact saving.
- One Java Memory Service using Redis for short-term state and MySQL for long-term records.
- Docker Compose for Java services and dependencies.
- A real DeerFlow-to-Java invocation with visible invalid-input, timeout and tool-failure paths.

Add acceptance for two tool calls, memory write/read, explicit errors, secret protection and a README explaining the boundary. Mark protocol/transport choices as validation tasks until tested.

- [ ] **Step 3: Preserve all Agent candidates**

Create `候选技术池（默认不启用）` with design section 5.2: more MCP servers, vector memory/compaction/multi-tenancy, distributed scheduler/DAG, tool gateway, Sentinel, Java Docker sandbox, RocketMQ message bus, Kafka logs and WebSocket/SSE progress.

Each candidate needs an activation gate tied to an observed limitation. Do not introduce it merely because it appears in the attachment.

- [ ] **Step 4: Add evidence and safety rules**

Link `[[AI使用公约|AI使用公约]]` and `[[AI项目开发参考手册|AI项目开发参考手册]]`. Require restricted tool inputs, least privilege, no production credentials, explicit network/data boundaries and truthful measurement.

- [ ] **Step 5: Verify the MVP boundary**

Run:

```powershell
rg -n "Python|编排|Java MCP|两个|Memory Service|Redis|MySQL|Docker Compose|超时|候选技术池|调度|沙箱|RocketMQ|不得" "02-项目管理/DeerFlow-Java扩展平台计划.md"
```

Expected: committed components, failure paths, evidence rules and retained candidates appear.

### Task 5: Pause Old Directions and Update Navigation

**Files:**
- Modify: `02-项目管理/图书管理系统架构演进.md`
- Modify: `03-领域与长期/全栈学习路线图.md`
- Modify: `README.md`

- [ ] **Step 1: Pause the library project without deleting history**

Add a banner in the first 10 lines:

```markdown
> **状态：暂停，不占用 P0。** 当前项目因实施条件不足停止推进，不保留原日期任务债。只有在核心 Java 路线稳定、重新确认交付物与恢复后的 15 分钟第一步时，才可按 [[项目SOP|项目 SOP]] 重新立项。
```

Use a correct relative link from the file's directory. Do not rewrite the historical architecture content or claim it was completed.

- [ ] **Step 2: Supersede the old full-stack roadmap**

Update its existing top banner to say the current career route is `[[Java后端学习与投递路线图]]`; all React/Node fixed stages and outcome claims below are historical reference, not active requirements.

- [ ] **Step 3: Add compact README navigation**

Keep the four-point daily rule. Add links to:

- `[[AI协作上下文]]`
- `[[Java后端学习与投递路线图]]`
- `[[黑马点评生产级改造计划]]`
- `[[DeerFlow-Java扩展平台计划]]`

Describe the current sequence in one sentence. Do not turn README into a daily checklist.

- [ ] **Step 4: Verify active and paused navigation**

Run:

```powershell
rg -n "暂停|不占用 P0|Java后端学习与投递路线图|AI协作上下文|黑马点评生产级改造计划|DeerFlow-Java扩展平台计划" "02-项目管理/图书管理系统架构演进.md" "03-领域与长期/全栈学习路线图.md" README.md
```

Expected: the library project is paused, old roadmap is superseded and new navigation is present.

### Task 6: Run System-Level Verification

**Files:**
- Verify all files from the File Map.

- [ ] **Step 1: Verify every target file exists**

Run:

```powershell
Get-Item "00-收件箱/00-当前状态面板.md","00-收件箱/AI协作上下文.md","03-领域与长期/Java后端学习与投递路线图.md","02-项目管理/黑马点评生产级改造计划.md","02-项目管理/DeerFlow-Java扩展平台计划.md","02-项目管理/图书管理系统架构演进.md","03-领域与长期/全栈学习路线图.md","README.md"
```

Expected: eight files, no missing path errors.

- [ ] **Step 2: Verify one active P0**

Run:

```powershell
rg -n "唯一目标|P0|考试冲刺|计算机组成原理|图书管理.*P0|比赛.*P0" "00-收件箱/00-当前状态面板.md" "00-收件箱/AI协作上下文.md"
```

Expected: JavaSE is the only active P0; exam, competition and library project do not appear as active goals.

- [ ] **Step 3: Verify committed and candidate scopes**

Run:

```powershell
rg -n "只承诺|候选技术池|默认不启用|验收|未知|不得编造" "02-项目管理/黑马点评生产级改造计划.md" "02-项目管理/DeerFlow-Java扩展平台计划.md"
```

Expected: each project has explicit commitments, candidate boundaries and evidence rules.

- [ ] **Step 4: Check live wiki links and Markdown fences**

Inspect wiki links outside fenced examples, resolve relative targets and report any missing target. Count Markdown fence openings/closures, including four-backtick wrappers around nested examples. Expected: zero broken live links and zero unclosed fences.

- [ ] **Step 5: Scan unsupported claims**

Run:

```powershell
rg -n "QPS|P99|延迟.*ms|提升.*%|降低.*%|录用率|通过率|绝对够|直接要" "03-领域与长期/Java后端学习与投递路线图.md" "02-项目管理/黑马点评生产级改造计划.md" "02-项目管理/DeerFlow-Java扩展平台计划.md" "00-收件箱/AI协作上下文.md"
```

Expected: no unsupported result claim. Any numeric target must be labeled as a test condition, unknown or candidate rather than an achieved result.

- [ ] **Step 6: Run reader scenarios**

Confirm from the actual files:

1. Reading only the current panel reveals today's JavaSE action, minimum and stop condition.
2. A fresh Codex session reading the context snapshot can state current P0, roadmap, project boundaries, paused directions and next action.
3. Reading Black Horse plan cannot place Elasticsearch, sharding, K8s or AI into the current commitment.
4. Reading DeerFlow plan cannot place distributed scheduler, sandbox or message bus into the MVP.

- [ ] **Step 7: Report evidence**

Report changed files, command outputs, unresolved questions and the fact that no project code was created. Do not claim a commit because the workspace has no Git repository.
