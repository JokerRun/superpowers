# 03 writing-plans

## 速查卡 (Quick Reference)
| 维度           | 内容 |
|----------------|------|
| 触发时机       | 拥有已批准的 spec / 需求文档，准备动手写代码之前 |
| 调用链上游     | brainstorming（输出 spec，创建 worktree） |
| 调用链下游     | using-git-worktrees、executing-plans、subagent-driven-development |
| 核心产出       | `docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md` — 可逐步执行的任务清单 |
| 关联 Subagents | plan-document-reviewer (via plan-document-reviewer-prompt.md) |

---

## 一、原文

### SKILL.md

````
---
name: writing-plans
description: Use when you have a spec or requirements for a multi-step task, before touching code
---

# Writing Plans

## Overview

Write comprehensive implementation plans assuming the engineer has zero context for our codebase and questionable taste. Document everything they need to know: which files to touch for each task, code, testing, docs they might need to check, how to test it. Give them the whole plan as bite-sized tasks. DRY. YAGNI. TDD. Frequent commits.

Assume they are a skilled developer, but know almost nothing about our toolset or problem domain. Assume they don't know good test design very well.

**Announce at start:** "I'm using the writing-plans skill to create the implementation plan."

**Context:** This should be run in a dedicated worktree (created by brainstorming skill).

**Save plans to:** `docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md`
- (User preferences for plan location override this default)

## Scope Check

If the spec covers multiple independent subsystems, it should have been broken into sub-project specs during brainstorming. If it wasn't, suggest breaking this into separate plans — one per subsystem. Each plan should produce working, testable software on its own.

## File Structure

Before defining tasks, map out which files will be created or modified and what each one is responsible for. This is where decomposition decisions get locked in.

- Design units with clear boundaries and well-defined interfaces. Each file should have one clear responsibility.
- You reason best about code you can hold in context at once, and your edits are more reliable when files are focused. Prefer smaller, focused files over large ones that do too much.
- Files that change together should live together. Split by responsibility, not by technical layer.
- In existing codebases, follow established patterns. If the codebase uses large files, don't unilaterally restructure - but if a file you're modifying has grown unwieldy, including a split in the plan is reasonable.

This structure informs the task decomposition. Each task should produce self-contained changes that make sense independently.

## Bite-Sized Task Granularity

**Each step is one action (2-5 minutes):**
- "Write the failing test" - step
- "Run it to make sure it fails" - step
- "Implement the minimal code to make the test pass" - step
- "Run the tests and make sure they pass" - step
- "Commit" - step

## Plan Document Header

**Every plan MUST start with this header:**

```markdown
# [Feature Name] Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** [One sentence describing what this builds]

**Architecture:** [2-3 sentences about approach]

**Tech Stack:** [Key technologies/libraries]

---
````

## Task Structure

````markdown
### Task N: [Component Name]

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

- [ ] **Step 1: Write the failing test**

```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pytest tests/path/test.py::test_name -v`
Expected: FAIL with "function not defined"

- [ ] **Step 3: Write minimal implementation**

```python
def function(input):
    return expected
```

- [ ] **Step 4: Run test to verify it passes**

Run: `pytest tests/path/test.py::test_name -v`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add tests/path/test.py src/path/file.py
git commit -m "feat: add specific feature"
```
````

## Remember
- Exact file paths always
- Complete code in plan (not "add validation")
- Exact commands with expected output
- Reference relevant skills with @ syntax
- DRY, YAGNI, TDD, frequent commits

## Plan Review Loop

After writing the complete plan:

1. Dispatch a single plan-document-reviewer subagent (see plan-document-reviewer-prompt.md) with precisely crafted review context — never your session history. This keeps the reviewer focused on the plan, not your thought process.
   - Provide: path to the plan document, path to spec document
2. If ❌ Issues Found: fix the issues, re-dispatch reviewer for the whole plan
3. If ✅ Approved: proceed to execution handoff

**Review loop guidance:**
- Same agent that wrote the plan fixes it (preserves context)
- If loop exceeds 3 iterations, surface to human for guidance
- Reviewers are advisory — explain disagreements if you believe feedback is incorrect

## Execution Handoff

After saving the plan, offer execution choice:

**"Plan complete and saved to `docs/superpowers/plans/<filename>.md`. Two execution options:**

**1. Subagent-Driven (recommended)** - I dispatch a fresh subagent per task, review between tasks, fast iteration

**2. Inline Execution** - Execute tasks in this session using executing-plans, batch execution with checkpoints

**Which approach?"**

**If Subagent-Driven chosen:**
- **REQUIRED SUB-SKILL:** Use superpowers:subagent-driven-development
- Fresh subagent per task + two-stage review

**If Inline Execution chosen:**
- **REQUIRED SUB-SKILL:** Use superpowers:executing-plans
- Batch execution with checkpoints for review
```

### plan-document-reviewer-prompt.md

```
# Plan Document Reviewer Prompt Template

Use this template when dispatching a plan document reviewer subagent.

**Purpose:** Verify the plan is complete, matches the spec, and has proper task decomposition.

**Dispatch after:** The complete plan is written.

```
Task tool (general-purpose):
  description: "Review plan document"
  prompt: |
    You are a plan document reviewer. Verify this plan is complete and ready for implementation.

    **Plan to review:** [PLAN_FILE_PATH]
    **Spec for reference:** [SPEC_FILE_PATH]

    ## What to Check

    | Category | What to Look For |
    |----------|------------------|
    | Completeness | TODOs, placeholders, incomplete tasks, missing steps |
    | Spec Alignment | Plan covers spec requirements, no major scope creep |
    | Task Decomposition | Tasks have clear boundaries, steps are actionable |
    | Buildability | Could an engineer follow this plan without getting stuck? |

    ## Calibration

    **Only flag issues that would cause real problems during implementation.**
    An implementer building the wrong thing or getting stuck is an issue.
    Minor wording, stylistic preferences, and "nice to have" suggestions are not.

    Approve unless there are serious gaps — missing requirements from the spec,
    contradictory steps, placeholder content, or tasks so vague they can't be acted on.

    ## Output Format

    ## Plan Review

    **Status:** Approved | Issues Found

    **Issues (if any):**
    - [Task X, Step Y]: [specific issue] - [why it matters for implementation]

    **Recommendations (advisory, do not block approval):**
    - [suggestions for improvement]
```

**Reviewer returns:** Status, Issues (if any), Recommendations
```

---

## 二、中文翻译

### SKILL.md 翻译

**元数据**
- name: writing-plans
- description: 当你拥有多步骤任务的 spec 或需求文档、尚未动手写代码时使用

---

**# Writing Plans**

**## Overview（概述）**

假设执行工程师对代码库毫无上下文、品味存疑，编写全面的实现计划。记录他们需要知道的一切：每个任务需要改动哪些文件、代码、测试、可能需要查阅的文档、如何测试。将完整计划分解为小粒度任务交给他们。DRY、YAGNI、TDD、频繁 commit。

假设他们是熟练开发者，但对我们的工具集和问题域几乎一无所知。假设他们对良好的测试设计不太熟悉。

**开始时声明：** "I'm using the writing-plans skill to create the implementation plan."

**上下文：** 应在专用 worktree 中运行（由 brainstorming skill 创建）。

**保存计划至：** `docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md`
- （用户偏好设置的计划位置优先于此默认值）

**## Scope Check（范围检查）**

若 spec 覆盖多个独立子系统，应在 brainstorming 阶段就拆分为子项目 spec。若未拆分，建议拆成多个计划——每个子系统一份。每份计划应能独立产出可运行、可测试的软件。

**## File Structure（文件结构）**

在定义任务之前，先列出哪些文件将被创建或修改，以及每个文件的职责。这是锁定分解决策的关键步骤。

- 设计具有清晰边界和良好定义接口的单元。每个文件应有一个明确的职责。
- 你对能一次性放入上下文的代码推理最好，文件聚焦时你的编辑也更可靠。偏好小而专注的文件，而非承担过多职责的大文件。
- 经常一起变更的文件应放在一起。按职责拆分，而非按技术层次。
- 在已有代码库中，遵循既有模式。若代码库使用大文件，不要单方面重构——但若你正在修改的文件已变得难以管理，在计划中纳入拆分是合理的。

文件结构指导任务分解。每个任务应产出独立的、有意义的变更。

**## Bite-Sized Task Granularity（小粒度任务粒度）**

**每个步骤是一个动作（2-5 分钟）：**
- "写失败测试" — 一个步骤
- "运行确认其失败" — 一个步骤
- "编写使测试通过的最小实现" — 一个步骤
- "运行测试确认通过" — 一个步骤
- "Commit" — 一个步骤

**## Plan Document Header（计划文档头部）**

**每份计划必须以此头部开头：**（格式见原文，内容不变）

**## Task Structure（任务结构）**

（格式见原文，包含精确文件路径、完整代码块、精确命令及预期输出）

**## Remember（注意事项）**
- 始终使用精确文件路径
- 计划中包含完整代码（而非"添加校验"之类的描述）
- 带预期输出的精确命令
- 使用 @ 语法引用相关 skills
- DRY、YAGNI、TDD、频繁 commit

**## Plan Review Loop（计划审查循环）**

完整计划写完后：

1. 分发一个 plan-document-reviewer subagent（见 plan-document-reviewer-prompt.md），提供精心构造的审查上下文——绝不是你的会话历史。这使审查者专注于计划本身，而非你的思考过程。
   - 提供：计划文档路径、spec 文档路径
2. 若 ❌ 发现问题：修复问题，重新分发审查者对整份计划进行审查
3. 若 ✅ 通过：进入执行交接

**审查循环指导：**
- 同一个写计划的 agent 来修复（保留上下文）
- 若循环超过 3 次，上报给人类寻求指导
- 审查者是建议性的——若你认为反馈有误，解释你的不同意见

**## Execution Handoff（执行交接）**

保存计划后，提供执行方式选择：

提供两个选项：
1. **Subagent-Driven（推荐）** — 每个任务分发一个新的 subagent，任务间审查，快速迭代
2. **Inline Execution** — 在本会话中使用 executing-plans 执行，带检查点的批量执行

---

### plan-document-reviewer-prompt.md 翻译

**# 计划文档审查者提示词模板**

分发 plan-document-reviewer subagent 时使用此模板。

**目的：** 验证计划完整、与 spec 匹配、任务分解合理。

**分发时机：** 完整计划写完后。

**Task tool（通用型）：**
- description: "审查计划文档"
- prompt: 你是一名计划文档审查者。验证此计划完整且已准备好进行实现。

**检查内容（四个维度）：**

| 类别 | 检查内容 |
|------|----------|
| Completeness（完整性） | TODO、占位符、未完成任务、缺失步骤 |
| Spec Alignment（与 spec 对齐） | 计划覆盖 spec 要求，无重大范围蔓延 |
| Task Decomposition（任务分解） | 任务边界清晰，步骤可操作 |
| Buildability（可构建性） | 工程师能否跟着计划做而不会卡住？ |

**校准原则：** 只标记那些在实现过程中会造成真实问题的 issue。实现者做出错误的东西或卡住是问题；措辞细节、风格偏好和"锦上添花"的建议不是。除非存在严重缺口，否则批准通过。

**输出格式：**
- Status: Approved | Issues Found
- Issues（若有）：具体到 Task X、Step Y 及原因
- Recommendations（建议性，不阻塞批准）

**审查者返回：** Status、Issues（若有）、Recommendations

---

## 三、剖析解读

### 3.1 功能与定位

writing-plans 的核心职责是将已批准的 spec **转化为可由 agent 逐步执行的任务清单**。

该 skill 有一个关键设计假设："**engineer has zero context**"——计划的读者（无论是人类工程师还是执行 agent）对代码库一无所知、对问题域几乎陌生、对测试设计掌握有限。这个假设驱动了所有设计决策：

- 计划必须**自包含**：包含精确文件路径、完整可运行代码、精确命令和预期输出
- 计划必须**原子化**：每个步骤 2-5 分钟，不可再拆
- 计划必须**可验证**：每个实现步骤后紧跟验证步骤（TDD 节奏）

三条核心原则贯穿全文：
- **DRY**（Don't Repeat Yourself）：文件结构设计时避免重复逻辑
- **YAGNI**（You Ain't Gonna Need It）：只实现 spec 要求的内容
- **TDD**（Test-Driven Development）：先写失败测试，再写最小实现

### 3.2 使用场景与案例

**典型触发场景：** brainstorming skill 输出了一份优惠券系统的 spec，并创建了专用 worktree，此时 writing-plans 被调用将 spec 转化为执行计划。

**假设 brainstorming 已输出优惠券系统的 spec，writing-plans 会将其拆解为以下粒度：**

```
Task 1: CouponValidator 模块
  Files:
    Create: src/coupons/validator.py
    Test:   tests/coupons/test_validator.py

  - [ ] Step 1: 写失败测试
        def test_valid_coupon_returns_discount():
            result = validate_coupon("SAVE10", order_total=100)
            assert result == {"valid": True, "discount": 10}

  - [ ] Step 2: 运行确认失败
        Run: pytest tests/coupons/test_validator.py -v
        Expected: FAIL with "cannot import name 'validate_coupon'"

  - [ ] Step 3: 写最小实现
        def validate_coupon(code, order_total): ...

  - [ ] Step 4: 运行确认通过
        Run: pytest tests/coupons/test_validator.py -v
        Expected: PASS

  - [ ] Step 5: Commit
        git commit -m "feat: add coupon validator"
```

每个 Task 之间有清晰边界，每个 Task 可独立提交，可独立由 subagent 执行。

**另一典型场景：Scope Check 的价值**

若 spec 同时涵盖"优惠券引擎"和"用户积分系统"两个独立子系统，writing-plans 会在开始前提示拆分为两份独立计划，而不是在同一计划里混合实现——这保证了每份计划产出可独立测试的软件。

### 3.3 Subagents / References 深度解读

**plan-document-reviewer** 是 writing-plans 内部 Plan Review Loop 所 dispatch 的 subagent，其完整提示词模板定义在 `plan-document-reviewer-prompt.md`。

**调用时机：** 完整计划写完之后、进入执行交接之前。

**四个检查维度的意义：**

| 维度 | 意义 | 典型失败案例 |
|------|------|-------------|
| Completeness | 计划不能有"TODO: 补充实现"之类的占位符 | agent 执行到某步骤发现内容缺失，无法继续 |
| Spec Alignment | 计划不能遗漏 spec 要求，也不能无端扩展范围 | agent 交付后发现某个需求没有覆盖 |
| Task Decomposition | 任务边界必须清晰，步骤必须可操作 | agent 面对"集成所有模块"这种模糊任务不知道从哪入手 |
| Buildability | 工程师/agent 跟着做不会卡住 | 命令缺少参数、路径写错、依赖未声明 |

**关键设计决策：精确的上下文隔离。** dispatch 时提供的是「计划文档路径 + spec 文档路径」，而**不是**会话历史。这确保审查者以"第一次看到这份计划"的视角进行评审，与真实的执行 agent 视角一致。

**审查循环上限为 3 次**，超出后上报人类——防止 agent 陷入无效的自我修正循环。

**校准原则值得特别注意：** 审查者被明确要求"只标记会造成真实问题的 issue"，避免过度审查阻塞流程。这是一个有意识的"宽松审查"设计，优先保证流程流畅。

### 3.4 流程图 / 示意图

**计划文档结构层次：**

```
Plan Document
├── Header (mandatory)
│   ├── Goal (one sentence)
│   ├── Architecture (2-3 sentences)
│   └── Tech Stack
│
├── File Map
│   ├── file-a.py  → responsibility A
│   ├── file-b.py  → responsibility B
│   └── test_a.py  → tests for A
│
└── Tasks (1..N)
    ├── Task 1
    │   ├── Files: Create / Modify / Test (exact paths)
    │   ├── Step 1: Write failing test  [code block]
    │   ├── Step 2: Run → verify FAIL   [exact command + expected output]
    │   ├── Step 3: Write minimal impl  [code block]
    │   ├── Step 4: Run → verify PASS   [exact command + expected output]
    │   └── Step 5: Commit              [exact git command]
    └── Task N
        └── ...
```

**writing-plans 完整工作流：**

```
brainstorming
  (spec + worktree)
       |
       v
  writing-plans
       |
       +--> [1] Scope Check
       |         多子系统? --> 建议拆分
       |
       +--> [2] File Structure Map
       |         锁定分解决策
       |
       +--> [3] Task Decomposition
       |         TDD节奏, 2-5min/step
       |
       +--> [4] Plan Review Loop
       |         dispatch plan-document-reviewer
       |              |
       |         Issues Found --> fix --> re-dispatch (max 3x)
       |              |
       |         Approved
       |
       +--> [5] Execution Handoff
                 |
          +------+------+
          |             |
    Subagent-Driven   Inline Execution
    (推荐)             executing-plans
    subagent-driven-
    development
```

**DRY / YAGNI / TDD 三原则关系：**

```
DRY ──────────→ File Structure 设计阶段
                (避免逻辑重复, 单一职责)

YAGNI ─────────→ Scope Check 阶段
                (只实现 spec 要求的内容)

TDD ───────────→ Task Step 设计阶段
                (先写失败测试, 再写最小实现)
```

### 3.5 与其他 Skills 的协作关系

| 协作 Skill | 关系类型 | 说明 |
|------------|----------|------|
| brainstorming | 上游 | 输出 spec 文档并创建 worktree，writing-plans 以此为输入 |
| plan-document-reviewer | 内部 subagent | writing-plans 在 Plan Review Loop 阶段 dispatch，不是独立 skill 而是内嵌审查机制 |
| using-git-worktrees | 下游（环境前提） | plan 完成后，worktree 是执行计划的隔离环境 |
| executing-plans | 下游（执行路径 B） | Inline Execution 选项，在当前会话批量执行计划任务 |
| subagent-driven-development | 下游（执行路径 A，推荐） | Subagent-Driven 选项，每个 Task 分发独立 subagent 执行 |
