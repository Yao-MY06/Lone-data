# 个人原则、生活节奏与双轨杂学整合实施计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 将已确认的三条个人原则、运动节奏、双轨杂学和投资学习边界接入个人管理系统，同时保持 Java 后端为唯一 P0。

**Architecture:** 新增一份长期生活系统总纲和一份投资学习安全边界文档；当前状态面板继续是唯一日常入口，只增加导航。杂学流程、晨间日志、周复盘和 README 仅做最小同步，停用的旧日程模板不重新激活。

**Tech Stack:** Obsidian-compatible Markdown、PowerShell、ripgrep。

**Workspace note:** `E:\个人管理系统2.0\.git` 不是有效 Git 仓库，本计划不包含 commit、merge 或 PR；每项以文件内容和链接校验为证据。

> **2026-07-18 实施修订：** 本计划的原始任务记录已经完成。其内关于“周日杂学复盘”和“睡前一小时无电子产品”的旧描述不再作为执行规则；现行规则为周一至周六 9.33 小时任务块、22:50–23:30 共 40 分钟无电子、周日只游泳和生活恢复。辅助线改为 LeetCode、Git/Linux/Docker、JavaGuide/小林 Coding（含 JUC/JVM）、有余力时的 MySQL/Redis 进阶四类；正式投递从 2026-09-15 起占用辅助线槽。权威入口是 [[个人原则与生活系统|个人原则与生活系统]] 和 [[00-当前状态面板|当前状态面板]]。

---

## File Map

- Create: `03-领域与长期/个人原则与生活系统.md` — 三条原则、每日节奏、运动和双轨杂学总纲。
- Create: `07-财务追踪/投资学习与安全边界.md` — 四阶段投资学习、Python 数据项目和风险边界。
- Modify: `03-领域与长期/杂学学习流程.md` — 标记当前两个杂学主题和输入到输出规则。
- Modify: `01-日记与周期/晨间日志模板.md` — 增加运动、10 分钟规划和创造性产出字段。
- Modify: `01-日记与周期/周复盘模板.md` — 增加运动、杂学、关系和财务学习复盘字段。
- Modify: `00-收件箱/00-当前状态面板.md` — 保留 Java 唯一 P0，只增加总纲链接。
- Modify: `README.md` — 增加生活系统和投资学习导航。
- Do not modify: `附-模板与工具/每日执行流程.md`、`附-模板与工具/每周时间块模板.md`、`03-领域与长期/飞轮系统整合方案.md`；三者继续作为停用/历史参考。

## Task 1: Create the personal principles and daily rhythm source of truth

**Files:**
- Create: `03-领域与长期/个人原则与生活系统.md`

- [ ] **Step 1: Write the document header and three principles**

Create an Obsidian page whose first sections state that it is a long-term reference page, not the daily entry, and include verbatim/paraphrased principles: future is created by present actions; creative work is higher-quality happiness than passive consumption; input counts only after behavior/output.

- [ ] **Step 2: Add the confirmed daily table**

Add the exact confirmed schedule: wake at 07:40, wash up and prepare for exercise; 07:50–08:20 run on Monday/Wednesday/Friday and strength train on Tuesday/Thursday/Saturday; after exercise, take the backpack directly to the lab and arrive ready to study by 09:00; Sunday swimming 45–60 minutes follows the day’s arrangement; 09:00–11:30 main focus; 13:30–17:10 coding; 17:10–19:00 meals, rest, and chores such as washing and hanging clothes; 19:10–19:55 one auxiliary line; 19:55–20:55 one full hour of alternating miscellany with output recorded in `04-知识笔记`; 20:55–21:40 combined main-line closeout, acceptance, next-day first step, and review; 21:40–22:20 review and organize the day without adding a new topic; 22:20–22:30 pack the backpack; 22:30–22:50 return to the dorm; 22:50–23:10 prepare and shower; 23:10–23:30 no electronics and bedtime preparation; sleep at 23:30.

- [ ] **Step 3: Add the two-track miscellany rules**

Record Monday/Wednesday/Friday for `《实践论》《矛盾论》`, Tuesday/Thursday/Saturday for investment basics, and Sunday for review only. State that at most two miscellany topics are active and miscellany cannot create a second P0.

- [ ] **Step 4: Add the behavior protocol**

Write four subsections with these rules:

1. Creation gate: exercise and the main-line minimum before games or short videos.
2. Input-to-output: programming creates runnable evidence; Marx reading analyzes a real problem; investment study creates a definition plus simulation/data practice.
3. Friction: remove games/short-video shortcuts and notifications; phone away during focus blocks; breaks use walking, water, distance viewing, meditation or doing nothing.
4. Recovery: after a relapse, do not punish with late-night debt; record the trigger and execute the current task for five minutes.

- [ ] **Step 5: Add the 14-item layered model**

Group the requested items into daily anchors (exercise, 10-minute planning, creation, micro-breaks/water, five-minute review, one-hour screen boundary), weekly anchors (decluttering, two hours of family conversation, relationships, miscellany review, finance learning), and long-term directions (AI/automation, monetizable Java skill, interdisciplinary thinking, buying time, emergency reserve and stable finance). Explicitly prohibit a 14-item daily checklist.

## Task 2: Create investment learning and safety boundary page

**Files:**
- Create: `07-财务追踪/投资学习与安全边界.md`

- [ ] **Step 1: Define purpose and non-advice boundary**

State that this is an education/simulation plan, not a buy/sell instruction. Record the user-provided background of monthly savings around 1500 yuan as context only; do not claim returns or generate a guaranteed allocation from it.

- [ ] **Step 2: Add the four stages**

Add: two-week anti-scam foundation; two-week simulated tools and market map; one-to-two-month systematic concepts plus Python data projects; later small-scale practice only after a separate review. Include the requested concepts (compound interest, annualized return, volatility, maximum drawdown, Sharpe ratio) and the Python projects (valuation percentile chart, annual-report indicators, broad-index backtest).

- [ ] **Step 3: Add safety boundaries and candidate allocation**

Record no paid signal groups, no copy trading, no leverage, no futures/options/crypto contracts, no real-money action during the first two stages. Mark 20% emergency / 70% stable / 10% high-risk as a candidate principle requiring emergency-fund and risk-capacity review, not an automatic command.

- [ ] **Step 4: Add the input-to-output learning record**

Provide a small table with date, concept/source, own definition, simulation/data action, evidence link, and unresolved question. State that recommendations and current product details require later verification before any action.

## Task 3: Synchronize the miscellany learning workflow

**Files:**
- Modify: `03-领域与长期/杂学学习流程.md`

- [ ] **Step 1: Add a current-status banner**

Keep the page as optional reference, but add a current active pair pointing to the personal-life and investment pages. State that the daily panel remains the only execution entry.

- [ ] **Step 2: Add two concrete minimum-practice recipes**

For Marx reading, require one real-life analysis paragraph. For investment, require one definition and one simulation/data action. Retain the existing anti-copying and AI-as-coach rules.

- [ ] **Step 3: Update miscellany capacity rules**

State that exactly these two topics are active for the current cycle, no third topic starts, and Sunday only reviews existing outputs.

## Task 4: Add minimal habit fields to periodic templates

**Files:**
- Modify: `01-日记与周期/晨间日志模板.md`
- Modify: `01-日记与周期/周复盘模板.md`

- [ ] **Step 1: Extend the morning template**

Add optional fields for planned exercise type, the first 10-minute plan, today’s creative/main-line output, and the first consumption boundary. Preserve the existing three-minute limit and single-goal rule.

- [ ] **Step 2: Extend the weekly review template**

Add compact fields for exercise days, miscellany outputs, family/deep-relationship time, decluttering, finance-learning evidence, and the most common consumption trigger. Keep the review under ten minutes.

- [ ] **Step 3: Preserve non-punitive recovery language**

Add that missed habits are recorded as facts, not debts; the next week shrinks the habit set instead of compensating with excessive hours.

## Task 5: Update navigation without creating a second daily surface

**Files:**
- Modify: `00-收件箱/00-当前状态面板.md`
- Modify: `README.md`

- [ ] **Step 1: Add one panel link only**

Keep the Java P0, current next action, minimum line and stop condition unchanged. Add one sentence linking to `[[个人原则与生活系统|个人原则与生活系统]]`; do not add the 14 items or a second daily checklist.

- [ ] **Step 2: Add README navigation**

Add links to the personal-life system and investment-learning page, state that the daily entry remains the current panel, and retain the Java sequence as the active career route.

- [ ] **Step 3: Verify deprecated pages remain untouched**

Use `Get-Item` timestamps or `git` only if available to confirm the three explicitly excluded pages were not modified by this implementation.

## Task 6: Verify the integrated system

**Files:** all files in the File Map.

- [ ] **Step 1: Verify target files exist**

Run:

```powershell
Get-Item '03-领域与长期/个人原则与生活系统.md','07-财务追踪/投资学习与安全边界.md','03-领域与长期/杂学学习流程.md','01-日记与周期/晨间日志模板.md','01-日记与周期/周复盘模板.md','00-收件箱/00-当前状态面板.md','README.md'
```

Expected: seven paths resolve without errors.

- [ ] **Step 2: Verify principle and schedule coverage**

Run `rg -n` over the new life-system page for `未来|创造|行为|07:40|07:50|跑步|无氧|游泳|09:00|21:30|22:00|实践论|投资`.

- [ ] **Step 3: Verify behavior anti-procrastination rules**

Run `rg -n` over the life-system and miscellany pages for `消费|短视频|游戏|输入到输出|5 分钟|最小输出|复盘|触发`.

- [ ] **Step 4: Verify investment safety boundaries**

Run `rg -n` over the investment page for `防骗|模拟|Python|不开户|不跟单|不碰杠杆|候选|不得|收益` and manually check that any result claims are prohibited or marked unknown.

- [ ] **Step 5: Verify active P0 and live links**

Confirm the panel contains exactly one `唯一目标`, contains JavaSE, and does not gain a second daily goal. Resolve all new `[[...]]` links and count Markdown fences; expected zero broken links and zero unclosed fences.

- [ ] **Step 6: Reader scenario check**

Confirm: reading only the panel reveals Java’s next action; reading the life-system page reveals the day schedule and two-track rules; reading the investment page cannot be interpreted as a live trading command; reading the weekly template reveals recovery rather than debt.
