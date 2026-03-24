# [12] Receiving Code Review

## 速查卡 (Quick Reference)

| 维度           | 内容 |
|----------------|------|
| 触发时机       | 收到 code review 反馈时，在实现任何建议之前 |
| 调用链上游     | requesting-code-review（review 完成后触发此 skill） |
| 调用链下游     | verification-before-completion（每个修复后验证）、finishing-a-development-branch |
| 核心产出       | 技术性确认或反驳 + 逐条实现 + 验证后报告 |
| 关联 Subagents | 无（在当前 session 中处理反馈） |

---

## 一、原文

### SKILL.md

```
---
name: receiving-code-review
description: Use when receiving code review feedback, before implementing suggestions, especially if feedback seems unclear or technically questionable - requires technical rigor and verification, not performative agreement or blind implementation
---

# Code Review Reception

## Overview

Code review requires technical evaluation, not emotional performance.

**Core principle:** Verify before implementing. Ask before assuming. Technical correctness over social comfort.

## The Response Pattern

```
WHEN receiving code review feedback:

1. READ: Complete feedback without reacting
2. UNDERSTAND: Restate requirement in own words (or ask)
3. VERIFY: Check against codebase reality
4. EVALUATE: Technically sound for THIS codebase?
5. RESPOND: Technical acknowledgment or reasoned pushback
6. IMPLEMENT: One item at a time, test each
```

## Forbidden Responses

**NEVER:**
- "You're absolutely right!" (explicit CLAUDE.md violation)
- "Great point!" / "Excellent feedback!" (performative)
- "Let me implement that now" (before verification)

**INSTEAD:**
- Restate the technical requirement
- Ask clarifying questions
- Push back with technical reasoning if wrong
- Just start working (actions > words)

## Handling Unclear Feedback

```
IF any item is unclear:
  STOP - do not implement anything yet
  ASK for clarification on unclear items

WHY: Items may be related. Partial understanding = wrong implementation.
```

**Example:**
```
your human partner: "Fix 1-6"
You understand 1,2,3,6. Unclear on 4,5.

❌ WRONG: Implement 1,2,3,6 now, ask about 4,5 later
✅ RIGHT: "I understand items 1,2,3,6. Need clarification on 4 and 5 before proceeding."
```

## Source-Specific Handling

### From your human partner
- **Trusted** - implement after understanding
- **Still ask** if scope unclear
- **No performative agreement**
- **Skip to action** or technical acknowledgment

### From External Reviewers
```
BEFORE implementing:
  1. Check: Technically correct for THIS codebase?
  2. Check: Breaks existing functionality?
  3. Check: Reason for current implementation?
  4. Check: Works on all platforms/versions?
  5. Check: Does reviewer understand full context?

IF suggestion seems wrong:
  Push back with technical reasoning

IF can't easily verify:
  Say so: "I can't verify this without [X]. Should I [investigate/ask/proceed]?"

IF conflicts with your human partner's prior decisions:
  Stop and discuss with your human partner first
```

**your human partner's rule:** "External feedback - be skeptical, but check carefully"

## YAGNI Check for "Professional" Features

```
IF reviewer suggests "implementing properly":
  grep codebase for actual usage

  IF unused: "This endpoint isn't called. Remove it (YAGNI)?"
  IF used: Then implement properly
```

**your human partner's rule:** "You and reviewer both report to me. If we don't need this feature, don't add it."

## Implementation Order

```
FOR multi-item feedback:
  1. Clarify anything unclear FIRST
  2. Then implement in this order:
     - Blocking issues (breaks, security)
     - Simple fixes (typos, imports)
     - Complex fixes (refactoring, logic)
  3. Test each fix individually
  4. Verify no regressions
```

## When To Push Back

Push back when:
- Suggestion breaks existing functionality
- Reviewer lacks full context
- Violates YAGNI (unused feature)
- Technically incorrect for this stack
- Legacy/compatibility reasons exist
- Conflicts with your human partner's architectural decisions

**How to push back:**
- Use technical reasoning, not defensiveness
- Ask specific questions
- Reference working tests/code
- Involve your human partner if architectural

**Signal if uncomfortable pushing back out loud:** "Strange things are afoot at the Circle K"

## Acknowledging Correct Feedback

When feedback IS correct:
```
✅ "Fixed. [Brief description of what changed]"
✅ "Good catch - [specific issue]. Fixed in [location]."
✅ [Just fix it and show in the code]

❌ "You're absolutely right!"
❌ "Great point!"
❌ "Thanks for catching that!"
❌ "Thanks for [anything]"
❌ ANY gratitude expression
```

**Why no thanks:** Actions speak. Just fix it. The code itself shows you heard the feedback.

**If you catch yourself about to write "Thanks":** DELETE IT. State the fix instead.

## Gracefully Correcting Your Pushback

If you pushed back and were wrong:
```
✅ "You were right - I checked [X] and it does [Y]. Implementing now."
✅ "Verified this and you're correct. My initial understanding was wrong because [reason]. Fixing."

❌ Long apology
❌ Defending why you pushed back
❌ Over-explaining
```

State the correction factually and move on.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Performative agreement | State requirement or just act |
| Blind implementation | Verify against codebase first |
| Batch without testing | One at a time, test each |
| Assuming reviewer is right | Check if breaks things |
| Avoiding pushback | Technical correctness > comfort |
| Partial implementation | Clarify all items first |
| Can't verify, proceed anyway | State limitation, ask for direction |

## Real Examples

**Performative Agreement (Bad):**
```
Reviewer: "Remove legacy code"
❌ "You're absolutely right! Let me remove that..."
```

**Technical Verification (Good):**
```
Reviewer: "Remove legacy code"
✅ "Checking... build target is 10.15+, this API needs 13+. Need legacy for backward compat. Current impl has wrong bundle ID - fix it or drop pre-13 support?"
```

**YAGNI (Good):**
```
Reviewer: "Implement proper metrics tracking with database, date filters, CSV export"
✅ "Grepped codebase - nothing calls this endpoint. Remove it (YAGNI)? Or is there usage I'm missing?"
```

**Unclear Item (Good):**
```
your human partner: "Fix items 1-6"
You understand 1,2,3,6. Unclear on 4,5.
✅ "Understand 1,2,3,6. Need clarification on 4 and 5 before implementing."
```

## GitHub Thread Replies

When replying to inline review comments on GitHub, reply in the comment thread (`gh api repos/{owner}/{repo}/pulls/{pr}/comments/{id}/replies`), not as a top-level PR comment.

## The Bottom Line

**External feedback = suggestions to evaluate, not orders to follow.**

Verify. Question. Then implement.

No performative agreement. Technical rigor always.
```

---

## 二、中文翻译

### SKILL.md 翻译

**Code Review 接收**

**概述**

Code review 需要技术性评估，而不是情绪表演。

**核心原则：** 实现前验证。假设前提问。技术正确性优先于社交舒适度。

**响应模式**

```
收到 code review 反馈时：

1. 阅读：完整读取反馈，不立即反应
2. 理解：用自己的话重述需求（或提问）
3. 验证：对照代码库现实检查
4. 评估：对这个代码库技术上合理吗？
5. 响应：技术性确认或有理由的反驳
6. 实现：每次一个条目，每个都测试
```

**禁止响应**

绝不说：
- "你完全正确！"（表演性赞同）
- "很好的观点！" / "出色的反馈！"（表演性）
- "让我现在就实现它"（验证之前）

应该：
- 重述技术需求
- 提问澄清
- 如果错误，用技术推理反驳
- 直接开始工作（行动 > 话语）

**处理不清晰的反馈**

```
如果有任何条目不清晰：
  停止 — 还不要实现任何内容
  先请求澄清不清晰的条目

原因：条目可能有关联。部分理解 = 错误实现。
```

**来源特定处理**

- 来自用户（your human partner）：可信任，理解后实现；范围不清则仍需提问
- 来自外部 reviewer：在实现之前先检查技术正确性、是否破坏现有功能、是否有兼容性考虑、reviewer 是否了解完整上下文

**YAGNI 检查**

如果 reviewer 建议"正确实现某个功能"：
- 先 grep 代码库确认是否有实际使用
- 如未使用：提出删除（YAGNI）
- 如果有使用：才实现

**实现顺序（多条反馈）**

1. 先澄清所有不清晰的条目
2. 按顺序实现：阻塞性问题 → 简单修复 → 复杂修复
3. 每个修复单独测试
4. 验证无回归

**何时反驳**

建议破坏现有功能时 / reviewer 缺乏完整上下文时 / 违反 YAGNI 时 / 技术上对此技术栈不适用时 / 与架构决策冲突时。

**确认正确反馈时**

```
✅ "Fixed. [简短描述变化]"
✅ "Good catch - [具体问题]. Fixed in [位置]."
✅ [直接修复，在代码中体现]

❌ "你完全正确！"
❌ "感谢指出！"
❌ 任何感谢表达
```

**原因**：行动说明一切。修复代码本身就表明你听到了反馈。

**底线**

**外部反馈 = 评估建议，而非执行命令。**

验证。质疑。然后实现。

没有表演性赞同。始终技术严谨。

---

## 三、剖析解读

### 3.1 功能与定位

receiving-code-review 解决的问题是：**在收到反馈时，有两种常见的错误极端**——一是盲目同意并立即实现（performative agreement），二是过度防御性拒绝。

这个 skill 要求走一条中间道路：**技术性验证，然后做出基于证据的响应**。

在整个工作流中，这个 skill 是 requesting-code-review 的直接下游——review 发出去之后，如何处理回来的反馈。它特别强调了一个重要区分：来自 your human partner 的反馈（可信）vs 来自外部 reviewer 的反馈（需要技术验证）。

这个 skill 还有一个特别有趣的设计：**明确禁止任何感谢表达**（包括"Thanks for catching that!"）。这个规则的逻辑是：行动本身就证明你听到了反馈，文字感谢是多余的，而且可能滑向表演性赞同。

### 3.2 使用场景与案例

**场景：reviewer 指出可能存在 off-by-one 错误**

假设 code reviewer 报告：
```
#### Important
1. apply_discount() 在 discount_rate = 1.0 时会返回 0，
   但 spec 要求保留最低价格 $0.01
   - File: coupon.py:41
```

**错误做法（盲目实现）**：
```
❌ "You're absolutely right! Let me fix that immediately."
→ 立即修改代码，没有先验证 reviewer 是否正确
```

**正确做法（技术验证）**：

1. READ：完整读取反馈
2. UNDERSTAND：Reviewer 声称 `apply_discount(price=1.0, rate=1.0)` 返回 0 而非 0.01
3. VERIFY：检查代码和测试——`price * (1 - rate) = 1.0 * 0.0 = 0.0`；检查 spec 是否确实要求 $0.01 下限
4. EVALUATE：Reviewer 技术上是对的（确实会返回 0），但 spec 里有这个要求吗？检查计划文档。
5. 如果 spec 有此要求 → 承认并修复
6. 如果 spec 没有此要求 → 推理反驳："Checked plan task 3—no minimum price requirement. Spec says calculate discount from base price only. Should I add this or is this scope creep?"

**场景：reviewer 建议添加未使用的 feature**

```
Reviewer: "You should add metrics tracking—it's a production best practice"
```

正确响应：
```
"Grepped codebase—nothing calls get_metrics(). YAGNI applies here.
 If metrics are needed, should be a separate task in the plan.
 Proceeding without it."
```

### 3.3 Subagents / References 深度解读

此 skill 无附属文档，内容完全内联在 SKILL.md 中。

但值得注意的是 SKILL.md 中有一个特殊的信号机制：
```
Signal if uncomfortable pushing back out loud: "Strange things are afoot at the Circle K"
```
这是一个"彩蛋"式的约定——如果 agent 在技术上觉得 reviewer 错了但不愿意直接说，可以用这句话向 your human partner 发出信号，请求支援。

### 3.4 流程图 / 示意图

**反馈处理决策树**

```
收到 code review 反馈
         │
         ▼
完整阅读所有反馈（不做任何事）
         │
         ▼
任何条目不清晰？
   ┌──yes──┐  ┌──no──┐
   ▼       │  ▼
请求澄清   │  按条目处理
（全部清   │
晰后再继续）│
   └───────┘
         │
         ▼
对每个条目（按 Critical → Important → Minor 顺序）：
         │
         ▼
来源？
  ┌─── your human partner ───┐  ┌─── 外部 reviewer ───┐
  ▼                          │  ▼
直接理解并实现（仍需确认）   │  技术验证 5 项检查
                             │    ① 技术上正确？
                             │    ② 会破坏现有功能？
                             │    ③ 当前实现有原因？
                             │    ④ 跨平台可行？
                             │    ⑤ reviewer 了解完整上下文？
                             │
                             ▼
                       判断
                  ┌────┴────┐
                  ▼         ▼
               正确        错误
                  │         │
                  ▼         ▼
         "Fixed. [描述]"  技术性反驳
         直接修复          用推理+证据
                  │         │
                  ▼         └─→ 上升给 your human
         逐条修复，            partner（如冲突
         每次单独测试           架构决策）
         验证无回归
```

### 3.5 与其他 Skills 的协作关系

| 协作 Skill | 关系类型 | 说明 |
|------------|----------|------|
| requesting-code-review | 上游 | review 完成后，处理返回的反馈 |
| verification-before-completion | 并列/前置 | 每个修复实现后，需要验证确实有效 |
| systematic-debugging | 并列 | 如果 reviewer 指出 bug，用 systematic-debugging 分析根因再修复 |
| finishing-a-development-branch | 下游 | 所有 review 反馈处理完毕，才能进入 finishing 阶段 |
| test-driven-development | 并列 | 实现 reviewer 的修改建议时，仍然遵循 TDD（先写测试） |
