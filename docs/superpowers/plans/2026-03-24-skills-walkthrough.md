# Skills Walkthrough Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create 15 walkthrough documents in `docs/superpowers/skill-walkthrough/` covering all 14 Superpowers skills with original content, Chinese translation, and multi-dimensional analysis.

**Architecture:** One file per skill ordered by natural invocation sequence, each following a fixed template (速查卡 → 原文 → 中文翻译 → 剖析解读). A `00-overview.md` provides global topology and quick-reference table. All source files are read-only references; no existing skill files are modified.

**Tech Stack:** Markdown only. Source files in `skills/*/`. Output to `docs/superpowers/skill-walkthrough/`.

---

## File Map

| Output File | Source Files |
|-------------|-------------|
| `00-overview.md` | All SKILL.md files (for topology data) |
| `01-using-superpowers.md` | `skills/using-superpowers/SKILL.md`, `skills/using-superpowers/references/codex-tools.md`, `skills/using-superpowers/references/gemini-tools.md` |
| `02-brainstorming.md` | `skills/brainstorming/SKILL.md`, `skills/brainstorming/spec-document-reviewer-prompt.md`, `skills/brainstorming/visual-companion.md` *(scripts/ excluded — implementation code, not doc content)* |
| `03-writing-plans.md` | `skills/writing-plans/SKILL.md`, `skills/writing-plans/plan-document-reviewer-prompt.md` |
| `04-using-git-worktrees.md` | `skills/using-git-worktrees/SKILL.md` |
| `05-executing-plans.md` | `skills/executing-plans/SKILL.md` |
| `06-subagent-driven-development.md` | `skills/subagent-driven-development/SKILL.md`, `skills/subagent-driven-development/implementer-prompt.md`, `skills/subagent-driven-development/spec-reviewer-prompt.md`, `skills/subagent-driven-development/code-quality-reviewer-prompt.md` |
| `07-dispatching-parallel-agents.md` | `skills/dispatching-parallel-agents/SKILL.md` |
| `08-test-driven-development.md` | `skills/test-driven-development/SKILL.md`, `skills/test-driven-development/testing-anti-patterns.md` |
| `09-systematic-debugging.md` | `skills/systematic-debugging/SKILL.md`, `skills/systematic-debugging/root-cause-tracing.md`, `skills/systematic-debugging/defense-in-depth.md`, `skills/systematic-debugging/condition-based-waiting.md`, `skills/systematic-debugging/condition-based-waiting-example.ts`, `skills/systematic-debugging/find-polluter.sh`, `skills/systematic-debugging/test-academic.md`, `skills/systematic-debugging/test-pressure-1.md`, `skills/systematic-debugging/test-pressure-2.md`, `skills/systematic-debugging/test-pressure-3.md` *(CREATION-LOG.md excluded — internal dev log, not doc content)* |
| `10-verification-before-completion.md` | `skills/verification-before-completion/SKILL.md` |
| `11-requesting-code-review.md` | `skills/requesting-code-review/SKILL.md`, `skills/requesting-code-review/code-reviewer.md` |
| `12-receiving-code-review.md` | `skills/receiving-code-review/SKILL.md` |
| `13-finishing-a-development-branch.md` | `skills/finishing-a-development-branch/SKILL.md` |
| `14-writing-skills.md` | `skills/writing-skills/SKILL.md`, `skills/writing-skills/anthropic-best-practices.md`, `skills/writing-skills/persuasion-principles.md`, `skills/writing-skills/graphviz-conventions.dot`, `skills/writing-skills/testing-skills-with-subagents.md`, `skills/writing-skills/render-graphs.js`, `skills/writing-skills/examples/CLAUDE_MD_TESTING.md` |

---

## Per-File Template (applies to tasks 2–15)

Every skill file must follow this exact structure:

```markdown
# [NN] Skill Name

## 速查卡 (Quick Reference)
| 维度           | 内容 |
|----------------|------|
| 触发时机       | ... |
| 调用链上游     | ... |
| 调用链下游     | ... |
| 核心产出       | ... |
| 关联 Subagents | ... |

---

## 一、原文

### SKILL.md

(完整原文，不删减)

### [filename] (每个附属文档单独一节)

(完整原文)

---

## 二、中文翻译

### SKILL.md 翻译

(按章节对照翻译，保留原文标题结构)

### [filename] 翻译

(同上)

---

## 三、剖析解读

### 3.1 功能与定位

### 3.2 使用场景与案例

### 3.3 Subagents / References 深度解读
(仅有附属文档时才写此节)

### 3.4 流程图 / 示意图

### 3.5 与其他 Skills 的协作关系
| 协作 Skill | 关系类型 | 说明 |
|------------|----------|------|
```

**Translation rules:**
- Technical terms stay in English: `worktree`, `SKILL.md`, skill names, CLI commands, file paths, code
- All prose translated to Chinese
- Section headings: keep English heading + add Chinese in parentheses, or use Chinese directly for new headings

---

## Task 1: Create `00-overview.md`

**Files:**
- Create: `docs/superpowers/skill-walkthrough/00-overview.md`
- Read: all 14 `skills/*/SKILL.md` files

- [ ] **Step 1: Read all SKILL.md files for topology data**

Read each of the 14 SKILL.md files, noting:
- `description:` frontmatter field (trigger condition)
- `## Integration` section (Called by / Pairs with / Transitions to)

- [ ] **Step 2: Write `00-overview.md`**

Structure:
```markdown
# Superpowers Skills — 全局总览

## 全局调用链拓扑图

(ASCII diagram showing all 14 skills and their directional relationships)

## 14 Skills 速查表

| # | Skill | 触发时机 | 核心产出 | 上游 | 下游 |
|---|-------|----------|----------|------|------|
| 01 | using-superpowers | ... | ... | — | all others |
...

## 典型主干路径：新功能开发 (End-to-End)

(Step-by-step narrative: brainstorming → writing-plans → using-git-worktrees → executing-plans/SDD → TDD → verification → code-review → finishing)

## 典型分支路径

### 调试分支 (遇到 Bug)
### 并行加速分支 (dispatching-parallel-agents)
### 创造新 Skill (writing-skills)
```

- [ ] **Step 3: Commit**

```bash
git add docs/superpowers/skill-walkthrough/00-overview.md
git commit -m "docs: add skills walkthrough 00-overview (topology + quick-ref table)"
```

---

## Task 2: `01-using-superpowers.md`

**Files:**
- Create: `docs/superpowers/skill-walkthrough/01-using-superpowers.md`
- Read: `skills/using-superpowers/SKILL.md`, `skills/using-superpowers/references/codex-tools.md`, `skills/using-superpowers/references/gemini-tools.md`

- [ ] **Step 1: Read all 3 source files**

- [ ] **Step 2: Write walkthrough file** following the per-file template

Key analysis points for 3.1–3.5:
- This is the **meta-skill / entry point** — its job is to tell Claude how to discover and use all other skills
- 3.2 案例: "假设你刚开始一个新会话，想要开发一个新功能——using-superpowers 确保 Claude 先检查可用 skills 再采取任何行动"
- 3.3: `codex-tools.md` 和 `gemini-tools.md` 分别描述在 Codex App 和 Gemini 环境下的工具差异
- 3.4: 简单流程图（session start → check skills → invoke appropriate skill）
- 3.5: 上游无，下游 = 所有其他 skills

- [ ] **Step 3: Commit**

```bash
git add docs/superpowers/skill-walkthrough/01-using-superpowers.md
git commit -m "docs: add skills walkthrough 01-using-superpowers"
```

---

## Task 3: `02-brainstorming.md`

**Files:**
- Create: `docs/superpowers/skill-walkthrough/02-brainstorming.md`
- Read: `skills/brainstorming/SKILL.md`, `skills/brainstorming/spec-document-reviewer-prompt.md`, `skills/brainstorming/visual-companion.md`
- **Exclude:** `skills/brainstorming/scripts/` (implementation code for visual companion server, not doc content)

- [ ] **Step 1: Read all 3 source files**

- [ ] **Step 2: Write walkthrough file** following the per-file template

Key analysis points:
- **HARD-GATE**: 这是最重要的约束——在设计获批前绝对不能写代码。理解为什么这个规则存在
- 3.2 案例: "假设你要为一个电商平台添加优惠券系统——brainstorming 会先问清楚优惠券类型、折扣逻辑、用户界面等，再输出设计文档，而不是立刻开始写代码"
- 3.3: `spec-document-reviewer-prompt.md` 是 subagent 提示词模板，用于 spec 写完后的自动审查；`visual-companion.md` 描述浏览器辅助工具的详细用法
- 3.4: 完整复现 SKILL.md 中的 graphviz 流程图为 ASCII 版本
- Process flow: 9步骤的完整闭环（explore → visual? → clarify → approaches → design → write spec → spec review → user review → writing-plans）

- [ ] **Step 3: Commit**

```bash
git add docs/superpowers/skill-walkthrough/02-brainstorming.md
git commit -m "docs: add skills walkthrough 02-brainstorming"
```

---

## Task 4: `03-writing-plans.md`

**Files:**
- Create: `docs/superpowers/skill-walkthrough/03-writing-plans.md`
- Read: `skills/writing-plans/SKILL.md`, `skills/writing-plans/plan-document-reviewer-prompt.md`

- [ ] **Step 1: Read both source files**

- [ ] **Step 2: Write walkthrough file** following the per-file template

Key analysis points:
- 3.2 案例: "假设 spec 已批准，需要将优惠券功能拆解为可执行任务——writing-plans 输出带 TDD 步骤、精确文件路径、预期输出的 bite-sized 任务清单"
- 3.3: `plan-document-reviewer-prompt.md` 的 review 维度（completeness、TDD compliance、task granularity、file paths）
- 3.4: 计划文档结构图（header → file map → tasks → each task: files/steps/commands）
- 关键原则表: DRY / YAGNI / TDD / frequent commits 各自含义和在计划中的体现

- [ ] **Step 3: Commit**

```bash
git add docs/superpowers/skill-walkthrough/03-writing-plans.md
git commit -m "docs: add skills walkthrough 03-writing-plans"
```

---

## Task 5: `04-using-git-worktrees.md`

**Files:**
- Create: `docs/superpowers/skill-walkthrough/04-using-git-worktrees.md`
- Read: `skills/using-git-worktrees/SKILL.md`

- [ ] **Step 1: Read source file**

- [ ] **Step 2: Write walkthrough file** following the per-file template

Key analysis points:
- 3.2 案例: "plan 已提交到 main，准备开始实现——using-git-worktrees 在 `.worktrees/feature-coupon` 创建隔离分支，运行 npm install + npm test 确认 baseline 干净，再开始实现"
- 3.2 正确 vs 错误对比: 错误=直接在 main 上开发；正确=worktree 隔离
- 3.4: 目录选择优先级决策树（existing dir > CLAUDE.md > ask user）
- 特别说明 spec/plan 在 main 的设计意图（决策记录 vs 代码实现的分离）

- [ ] **Step 3: Commit**

```bash
git add docs/superpowers/skill-walkthrough/04-using-git-worktrees.md
git commit -m "docs: add skills walkthrough 04-using-git-worktrees"
```

---

## Task 6: `05-executing-plans.md`

**Files:**
- Create: `docs/superpowers/skill-walkthrough/05-executing-plans.md`
- Read: `skills/executing-plans/SKILL.md`

- [ ] **Step 1: Read source file**

- [ ] **Step 2: Write walkthrough file** following the per-file template

Key analysis points:
- 3.2 案例: "有一份 10 任务的实现计划，选择 inline execution——executing-plans 按 checkpoint 批次执行，每批完成后等待 review"
- 3.2: 与 `subagent-driven-development` 的选择时机对比（inline vs fresh subagent per task）
- 3.4: checkpoint 机制示意图

- [ ] **Step 3: Commit**

```bash
git add docs/superpowers/skill-walkthrough/05-executing-plans.md
git commit -m "docs: add skills walkthrough 05-executing-plans"
```

---

## Task 7: `06-subagent-driven-development.md`

**Files:**
- Create: `docs/superpowers/skill-walkthrough/06-subagent-driven-development.md`
- Read: `skills/subagent-driven-development/SKILL.md`, `skills/subagent-driven-development/implementer-prompt.md`, `skills/subagent-driven-development/spec-reviewer-prompt.md`, `skills/subagent-driven-development/code-quality-reviewer-prompt.md`

- [ ] **Step 1: Read all 4 source files**

- [ ] **Step 2: Write walkthrough file** following the per-file template

Key analysis points:
- 3.2 案例: "10 个独立任务，选择 subagent-driven——每个任务 dispatch 一个 fresh implementer subagent，完成后 dispatch code-quality-reviewer，再执行下一个"
- 3.3: 三个 prompt 文件详解：
  - `implementer-prompt.md`: 给实现 subagent 的指令模板（读 plan → 实现 → TDD → commit）
  - `spec-reviewer-prompt.md`: 实现前校验 task 与 spec 一致性
  - `code-quality-reviewer-prompt.md`: 实现后审查代码质量
- 3.4: subagent 调度循环图（main agent → dispatch implementer → dispatch reviewer → loop）
- 对比表: SDD vs executing-plans（上下文隔离、review 频率、适用场景）

- [ ] **Step 3: Commit**

```bash
git add docs/superpowers/skill-walkthrough/06-subagent-driven-development.md
git commit -m "docs: add skills walkthrough 06-subagent-driven-development"
```

---

## Task 8: `07-dispatching-parallel-agents.md`

**Files:**
- Create: `docs/superpowers/skill-walkthrough/07-dispatching-parallel-agents.md`
- Read: `skills/dispatching-parallel-agents/SKILL.md`

- [ ] **Step 1: Read source file**

- [ ] **Step 2: Write walkthrough file** following the per-file template

Key analysis points:
- 3.2 案例: "需要同时实现登录模块和商品模块，两者无共享状态——dispatching-parallel-agents 同时 dispatch 两个 implementer，节省时间"
- 3.2: 判断是否可并行的条件（无共享状态、无顺序依赖）
- 3.4: 串行 vs 并行对比示意图

- [ ] **Step 3: Commit**

```bash
git add docs/superpowers/skill-walkthrough/07-dispatching-parallel-agents.md
git commit -m "docs: add skills walkthrough 07-dispatching-parallel-agents"
```

---

## Task 9: `08-test-driven-development.md`

**Files:**
- Create: `docs/superpowers/skill-walkthrough/08-test-driven-development.md`
- Read: `skills/test-driven-development/SKILL.md`, `skills/test-driven-development/testing-anti-patterns.md`

- [ ] **Step 1: Read both source files**

- [ ] **Step 2: Write walkthrough file** following the per-file template

Key analysis points:
- 3.2 案例: "实现优惠券折扣计算函数——TDD 要求先写 `test_apply_coupon_discount()`，运行确认 FAIL，再写最小实现，再确认 PASS，再 commit"
- 3.3: `testing-anti-patterns.md` 逐条解读，每个 anti-pattern 给出正确做法对比
- 3.4: Red-Green-Refactor 循环图
- 强调"测试先于实现"的原因（设计约束、可测性、防回归）

- [ ] **Step 3: Commit**

```bash
git add docs/superpowers/skill-walkthrough/08-test-driven-development.md
git commit -m "docs: add skills walkthrough 08-test-driven-development"
```

---

## Task 10: `09-systematic-debugging.md`

**Files:**
- Create: `docs/superpowers/skill-walkthrough/09-systematic-debugging.md`
- Read: `skills/systematic-debugging/SKILL.md`, `skills/systematic-debugging/root-cause-tracing.md`, `skills/systematic-debugging/defense-in-depth.md`, `skills/systematic-debugging/condition-based-waiting.md`, `skills/systematic-debugging/condition-based-waiting-example.ts`, `skills/systematic-debugging/find-polluter.sh`, `skills/systematic-debugging/test-academic.md`, `skills/systematic-debugging/test-pressure-1.md`, `skills/systematic-debugging/test-pressure-2.md`, `skills/systematic-debugging/test-pressure-3.md`
- **Exclude:** `skills/systematic-debugging/CREATION-LOG.md` (internal development log, not doc content)

- [ ] **Step 1: Read all 10 source files**

- [ ] **Step 2: Write walkthrough file** following the per-file template

Key analysis points (this is the most content-rich skill):
- 3.2 案例: "测试突然开始随机失败——systematic-debugging 要求先重现、再隔离、再找根因，而不是猜测性地修改代码"
- 3.3 各文档详解:
  - `root-cause-tracing.md`: 根因追踪方法论（5 Whys、bisect 等）
  - `defense-in-depth.md`: 防御性调试策略
  - `condition-based-waiting.md` + `condition-based-waiting-example.ts`: 条件等待模式（替代 sleep）
  - `find-polluter.sh`: 测试污染定位脚本的使用方法
  - `test-academic.md` / `test-pressure-*.md`: 测试场景案例（学术型 vs 压力测试）
- 3.4: 调试决策树（reproduce → isolate → hypothesize → verify → fix → regression test）

- [ ] **Step 3: Commit**

```bash
git add docs/superpowers/skill-walkthrough/09-systematic-debugging.md
git commit -m "docs: add skills walkthrough 09-systematic-debugging"
```

---

## Task 11: `10-verification-before-completion.md`

**Files:**
- Create: `docs/superpowers/skill-walkthrough/10-verification-before-completion.md`
- Read: `skills/verification-before-completion/SKILL.md`

- [ ] **Step 1: Read source file**

- [ ] **Step 2: Write walkthrough file** following the per-file template

Key analysis points:
- 3.2 案例: "准备声明'功能完成'——verification-before-completion 要求先跑完整测试套件并截取实际输出，再对比预期，证据在前，断言在后"
- 核心原则: **evidence before assertions** — 不能凭感觉说"应该没问题"
- 3.4: verification checklist 流程图
- 常见错误: 跑了部分测试就说全部通过；修了一个 test 就说 bug 修好了

- [ ] **Step 3: Commit**

```bash
git add docs/superpowers/skill-walkthrough/10-verification-before-completion.md
git commit -m "docs: add skills walkthrough 10-verification-before-completion"
```

---

## Task 12: `11-requesting-code-review.md`

**Files:**
- Create: `docs/superpowers/skill-walkthrough/11-requesting-code-review.md`
- Read: `skills/requesting-code-review/SKILL.md`, `skills/requesting-code-review/code-reviewer.md`

- [ ] **Step 1: Read both source files**

- [ ] **Step 2: Write walkthrough file** following the per-file template

Key analysis points:
- 3.2 案例: "实现完成、测试通过——requesting-code-review dispatch code-reviewer subagent，提供 diff + 原始需求 + 实现说明"
- 3.3: `code-reviewer.md` 的 review 维度（correctness、security、design、tests）
- 3.5: 与 `receiving-code-review` 的串联关系（request → reviewer returns feedback → receive）

- [ ] **Step 3: Commit**

```bash
git add docs/superpowers/skill-walkthrough/11-requesting-code-review.md
git commit -m "docs: add skills walkthrough 11-requesting-code-review"
```

---

## Task 13: `12-receiving-code-review.md`

**Files:**
- Create: `docs/superpowers/skill-walkthrough/12-receiving-code-review.md`
- Read: `skills/receiving-code-review/SKILL.md`

- [ ] **Step 1: Read source file**

- [ ] **Step 2: Write walkthrough file** following the per-file template

Key analysis points:
- 3.2 案例: "reviewer 指出某函数可能有 off-by-one 错误——receiving-code-review 要求先验证 reviewer 的技术判断是否正确，而不是盲目执行"
- 核心原则: **technical rigor, not performative agreement** — 不是收到 feedback 就改，要先验证
- 3.4: feedback 处理决策树（agree → implement；disagree → explain with evidence；unclear → ask）

- [ ] **Step 3: Commit**

```bash
git add docs/superpowers/skill-walkthrough/12-receiving-code-review.md
git commit -m "docs: add skills walkthrough 12-receiving-code-review"
```

---

## Task 14: `13-finishing-a-development-branch.md`

**Files:**
- Create: `docs/superpowers/skill-walkthrough/13-finishing-a-development-branch.md`
- Read: `skills/finishing-a-development-branch/SKILL.md`

- [ ] **Step 1: Read source file**

- [ ] **Step 2: Write walkthrough file** following the per-file template

Key analysis points:
- 3.2 案例: "功能开发完成，code review 通过——finishing-a-development-branch 提供结构化选项（merge / PR / cleanup），引导完成收尾"
- 3.4: 分支收尾决策树（merge to main? / create PR? / squash? / delete worktree?）
- 3.5: 与 `using-git-worktrees` 配对（开始=worktrees，结束=finishing）

- [ ] **Step 3: Commit**

```bash
git add docs/superpowers/skill-walkthrough/13-finishing-a-development-branch.md
git commit -m "docs: add skills walkthrough 13-finishing-a-development-branch"
```

---

## Task 15: `14-writing-skills.md`

**Files:**
- Create: `docs/superpowers/skill-walkthrough/14-writing-skills.md`
- Read: `skills/writing-skills/SKILL.md`, `anthropic-best-practices.md`, `persuasion-principles.md`, `graphviz-conventions.dot`, `testing-skills-with-subagents.md`, `render-graphs.js`, `examples/CLAUDE_MD_TESTING.md`

- [ ] **Step 1: Read all 7 source files**

- [ ] **Step 2: Write walkthrough file** following the per-file template

Key analysis points:
- 3.2 案例: "需要为团队创建一个新的 `code-formatting` skill——writing-skills 指导如何设计 frontmatter、写 checklist、测试 skill 行为"
- 3.3 各文档详解:
  - `anthropic-best-practices.md`: Anthropic 官方推荐的 skill 写作规范
  - `persuasion-principles.md`: 如何写出让 Claude 真正遵循的指令（persuasion vs instruction）
  - `graphviz-conventions.dot`: 流程图的标准样式约定
  - `testing-skills-with-subagents.md`: 用 subagent 测试 skill 的方法
  - `render-graphs.js`: 图形渲染工具（说明用途即可，不深入代码）
  - `examples/CLAUDE_MD_TESTING.md`: CLAUDE.md 测试示例
- 3.4: skill 文件结构图（frontmatter → overview → checklist → sections → examples）
- 这是"元级别"的 skill，理解它意味着理解整个 superpowers 系统的可扩展性

- [ ] **Step 3: Commit**

```bash
git add docs/superpowers/skill-walkthrough/14-writing-skills.md
git commit -m "docs: add skills walkthrough 14-writing-skills"
```

---

## Final Task: Verify completeness

- [ ] Confirm all 15 files exist in `docs/superpowers/skill-walkthrough/`
- [ ] Confirm `00-overview.md` topology table covers all 14 skills
- [ ] Confirm each skill file has all 5 sections (速查卡, 原文, 翻译, 剖析解读 with 3.1–3.5)
