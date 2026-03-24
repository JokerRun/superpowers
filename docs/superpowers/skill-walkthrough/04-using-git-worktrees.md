# 04 using-git-worktrees

## 速查卡 (Quick Reference)
| 维度           | 内容 |
|----------------|------|
| 触发时机       | plan 提交到 main 之后、开始实现之前；或任何需要隔离工作区的时机 |
| 调用链上游     | brainstorming (Phase 4 REQUIRED)、writing-plans（计划写完后触发） |
| 调用链下游     | executing-plans、subagent-driven-development（建立 worktree 后进入执行阶段） |
| 核心产出       | 隔离的 git worktree，含干净的依赖安装与通过的 baseline 测试 |
| 关联 Subagents | 无 |

---

## 一、原文

### SKILL.md

---
name: using-git-worktrees
description: Use when starting feature work that needs isolation from current workspace or before executing implementation plans - creates isolated git worktrees with smart directory selection and safety verification
---

# Using Git Worktrees

## Overview

Git worktrees create isolated workspaces sharing the same repository, allowing work on multiple branches simultaneously without switching.

**Core principle:** Systematic directory selection + safety verification = reliable isolation.

**Announce at start:** "I'm using the using-git-worktrees skill to set up an isolated workspace."

## Directory Selection Process

Follow this priority order:

### 1. Check Existing Directories

```bash
# Check in priority order
ls -d .worktrees 2>/dev/null     # Preferred (hidden)
ls -d worktrees 2>/dev/null      # Alternative
```

**If found:** Use that directory. If both exist, `.worktrees` wins.

### 2. Check CLAUDE.md

```bash
grep -i "worktree.*director" CLAUDE.md 2>/dev/null
```

**If preference specified:** Use it without asking.

### 3. Ask User

If no directory exists and no CLAUDE.md preference:

```
No worktree directory found. Where should I create worktrees?

1. .worktrees/ (project-local, hidden)
2. ~/.config/superpowers/worktrees/<project-name>/ (global location)

Which would you prefer?
```

## Safety Verification

### For Project-Local Directories (.worktrees or worktrees)

**MUST verify directory is ignored before creating worktree:**

```bash
# Check if directory is ignored (respects local, global, and system gitignore)
git check-ignore -q .worktrees 2>/dev/null || git check-ignore -q worktrees 2>/dev/null
```

**If NOT ignored:**

Per Jesse's rule "Fix broken things immediately":
1. Add appropriate line to .gitignore
2. Commit the change
3. Proceed with worktree creation

**Why critical:** Prevents accidentally committing worktree contents to repository.

### For Global Directory (~/.config/superpowers/worktrees)

No .gitignore verification needed - outside project entirely.

## Creation Steps

### 1. Detect Project Name

```bash
project=$(basename "$(git rev-parse --show-toplevel)")
```

### 2. Create Worktree

```bash
# Determine full path
case $LOCATION in
  .worktrees|worktrees)
    path="$LOCATION/$BRANCH_NAME"
    ;;
  ~/.config/superpowers/worktrees/*)
    path="~/.config/superpowers/worktrees/$project/$BRANCH_NAME"
    ;;
esac

# Create worktree with new branch
git worktree add "$path" -b "$BRANCH_NAME"
cd "$path"
```

### 3. Run Project Setup

Auto-detect and run appropriate setup:

```bash
# Node.js
if [ -f package.json ]; then npm install; fi

# Rust
if [ -f Cargo.toml ]; then cargo build; fi

# Python
if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
if [ -f pyproject.toml ]; then poetry install; fi

# Go
if [ -f go.mod ]; then go mod download; fi
```

### 4. Verify Clean Baseline

Run tests to ensure worktree starts clean:

```bash
# Examples - use project-appropriate command
npm test
cargo test
pytest
go test ./...
```

**If tests fail:** Report failures, ask whether to proceed or investigate.

**If tests pass:** Report ready.

### 5. Report Location

```
Worktree ready at <full-path>
Tests passing (<N> tests, 0 failures)
Ready to implement <feature-name>
```

## Quick Reference

| Situation | Action |
|-----------|--------|
| `.worktrees/` exists | Use it (verify ignored) |
| `worktrees/` exists | Use it (verify ignored) |
| Both exist | Use `.worktrees/` |
| Neither exists | Check CLAUDE.md → Ask user |
| Directory not ignored | Add to .gitignore + commit |
| Tests fail during baseline | Report failures + ask |
| No package.json/Cargo.toml | Skip dependency install |

## Common Mistakes

### Skipping ignore verification

- **Problem:** Worktree contents get tracked, pollute git status
- **Fix:** Always use `git check-ignore` before creating project-local worktree

### Assuming directory location

- **Problem:** Creates inconsistency, violates project conventions
- **Fix:** Follow priority: existing > CLAUDE.md > ask

### Proceeding with failing tests

- **Problem:** Can't distinguish new bugs from pre-existing issues
- **Fix:** Report failures, get explicit permission to proceed

### Hardcoding setup commands

- **Problem:** Breaks on projects using different tools
- **Fix:** Auto-detect from project files (package.json, etc.)

## Example Workflow

```
You: I'm using the using-git-worktrees skill to set up an isolated workspace.

[Check .worktrees/ - exists]
[Verify ignored - git check-ignore confirms .worktrees/ is ignored]
[Create worktree: git worktree add .worktrees/auth -b feature/auth]
[Run npm install]
[Run npm test - 47 passing]

Worktree ready at /Users/jesse/myproject/.worktrees/auth
Tests passing (47 tests, 0 failures)
Ready to implement auth feature
```

## Red Flags

**Never:**
- Create worktree without verifying it's ignored (project-local)
- Skip baseline test verification
- Proceed with failing tests without asking
- Assume directory location when ambiguous
- Skip CLAUDE.md check

**Always:**
- Follow directory priority: existing > CLAUDE.md > ask
- Verify directory is ignored for project-local
- Auto-detect and run project setup
- Verify clean test baseline

## Integration

**Called by:**
- **brainstorming** (Phase 4) - REQUIRED when design is approved and implementation follows
- **subagent-driven-development** - REQUIRED before executing any tasks
- **executing-plans** - REQUIRED before executing any tasks
- Any skill needing isolated workspace

**Pairs with:**
- **finishing-a-development-branch** - REQUIRED for cleanup after work complete

---

## 二、中文翻译

### SKILL.md 翻译

---
name: using-git-worktrees
description: 在开始需要与当前工作区隔离的功能开发时，或在执行实现计划之前使用——通过智能目录选择和安全验证来创建隔离的 git worktree
---

# 使用 Git Worktrees

## 概述

Git worktrees 可在共享同一仓库的同时创建隔离的工作空间，无需切换分支即可同时处理多个分支。

**核心原则：** 系统化的目录选择 + 安全验证 = 可靠的隔离。

**开始时声明：** "I'm using the using-git-worktrees skill to set up an isolated workspace."

## 目录选择流程

按以下优先级顺序执行：

### 1. 检查已有目录

```bash
# 按优先级检查
ls -d .worktrees 2>/dev/null     # 首选（隐藏目录）
ls -d worktrees 2>/dev/null      # 备选
```

**如果找到：** 使用该目录。如果两者都存在，`.worktrees` 优先。

### 2. 检查 CLAUDE.md

```bash
grep -i "worktree.*director" CLAUDE.md 2>/dev/null
```

**如果指定了偏好：** 直接使用，无需询问。

### 3. 询问用户

如果没有已有目录且 CLAUDE.md 中也没有偏好设置：

```
未找到 worktree 目录。应该在哪里创建 worktrees？

1. .worktrees/（项目本地，隐藏目录）
2. ~/.config/superpowers/worktrees/<project-name>/（全局位置）

您更倾向于哪种？
```

## 安全验证

### 针对项目本地目录（.worktrees 或 worktrees）

**创建 worktree 前必须验证目录已被忽略：**

```bash
# 检查目录是否被忽略（遵循本地、全局和系统级 gitignore）
git check-ignore -q .worktrees 2>/dev/null || git check-ignore -q worktrees 2>/dev/null
```

**如果未被忽略：**

根据 Jesse 的原则"立即修复已损坏的东西"：
1. 在 .gitignore 中添加相应的行
2. 提交该变更
3. 继续创建 worktree

**为何关键：** 防止意外将 worktree 内容提交到仓库。

### 针对全局目录（~/.config/superpowers/worktrees）

无需 .gitignore 验证——该路径完全位于项目外部。

## 创建步骤

### 1. 检测项目名称

```bash
project=$(basename "$(git rev-parse --show-toplevel)")
```

### 2. 创建 Worktree

```bash
# 确定完整路径
case $LOCATION in
  .worktrees|worktrees)
    path="$LOCATION/$BRANCH_NAME"
    ;;
  ~/.config/superpowers/worktrees/*)
    path="~/.config/superpowers/worktrees/$project/$BRANCH_NAME"
    ;;
esac

# 创建带新分支的 worktree
git worktree add "$path" -b "$BRANCH_NAME"
cd "$path"
```

### 3. 运行项目初始化

自动检测并运行相应的初始化命令：

```bash
# Node.js
if [ -f package.json ]; then npm install; fi

# Rust
if [ -f Cargo.toml ]; then cargo build; fi

# Python
if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
if [ -f pyproject.toml ]; then poetry install; fi

# Go
if [ -f go.mod ]; then go mod download; fi
```

### 4. 验证干净的 Baseline

运行测试以确保 worktree 启动时是干净状态：

```bash
# 示例——使用适合项目的命令
npm test
cargo test
pytest
go test ./...
```

**如果测试失败：** 报告失败情况，询问是继续还是调查原因。

**如果测试通过：** 报告就绪。

### 5. 报告位置

```
Worktree ready at <完整路径>
Tests passing (<N> tests, 0 failures)
Ready to implement <功能名称>
```

## 速查表

| 情况 | 操作 |
|------|------|
| `.worktrees/` 存在 | 使用它（验证 gitignore） |
| `worktrees/` 存在 | 使用它（验证 gitignore） |
| 两者都存在 | 使用 `.worktrees/` |
| 两者都不存在 | 检查 CLAUDE.md → 询问用户 |
| 目录未被忽略 | 添加到 .gitignore 并提交 |
| baseline 测试失败 | 报告失败 + 询问 |
| 无 package.json/Cargo.toml | 跳过依赖安装 |

## 常见错误

### 跳过 ignore 验证

- **问题：** Worktree 内容被追踪，污染 git 状态
- **修复：** 创建项目本地 worktree 前始终使用 `git check-ignore`

### 假设目录位置

- **问题：** 造成不一致，违反项目约定
- **修复：** 遵循优先级：已有目录 > CLAUDE.md > 询问用户

### 在测试失败时继续

- **问题：** 无法区分新引入的 bug 与已有问题
- **修复：** 报告失败，获得明确许可后再继续

### 硬编码初始化命令

- **问题：** 在使用不同工具的项目上会出错
- **修复：** 从项目文件（package.json 等）自动检测

## 示例工作流

```
You: I'm using the using-git-worktrees skill to set up an isolated workspace.

[检查 .worktrees/ - 存在]
[验证已忽略 - git check-ignore 确认 .worktrees/ 已被忽略]
[创建 worktree: git worktree add .worktrees/auth -b feature/auth]
[运行 npm install]
[运行 npm test - 47 通过]

Worktree ready at /Users/jesse/myproject/.worktrees/auth
Tests passing (47 tests, 0 failures)
Ready to implement auth feature
```

## 红线

**绝不：**
- 未验证 gitignore 就创建 worktree（项目本地）
- 跳过 baseline 测试验证
- 未询问就在测试失败时继续
- 在路径不明确时假设目录位置
- 跳过 CLAUDE.md 检查

**始终：**
- 遵循目录优先级：已有目录 > CLAUDE.md > 询问用户
- 为项目本地目录验证 gitignore
- 自动检测并运行项目初始化
- 验证干净的测试 baseline

## 集成关系

**被以下调用：**
- **brainstorming**（Phase 4）- 设计被批准且后续需要实现时为必需
- **subagent-driven-development** - 执行任何任务前为必需
- **executing-plans** - 执行任何任务前为必需
- 任何需要隔离工作区的 skill

**配对使用：**
- **finishing-a-development-branch** - 工作完成后清理时为必需

---

## 三、剖析解读

### 3.1 功能与定位

using-git-worktrees 的核心职责是：**在 plan 提交到 main 之后、实现开始之前，建立一个经过验证的隔离代码实现工作区。**

这里有一个关键认知需要厘清：spec/plan 文件是**有意提交到 main** 的——它们是决策记录，属于持久化的文档资产。worktree 隔离的对象是**代码实现**，而不是规划本身。

```
main 分支
  ├── spec/     ← 决策记录，永久留存于 main
  ├── plans/    ← 实现计划，永久留存于 main
  └── src/      ← 不在这里直接写实现代码！
                         |
                         v
  .worktrees/feature-xxx/  ← 在这里写实现代码（隔离）
```

该 skill 的两大支柱：

1. **系统化的目录选择**：不依赖猜测，而是遵循明确的优先级（已有目录 > CLAUDE.md 偏好 > 询问用户），确保与项目约定保持一致。
2. **安全验证**：`git check-ignore` + baseline 测试，确保 worktree 既不污染主仓库状态，也从一个干净可知的起点开始。

### 3.2 使用场景与案例

**典型触发场景：** writing-plans 已将优惠券系统的实现计划提交到 main，现在需要开始实现。

using-git-worktrees 的完整执行序列：

1. 检查 `.worktrees/` 是否存在 → 存在
2. 验证已被 gitignore 忽略 → `git check-ignore -q .worktrees` 返回成功
3. 执行 `git worktree add .worktrees/feature-coupon -b feature/coupon`
4. 自动检测到 `package.json` → 运行 `npm install`
5. 运行 `npm test` → 47 tests passing
6. 报告：`"Worktree ready at .worktrees/feature-coupon, 47 tests passing"`

**正确 vs 错误实践对比：**

| 做法 | 结果 |
|------|------|
| 直接在 main 分支上写实现代码 | 错误：污染主分支，无法隔离回滚 |
| 创建 worktree 但跳过 .gitignore 验证 | 错误：worktree 内容被 git track，污染 git status |
| 测试失败但不询问直接继续 | 错误：新 bug 与既有问题混淆，难以排查 |
| 按优先级选目录 → 验证 ignore → 运行 setup → 验证 baseline | 正确：可靠隔离，干净起点 |

### 3.4 流程图 / 示意图

**目录选择优先级决策树：**

```
START: 需要创建 worktree
         |
         v
  .worktrees/ 存在？
    /         \
  YES          NO
   |            |
   v            v
使用它       worktrees/ 存在？
(验证 gitignore)   /         \
               YES          NO
                |            |
                v            v
             使用它      CLAUDE.md 中有偏好？
           (验证 gitignore)  /         \
                           YES          NO
                            |            |
                            v            v
                        使用指定路径   询问用户
                                    (选项A或B)
```

**完整工作流时序：**

```
writing-plans          using-git-worktrees       executing-plans
      |                        |                       |
      | plan committed         |                       |
      |----------------------->|                       |
      |                        | select dir            |
      |                        |---.                   |
      |                        |   | (priority logic)  |
      |                        |<--'                   |
      |                        | verify gitignore      |
      |                        |---.                   |
      |                        |<--'                   |
      |                        | git worktree add      |
      |                        |---.                   |
      |                        |<--'                   |
      |                        | run setup             |
      |                        |---.                   |
      |                        |<--'                   |
      |                        | verify baseline       |
      |                        |---.                   |
      |                        |<--'                   |
      |                        | "Worktree ready"      |
      |                        |---------------------->|
      |                        |                  begin impl
```

### 3.5 与其他 Skills 的协作关系

| 协作 Skill | 关系类型 | 说明 |
|------------|----------|------|
| brainstorming | 上游（触发方） | Phase 4 中设计被批准时，REQUIRED 调用本 skill |
| writing-plans | 上游（触发方） | 计划写完并提交到 main 后，触发本 skill 建立实现环境 |
| executing-plans | 下游（被启动） | worktree 就绪后，在隔离环境中开始执行任务 |
| subagent-driven-development | 下游（被启动） | worktree 就绪后，subagent 在隔离环境中并行执行 |
| finishing-a-development-branch | 配对（收尾方） | 本 skill 是开始（建立隔离），finishing 是结束（清理合并），形成完整生命周期 |
