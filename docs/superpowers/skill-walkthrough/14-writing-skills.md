# [14] Writing Skills

## 速查卡 (Quick Reference)

| 维度           | 内容 |
|----------------|------|
| 触发时机       | 创建新 skill、编辑现有 skill、或在部署前验证 skill 是否工作 |
| 调用链上游     | 任何时候发现可重用的模式/技术，且不想在下次遇到同样问题时重新摸索 |
| 调用链下游     | 所有其他 skills（writing-skills 是元级别 skill，产出的 skill 文件会影响所有工作流） |
| 核心产出       | 通过测试的 SKILL.md 文件（经历了 RED-GREEN-REFACTOR 的完整 TDD 周期） |
| 关联 Subagents | pressure scenario subagents（用于测试 skill 合规性） |

---

## 一、原文

### SKILL.md

````
---
name: writing-skills
description: Use when creating new skills, editing existing skills, or verifying skills work before deployment
---

# Writing Skills

## Overview

**Writing skills IS Test-Driven Development applied to process documentation.**

**Personal skills live in agent-specific directories (`~/.claude/skills` for Claude Code, `~/.agents/skills/` for Codex)**

You write test cases (pressure scenarios with subagents), watch them fail (baseline behavior), write the skill (documentation), watch tests pass (agents comply), and refactor (close loopholes).

**Core principle:** If you didn't watch an agent fail without the skill, you don't know if the skill teaches the right thing.

**REQUIRED BACKGROUND:** You MUST understand superpowers:test-driven-development before using this skill. That skill defines the fundamental RED-GREEN-REFACTOR cycle. This skill adapts TDD to documentation.

**Official guidance:** For Anthropic's official skill authoring best practices, see anthropic-best-practices.md. This document provides additional patterns and guidelines that complement the TDD-focused approach in this skill.

## What is a Skill?

A **skill** is a reference guide for proven techniques, patterns, or tools. Skills help future Claude instances find and apply effective approaches.

**Skills are:** Reusable techniques, patterns, tools, reference guides

**Skills are NOT:** Narratives about how you solved a problem once

## TDD Mapping for Skills

| TDD Concept | Skill Creation |
|-------------|----------------|
| **Test case** | Pressure scenario with subagent |
| **Production code** | Skill document (SKILL.md) |
| **Test fails (RED)** | Agent violates rule without skill (baseline) |
| **Test passes (GREEN)** | Agent complies with skill present |
| **Refactor** | Close loopholes while maintaining compliance |
| **Write test first** | Run baseline scenario BEFORE writing skill |
| **Watch it fail** | Document exact rationalizations agent uses |
| **Minimal code** | Write skill addressing those specific violations |
| **Watch it pass** | Verify agent now complies |
| **Refactor cycle** | Find new rationalizations → plug → re-verify |

The entire skill creation process follows RED-GREEN-REFACTOR.

## When to Create a Skill

**Create when:**
- Technique wasn't intuitively obvious to you
- You'd reference this again across projects
- Pattern applies broadly (not project-specific)
- Others would benefit

**Don't create for:**
- One-off solutions
- Standard practices well-documented elsewhere
- Project-specific conventions (put in CLAUDE.md)
- Mechanical constraints (if it's enforceable with regex/validation, automate it)

## Skill Types

### Technique
Concrete method with steps to follow (condition-based-waiting, root-cause-tracing)

### Pattern
Way of thinking about problems (flatten-with-flags, test-invariants)

### Reference
API docs, syntax guides, tool documentation (office docs)

## Directory Structure

````
skills/
  skill-name/
    SKILL.md              # Main reference (required)
    supporting-file.*     # Only if needed
````

**Flat namespace** - all skills in one searchable namespace

**Separate files for:**
1. **Heavy reference** (100+ lines) - API docs, comprehensive syntax
2. **Reusable tools** - Scripts, utilities, templates

**Keep inline:**
- Principles and concepts
- Code patterns (< 50 lines)
- Everything else

## SKILL.md Structure

**Frontmatter (YAML):**
- Only two fields supported: `name` and `description`
- Max 1024 characters total
- `name`: Use letters, numbers, and hyphens only (no parentheses, special chars)
- `description`: Third-person, describes ONLY when to use (NOT what it does)
  - Start with "Use when..." to focus on triggering conditions
  - Include specific symptoms, situations, and contexts
  - **NEVER summarize the skill's process or workflow**
  - Keep under 500 characters if possible

[...完整结构模板见原文...]

## Claude Search Optimization (CSO)

**Critical for discovery:** Future Claude needs to FIND your skill

### 1. Rich Description Field

**CRITICAL: Description = When to Use, NOT What the Skill Does**

Testing revealed that when a description summarizes the skill's workflow, Claude may follow the description instead of reading the full skill content. A description saying "code review between tasks" caused Claude to do ONE review, even though the skill's flowchart clearly showed TWO reviews.

```yaml
# ❌ BAD: Summarizes workflow - Claude may follow this instead of reading skill
description: Use when executing plans - dispatches subagent per task with code review between tasks

# ✅ GOOD: Just triggering conditions, no workflow summary
description: Use when executing implementation plans with independent tasks in the current session
````

### 4. Token Efficiency (Critical)

**Target word counts:**
- getting-started workflows: <150 words each
- Frequently-loaded skills: <200 words total
- Other skills: <500 words (still be concise)

## Flowchart Usage

[Graphviz dot 格式流程图，ASCII 重绘版见 3.4 节]

**Use flowcharts ONLY for:**
- Non-obvious decision points
- Process loops where you might stop too early
- "When to use A vs B" decisions

## The Iron Law (Same as TDD)

````
NO SKILL WITHOUT A FAILING TEST FIRST
````

This applies to NEW skills AND EDITS to existing skills.

Write skill before testing? Delete it. Start over.

**No exceptions:**
- Not for "simple additions"
- Not for "just adding a section"
- Not for "documentation updates"

## RED-GREEN-REFACTOR for Skills

### RED: Write Failing Test (Baseline)

Run pressure scenario with subagent WITHOUT the skill. Document exact behavior.

### GREEN: Write Minimal Skill

Write skill that addresses those specific rationalizations.

### REFACTOR: Close Loopholes

Agent found new rationalization? Add explicit counter. Re-test until bulletproof.

## Anti-Patterns

### ❌ Narrative Example
"In session 2025-10-03, we found empty projectDir caused..."
**Why bad:** Too specific, not reusable

### ❌ Code in Flowcharts
**Why bad:** Can't copy-paste, hard to read

### ❌ Generic Labels
helper1, helper2, step3, pattern4
**Why bad:** Labels should have semantic meaning

## Skill Creation Checklist (TDD Adapted)

**RED Phase:**
- [ ] Create pressure scenarios (3+ combined pressures for discipline skills)
- [ ] Run scenarios WITHOUT skill - document baseline behavior verbatim
- [ ] Identify patterns in rationalizations/failures

**GREEN Phase:**
- [ ] Name uses only letters, numbers, hyphens
- [ ] YAML frontmatter with only name and description (max 1024 chars)
- [ ] Description starts with "Use when..." and includes specific triggers
- [ ] Keywords throughout for search
- [ ] Address specific baseline failures

**REFACTOR Phase:**
- [ ] Identify NEW rationalizations from testing
- [ ] Add explicit counters
- [ ] Build rationalization table
- [ ] Create red flags list
- [ ] Re-test until bulletproof
````

---

### anthropic-best-practices.md

（完整原文为 Anthropic 官方文档，核心要点摘录）

```markdown
# Skill authoring best practices

> Learn how to write effective Skills that Claude can discover and use successfully.

## Core principles

### Concise is key

The context window is a public good. Only the metadata (name and description) is pre-loaded. Claude reads SKILL.md only when relevant.

**Good example: Concise** (~50 tokens):
```python
with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
````

**Bad example: Too verbose** (~150 tokens):
````
PDF (Portable Document Format) files are a common file format...
````

````

### Set appropriate degrees of freedom

- **High freedom** (text-based): Multiple valid approaches, decisions depend on context
- **Medium freedom** (pseudocode): Preferred pattern, some variation acceptable
- **Low freedom** (exact scripts): Fragile operations, consistency critical

Analogy: "Narrow bridge" (specific guardrails) vs "Open field" (general direction).

### Test with all models you plan to use

- Haiku: Does the skill provide enough guidance?
- Sonnet: Is it clear and efficient?
- Opus: Does it avoid over-explaining?

## Skill structure

### Writing effective descriptions

**Be specific and include key terms.** The description is critical for skill selection from 100+ available skills.

**Always write in third person** (injected into system prompt).

Good examples:
```yaml
description: Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files.
description: Generate descriptive commit messages by analyzing git diffs. Use when the user asks for help writing commit messages.
````

### Progressive disclosure patterns

Keep SKILL.md body under 500 lines. Use separate files for heavy reference.

### Avoid deeply nested references

Keep references one level deep from SKILL.md. Nested references cause partial reads.

## Workflows and feedback loops

Use workflows for complex tasks with clear sequential steps. Implement feedback loops (run validator → fix errors → repeat).

## Content guidelines

- Avoid time-sensitive information
- Use consistent terminology throughout

## Evaluation and iteration

**Build evaluations first.** Evaluation-driven development:
1. Run Claude on tasks WITHOUT a skill → document failures
2. Create 3 evaluation scenarios
3. Establish baseline (without skill)
4. Write minimal instructions to pass evaluations
5. Iterate

Develop skills iteratively with Claude (Claude A helps write, Claude B tests in real tasks).

## Anti-patterns to avoid

- Windows-style paths (use forward slashes)
- Too many options (provide a default with escape hatch)

## Checklist for effective Skills

- [ ] Description is specific and includes key terms
- [ ] SKILL.md body is under 500 lines
- [ ] No time-sensitive information
- [ ] Consistent terminology
- [ ] File references are one level deep
- [ ] At least three evaluations created
- [ ] Tested with Haiku, Sonnet, Opus
````

---

### persuasion-principles.md

```markdown
# Persuasion Principles for Skill Design

## Overview

LLMs respond to the same persuasion principles as humans.

**Research foundation:** Meincke et al. (2025) tested 7 persuasion principles with N=28,000 AI conversations. Compliance rates more than doubled (33% → 72%).

## The Seven Principles

### 1. Authority
Imperative language: "YOU MUST", "Never", "Always"
Non-negotiable framing: "No exceptions"
→ Best for: Discipline-enforcing skills (TDD, verification)

### 2. Commitment
Require announcements: "Announce skill usage"
Force explicit choices
Use tracking: TodoWrite for checklists
→ Best for: Ensuring skills are actually followed

### 3. Scarcity
Time-bound requirements: "Before proceeding"
Sequential dependencies: "Immediately after X"
→ Best for: Immediate verification requirements

### 4. Social Proof
Universal patterns: "Every time", "Always"
Failure modes: "X without Y = failure"
→ Best for: Documenting universal practices

### 5. Unity
Collaborative language: "our codebase", "we're colleagues"
→ Best for: Collaborative workflows

### 6. Reciprocity
Use sparingly - can feel manipulative

### 7. Liking
DON'T USE for compliance - creates sycophancy

## Principle Combinations by Skill Type

| Skill Type | Use | Avoid |
|------------|-----|-------|
| Discipline-enforcing | Authority + Commitment + Social Proof | Liking, Reciprocity |
| Guidance/technique | Moderate Authority + Unity | Heavy authority |
| Reference | Clarity only | All persuasion |

## Why This Works

- Bright-line rules reduce rationalization
- Implementation intentions create automatic behavior ("When X, do Y")
- LLMs are parahuman: trained on human text containing these patterns

## Research Citations

Meincke et al. (2025): Compliance 33% → 72% with persuasion techniques; Authority + Commitment + Scarcity most effective.
Cialdini (2021): Seven principles of persuasion empirical foundation.
```

---

### graphviz-conventions.dot

```dot
digraph STYLE_GUIDE {
    // NODE TYPES AND SHAPES:
    // Questions → diamond
    // Actions → box (default)
    // Commands → plaintext
    // States → ellipse
    // Warnings → octagon (filled red)
    // Entry/exit → doublecircle

    // EDGE LABELS:
    // Binary: "yes" / "no"
    // Multiple: "condition A" / "condition B" / "otherwise"
    // Triggers: style=dotted, label="triggers"

    // NAMING PATTERNS:
    // Questions end with ?
    // Actions start with verb
    // Commands are literal bash commands
    // States describe situation

    // SHAPE SELECTION GUIDE:
    // Decision? → diamond
    // Command? → plaintext
    // Warning? → octagon (filled red)
    // Entry/exit? → doublecircle
    // State? → ellipse
    // Default → box
}
```

---

### testing-skills-with-subagents.md

```markdown
# Testing Skills With Subagents

**Testing skills is just TDD applied to process documentation.**

## TDD Mapping for Skill Testing

| TDD Phase | Skill Testing |
|-----------|---------------|
| **RED** | Run scenario WITHOUT skill, watch agent fail |
| **GREEN** | Write skill addressing failures, verify compliance |
| **REFACTOR** | Close loopholes, add counters for new rationalizations |

## Pressure Types

| Pressure | Example |
|----------|---------|
| Time | Emergency, deadline |
| Sunk cost | Hours of work, "waste" to delete |
| Authority | Senior says skip it |
| Economic | Job, promotion at stake |
| Exhaustion | End of day |
| Social | Looking dogmatic |

**Best tests combine 3+ pressures.**

## Writing Good Scenarios

Key elements:
1. **Concrete options** - Force A/B/C choice
2. **Real constraints** - Specific times, actual consequences
3. **Make agent act** - "What do you do?" not "What should you do?"
4. **No easy outs** - Can't defer without choosing

## REFACTOR Phase: Plugging Holes

For each new rationalization, add:
1. Explicit negation in rules
2. Entry in rationalization table
3. Red flag entry
4. Update description with violation symptoms

## Meta-Testing

After agent violates rule despite having skill:
"How could that skill have been written differently to make it crystal clear Option A was the only acceptable answer?"

Three responses:
1. "Skill WAS clear, I chose to ignore it" → Need stronger foundational principle
2. "Skill should have said X" → Documentation problem, add suggestion
3. "I didn't see section Y" → Organization problem, restructure
```

---

### render-graphs.js

````javascript
#!/usr/bin/env node

/**
 * Render graphviz diagrams from a skill's SKILL.md to SVG files.
 *
 * Usage:
 *   ./render-graphs.js <skill-directory>           # Render each diagram separately
 *   ./render-graphs.js <skill-directory> --combine # Combine all into one diagram
 *
 * Extracts all ```dot blocks from SKILL.md and renders to SVG.
 * Requires: graphviz (dot) installed on system
 */

// [完整实现代码见源文件]
// 功能：从 SKILL.md 的 ```dot 块提取 graphviz 图，渲染为 SVG
// 输出到 skill 目录下的 diagrams/ 目录
// 支持 --combine 参数将所有图合并为一个 SVG
````

---

### examples/CLAUDE_MD_TESTING.md

```markdown
# Testing CLAUDE.md Skills Documentation

Testing different documentation variants to find what actually makes agents discover and use skills under pressure.

## Test Scenarios (4 scenarios × variants)

- **Scenario 1**: Time Pressure + Confidence — production down, $5k/min
- **Scenario 2**: Sunk Cost + Works Already — 45 min investment, working code
- **Scenario 3**: Authority + Speed Bias — human partner wants speed
- **Scenario 4**: Familiarity + Efficiency — "I've done this many times"

## Documentation Variants to Test

- **NULL**: No skills mention at all (baseline)
- **Variant A**: Soft Suggestion ("Consider checking...")
- **Variant B**: Directive ("Before working on any task, check...")
- **Variant C**: Emphatic Style (<important_info_about_skills>, "THIS IS EXTREMELY IMPORTANT")
- **Variant D**: Process-Oriented (step-by-step workflow)

## Expected Results

- NULL: No skill awareness
- Variant A: Checks if no pressure, skips under pressure
- Variant B: Sometimes checks, easily rationalized away
- Variant C: Strong compliance, might feel too rigid
- Variant D: Balanced, but agents need to internalize
```

---

## 二、中文翻译

### SKILL.md 翻译

**编写 Skills**

**概述**

**编写 skills 就是将测试驱动开发应用于流程文档。**

你编写测试用例（用 subagent 的压力场景），观察测试失败（基准行为），编写 skill（文档），观察测试通过（agents 合规），然后重构（关闭漏洞）。

**核心原则：** 如果你没有观察到 agent 在没有 skill 的情况下失败，你就不知道 skill 是否教了正确的东西。

**什么是 Skill？**

Skill 是经过验证的技术、模式或工具的参考指南。Skills **不是**关于你某次解决问题的叙述。

**Skill 类型：**
- Technique（技术）：有具体步骤可遵循的方法
- Pattern（模式）：思考问题的方式
- Reference（参考）：API 文档、语法指南

**TDD 映射关系**

| TDD 概念 | Skill 创建 |
|----------|-----------|
| 测试用例 | 压力场景（subagent） |
| 生产代码 | SKILL.md |
| 测试失败（RED） | Agent 在没有 skill 时违反规则（基准） |
| 测试通过（GREEN） | Agent 在有 skill 时遵规 |
| 重构 | 关闭漏洞，同时维持合规性 |

**SKILL.md 结构**

frontmatter：只有 `name` 和 `description` 两个字段（最多 1024 字符）
- `name`：只用字母、数字、连字符
- `description`：第三人称，只描述**何时使用**（不描述做什么），以 "Use when..." 开头

**Claude 搜索优化（CSO）**

**关键洞见**：description 字段只描述触发条件，不描述工作流程。原因：测试发现，如果 description 概括了工作流，Claude 可能跟随 description 而不读完整的 skill 内容。

例如，一个说"tasks 之间做 code review"的 description 让 Claude 只做了一次 review，即使 skill 的流程图清楚地显示需要两次（spec compliance + code quality）。

**Token 效率：**
- getting-started 工作流：每个 <150 词
- 频繁加载的 skills：<200 词
- 其他 skills：<500 词

**铁律**

```
没有 failing test 的 skill = 违规
```

适用于新 skill 和对现有 skill 的修改。先写再测试？删掉，重来。

**RED-GREEN-REFACTOR for Skills**

- **RED**：在没有 skill 的情况下运行压力场景，记录 agent 的确切失败和合理化借口
- **GREEN**：写 skill 解决这些具体的失败
- **REFACTOR**：发现新的合理化？加明确的对抗条目。重测直到无懈可击

---

### anthropic-best-practices.md 翻译

**Skill 编写最佳实践（Anthropic 官方）**

**简洁是关键**：context window 是公共资源。只添加 Claude 没有的信息。

**适度自由**：根据任务的脆弱性和可变性匹配具体程度。
- 高自由度（文本指令）：多种方法都有效
- 中等自由度（伪代码）：有首选模式但允许变化
- 低自由度（精确脚本）：fragile 操作，一致性关键

**与所有模型测试**：Haiku 需要更多指导，Opus 不需要过度解释。

**Progressive disclosure（渐进式披露）**：SKILL.md 保持 500 行以下。详情放在单独文件。

**建立评估（Evaluations）**：先创建测试场景，建立基准，再写 skill。观察 Claude B 在实际任务中如何使用，带反馈回给 Claude A 改进。

---

### persuasion-principles.md 翻译

**Skill 设计的说服原则**

**研究基础**：Meincke et al. (2025) 用 N=28,000 次 AI 对话测试了 7 个说服原则。说服技术将合规率从 33% 提升到 72%（p < .001）。

**七大原则**：
1. **权威（Authority）**：命令式语言（"YOU MUST"，"No exceptions"）— 用于纪律执行类 skill
2. **承诺（Commitment）**：要求声明、强制明确选择 — 确保 skill 被真正遵守
3. **稀缺性（Scarcity）**：时间约束（"Before proceeding"）— 防止拖延
4. **社会证明（Social Proof）**：普遍模式（"Every time"）— 建立规范
5. **一体性（Unity）**：协作语言（"our codebase"）— 用于协作工作流
6. **互惠（Reciprocity）**：少用，容易感觉被操纵
7. **喜好（Liking）**：**不用于合规执行**——创造奉承行为

**伦理使用检验**：如果用户完全理解这种技术，它是否服务于他们的真实利益？

---

### graphviz-conventions.dot 翻译

这是 Graphviz 流程图的样式约定文件，定义了各种节点类型的标准形状：
- 问题/决策 → diamond（菱形）
- 操作步骤 → box（方框，默认）
- 命令（如 `git commit`）→ plaintext（无边框）
- 状态 → ellipse（椭圆）
- 警告 → octagon（八角形，红色填充）
- 开始/结束 → doublecircle（双圆）

---

### testing-skills-with-subagents.md 翻译

**用 Subagents 测试 Skills**

这是 TDD 应用于流程文档的完整测试方法论。与编写代码的 TDD 完全对应：

- **RED（失败）**：在没有 skill 的情况下运行压力场景，观察 agent 失败并记录确切的合理化借口
- **GREEN（通过）**：写 skill，针对这些具体失败，验证 agent 现在合规
- **REFACTOR（重构）**：发现新漏洞，关闭它们

最有效的压力测试结合 3+ 种压力类型（时间压力 + 沉没成本 + 疲惫 + 权威）。

**Meta-testing**：当 agent 在有 skill 的情况下还是违规时，问它"这个 skill 怎么写才能让 Option A 更清晰？"——这会揭示是文档问题还是需要更强的基础原则。

---

### render-graphs.js 翻译

这是一个 Node.js 工具脚本，用途：从 SKILL.md 中提取所有 ` ```dot ` 格式的 graphviz 图，渲染为 SVG 文件，帮助用户可视化流程图。

使用方法：
```bash
./render-graphs.js ../some-skill            # 每个图单独渲染
./render-graphs.js ../some-skill --combine  # 所有图合并为一个 SVG
```

依赖：系统需安装 graphviz（`brew install graphviz`）。

---

### examples/CLAUDE_MD_TESTING.md 翻译

这是一个完整的测试活动文档，测试不同 CLAUDE.md 文档变体对 agent 发现和使用 skills 的影响。

4 个压力场景（生产宕机 / 沉没成本 / 权威压力 / 熟悉度偏见）× 5 个文档变体（NULL 基准 / 软建议 / 指令式 / 强调式 / 流程导向）。

测试协议：先运行 NULL 基准，记录 agent 的选择和合理化；再用各变体测试；在压力场景下测试。

---

## 三、剖析解读

### 3.1 功能与定位

writing-skills 是整个 Superpowers 系统中**唯一的元级别 skill**——它的产出不是代码或文档，而是**新的 skills**。理解这个 skill 意味着理解整个系统的可扩展性。

它的核心洞见是：**skill 创建和代码开发面临同样的问题——没有测试就不知道它是否工作**。一个没有经过压力测试的 skill 就像没有测试的代码——可能在理想条件下工作，但在压力下（时间紧迫、沉没成本、权威压力）会被绕过。

这个 skill 绑定了 7 个支撑文件：
- **anthropic-best-practices.md**：Anthropic 官方指南，关注 token 效率和可发现性
- **persuasion-principles.md**：研究基础，解释为什么 skill 的措辞方式会影响 agent 合规率
- **graphviz-conventions.dot**：标准化流程图样式
- **testing-skills-with-subagents.md**：完整的 TDD-for-skills 测试方法论
- **render-graphs.js**：将 dot 图渲染为 SVG 的工具
- **examples/CLAUDE_MD_TESTING.md**：一个完整的测试活动案例

### 3.2 使用场景与案例

**场景：为团队创建 `code-formatting` skill**

假设在项目中反复遇到一个问题：AI agent 格式化代码时不一致，有时用 Prettier，有时不用，有时混用。你想创建一个 skill 规范这个行为。

**错误做法（跳过测试）**：
- 直接写 SKILL.md 描述"用 Prettier 格式化"
- 部署，认为完成了

**正确做法（RED-GREEN-REFACTOR）**：

**RED phase**：
```
创建压力场景：
- 时间压力："只有 5 分钟，改一个小 bug，不想等 Prettier"
- 已有惯例："其他文件都没用 Prettier"
- 权威："项目创始人说不需要 Prettier"
运行场景（没有 code-formatting skill）
记录：Agent 选择不运行 Prettier，理由是"这是小改动"
```

**GREEN phase**：
```
写 skill，针对这个具体失败：
- 标题：code-formatting
- Iron Law："任何代码修改后必须运行 Prettier，无例外"
- 禁止合理化表：
  | 借口 | 现实 |
  |------|------|
  | "这是小改动" | 所有改动都需要格式化 |
  | "其他文件没用" | 是我们要修复的问题 |

运行相同场景 → Agent 现在运行 Prettier ✓
```

**REFACTOR phase**：
```
发现新漏洞："我已经手动格式化了"
→ 加入规则："手动格式化 ≠ Prettier 通过"
→ 要求运行 prettier --check 看输出
再测试 → 合规 ✓
```

### 3.3 Subagents / References 深度解读

**anthropic-best-practices.md**

这是 Anthropic 官方文档的完整引用，与内部的 writing-skills SKILL.md 互补：
- 官方文档强调 token 效率、progressive disclosure、evaluations
- 内部 SKILL.md 强调 TDD 方法论（RED-GREEN-REFACTOR）、压力测试、说服原则

两者的关键一致点：**description 字段只描述触发条件**，不描述工作流。

官方文档的独特贡献：**"freedom spectrum"**（高/中/低自由度）概念——窄桥比喻（低自由度，有危险时提供精确护栏）vs 开阔地比喻（高自由度，多条路都能成功时给方向）。

**persuasion-principles.md**

这是整个系统中最有学术深度的文档，基于 Cialdini (2021) 和 Meincke et al. (2025) 的研究。

核心发现：LLMs 对人类说服原则有响应（因为训练数据中包含这些模式），且使用说服技术后合规率从 33% 提升到 72%。

对 skill 设计的实际影响：
- 纪律执行类 skill → 权威 + 承诺 + 社会证明（用 "YOU MUST", "No exceptions"）
- 协作类 skill → 一体性（用 "our codebase", "we're colleagues"）
- 参考类 skill → 只用清晰度，不用说服技术

**特别禁止**：不要用"喜好"原则（不要让 Claude 喜欢某件事所以才做）——这会创造奉承行为，破坏诚实文化。

**graphviz-conventions.dot**

这个文件自身就是用 Graphviz DOT 语言写的，而内容是关于如何使用 Graphviz 的规范——一个优雅的自引用设计。它定义了标准节点形状，帮助保持所有 skill 中流程图的一致性。

**testing-skills-with-subagents.md**

这是 RED-GREEN-REFACTOR 方法论的完整操作手册，包含：
- 如何写好的压力场景（具体选项、真实约束、强迫行动）
- 各种压力类型的组合
- 如何通过 meta-testing 诊断 skill 的哪个部分不清晰
- 完整的 skill 测试 checklist

**render-graphs.js**

这是一个实用工具，解决了一个实际痛点：SKILL.md 中的 Graphviz dot 图在大多数编辑器中无法直接预览。这个脚本：
1. 从 SKILL.md 提取所有 ` ```dot ` 块
2. 调用系统的 `dot` 命令渲染为 SVG
3. 支持 `--combine` 将所有图合并为一张全局视图

这对于 writing-skills 作者来说是"让用户看到你的流程图"的关键工具。

**examples/CLAUDE_MD_TESTING.md**

这是一个完整的测试活动记录，展示了如何系统性地测试文档变体的效果。4 种压力场景 × 5 种文档变体的矩阵测试，是 RED-GREEN-REFACTOR 方法在 CLAUDE.md 本身（而非 SKILL.md）上的应用。

### 3.4 流程图 / 示意图

**Skill 创建 TDD 循环**

```
准备创建新 skill
       │
       ▼
RED phase：先写 failing test
  创建 3+ 组压力场景
  （时间 + 沉没成本 + 权威 + 疲惫）
  在没有 skill 的情况下运行
  记录 agent 的确切失败和合理化
       │
       ▼
分析失败模式
  哪些借口重复出现？
  哪些压力场景最有效触发违规？
       │
       ▼
GREEN phase：写最小化 skill
  SKILL.md 结构：
    frontmatter (name + description)
    Overview + Core principle
    When to Use
    The Process / Core Pattern
    Red Flags（来自 baseline 失败）
    Rationalization table（来自 baseline 借口）
  用相同场景测试 WITH skill
  Agent 现在合规？
     ┌──no──┐   ┌──yes──┐
     ▼      │   ▼
  修订 skill  REFACTOR phase：
  重测         运行更多场景
               发现新合理化？
               ┌──yes──┐  ┌──no──┐
               ▼       │  ▼
           加明确对     │  Skill 通过测试
           抗条目       │  部署 ✓
           重测         │
               └────────┘
```

**Description 字段决策**

```
写 description 字段时：
       │
       ▼
描述的是触发条件（When to Use）吗？
   ┌──yes──┐  ┌──no——包含了工作流摘要──┐
   ▼       │  ▼
保留        │  危险！Claude 可能跟随
            │  description 而不读 skill 内容
            │  → 删除工作流部分
            │  → 只保留触发条件
            └──→ 重写

✅ "Use when executing implementation plans with independent tasks"
❌ "Use when executing plans - dispatches subagent per task with code review between tasks"
````

### 3.5 与其他 Skills 的协作关系

| 协作 Skill | 关系类型 | 说明 |
|------------|----------|------|
| test-driven-development | 前置/必须 | REQUIRED BACKGROUND：必须先理解 TDD 才能用此 skill；TDD 的 RED-GREEN-REFACTOR 直接映射到 skill 创建 |
| 所有其他 skills | 下游 | writing-skills 的产出会影响所有其他 skill 的行为；它是系统的可扩展性机制 |
| brainstorming | 元关系 | 发现一个需要 skill 的模式时，可以先 brainstorm skill 的设计 |
| subagent-driven-development | 工具使用 | 压力测试场景本身就是 subagent 的应用；testing-skills-with-subagents.md 和 SDD 的思路共通 |

> **注意**：writing-skills 是整个系统中唯一没有"上游"的 skill——你不需要先经历其他 skill 才能使用它。它可以在任何时候、任何场景下被触发，只要你发现了一个值得被编码为 skill 的可重用模式。
