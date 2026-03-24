# [10] Verification Before Completion

## 速查卡 (Quick Reference)

| 维度           | 内容 |
|----------------|------|
| 触发时机       | 准备声明工作完成、测试通过、bug 已修复，或任何成功/完成断言之前 |
| 调用链上游     | executing-plans、subagent-driven-development、systematic-debugging（fix 后） |
| 调用链下游     | requesting-code-review（验证通过后才发起 code review） |
| 核心产出       | 运行命令的实际输出 + 基于证据的状态断言 |
| 关联 Subagents | 无（在当前 session 中直接执行验证命令） |

---

## 一、原文

### SKILL.md

```
---
name: verification-before-completion
description: Use when about to claim work is complete, fixed, or passing, before committing or creating PRs - requires running verification commands and confirming output before making any success claims; evidence before assertions always
---

# Verification Before Completion

## Overview

Claiming work is complete without verification is dishonesty, not efficiency.

**Core principle:** Evidence before claims, always.

**Violating the letter of this rule is violating the spirit of this rule.**

## The Iron Law

```
NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE
```

If you haven't run the verification command in this message, you cannot claim it passes.

## The Gate Function

```
BEFORE claiming any status or expressing satisfaction:

1. IDENTIFY: What command proves this claim?
2. RUN: Execute the FULL command (fresh, complete)
3. READ: Full output, check exit code, count failures
4. VERIFY: Does output confirm the claim?
   - If NO: State actual status with evidence
   - If YES: State claim WITH evidence
5. ONLY THEN: Make the claim

Skip any step = lying, not verifying
```

## Common Failures

| Claim | Requires | Not Sufficient |
|-------|----------|----------------|
| Tests pass | Test command output: 0 failures | Previous run, "should pass" |
| Linter clean | Linter output: 0 errors | Partial check, extrapolation |
| Build succeeds | Build command: exit 0 | Linter passing, logs look good |
| Bug fixed | Test original symptom: passes | Code changed, assumed fixed |
| Regression test works | Red-green cycle verified | Test passes once |
| Agent completed | VCS diff shows changes | Agent reports "success" |
| Requirements met | Line-by-line checklist | Tests passing |

## Red Flags - STOP

- Using "should", "probably", "seems to"
- Expressing satisfaction before verification ("Great!", "Perfect!", "Done!", etc.)
- About to commit/push/PR without verification
- Trusting agent success reports
- Relying on partial verification
- Thinking "just this once"
- Tired and wanting work over
- **ANY wording implying success without having run verification**

## Rationalization Prevention

| Excuse | Reality |
|--------|---------|
| "Should work now" | RUN the verification |
| "I'm confident" | Confidence ≠ evidence |
| "Just this once" | No exceptions |
| "Linter passed" | Linter ≠ compiler |
| "Agent said success" | Verify independently |
| "I'm tired" | Exhaustion ≠ excuse |
| "Partial check is enough" | Partial proves nothing |
| "Different words so rule doesn't apply" | Spirit over letter |

## Key Patterns

**Tests:**
```
✅ [Run test command] [See: 34/34 pass] "All tests pass"
❌ "Should pass now" / "Looks correct"
```

**Regression tests (TDD Red-Green):**
```
✅ Write → Run (pass) → Revert fix → Run (MUST FAIL) → Restore → Run (pass)
❌ "I've written a regression test" (without red-green verification)
```

**Build:**
```
✅ [Run build] [See: exit 0] "Build passes"
❌ "Linter passed" (linter doesn't check compilation)
```

**Requirements:**
```
✅ Re-read plan → Create checklist → Verify each → Report gaps or completion
❌ "Tests pass, phase complete"
```

**Agent delegation:**
```
✅ Agent reports success → Check VCS diff → Verify changes → Report actual state
❌ Trust agent report
```

## Why This Matters

From 24 failure memories:
- your human partner said "I don't believe you" - trust broken
- Undefined functions shipped - would crash
- Missing requirements shipped - incomplete features
- Time wasted on false completion → redirect → rework
- Violates: "Honesty is a core value. If you lie, you'll be replaced."

## When To Apply

**ALWAYS before:**
- ANY variation of success/completion claims
- ANY expression of satisfaction
- ANY positive statement about work state
- Committing, PR creation, task completion
- Moving to next task
- Delegating to agents

**Rule applies to:**
- Exact phrases
- Paraphrases and synonyms
- Implications of success
- ANY communication suggesting completion/correctness

## The Bottom Line

**No shortcuts for verification.**

Run the command. Read the output. THEN claim the result.

This is non-negotiable.
```

---

## 二、中文翻译

### SKILL.md 翻译

**完成前验证**

**概述**

在没有验证的情况下声称工作完成是不诚实，而不是效率。

**核心原则：** 证据先于断言，始终如此。

**违反这个规则的字面规定就是违反其精神。**

**铁律**

```
没有新鲜验证证据，不得声称完成
```

如果你在这条消息中还没有运行验证命令，就不能声称它通过了。

**关卡函数**

```
在声明任何状态或表达满意之前：

1. 识别：什么命令能证明这个断言？
2. 运行：执行完整命令（新鲜的、完整的）
3. 阅读：完整输出，检查退出码，计算失败次数
4. 验证：输出是否确认了断言？
   - 如果否：用证据说明实际状态
   - 如果是：用证据说明断言
5. 只有这时：才能做出断言

跳过任何步骤 = 撒谎，而不是验证
```

**常见失败场景对照**

| 断言 | 需要 | 不充分 |
|------|------|--------|
| 测试通过 | 测试命令输出：0 failures | 之前的运行、"应该通过" |
| Linter 干净 | Linter 输出：0 errors | 部分检查、推断 |
| 构建成功 | 构建命令：exit 0 | Linter 通过、日志看起来好 |
| Bug 已修复 | 测试原始症状：通过 | 代码已改变、假设已修复 |
| 回归测试有效 | Red-Green 循环验证 | 测试通过一次 |
| Agent 完成了 | VCS diff 显示变更 | Agent 报告"成功" |
| 满足需求 | 逐行 checklist | 测试通过 |

**红旗 — 停止**

- 使用"应该"、"可能"、"似乎"
- 在验证之前表达满意（"太好了！"、"完美！"、"完成了！"等）
- 即将 commit/push/PR 而未验证
- 相信 agent 的成功报告
- 依赖部分验证
- 想着"就这一次"
- 累了想结束工作
- **任何暗示成功却未运行验证的措辞**

**合理化借口预防**

| 借口 | 现实 |
|------|------|
| "现在应该能工作了" | 运行验证 |
| "我很有信心" | 信心 ≠ 证据 |
| "就这一次" | 没有例外 |
| "Linter 通过了" | Linter ≠ 编译器 |
| "Agent 说成功了" | 独立验证 |
| "我累了" | 疲惫 ≠ 借口 |
| "部分检查够了" | 部分证明不了任何事 |
| "不同的措辞所以规则不适用" | 精神优先于字面 |

**关键模式**

```
✅ 测试：[运行测试命令] [看到：34/34 通过] → "所有测试通过"
❌ "现在应该通过了" / "看起来正确"

✅ 回归测试：写 → 运行（通过）→ 还原 fix → 运行（必须失败）→ 恢复 → 运行（通过）
❌ "我写了一个回归测试"（没有 red-green 验证）

✅ 需求：重读计划 → 创建 checklist → 逐条验证 → 报告差距或完成
❌ "测试通过，阶段完成"
```

**为什么这很重要**

来自 24 次失败记忆：
- 用户说"我不相信你"——信任破裂
- 未定义的函数已发布——会崩溃
- 未满足的需求已发布——不完整的功能
- 虚假完成浪费时间 → 重定向 → 返工

**适用时机**

始终在以下情况之前：
- 任何形式的成功/完成断言
- 任何满意表达
- 任何关于工作状态的正面陈述
- Commit、创建 PR、完成任务
- 进入下一个任务
- 向 agent 委派

**底线**

**没有验证的捷径。**

运行命令。读取输出。然后才能声称结果。

这是不可谈判的。

---

## 三、剖析解读

### 3.1 功能与定位

verification-before-completion 解决的根本问题是：**AI agent 和工程师都有虚报完成状态的倾向**——或因为懒惰，或因为疲惫，或因为"感觉应该对"。

这个 skill 是整个工作流中一个强制性的**质量关卡**，在任何声称"完成"之前必须执行。它的定位类似一个检查清单规则："你说完成了，但有证据吗？"

特别重要的是它对 agent 委派场景的覆盖：当一个子 agent 报告"DONE"时，controller 不能直接相信报告——必须通过 VCS diff 验证实际变更。这个规则在 subagent-driven-development 工作流中尤为关键。

这个 skill 非常简洁（只有一个源文件），但设计极其严谨——它明确列出了所有常见的合理化借口，并用对照表逐一驳斥。这是为了防止 agent 在压力下找到"例外"。

### 3.2 使用场景与案例

**场景：准备声明"功能完成"**

假设 implementer subagent 刚刚实现了优惠券折扣计算功能，并报告：
```
Status: DONE
已实现 apply_coupon_discount() 函数
测试看起来都通过了
```

**错误做法**（违反此 skill）：
- controller 直接标记 task 为 completed
- 说"测试通过了，可以 code review 了"
- 在没有运行任何命令的情况下 commit

**正确做法**（按 Gate Function 执行）：
1. IDENTIFY：证明测试通过的命令是 `pytest tests/test_coupon.py -v`
2. RUN：执行命令，得到实际输出
3. READ：`3 passed, 0 failed` → exit code 0
4. VERIFY：输出确认了断言
5. ONLY THEN：报告"测试通过：3/3，exit 0"

**关键：** 不是因为 subagent 说"DONE"，而是因为测试命令输出了 `3 passed`，才能声称完成。

**场景：回归测试 Red-Green 验证**

修复了一个 bug 后，说"我写了回归测试"是不够的：
```
✅ 正确序列：
1. 写测试（应该捕获 bug）
2. 运行 → 通过（这是错误！说明测试没有捕获 bug）
3. 发现问题，修改测试
4. 运行 → 失败（RED ✓）
5. 恢复 fix
6. 运行 → 通过（GREEN ✓）
```

只有看到了完整的 RED-GREEN 循环，才能声称回归测试有效。

### 3.3 Subagents / References 深度解读

此 skill 无附属文档。但它与多个工作流 skill 有深度集成：

- 在 **subagent-driven-development** 中，spec compliance reviewer 和 code quality reviewer 的工作结果，以及 implementer 的 DONE 报告，都需要通过此 skill 进行独立验证。
- 在 **executing-plans** 中，每个 checkpoint 的"批次完成"声明都需要此 skill 验证。
- 在 **systematic-debugging** 中，声称"bug 已修复"之前，必须运行原始症状的测试并看到通过输出。

### 3.4 流程图 / 示意图

**Gate Function（关卡函数）决策流**

```
即将做出完成/成功断言
          │
          ▼
IDENTIFY：哪个命令能证明这个断言？
          │
          ▼ 执行完整命令（不是部分、不是之前的结果）
RUN：运行命令，获取本次新鲜输出
          │
          ▼
READ：完整读取输出
  检查退出码是否为 0？
  计算失败数是否为 0？
  逐行确认关键指标？
          │
     ┌────┴────┐
     ▼         ▼
  通过         失败
     │         │
     ▼         ▼
 VERIFY ✓  VERIFY ✗
带证据的    报告实际
成功断言    状态+证据
```

**常见断言的证据要求：**

```
断言 "测试通过"
  → 证据：pytest/npm test 输出 "N passed, 0 failed"
  → 不够：之前跑过 | agent 说 passed | 部分测试通过

断言 "Bug 已修复"
  → 证据：原始症状的测试 → 红色（复现）→ 绿色（修复后）
  → 不够：代码已改动 | 相关测试通过

断言 "需求满足"
  → 证据：逐条对照计划，每条都有验证输出
  → 不够：测试通过 | 功能看起来对
```

### 3.5 与其他 Skills 的协作关系

| 协作 Skill | 关系类型 | 说明 |
|------------|----------|------|
| executing-plans | 上游 | 每个任务完成前、每个 checkpoint 前必须应用此 skill |
| subagent-driven-development | 上游 | implementer DONE 报告后、spec reviewer 审查后，controller 必须独立验证 |
| systematic-debugging | 上游 | 声称 bug 已修复之前，运行验证命令确认 |
| requesting-code-review | 下游 | 验证通过（有证据）才能进入 code review 阶段 |
| test-driven-development | 并列 | TDD 的 RED-GREEN 验证本身就是一种 verification-before-completion 的应用 |
