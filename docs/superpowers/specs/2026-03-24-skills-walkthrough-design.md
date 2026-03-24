# Superpowers Skills Walkthrough — Design Spec

**Date:** 2026-03-24
**Status:** Approved
**Location:** `docs/superpowers/skill-walkthrough/`

---

## Goal

Systematically document all 14 Superpowers skills as a personal knowledge base for deep mastery. Each skill is covered with: original content, Chinese translation, and multi-dimensional analysis (function, positioning, use cases, diagrams, tables).

---

## Directory Structure

```
docs/superpowers/skill-walkthrough/
├── 00-overview.md                        # Global topology + 14-skill quick-ref table + typical workflow paths
├── 01-using-superpowers.md               # SKILL.md + references/codex-tools.md + references/gemini-tools.md
├── 02-brainstorming.md                   # SKILL.md + spec-document-reviewer-prompt.md + visual-companion.md
├── 03-writing-plans.md                   # SKILL.md + plan-document-reviewer-prompt.md
├── 04-using-git-worktrees.md             # SKILL.md
├── 05-executing-plans.md                 # SKILL.md
├── 06-subagent-driven-development.md     # SKILL.md + implementer-prompt.md + spec-reviewer-prompt.md + code-quality-reviewer-prompt.md
├── 07-dispatching-parallel-agents.md     # SKILL.md
├── 08-test-driven-development.md         # SKILL.md + testing-anti-patterns.md
├── 09-systematic-debugging.md            # SKILL.md + root-cause-tracing.md + defense-in-depth.md + condition-based-waiting.md + find-polluter.sh
├── 10-verification-before-completion.md  # SKILL.md
├── 11-requesting-code-review.md          # SKILL.md + code-reviewer.md
├── 12-receiving-code-review.md           # SKILL.md
├── 13-finishing-a-development-branch.md  # SKILL.md
└── 14-writing-skills.md                  # SKILL.md + anthropic-best-practices.md + persuasion-principles.md + graphviz-conventions.dot + testing-skills-with-subagents.md + examples/
```

---

## Skill Ordering Rationale

Skills are ordered by **natural invocation sequence** in a typical development task:

```
[Entry Point]
using-superpowers (meta-skill, how to find/invoke all others)
    ↓
[Planning Phase]
brainstorming → writing-plans
    ↓
[Isolation Setup]
using-git-worktrees  ← after plan committed to main, before implementation
    ↓
[Execution Phase]
executing-plans  OR  subagent-driven-development
    + dispatching-parallel-agents (acceleration, called within execution)
    + test-driven-development (before writing implementation code)
    + systematic-debugging (when bugs/failures encountered)
    + verification-before-completion (before claiming done)
    ↓
[Review Phase]
requesting-code-review → receiving-code-review
    ↓
[Completion Phase]
finishing-a-development-branch
    ↓
[Meta Level]
writing-skills (creating/editing skills themselves)
```

**Note on spec/plan + worktrees:** `brainstorming` and `writing-plans` commit spec and plan docs directly to main — this is intentional. These are decision records, not code pollution. `using-git-worktrees` then forks from that main HEAD to create an isolated implementation branch. If you're exploring without intent to implement immediately, don't commit the plan yet — keep it as a local draft.

---

## Per-File Template

Every skill file (`01-` through `14-`) follows this fixed structure:

```markdown
# [NN] Skill Name

## 速查卡 (Quick Reference)
| 维度           | 内容 |
|----------------|------|
| 触发时机       | When to invoke this skill |
| 调用链上游     | Skills that call this one |
| 调用链下游     | Skills this one calls/transitions to |
| 核心产出       | Primary output/artifact |
| 关联 Subagents | Agent prompts bundled with this skill |

---

## 一、原文

> SKILL.md 原始内容（完整保留，不删减）

### References / Subagent Prompts

> 每个附属文档的完整原文，标注文件名

---

## 二、中文翻译

> 按章节对照翻译，保留原文结构

---

## 三、剖析解读

### 3.1 功能与定位
- 这个 skill 解决什么问题
- 在整个工作流中的角色

### 3.2 使用场景与案例
- 典型场景描述（构造案例，如"假设你要开发一个登录功能..."）
- 正确使用 vs 错误使用对比

### 3.3 Subagents / References 深度解读
- 每个附属文档的作用、设计意图
- 与主 SKILL.md 的关系

### 3.4 流程图 / 示意图
- ASCII 或 Mermaid 流程图
- 数据流、决策树等

### 3.5 与其他 Skills 的协作关系
| 协作 Skill | 关系类型 | 说明 |
|------------|----------|------|
| ...        | 上游/下游/并列 | ... |
```

---

## `00-overview.md` Structure

1. **全局拓扑图** — ASCII diagram showing all 14 skills and their call relationships
2. **14 Skills 速查表** — one-row-per-skill: trigger condition, input, output, upstream, downstream
3. **典型主干路径** — end-to-end walkthrough of a new feature (brainstorming → finishing)
4. **典型分支路径** — debugging detour, parallel agent acceleration, writing a new skill

---

## Depth Policy

All 14 skills use the **same template depth** (Option A: uniform depth). Complex skills (brainstorming, subagent-driven-development, systematic-debugging) will naturally produce longer sections due to more content — no artificial truncation.

---

## Content Sources

- **Original content:** All files under `skills/*/` in this repo:
  - Every skill: `skills/<name>/SKILL.md`
  - Subagent prompts: `skills/<name>/*-prompt.md`
  - Reference docs: `skills/<name>/references/*.md`
  - Supplementary: any additional `.md`, `.sh`, `.ts`, `.dot` files in the skill directory
  - `writing-skills` only: `skills/writing-skills/examples/CLAUDE_MD_TESTING.md`
- **Translation style:** Technical terms and code identifiers stay in English (e.g., `worktree`, `SKILL.md`, skill names). All prose translated to Chinese.
- **Examples/cases:** Purely theoretical, constructed from skill doc content (no real project cases)
- **Audience:** Self-reference + others learning the system; both quick-lookup and deep-read modes supported

---

## Out of Scope

- Implementing any new skill functionality
- Modifying existing SKILL.md files
- Creating a "Phase 2" workflow-based restructure (reserved for future version)
