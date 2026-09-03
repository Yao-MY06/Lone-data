# Personal System Progressive Integration Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Integrate the four AI learning and engineering documents into the existing Obsidian system while preserving one daily entry point and introducing new behavior over two weeks.

**Architecture:** Keep `00-收件箱/00-当前状态面板.md` as the only daily control surface. Put activation rules in `启动计划.md`, reusable workflows in existing SOP files, and long engineering templates in one background reference manual. Validate the result through link, contradiction, duplicate-routine, and recovery-scenario checks.

**Tech Stack:** Obsidian-compatible Markdown, Mermaid, PowerShell and ripgrep for verification.

---

## File Map

- Modify `00-收件箱/00-当前状态面板.md`: current exam mode and the single daily control surface.
- Modify `00-收件箱/启动计划.md`: July 14-27 activation and downgrade rules.
- Modify `01-日记与周期/周复盘模板.md`: ten-minute weekly review.
- Modify `02-项目管理/项目SOP.md`: project entry gate, DLARL loop, incremental MVP, acceptance and optional engineering upgrades.
- Modify `04-知识笔记/知识处理SOP.md`: goal-first learning, retrieval practice, project validation and lightweight capture.
- Modify `05-资源库/AI使用公约.md`: risk-based human/AI responsibilities and comprehension gate.
- Create `05-资源库/AI项目开发参考手册.md`: dormant engineering, Prompt and open-source templates.
- Modify `README.md`: single-entry navigation and activation boundaries.
- Verify all changed Markdown files: links, stale rules, duplicate daily requirements and usage scenarios.

The workspace is not a Git repository. Commit steps are intentionally omitted; verification output is the completion evidence.

### Task 1: Replace the Daily Control Surface

**Files:**
- Modify: `00-收件箱/00-当前状态面板.md`

- [ ] **Step 1: Record the baseline facts**

Run:

```powershell
rg -n "日期|今日唯一目标|本周唯一重点|考试日期|恢复协议" "00-收件箱/00-当前状态面板.md"
```

Expected: the file still contains June dates, probability theory as P0 and a July 13 computer organization exam entry.

- [ ] **Step 2: Replace the page with the four-field daily surface**

The page must contain, in this order:

1. A banner saying this is the only page maintained daily.
2. `当前模式` with `考试冲刺（截至 2026-07-13）`.
3. Four fields: `唯一目标`, `下一步动作`, `最低完成线`, `停止条件`.
4. July 12 action: do questions, active recall and gap filling; do not add resources.
5. Pre-exam action: review only mistakes, formulas and problem types that still cannot be started.
6. A compact end-of-day log with `完成`, `卡点`, `明天首先打开`.
7. A recovery rule: after a missed day, do a five-minute action without repaying task debt.
8. A pointer to `[[启动计划]]`, explicitly saying not to open it before the exam unless needed.

Use these exact operational rules:

```markdown
| 字段 | 当前内容 |
|---|---|
| 唯一目标 | 计算机组成原理稳过 |
| 下一步动作 | 做一组题；不会的题先写已知条件和第一步，再查漏 |
| 最低完成线 | 独立写出 3 道常见题的主要步骤 |
| 停止条件 | 常见题能起步，错题已标记；到计划睡眠时间立即停止 |
```

- [ ] **Step 3: Verify that stale control data is gone**

Run:

```powershell
rg -n "2026-06-15|概率论|距离考试：20天|P1：Java" "00-收件箱/00-当前状态面板.md"
```

Expected: no matches.

- [ ] **Step 4: Verify the four fields and recovery rule**

Run:

```powershell
rg -n "唯一目标|下一步动作|最低完成线|停止条件|5分钟|不补" "00-收件箱/00-当前状态面板.md"
```

Expected: all six concepts appear.

### Task 2: Replace the Activation Plan and Weekly Review

**Files:**
- Modify: `00-收件箱/启动计划.md`
- Modify: `01-日记与周期/周复盘模板.md`

- [ ] **Step 1: Replace the old eight-week schedule**

`启动计划.md` must define:

- July 14-20: current panel only; one deliverable per day; five-minute floor; 30-90 minute normal target; three-line shutdown log.
- July 21 is the earliest second-stage date, not an automatic switch.
- Stage two unlock condition: at least three actual execution days under stage-one rules.
- July 21-27: one project or visible learning outcome; loop `定目标 -> 自己尝试 -> AI辅助 -> 验收 -> 记一个坑`.
- Downgrade after two consecutive missed days: return to the current panel only.
- No debt repayment, no forced blog, no Token tracking, no morning journal and no Pomodoro accounting during recovery.
- July 27 review criteria: 10 execution days, one visible result, next-day recovery after one missed day and one daily entry point.

- [ ] **Step 2: Replace the weekly review with four decisions**

`周复盘模板.md` must contain only:

```markdown
## 1. 可见成果
- 本周真正完成了什么：

## 2. 执行与恢复
- 实际推进：__ 天
- 中断后是否在第二天恢复：是 / 否

## 3. 最大阻力
- 最常阻止我开始或继续的事情：
- 下周删除或缩小什么：

## 4. 下周唯一主线
- P0：
- 可验收结果：
- 第一个 5 分钟动作：
```

End with: `超过 10 分钟仍未写完，就停止；复盘的产出是一个决定，不是一篇报告。`

- [ ] **Step 3: Verify obsolete activation requirements are removed**

Run:

```powershell
rg -n "第3周|第4周|第5周|第6周|第7周|第8周|每天1次|目标7天|GitHub有.*天|技术博客" "00-收件箱/启动计划.md" "01-日记与周期/周复盘模板.md"
```

Expected: no matches.

### Task 3: Refactor the Project SOP Around Progressive Gates

**Files:**
- Modify: `02-项目管理/项目SOP.md`

- [ ] **Step 1: Add a lightweight project entry gate before the full SOP**

The first operational section must say:

- Recovery period allows one P0 project only.
- A project needs one user/problem, one visible deliverable, one deadline and one first action under 15 minutes.
- Commercial questions are optional for pure learning or course projects; label the project type instead of blocking it.
- Start with a one-page brief; open the background reference manual only when the project needs stronger engineering evidence.

- [ ] **Step 2: Define the minimal DLARL loop**

Use this exact mapping:

```markdown
Design：写清本轮交付物、约束和验收标准。
AI落地：一次只实现一个可验证步骤。
Review：先运行和检查，再判断是否接受 AI 产出。
Learn：只补当前交付物所需的知识，默认限时 15 分钟。
Adjust：记录一个改变，下轮继续验证。
```

The SOP must state that architecture, PRD and ADR depth scales with risk; they are not all mandatory for a two-week learning result.

- [ ] **Step 3: Define incremental MVP stages**

Provide the optional sequence `可运行骨架 -> 数据/状态 -> 核心规则 -> 外部或 AI 集成 -> 界面 -> 日志与发布`. Each stage must have one observable acceptance result before the next begins.

- [ ] **Step 4: Replace fixed activity metrics with outcome checks**

Remove requirements for three commits per week, daily commits and mandatory blog output. Project review must instead check:

- Does the current version run or can the result be shown?
- Does it satisfy this iteration's acceptance condition?
- Can the user explain the accepted AI-generated portion?
- What is the smallest next testable step?

- [ ] **Step 5: Link the reference manual**

Add `[[../05-资源库/AI项目开发参考手册|AI项目开发参考手册]]` for PRD, ADR, Prompt versioning, engineering checks and open-source release details.

- [ ] **Step 6: Verify contradictory project limits and fixed metrics are gone**

Run:

```powershell
rg -n "项目.*≤2|不超过2个|至少3次commit|每天1次|总分>15|全部11项" "02-项目管理/项目SOP.md"
```

Expected: no matches.

### Task 4: Make Knowledge Processing Action-First

**Files:**
- Modify: `04-知识笔记/知识处理SOP.md`

- [ ] **Step 1: Put the learning loop before classification**

The first operational section must use:

```markdown
1. 定一个可验证问题或成果。
2. 不看答案，先回忆、解题或实现。
3. 定向查官方文档、教材或源码。
4. 回到题目或项目验证。
5. 只记录一个以后会复用的结论、例子或坑。
```

State that video is for overview or a blocked concept, not a default learning path.

- [ ] **Step 2: Reduce capture and note requirements**

Keep the routing decision but remove fixed 22:00 processing and mandatory daily inbox clearing. A note is created only when it supports the current project, explains a recurring mistake or is likely to be reused. The minimum note fields are `一句话理解`, `一个例子或题型`, `一个易错点`, `来源`.

- [ ] **Step 3: Add active recall and spaced review triggers**

Define review by use:

- Review once the next day for exam-critical or repeatedly missed knowledge.
- Review again when the project or a new problem calls it.
- Do not schedule all saved information for review.

- [ ] **Step 4: Remove maintenance-heavy rules**

Remove mandatory weekly link building, the fixed extra 15-minute cleanup, mandatory tags and claims that an unprocessed day creates failure.

- [ ] **Step 5: Verify action precedes capture**

Run:

```powershell
rg -n "定一个可验证|先回忆|回到.*验证|一句话理解|一个易错点" "04-知识笔记/知识处理SOP.md"
```

Expected: all five concepts appear.

### Task 5: Replace the AI Covenant with Risk-Based Rules

**Files:**
- Modify: `05-资源库/AI使用公约.md`

- [ ] **Step 1: Replace the universal 30-minute rule**

Use three levels:

- Low-risk repetitive work: AI may be used immediately, but output must be verified.
- Learning or core project decisions: write the goal, known facts and first approach before asking AI; use 5-30 minutes based on difficulty.
- Exams, graded core work, factual claims and irreversible actions: keep human responsibility and independently verify against authoritative sources.

- [ ] **Step 2: Define the comprehension gate**

AI output is accepted only if the user can explain its purpose, run or test it, identify what changed and undo or revise it. Unknown parts become a bounded learning item; they do not force abandonment of the whole project.

- [ ] **Step 3: Define model/tool escalation without daily accounting**

Use the sequence `local/cheap tool -> stronger model -> specialist review` only when the current level fails a clearly written acceptance condition. Record cost only for a real paid project or when monthly spend approaches a chosen budget; remove daily dependence scoring and the mandatory AI detox week.

- [ ] **Step 4: Verify extreme and maintenance-heavy rules are gone**

Run:

```powershell
rg -n "30分钟是底线|每天晚上.*5个问题|依赖度评分|AI戒断周|看不懂.*绝对不能直接运行" "05-资源库/AI使用公约.md"
```

Expected: no matches.

### Task 6: Create the Background Reference Manual

**Files:**
- Create: `05-资源库/AI项目开发参考手册.md`

- [ ] **Step 1: Create a clearly dormant reference page**

The opening must state: `这不是每日清单。只有项目SOP明确指向某一节时才打开。`

- [ ] **Step 2: Add reusable templates**

Include complete compact templates for:

- One-page PRD: problem, user, Must/Should/Could, constraints, acceptance.
- ADR: context, decision, alternatives, consequences, review trigger.
- Iteration Prompt: goal, context, constraints, in-scope, out-of-scope, acceptance, output format.
- Review note: observed behavior, accepted/rejected change, unknown point, next test.
- Prompt version record: version, changed rule, test case, observed result, decision.

- [ ] **Step 3: Add optional engineering and release gates**

Organize optional checks by risk: tests and error paths; performance baseline; security and secrets; logs and diagnostics; README, setup steps, architecture explanation and demo. Explicitly prohibit invented coverage, performance, cost-saving, completion-rate, star or interview metrics.

- [ ] **Step 4: Preserve candidate project ideas without activating them**

Summarize AI task scheduler, code review assistant and knowledge-base assistant as candidates. Each entry must contain the user problem, smallest two-week proof and reason not to start automatically.

- [ ] **Step 5: Verify the manual is not framed as daily work**

Run:

```powershell
rg -n "不是每日清单|按需|不得编造|两周" "05-资源库/AI项目开发参考手册.md"
```

Expected: all four concepts appear.

### Task 7: Update Navigation and Run System-Level Verification

**Files:**
- Modify: `README.md`
- Verify: all files listed in the File Map

- [ ] **Step 1: Put the single-entry rule at the top of README**

Add a compact `现在怎么用` section immediately after the introduction:

```markdown
1. 每天只打开 [[00-收件箱/00-当前状态面板]]。
2. 只推进一个 P0；其他想法进入收件箱。
3. SOP 和参考手册只有在当前面板或项目明确需要时才打开。
4. 中断后执行 5 分钟恢复动作，不补债。
```

Replace stale 24-hour startup instructions and fixed productivity targets with links to the current panel and activation plan. Keep the broader PARA explanation as reference.

- [ ] **Step 2: Check internal Markdown and wiki links manually**

Run:

```powershell
rg -n "\[\[[^]]+\]\]|\]\([^)]*\.md" README.md "00-收件箱" "01-日记与周期" "02-项目管理" "04-知识笔记" "05-资源库"
```

Expected: inspect each changed-file link and confirm its target exists.

- [ ] **Step 3: Scan for stale dates and conflicting daily obligations**

Run:

```powershell
rg -n "2026-06-15|概率论|距离考试：20天|每天.*番茄|每天.*博客|每天.*Token|每天晚上.*AI|同时.*项目.*2" README.md "00-收件箱" "01-日记与周期/周复盘模板.md" "02-项目管理/项目SOP.md" "04-知识笔记/知识处理SOP.md" "05-资源库/AI使用公约.md"
```

Expected: no active-rule matches. Historical journal entries outside these paths are out of scope.

- [ ] **Step 4: Run the missed-day recovery scenario**

Read only `README.md` and `00-收件箱/00-当前状态面板.md`. Confirm the answer to “I missed yesterday; what do I do now?” is visible without opening another file: perform the current five-minute action and do not repay yesterday's task debt.

- [ ] **Step 5: Run the project-start scenario**

Read `02-项目管理/项目SOP.md`. Confirm a user can create a small learning project using the lightweight gate without completing PRD, ADR, multi-Agent, monetization and release checklists. Confirm the reference manual is linked for optional depth.

- [ ] **Step 6: Run the first-time-reader scenario**

Read only `README.md` and the current panel. Confirm the reader can identify today's single goal, next action, minimum line, stopping condition and recovery action.

- [ ] **Step 7: Report completion evidence**

Report changed files, the verification commands and any remaining intentionally inactive modules. Do not claim Git history or a commit because this workspace has no `.git` repository.

