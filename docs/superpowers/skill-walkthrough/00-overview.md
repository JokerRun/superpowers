# Superpowers Skills — 全局总览

## 全局调用链拓扑图

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         using-superpowers (01)                              │
│              Session 启动时加载，所有 skill 的调用前提                        │
└──────────────────────────────────┬──────────────────────────────────────────┘
                                   │ 触发所有下游 skill
                                   ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                         brainstorming (02)                                   │
│            创意/功能/需求 → 设计稿 → spec doc → spec-review-loop              │
└──────┬───────────────────────────────────────────────────────────────────────┘
       │ 设计通过后必须调用
       ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                         writing-plans (03)                                   │
│              spec → 实现计划文档 → plan-review-loop → 执行方式选择            │
└──────┬──────────────────────────────────┬────────────────────────────────────┘
       │ 用户选"Subagent 模式"            │ 用户选"Inline 模式"
       ▼                                  ▼
┌──────────────────────┐     ┌────────────────────────────┐
│ subagent-driven-     │     │    executing-plans (05)    │
│ development (06)     │     │                            │
│                      │     │                            │
│ 每任务 dispatch       │     │ 同 session 批量执行        │
│ 新 subagent           │     │ with review checkpoints   │
└──────┬───────────────┘     └──────────┬─────────────────┘
       │                               │
       │  两者都需要在开始前调用         │
       ▼                               ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                       using-git-worktrees (04)                               │
│           隔离工作区 → branch → setup → baseline test verification           │
└──────────────────────────────────────────────────────────────────────────────┘
       │ subagent 执行每个任务时
       ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                    test-driven-development (08)                              │
│               RED (写失败测试) → GREEN (最小实现) → REFACTOR                  │
└──────────────────────────────────────────────────────────────────────────────┘
       │ 遇到 bug/test failure 时
       ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                      systematic-debugging (09)                               │
│      Phase1:RootCause → Phase2:Pattern → Phase3:Hypothesis → Phase4:Fix     │
│      (Phase4 内调用 TDD 写 failing test)                                      │
└──────────────────────────────────────────────────────────────────────────────┘

       ┌────────────────────────────────────────────────────────┐
       │            dispatching-parallel-agents (07)            │
       │  当存在 2+ 个独立任务/bug 时，可在执行阶段并行 dispatch  │
       │  subagent-driven-dev 或 executing-plans 内部均可调用   │
       └────────────────────────────────────────────────────────┘

       │ 每个任务完成 / 准备 claim "done" 前
       ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                  verification-before-completion (10)                         │
│          证据优先：运行验证命令 → 读输出 → 确认后才能声明完成                  │
└──────────────────────────────────────────────────────────────────────────────┘
       │ 每个任务完成后 / merge 前
       ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                    requesting-code-review (11)                               │
│            dispatch code-reviewer subagent，精确传递上下文                   │
└──────────────────────────────────────────────────────────────────────────────┘
       │ 收到 review 反馈后
       ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                     receiving-code-review (12)                               │
│         技术评估（非情绪表演）→ 验证 → 逐条实现 or 有理由 pushback           │
└──────────────────────────────────────────────────────────────────────────────┘
       │ 所有任务完成 + 测试通过后
       ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                 finishing-a-development-branch (13)                          │
│     验证测试 → 4选1(本地merge/PR/保留/丢弃) → 清理 worktree                   │
└──────────────────────────────────────────────────────────────────────────────┘

       ┌────────────────────────────────────────────────────────┐
       │                  writing-skills (14)                   │
       │  元级别：TDD 应用于文档 — RED(baseline) → GREEN(写skill)│
       │  → REFACTOR(关闭漏洞)；可在任何时机创建/迭代 skill      │
       └────────────────────────────────────────────────────────┘
```

**关键横向关系（不在主干链上）：**

```
subagent-driven-development ──(内部每任务)──► requesting-code-review
subagent-driven-development ──(subagent 执行)──► test-driven-development
systematic-debugging ──(Phase4)──► test-driven-development
writing-skills ──(前提)──► test-driven-development (REQUIRED BACKGROUND)
using-git-worktrees ◄──────────────────────── finishing-a-development-branch (配对清理)
```

---

## 14 Skills 速查表

| # | Skill | 触发时机 | 核心产出 | 上游调用方 | 下游跳转 |
|---|-------|----------|----------|-----------|---------|
| 01 | using-superpowers | 任何 session 开始时 | Skill 调用规则 + 优先级框架 | — (入口) | 所有其他 skill |
| 02 | brainstorming | 任何创意/功能/需求出现时，写代码之前 | 设计稿 + spec doc (含 spec-review-loop) | using-superpowers | writing-plans |
| 03 | writing-plans | 有 spec 或需求、开始动代码之前 | 实现计划文档 (含 plan-review-loop) | brainstorming | subagent-driven-development 或 executing-plans |
| 04 | using-git-worktrees | 开始功能开发 / 执行计划之前需要隔离工作区时 | 隔离 worktree + baseline 测试通过 | brainstorming, subagent-driven-development, executing-plans | finishing-a-development-branch (清理) |
| 05 | executing-plans | 有计划文档、选择 inline 执行时 | 任务逐步执行 + review checkpoints | writing-plans | finishing-a-development-branch |
| 06 | subagent-driven-development | 有计划文档、选择 subagent 模式执行时 | 每任务 dispatch 新 subagent + 两阶段 review | writing-plans | finishing-a-development-branch |
| 07 | dispatching-parallel-agents | 存在 2+ 个独立任务/bug/test failures 可并行时 | 多 subagent 并行执行 + 整合结果 | 执行阶段内部（subagent-driven-dev / executing-plans）| 汇总后继续执行流 |
| 08 | test-driven-development | 实现任何功能或 bugfix、写实现代码之前 | RED-GREEN-REFACTOR 循环 + 通过测试 | subagent（执行任务时）, systematic-debugging (Phase4) | — |
| 09 | systematic-debugging | 遇到 bug / test failure / 意外行为时，提出修复前 | 根因 + 最小 fix + 回归测试 | 执行阶段遇到问题时 | test-driven-development (Phase4) |
| 10 | verification-before-completion | 准备声明工作完成 / commit / PR 之前 | 执行验证命令 + 证据输出 | 每个任务完成节点 | requesting-code-review |
| 11 | requesting-code-review | 任务完成后 / 主要功能完成 / merge 前 | code-reviewer subagent 反馈报告 | subagent-driven-development (每任务后), executing-plans (每批次后) | receiving-code-review |
| 12 | receiving-code-review | 收到 code review 反馈时 | 技术验证 + 修复 or 有据 pushback | requesting-code-review | 继续执行或 finishing-a-development-branch |
| 13 | finishing-a-development-branch | 实现完成、测试通过、需要整合时 | merge/PR/保留/丢弃 + worktree 清理 | subagent-driven-development, executing-plans | — (终态) |
| 14 | writing-skills | 创建新 skill / 编辑现有 skill / 部署前验证时 | 通过测试的 SKILL.md 文档 | 任意时机（元操作）| test-driven-development (前提依赖) |

---

## 典型主干路径：新功能开发 (End-to-End)

以下为一次完整新功能开发的标准流程：

**1. brainstorming** — 用户描述想法后，先探索项目上下文，逐一提问澄清需求，提出 2-3 方案，展示设计后用户审批，写入 spec doc，dispatch spec-review 子代理，用户最终确认 spec。

**2. writing-plans** — 基于 spec 生成详细实现计划，每步 2-5 分钟粒度，包含精确文件路径和代码。dispatch plan-reviewer 子代理，通过后提供执行方式选择（Subagent 模式 vs Inline 模式）。

**3. using-git-worktrees** — 无论选哪种执行模式，都必须先创建隔离 worktree，验证目录已被 .gitignore 忽略，运行 baseline 测试确认干净起点。

**4. subagent-driven-development / executing-plans** — 按计划逐任务执行。Subagent 模式下每任务 dispatch 新的 implementer subagent，执行完后做两阶段 review（spec compliance → code quality）。Inline 模式下在当前 session 批量执行，每批次后做 review checkpoint。

**5. test-driven-development** — 每个任务内，subagent 必须遵循 RED-GREEN-REFACTOR：先写失败测试，看着它失败，再写最小实现使测试通过，最后 refactor。

**6. verification-before-completion** — 每个任务声明完成前，必须运行验证命令、读取完整输出，用证据支撑完成声明，不能靠"应该通过"的主观判断。

**7. requesting-code-review** — subagent-driven-development 每任务后（spec + quality 两次）、executing-plans 每批次后，dispatch code-reviewer subagent，提供精确的 git SHA 范围和上下文（不传递 session 历史）。

**8. receiving-code-review** — 收到 review 反馈后，技术评估每条建议：理解 → 验证 → 实现或有据 pushback。不做情绪表演，不盲目实现。

**9. finishing-a-development-branch** — 所有任务完成且测试通过后，选择整合方式（本地 merge / 创建 PR / 保留分支 / 丢弃），执行后清理 worktree（Option 1 和 4 需清理，Option 2 和 3 保留）。

---

## 典型分支路径

### 调试分支 (遇到 Bug 时)

在 executing-plans 或 subagent-driven-development 执行期间，任何任务遇到 bug / test failure / 意外行为时，**立即**调用 `systematic-debugging`，而不是直接猜测并修复。

流程：
- **Phase 1 (Root Cause)** — 读错误信息、复现、检查 recent changes、在多组件系统中添加诊断 instrumentation
- **Phase 2 (Pattern)** — 找同类可工作的代码，对比差异
- **Phase 3 (Hypothesis)** — 形成单一假设并最小化测试
- **Phase 4 (Implementation)** — 调用 `test-driven-development` 写 failing test 复现 bug，再实现 fix

若连续 3 次 fix 都失败，停下来讨论架构问题，不能继续第 4 次猜测修复。

调试完成后回归执行主干，继续 `verification-before-completion` → `requesting-code-review`。

### 并行加速分支 (dispatching-parallel-agents)

当执行阶段遇到以下情况时触发：
- 多个独立 test failures（不同文件/子系统，互不影响）
- 多个独立任务可以无共享状态地并行处理

此时 `dispatching-parallel-agents` 作为**加速器**嵌入执行流程内部：一个问题一个 agent，并发执行，结果汇总后验证无冲突，再继续主干。

不适用场景：failures 相互关联（修一个可能影响另一个），或需要全局系统状态理解。

### 创造新 Skill (writing-skills)

这是一条**元级别**的工作流，可在任意时机发起：

1. **识别触发** — 遇到非直觉显见的技术、跨项目可复用的模式、他人也能受益的方法时
2. **RED（基线测试）** — 先在没有 skill 的情况下运行压力场景，记录 subagent 的失败行为和理由（必须先看失败）
3. **GREEN（写最小 skill）** — 针对基线测试暴露的具体漏洞写 SKILL.md；description 只写触发条件，不写流程摘要（避免 CSO 陷阱）
4. **REFACTOR（关闭漏洞）** — 找新的合理化借口，补充反驳表、Red Flags 列表，反复测试直到无漏洞

前提依赖：必须先掌握 `test-driven-development`，因为 writing-skills 本质是 TDD 应用于文档。
