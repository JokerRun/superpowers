# 05 executing-plans

## 速查卡 (Quick Reference)
| 维度           | 内容 |
|----------------|------|
| 触发时机       | 已有一份写好的实现计划，需要在当前会话内逐步执行 |
| 调用链上游     | writing-plans（产出计划）、using-git-worktrees（建立工作区） |
| 调用链下游     | finishing-a-development-branch（所有任务完成后） |
| 核心产出       | 按 checkpoint 批次完整执行计划中的所有任务，每批暂停等待 review |
| 关联 Subagents | 无 |

---

## 一、原文

### SKILL.md

```
---
name: executing-plans
description: Use when you have a written implementation plan to execute in a separate session with review checkpoints
---

# Executing Plans

## Overview

Load plan, review critically, execute all tasks, report when complete.

**Announce at start:** "I'm using the executing-plans skill to implement this plan."

**Note:** Tell your human partner that Superpowers works much better with access to subagents. The quality of its work will be significantly higher if run on a platform with subagent support (such as Claude Code or Codex). If subagents are available, use superpowers:subagent-driven-development instead of this skill.

## The Process

### Step 1: Load and Review Plan
1. Read plan file
2. Review critically - identify any questions or concerns about the plan
3. If concerns: Raise them with your human partner before starting
4. If no concerns: Create TodoWrite and proceed

### Step 2: Execute Tasks

For each task:
1. Mark as in_progress
2. Follow each step exactly (plan has bite-sized steps)
3. Run verifications as specified
4. Mark as completed

### Step 3: Complete Development

After all tasks complete and verified:
- Announce: "I'm using the finishing-a-development-branch skill to complete this work."
- **REQUIRED SUB-SKILL:** Use superpowers:finishing-a-development-branch
- Follow that skill to verify tests, present options, execute choice

## When to Stop and Ask for Help

**STOP executing immediately when:**
- Hit a blocker (missing dependency, test fails, instruction unclear)
- Plan has critical gaps preventing starting
- You don't understand an instruction
- Verification fails repeatedly

**Ask for clarification rather than guessing.**

## When to Revisit Earlier Steps

**Return to Review (Step 1) when:**
- Partner updates the plan based on your feedback
- Fundamental approach needs rethinking

**Don't force through blockers** - stop and ask.

## Remember
- Review plan critically first
- Follow plan steps exactly
- Don't skip verifications
- Reference skills when plan says to
- Stop when blocked, don't guess
- Never start implementation on main/master branch without explicit user consent

## Integration

**Required workflow skills:**
- **superpowers:using-git-worktrees** - REQUIRED: Set up isolated workspace before starting
- **superpowers:writing-plans** - Creates the plan this skill executes
- **superpowers:finishing-a-development-branch** - Complete development after all tasks
```

---

## 二、中文翻译

### SKILL.md 翻译

```
---
name: executing-plans
description: 当你有一份已写好的实现计划，需要在独立会话中按 review checkpoint 执行时使用
---
```

**执行计划 (Executing Plans)**

**概述**

加载计划，批判性审阅，执行所有任务，完成后汇报。

**开始时宣告：** "I'm using the executing-plans skill to implement this plan."

**注意：** 告知你的人类伙伴，Superpowers 在有 subagent 访问权限的情况下效果更好。若在支持 subagent 的平台（如 Claude Code 或 Codex）上运行，产出质量将显著更高。如果 subagent 可用，请使用 superpowers:subagent-driven-development 替代本 skill。

---

**流程**

**Step 1：加载并审阅计划**
1. 读取计划文件
2. 批判性审阅——识别计划中的任何疑问或顾虑
3. 若有顾虑：在开始前与人类伙伴沟通确认
4. 若无顾虑：创建 TodoWrite 并继续执行

**Step 2：执行任务**

对每个任务：
1. 标记为 in_progress
2. 严格按照每个步骤执行（计划已拆解为小步骤）
3. 按说明运行验证
4. 标记为 completed

**Step 3：完成开发**

所有任务完成并验证后：
- 宣告："I'm using the finishing-a-development-branch skill to complete this work."
- **必须调用的子 Skill：** 使用 superpowers:finishing-a-development-branch
- 按该 skill 的指引验证测试、展示选项、执行选择

---

**何时停下来寻求帮助**

**立即停止执行，当遇到：**
- 阻塞问题（缺少依赖、测试失败、指令不清晰）
- 计划存在关键缺口，无法开始
- 不理解某条指令
- 验证反复失败

**请求澄清，而非猜测。**

---

**何时回溯到之前步骤**

**返回 Review（Step 1），当：**
- 伙伴根据你的反馈更新了计划
- 基本方案需要重新思考

**不要强行突破阻塞** ——停下来，询问。

---

**注意事项**
- 先批判性地审阅计划
- 严格按计划步骤执行
- 不要跳过验证
- 计划要求时调用相关 skills
- 遇到阻塞时停止，不要猜测
- 未经用户明确同意，不得在 main/master 分支上开始实现

---

**集成 (Integration)**

**必需的工作流 Skills：**
- **superpowers:using-git-worktrees** — 必须：开始前建立隔离工作区
- **superpowers:writing-plans** — 产出本 skill 所执行的计划
- **superpowers:finishing-a-development-branch** — 所有任务完成后收尾开发

---

## 三、剖析解读

### 3.1 功能与定位

executing-plans 是 Superpowers 工作流的**执行引擎**，定位非常聚焦：在"已有一份写好的实现计划"的前提下，在**当前会话内**按顺序执行所有任务，每批完成后暂停等待人工 review，再继续。

它不负责生成计划（那是 writing-plans 的职责），也不负责收尾分支（那是 finishing-a-development-branch 的职责）。它只做一件事：**忠实地执行计划**。

与 subagent-driven-development 的核心区别：

- executing-plans：**inline**，在当前会话内执行，上下文连续，每批 checkpoint 后暂停等待人工确认
- subagent-driven-development：**fresh subagent per task**，每个任务启动全新 subagent，上下文隔离，自动循环

SKILL.md 本身也明确建议：**如果平台支持 subagent，优先选择 subagent-driven-development**——executing-plans 是 subagent 不可用时的备选方案。

### 3.2 使用场景与案例

**典型案例：** 假设有一份 10 任务的优惠券系统实现计划，由于运行环境不支持 subagent，选择 inline execution——executing-plans 将任务分为每批 2-3 个 checkpoint，执行 Task 1-3 后暂停，展示完成情况等待 review，用户确认后继续执行 Task 4-6，以此类推直到计划完成，最后调用 finishing-a-development-branch 收尾。

**选择时机对比表：**

| 维度 | executing-plans | subagent-driven-development |
|------|----------------|----------------------------|
| 执行位置 | 当前会话 (inline) | 当前会话（但 fresh subagent） |
| 上下文 | 继承当前上下文 | 每任务全新上下文 |
| Review 时机 | 每批 checkpoint 后暂停 | 每任务后（spec + quality 两次）|
| 适用场景 | 紧密依赖的任务序列 | 相对独立的任务集合 |
| 人工介入 | 每批次等待确认 | 自动循环到下一任务 |
| 推荐条件 | subagent 不可用时 | subagent 可用时（优先） |

**何时该用 executing-plans：**
1. 运行平台不支持 subagent（如部分 API 接入场景）
2. 任务间依赖极为紧密，需要共享上下文状态
3. 希望更直接、更细粒度地介入每个执行批次

### 3.4 流程图 / 示意图

**整体执行流程：**

```
[有一份写好的计划]
        |
        v
  Step 1: 加载 & 批判性审阅
        |
   有顾虑? ——是——> 与用户沟通澄清 ——> [用户更新计划] ——> 返回 Step 1
        |
        否
        |
        v
  创建 TodoWrite
        |
        v
  Step 2: 执行任务（逐个 checkpoint 批次）
        |
   遇到阻塞? ——是——> 立即停止，请求帮助
        |
        否
        |
        v
  所有任务完成 & 验证通过
        |
        v
  Step 3: 调用 finishing-a-development-branch
        |
        v
     [完成]
```

**checkpoint 机制示意图：**

```
Plan: [T1][T2][T3] | [T4][T5][T6] | [T7][T8][T9][T10]
                   ^checkpoint     ^checkpoint      ^完成
      执行T1-T3 -> 暂停review -> 执行T4-T6 -> 暂停review -> 执行T7-T10 -> 完成
                  (等待用户确认)              (等待用户确认)
```

**任务状态机：**

```
[pending] -> [in_progress] -> [completed]
                  |
                  v (遇到阻塞)
             [STOP & ASK]
```

### 3.5 与其他 Skills 的协作关系

| 协作 Skill | 关系类型 | 说明 |
|------------|----------|------|
| writing-plans | 上游 | 产出 executing-plans 所执行的计划文件 |
| using-git-worktrees | 上游（必需） | 执行计划前必须先建立隔离的 git worktree 工作区 |
| finishing-a-development-branch | 下游（必需） | 所有任务完成并验证后，强制调用此 skill 收尾 |
| subagent-driven-development | 并列 / 替代 | 两者均为计划执行方案，subagent 可用时优先选后者 |
| verification-before-completion | 下游（间接） | 通过 finishing-a-development-branch 间接触发验证 |
| requesting-code-review | 下游（间接） | 通过 finishing-a-development-branch 间接触发 code review |
| test-driven-development | 内部调用 | 执行计划中涉及测试任务时按需调用 |
| systematic-debugging | 内部调用 | 执行中遇到重复失败的验证时调用辅助排查 |
