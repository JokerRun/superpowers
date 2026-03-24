# [11] Requesting Code Review

## 速查卡 (Quick Reference)

| 维度           | 内容 |
|----------------|------|
| 触发时机       | 完成任务/主要功能实现、测试通过后、merge 前 |
| 调用链上游     | subagent-driven-development（每个任务后）、executing-plans（每批后）、systematic-debugging（bug 修复后） |
| 调用链下游     | receiving-code-review（reviewer 返回反馈后处理） |
| 核心产出       | code-reviewer subagent 发出的审查报告（Strengths / Issues / Assessment） |
| 关联 Subagents | superpowers:code-reviewer（专门的 code review subagent） |

---

## 一、原文

### SKILL.md

```
---
name: requesting-code-review
description: Use when completing tasks, implementing major features, or before merging to verify work meets requirements
---

# Requesting Code Review

Dispatch superpowers:code-reviewer subagent to catch issues before they cascade. The reviewer gets precisely crafted context for evaluation — never your session's history. This keeps the reviewer focused on the work product, not your thought process, and preserves your own context for continued work.

**Core principle:** Review early, review often.

## When to Request Review

**Mandatory:**
- After each task in subagent-driven development
- After completing major feature
- Before merge to main

**Optional but valuable:**
- When stuck (fresh perspective)
- Before refactoring (baseline check)
- After fixing complex bug

## How to Request

**1. Get git SHAs:**
```bash
BASE_SHA=$(git rev-parse HEAD~1)  # or origin/main
HEAD_SHA=$(git rev-parse HEAD)
```

**2. Dispatch code-reviewer subagent:**

Use Task tool with superpowers:code-reviewer type, fill template at `code-reviewer.md`

**Placeholders:**
- `{WHAT_WAS_IMPLEMENTED}` - What you just built
- `{PLAN_OR_REQUIREMENTS}` - What it should do
- `{BASE_SHA}` - Starting commit
- `{HEAD_SHA}` - Ending commit
- `{DESCRIPTION}` - Brief summary

**3. Act on feedback:**
- Fix Critical issues immediately
- Fix Important issues before proceeding
- Note Minor issues for later
- Push back if reviewer is wrong (with reasoning)

## Example

```
[Just completed Task 2: Add verification function]

You: Let me request code review before proceeding.

BASE_SHA=$(git log --oneline | grep "Task 1" | head -1 | awk '{print $1}')
HEAD_SHA=$(git rev-parse HEAD)

[Dispatch superpowers:code-reviewer subagent]
  WHAT_WAS_IMPLEMENTED: Verification and repair functions for conversation index
  PLAN_OR_REQUIREMENTS: Task 2 from docs/superpowers/plans/deployment-plan.md
  BASE_SHA: a7981ec
  HEAD_SHA: 3df7661
  DESCRIPTION: Added verifyIndex() and repairIndex() with 4 issue types

[Subagent returns]:
  Strengths: Clean architecture, real tests
  Issues:
    Important: Missing progress indicators
    Minor: Magic number (100) for reporting interval
  Assessment: Ready to proceed

You: [Fix progress indicators]
[Continue to Task 3]
```

## Integration with Workflows

**Subagent-Driven Development:**
- Review after EACH task
- Catch issues before they compound
- Fix before moving to next task

**Executing Plans:**
- Review after each batch (3 tasks)
- Get feedback, apply, continue

**Ad-Hoc Development:**
- Review before merge
- Review when stuck

## Red Flags

**Never:**
- Skip review because "it's simple"
- Ignore Critical issues
- Proceed with unfixed Important issues
- Argue with valid technical feedback

**If reviewer wrong:**
- Push back with technical reasoning
- Show code/tests that prove it works
- Request clarification

See template at: requesting-code-review/code-reviewer.md
```

---

### code-reviewer.md

```markdown
# Code Review Agent

You are reviewing code changes for production readiness.

**Your task:**
1. Review {WHAT_WAS_IMPLEMENTED}
2. Compare against {PLAN_OR_REQUIREMENTS}
3. Check code quality, architecture, testing
4. Categorize issues by severity
5. Assess production readiness

## What Was Implemented

{DESCRIPTION}

## Requirements/Plan

{PLAN_REFERENCE}

## Git Range to Review

**Base:** {BASE_SHA}
**Head:** {HEAD_SHA}

```bash
git diff --stat {BASE_SHA}..{HEAD_SHA}
git diff {BASE_SHA}..{HEAD_SHA}
```

## Review Checklist

**Code Quality:**
- Clean separation of concerns?
- Proper error handling?
- Type safety (if applicable)?
- DRY principle followed?
- Edge cases handled?

**Architecture:**
- Sound design decisions?
- Scalability considerations?
- Performance implications?
- Security concerns?

**Testing:**
- Tests actually test logic (not mocks)?
- Edge cases covered?
- Integration tests where needed?
- All tests passing?

**Requirements:**
- All plan requirements met?
- Implementation matches spec?
- No scope creep?
- Breaking changes documented?

**Production Readiness:**
- Migration strategy (if schema changes)?
- Backward compatibility considered?
- Documentation complete?
- No obvious bugs?

## Output Format

### Strengths
[What's well done? Be specific.]

### Issues

#### Critical (Must Fix)
[Bugs, security issues, data loss risks, broken functionality]

#### Important (Should Fix)
[Architecture problems, missing features, poor error handling, test gaps]

#### Minor (Nice to Have)
[Code style, optimization opportunities, documentation improvements]

**For each issue:**
- File:line reference
- What's wrong
- Why it matters
- How to fix (if not obvious)

### Recommendations
[Improvements for code quality, architecture, or process]

### Assessment

**Ready to merge?** [Yes/No/With fixes]

**Reasoning:** [Technical assessment in 1-2 sentences]

## Critical Rules

**DO:**
- Categorize by actual severity (not everything is Critical)
- Be specific (file:line, not vague)
- Explain WHY issues matter
- Acknowledge strengths
- Give clear verdict

**DON'T:**
- Say "looks good" without checking
- Mark nitpicks as Critical
- Give feedback on code you didn't review
- Be vague ("improve error handling")
- Avoid giving a clear verdict

## Example Output

```
### Strengths
- Clean database schema with proper migrations (db.ts:15-42)
- Comprehensive test coverage (18 tests, all edge cases)
- Good error handling with fallbacks (summarizer.ts:85-92)

### Issues

#### Important
1. **Missing help text in CLI wrapper**
   - File: index-conversations:1-31
   - Issue: No --help flag, users won't discover --concurrency
   - Fix: Add --help case with usage examples

2. **Date validation missing**
   - File: search.ts:25-27
   - Issue: Invalid dates silently return no results
   - Fix: Validate ISO format, throw error with example

#### Minor
1. **Progress indicators**
   - File: indexer.ts:130
   - Issue: No "X of Y" counter for long operations
   - Impact: Users don't know how long to wait

### Recommendations
- Add progress reporting for user experience
- Consider config file for excluded projects (portability)

### Assessment

**Ready to merge: With fixes**

**Reasoning:** Core implementation is solid with good architecture and tests. Important issues (help text, date validation) are easily fixed and don't affect core functionality.
```
```

---

## 二、中文翻译

### SKILL.md 翻译

**请求 Code Review**

dispatch superpowers:code-reviewer subagent，在问题扩散之前捕获它们。reviewer 获得精确定制的上下文进行评估——永远不是你的 session 历史记录。这保持 reviewer 专注于工作产出，而不是你的思考过程，同时也保护了你自己的上下文用于继续工作。

**核心原则：** 早 review，常 review。

**何时请求 review**

**必须：**
- subagent-driven development 中每个任务完成后
- 完成主要功能后
- merge 到 main 之前

**可选但有价值：**
- 卡住时（换个视角）
- 重构之前（建立 baseline）
- 修复复杂 bug 之后

**如何请求**

1. 获取 git SHA：
   ```bash
   BASE_SHA=$(git rev-parse HEAD~1)  # 或 origin/main
   HEAD_SHA=$(git rev-parse HEAD)
   ```

2. dispatch code-reviewer subagent，使用 `code-reviewer.md` 模板填写占位符

3. 处理反馈：
   - 立即修复 Critical 问题
   - 继续之前修复 Important 问题
   - 记录 Minor 问题留待后续
   - 如果 reviewer 错了，用推理反驳

**与工作流的集成**

- **Subagent-Driven Development**：每个任务后都 review，在问题叠加之前捕获
- **Executing Plans**：每批（3 个任务）后 review
- **Ad-Hoc 开发**：merge 前 review，卡住时 review

---

### code-reviewer.md 翻译

**Code Review Agent**

你正在审查代码变更的生产就绪性。

**你的任务：**
1. 审查 {WHAT_WAS_IMPLEMENTED}
2. 与 {PLAN_OR_REQUIREMENTS} 对比
3. 检查代码质量、架构、测试
4. 按严重程度分类问题
5. 评估生产就绪性

**审查清单：**

- **代码质量**：关注点清晰分离？错误处理正确？DRY 原则遵守？边界情况处理？
- **架构**：设计决策合理？可扩展性？性能影响？安全顾虑？
- **测试**：测试实际验证逻辑（而非只是 mock）？边界情况覆盖？
- **需求**：所有计划需求都满足？实现匹配 spec？无 scope creep？
- **生产就绪性**：migration 策略？向后兼容性？文档完整？

**输出格式**：Strengths（优点）→ Issues（Critical / Important / Minor）→ Recommendations → Assessment（Ready to merge? Yes/No/With fixes）

**关键规则**：按实际严重程度分类（不是所有问题都是 Critical）；具体指出 file:line；解释为什么问题重要；给出明确的裁定。

---

## 三、剖析解读

### 3.1 功能与定位

requesting-code-review 解决的问题是：**代码实现者（无论是人还是 AI）天然存在确认偏误——倾向于觉得自己的实现是正确的**。一个独立的 reviewer 能发现实现者看不到的问题。

这个 skill 的设计有一个关键工程决策：**reviewer 不继承 controller 的 session 历史**。通过向 reviewer subagent 提供精心构建的上下文（diff + 需求 + 描述），而不是传递整个对话历史，可以：
1. 保持 reviewer 专注于代码本身，而非实现过程
2. 避免 reviewer 受实现者思路的影响
3. 保护 controller 的上下文窗口

在 subagent-driven-development 工作流中，这是 Two-Stage Review 的第二阶段（spec compliance 审查是第一阶段），专注于代码质量。

### 3.2 使用场景与案例

**场景：完成了优惠券模块，准备进入 code review**

假设 Task 3（实现 apply_coupon 函数）的 implementer 报告 DONE，spec compliance 已通过，现在进入代码质量 review：

```bash
# 1. 获取 SHA 范围
BASE_SHA=$(git rev-parse HEAD~2)   # Task 3 开始前的提交
HEAD_SHA=$(git rev-parse HEAD)     # Task 3 的最终提交

# 2. dispatch code-reviewer subagent，填写模板：
WHAT_WAS_IMPLEMENTED: apply_coupon_discount() 和相关测试
PLAN_OR_REQUIREMENTS: docs/superpowers/plans/coupon-plan.md Task 3
BASE_SHA: abc123
HEAD_SHA: def456
DESCRIPTION: 实现了百分比折扣、固定金额折扣和最大折扣上限功能
```

Reviewer 返回：
```
### Strengths
- 清晰的单一职责函数 (coupon.py:15-45)
- 9/9 测试覆盖边界情况

### Issues

#### Important
1. **缺少负值折扣验证**
   - File: coupon.py:23
   - Issue: discount_rate 可以是负数，会增加价格
   - Fix: assert discount_rate >= 0

#### Minor
1. **魔法数字**
   - File: coupon.py:41
   - Issue: MAX_DISCOUNT_RATE = 0.99 未命名
   - Fix: 提取为常量

### Assessment: With fixes
```

**处理流程**：
- Important（缺少验证）→ 立即修复，然后 re-review
- Minor（魔法数字）→ 本次一并修复
- Reviewer 批准后 → 标记 Task 3 完成，继续 Task 4

### 3.3 Subagents / References 深度解读

**code-reviewer.md**

这是 reviewer subagent 的完整 prompt 模板，包含：

1. **五维 Review Checklist**：代码质量 / 架构 / 测试 / 需求 / 生产就绪性。每个维度都有具体的检查项，而不是模糊的"代码要好"。

2. **三级严重程度分类**：
   - Critical（必须修复）：bugs、安全问题、数据丢失风险、功能中断
   - Important（应该修复）：架构问题、缺失功能、测试缺口
   - Minor（Nice to Have）：代码风格、优化机会

3. **关键规则**：
   - 每个问题必须有 `file:line` 引用（具体，不模糊）
   - 必须给出明确的 verdict（Yes/No/With fixes），不能含糊
   - 不是所有问题都是 Critical——按实际严重程度分类

4. **完整示例输出**：展示了理想输出格式，帮助 reviewer 理解期望的粒度。

### 3.4 流程图 / 示意图

**Code Review 请求流程**

```
任务/功能实现完成
      │
      ▼
verification-before-completion
（确认测试通过有证据）
      │
      ▼
获取 git SHA 范围
BASE_SHA: 起始提交
HEAD_SHA: 最终提交
      │
      ▼
dispatch code-reviewer subagent
提供：WHAT / REQUIREMENTS / BASE_SHA / HEAD_SHA / DESCRIPTION
（精确构建上下文，不传递 session 历史）
      │
      ▼
Reviewer 审查
  ├── 运行 git diff BASE_SHA..HEAD_SHA
  ├── 对照 5 维 checklist
  └── 分类问题（Critical / Important / Minor）
      │
      ▼
Reviewer 返回报告
  ├── Strengths（优点）
  ├── Issues（分级）
  └── Assessment（Ready to merge?）
      │
 ┌────┴────┐
 ▼         ▼
无问题    有问题
  │         │
  ▼         ▼
继续下一   Critical → 立即修复
个任务     Important → 修复后继续
           Minor → 记录，视情况修复
               │
               ▼
           修复后重新 dispatch reviewer
```

### 3.5 与其他 Skills 的协作关系

| 协作 Skill | 关系类型 | 说明 |
|------------|----------|------|
| subagent-driven-development | 上游 | SDD 在每个任务的 code quality review 阶段调用此 skill |
| executing-plans | 上游 | 每个 checkpoint 批次完成后调用此 skill |
| verification-before-completion | 前置 | 在 request review 之前必须先有验证证据 |
| receiving-code-review | 下游 | reviewer 返回反馈后，用 receiving-code-review skill 处理反馈 |
| finishing-a-development-branch | 间接下游 | 所有 review 通过后才能 finishing branch |
