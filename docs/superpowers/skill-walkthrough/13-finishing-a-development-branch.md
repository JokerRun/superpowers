# [13] Finishing a Development Branch

## 速查卡 (Quick Reference)

| 维度           | 内容 |
|----------------|------|
| 触发时机       | 实现完成、所有测试通过、code review 通过，需要决定如何整合工作时 |
| 调用链上游     | subagent-driven-development（所有任务完成后）、executing-plans（所有批次完成后） |
| 调用链下游     | 无（这是工作流的最终阶段） |
| 核心产出       | merge 到主分支 / 创建 PR / 保留分支 / 废弃工作（四选一） |
| 关联 Subagents | 无（在当前 session 中直接执行 git 操作） |

---

## 一、原文

### SKILL.md

````
---
name: finishing-a-development-branch
description: Use when implementation is complete, all tests pass, and you need to decide how to integrate the work - guides completion of development work by presenting structured options for merge, PR, or cleanup
---

# Finishing a Development Branch

## Overview

Guide completion of development work by presenting clear options and handling chosen workflow.

**Core principle:** Verify tests → Present options → Execute choice → Clean up.

**Announce at start:** "I'm using the finishing-a-development-branch skill to complete this work."

## The Process

### Step 1: Verify Tests

**Before presenting options, verify tests pass:**

```bash
# Run project's test suite
npm test / cargo test / pytest / go test ./...
````

**If tests fail:**
````
Tests failing (<N> failures). Must fix before completing:

[Show failures]

Cannot proceed with merge/PR until tests pass.
```

Stop. Don't proceed to Step 2.

**If tests pass:** Continue to Step 2.

### Step 2: Determine Base Branch

```bash
# Try common base branches
git merge-base HEAD main 2>/dev/null || git merge-base HEAD master 2>/dev/null
```

Or ask: "This branch split from main - is that correct?"

### Step 3: Present Options

Present exactly these 4 options:

```
Implementation complete. What would you like to do?

1. Merge back to <base-branch> locally
2. Push and create a Pull Request
3. Keep the branch as-is (I'll handle it later)
4. Discard this work

Which option?
```

**Don't add explanation** - keep options concise.

### Step 4: Execute Choice

#### Option 1: Merge Locally

```bash
# Switch to base branch
git checkout <base-branch>

# Pull latest
git pull

# Merge feature branch
git merge <feature-branch>

# Verify tests on merged result
<test command>

# If tests pass
git branch -d <feature-branch>
```

Then: Cleanup worktree (Step 5)

#### Option 2: Push and Create PR

```bash
# Push branch
git push -u origin <feature-branch>

# Create PR
gh pr create --title "<title>" --body "$(cat <<'EOF'
## Summary
<2-3 bullets of what changed>

## Test Plan
- [ ] <verification steps>
EOF
)"
```

Then: Cleanup worktree (Step 5)

#### Option 3: Keep As-Is

Report: "Keeping branch <name>. Worktree preserved at <path>."

**Don't cleanup worktree.**

#### Option 4: Discard

**Confirm first:**
```
This will permanently delete:
- Branch <name>
- All commits: <commit-list>
- Worktree at <path>

Type 'discard' to confirm.
```

Wait for exact confirmation.

If confirmed:
```bash
git checkout <base-branch>
git branch -D <feature-branch>
```

Then: Cleanup worktree (Step 5)

### Step 5: Cleanup Worktree

**For Options 1, 2, 4:**

Check if in worktree:
```bash
git worktree list | grep $(git branch --show-current)
```

If yes:
```bash
git worktree remove <worktree-path>
```

**For Option 3:** Keep worktree.

## Quick Reference

| Option | Merge | Push | Keep Worktree | Cleanup Branch |
|--------|-------|------|---------------|----------------|
| 1. Merge locally | ✓ | - | - | ✓ |
| 2. Create PR | - | ✓ | ✓ | - |
| 3. Keep as-is | - | - | ✓ | - |
| 4. Discard | - | - | - | ✓ (force) |

## Common Mistakes

**Skipping test verification**
- **Problem:** Merge broken code, create failing PR
- **Fix:** Always verify tests before offering options

**Open-ended questions**
- **Problem:** "What should I do next?" → ambiguous
- **Fix:** Present exactly 4 structured options

**Automatic worktree cleanup**
- **Problem:** Remove worktree when might need it (Option 2, 3)
- **Fix:** Only cleanup for Options 1 and 4

**No confirmation for discard**
- **Problem:** Accidentally delete work
- **Fix:** Require typed "discard" confirmation

## Red Flags

**Never:**
- Proceed with failing tests
- Merge without verifying tests on result
- Delete work without confirmation
- Force-push without explicit request

**Always:**
- Verify tests before offering options
- Present exactly 4 options
- Get typed confirmation for Option 4
- Clean up worktree for Options 1 & 4 only

## Integration

**Called by:**
- **subagent-driven-development** (Step 7) - After all tasks complete
- **executing-plans** (Step 5) - After all batches complete

**Pairs with:**
- **using-git-worktrees** - Cleans up worktree created by that skill
```

---

## 二、中文翻译

### SKILL.md 翻译

**完成开发分支**

**概述**

通过呈现清晰选项并处理所选工作流，引导开发工作的完成。

**核心原则：** 验证测试 → 呈现选项 → 执行选择 → 清理。

**开始时声明：** "我正在使用 finishing-a-development-branch skill 来完成这项工作。"

**流程**

**Step 1：验证测试**

在呈现选项之前，验证测试通过：
```bash
npm test / cargo test / pytest / go test ./...
```

如果测试失败：停止。不进入 Step 2。报告失败并要求先修复。

如果测试通过：继续 Step 2。
````

**Step 2：确定基础分支**

使用 `git merge-base` 确认从哪个分支分叉，或直接询问用户。

**Step 3：呈现恰好 4 个选项**

```
实现完成。你想如何处理？

1. 本地 merge 回 <base-branch>
2. Push 并创建 Pull Request
3. 保持分支不变（我稍后处理）
4. 废弃这项工作

选择哪个？
```

**不要添加额外解释** — 保持选项简洁。

**Step 4：执行选择**

- **Option 1（本地 merge）**：checkout → pull → merge → 验证测试 → 删除分支 → 清理 worktree
- **Option 2（创建 PR）**：push → `gh pr create` → 清理 worktree
- **Option 3（保持不变）**：报告状态，不清理 worktree
- **Option 4（废弃）**：先显示将删除的内容，等待用户输入确切的 'discard' 确认，然后执行

**Step 5：清理 Worktree**

Options 1、2、4：执行 `git worktree remove <path>`
Option 3：保留 worktree

**快速参考**

| 选项 | Merge | Push | 保留 Worktree | 清理分支 |
|------|-------|------|--------------|---------|
| 1. 本地 merge | ✓ | — | — | ✓ |
| 2. 创建 PR | — | ✓ | ✓ | — |
| 3. 保持不变 | — | — | ✓ | — |
| 4. 废弃 | — | — | — | ✓ (force) |

---

## 三、剖析解读

### 3.1 功能与定位

finishing-a-development-branch 解决的问题是：**开发完成后，工程师面临多种不同的收尾方式（merge / PR / 暂存 / 废弃），容易做出错误或不完整的操作**。

这个 skill 的定位是**开发工作流的最后一个关卡**，它强制：
1. **先验证测试**——不能在测试失败时进行任何合并或 PR 操作
2. **结构化选项**——呈现恰好 4 个选项（不多不少），消除歧义
3. **安全保护**——废弃操作需要输入确认，防止意外删除
4. **worktree 一致性**——与 using-git-worktrees skill 配对，确保开始时创建的 worktree 在结束时被正确处理

它与 using-git-worktrees 形成了**开始-结束对称**：using-git-worktrees 在开发开始时创建隔离工作空间，finishing-a-development-branch 在结束时清理这个空间。

### 3.2 使用场景与案例

**场景：优惠券功能开发完成，code review 通过**

流程：
```
1. 运行测试
   npm test
   → 47 tests, 0 failures ✓

2. 确认基础分支
   git merge-base HEAD main → 基于 main

3. 呈现 4 个选项
   "实现完成。你想如何处理？
    1. 本地 merge 回 main
    2. Push 并创建 Pull Request
    3. 保持分支不变
    4. 废弃这项工作"

4. 用户选 2（创建 PR）：
   git push -u origin feature/coupon
   gh pr create --title "feat: add coupon discount system" \
     --body "$(cat <<'EOF'
   ## Summary
   - 实现百分比和固定金额折扣
   - 添加最大折扣上限验证
   - 47 tests, 0 failures

   ## Test Plan
   - [ ] 验证折扣计算正确
   - [ ] 验证边界情况处理
   EOF
   )"

5. 清理 worktree（Option 2 也清理）：
   git worktree remove .worktrees/feature-coupon
```

**错误案例对比**

```
❌ 错误：测试还没跑就 merge
→ 可能把失败的代码合并进主分支

❌ 错误：自动清理 worktree（即使选了 Option 2）
→ PR 还在 review 中，可能还需要修改

❌ 错误：废弃时不要求确认
→ 意外删除数小时的工作

✅ 正确：先验证测试，再呈现选项，根据选择决定是否清理 worktree
```

### 3.3 Subagents / References 深度解读

此 skill 无附属文档，所有内容在 SKILL.md 中完整内联。

值得注意的设计决策：
1. **"不要添加额外解释"** — 4 个选项就是 4 个选项，不解释每个的优缺点。这是故意的：让用户基于上下文做决定，而不是 AI 引导他们倾向某个选项。
2. **Options 1 和 4 清理 worktree，Options 2 和 3 不清理** — 这个规则的逻辑：Option 1 合并完了不再需要分支；Option 4 废弃了更不需要；Option 2 PR 还在 review，可能还要修改；Option 3 明确要保留。
3. **废弃确认使用精确字符串 'discard'** — 比"yes/no"更难误触，确保用户是有意识地决定废弃。

### 3.4 流程图 / 示意图

**完成开发分支决策树**

```
subagent-driven-development / executing-plans 全部完成
              │
              ▼
Step 1：验证测试
   npm test / pytest / cargo test
              │
      ┌───────┴───────┐
      ▼               ▼
   失败              通过
      │               │
      ▼               ▼
  停止！          Step 2：确认基础分支
  先修复 bug      (main / master / develop?)
                       │
                       ▼
              Step 3：呈现 4 个选项
                       │
     ┌─────────────────┼─────────────────┐
     ▼                 ▼                 ▼                 ▼
  Option 1         Option 2          Option 3          Option 4
  本地 merge       创建 PR           保持不变          废弃
     │                 │                 │                 │
     ▼                 ▼                 ▼                 ▼
checkout main    git push          报告状态          显示将删除
git pull         gh pr create      保留 worktree     内容清单
git merge        保留 worktree          │                 │
验证测试         清理 worktree          ▼                 ▼
删除分支               │            END           等待输入 'discard'
清理 worktree          │                                  │
     │                 │                                  ▼
     └─────────────────┘                          git branch -D
              │                                   清理 worktree
              ▼
            END
```

### 3.5 与其他 Skills 的协作关系

| 协作 Skill | 关系类型 | 说明 |
|------------|----------|------|
| using-git-worktrees | 配对（开始/结束） | using-git-worktrees 创建 worktree，此 skill 清理 worktree |
| subagent-driven-development | 上游 | SDD 的最后一步（所有任务 + final review 完成后）调用此 skill |
| executing-plans | 上游 | executing-plans 的最后一步（所有批次完成后）调用此 skill |
| verification-before-completion | 前置 | 步骤 1 的测试验证就是 verification-before-completion 的一个应用 |
| requesting-code-review | 前置 | 通常在 code review 通过后才使用此 skill |
