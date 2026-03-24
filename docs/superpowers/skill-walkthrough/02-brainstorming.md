# 02 brainstorming

## 速查卡 (Quick Reference)

| 维度           | 内容 |
|----------------|------|
| 触发时机       | 任何创意性工作开始前：新增功能、构建组件、修改行为——无一例外 |
| 调用链上游     | using-superpowers（入口 skill，负责 dispatch 本 skill） |
| 调用链下游     | writing-plans（唯一合法的下一步，设计获批后立即调用） |
| 核心产出       | `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md` — 经过审查并获用户确认的设计文档 |
| 关联 Subagents | spec-document-reviewer (via spec-document-reviewer-prompt.md) |

---

## 一、原文

### SKILL.md

```
---
name: brainstorming
description: "You MUST use this before any creative work - creating features, building components, adding functionality, or modifying behavior. Explores user intent, requirements and design before implementation."
---

# Brainstorming Ideas Into Designs

Help turn ideas into fully formed designs and specs through natural collaborative dialogue.

Start by understanding the current project context, then ask questions one at a time to refine the idea. Once you understand what you're building, present the design and get user approval.

<HARD-GATE>
Do NOT invoke any implementation skill, write any code, scaffold any project, or take any implementation action until you have presented a design and the user has approved it. This applies to EVERY project regardless of perceived simplicity.
</HARD-GATE>

## Anti-Pattern: "This Is Too Simple To Need A Design"

Every project goes through this process. A todo list, a single-function utility, a config change — all of them. "Simple" projects are where unexamined assumptions cause the most wasted work. The design can be short (a few sentences for truly simple projects), but you MUST present it and get approval.

## Checklist

You MUST create a task for each of these items and complete them in order:

1. **Explore project context** — check files, docs, recent commits
2. **Offer visual companion** (if topic will involve visual questions) — this is its own message, not combined with a clarifying question. See the Visual Companion section below.
3. **Ask clarifying questions** — one at a time, understand purpose/constraints/success criteria
4. **Propose 2-3 approaches** — with trade-offs and your recommendation
5. **Present design** — in sections scaled to their complexity, get user approval after each section
6. **Write design doc** — save to `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md` and commit
7. **Spec review loop** — dispatch spec-document-reviewer subagent with precisely crafted review context (never your session history); fix issues and re-dispatch until approved (max 3 iterations, then surface to human)
8. **User reviews written spec** — ask user to review the spec file before proceeding
9. **Transition to implementation** — invoke writing-plans skill to create implementation plan

## Process Flow

[Graphviz dot diagram — ASCII 重绘版本见"剖析解读"章节]

**The terminal state is invoking writing-plans.** Do NOT invoke frontend-design, mcp-builder, or any other implementation skill. The ONLY skill you invoke after brainstorming is writing-plans.

## The Process

**Understanding the idea:**

- Check out the current project state first (files, docs, recent commits)
- Before asking detailed questions, assess scope: if the request describes multiple independent subsystems (e.g., "build a platform with chat, file storage, billing, and analytics"), flag this immediately. Don't spend questions refining details of a project that needs to be decomposed first.
- If the project is too large for a single spec, help the user decompose into sub-projects: what are the independent pieces, how do they relate, what order should they be built? Then brainstorm the first sub-project through the normal design flow. Each sub-project gets its own spec → plan → implementation cycle.
- For appropriately-scoped projects, ask questions one at a time to refine the idea
- Prefer multiple choice questions when possible, but open-ended is fine too
- Only one question per message - if a topic needs more exploration, break it into multiple questions
- Focus on understanding: purpose, constraints, success criteria

**Exploring approaches:**

- Propose 2-3 different approaches with trade-offs
- Present options conversationally with your recommendation and reasoning
- Lead with your recommended option and explain why

**Presenting the design:**

- Once you believe you understand what you're building, present the design
- Scale each section to its complexity: a few sentences if straightforward, up to 200-300 words if nuanced
- Ask after each section whether it looks right so far
- Cover: architecture, components, data flow, error handling, testing
- Be ready to go back and clarify if something doesn't make sense

**Design for isolation and clarity:**

- Break the system into smaller units that each have one clear purpose, communicate through well-defined interfaces, and can be understood and tested independently
- For each unit, you should be able to answer: what does it do, how do you use it, and what does it depend on?
- Can someone understand what a unit does without reading its internals? Can you change the internals without breaking consumers? If not, the boundaries need work.
- Smaller, well-bounded units are also easier for you to work with - you reason better about code you can hold in context at once, and your edits are more reliable when files are focused. When a file grows large, that's often a signal that it's doing too much.

**Working in existing codebases:**

- Explore the current structure before proposing changes. Follow existing patterns.
- Where existing code has problems that affect the work (e.g., a file that's grown too large, unclear boundaries, tangled responsibilities), include targeted improvements as part of the design - the way a good developer improves code they're working in.
- Don't propose unrelated refactoring. Stay focused on what serves the current goal.

## After the Design

**Documentation:**

- Write the validated design (spec) to `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md`
  - (User preferences for spec location override this default)
- Use elements-of-style:writing-clearly-and-concisely skill if available
- Commit the design document to git

**Spec Review Loop:**
After writing the spec document:

1. Dispatch spec-document-reviewer subagent (see spec-document-reviewer-prompt.md)
2. If Issues Found: fix, re-dispatch, repeat until Approved
3. If loop exceeds 3 iterations, surface to human for guidance

**User Review Gate:**
After the spec review loop passes, ask the user to review the written spec before proceeding:

> "Spec written and committed to `<path>`. Please review it and let me know if you want to make any changes before we start writing out the implementation plan."

Wait for the user's response. If they request changes, make them and re-run the spec review loop. Only proceed once the user approves.

**Implementation:**

- Invoke the writing-plans skill to create a detailed implementation plan
- Do NOT invoke any other skill. writing-plans is the next step.

## Key Principles

- **One question at a time** - Don't overwhelm with multiple questions
- **Multiple choice preferred** - Easier to answer than open-ended when possible
- **YAGNI ruthlessly** - Remove unnecessary features from all designs
- **Explore alternatives** - Always propose 2-3 approaches before settling
- **Incremental validation** - Present design, get approval before moving on
- **Be flexible** - Go back and clarify when something doesn't make sense

## Visual Companion

A browser-based companion for showing mockups, diagrams, and visual options during brainstorming. Available as a tool — not a mode. Accepting the companion means it's available for questions that benefit from visual treatment; it does NOT mean every question goes through the browser.

**Offering the companion:** When you anticipate that upcoming questions will involve visual content (mockups, layouts, diagrams), offer it once for consent:
> "Some of what we're working on might be easier to explain if I can show it to you in a web browser. I can put together mockups, diagrams, comparisons, and other visuals as we go. This feature is still new and can be token-intensive. Want to try it? (Requires opening a local URL)"

**This offer MUST be its own message.** Do not combine it with clarifying questions, context summaries, or any other content. The message should contain ONLY the offer above and nothing else. Wait for the user's response before continuing. If they decline, proceed with text-only brainstorming.

**Per-question decision:** Even after the user accepts, decide FOR EACH QUESTION whether to use the browser or the terminal. The test: **would the user understand this better by seeing it than reading it?**

- **Use the browser** for content that IS visual — mockups, wireframes, layout comparisons, architecture diagrams, side-by-side visual designs
- **Use the terminal** for content that is text — requirements questions, conceptual choices, tradeoff lists, A/B/C/D text options, scope decisions

A question about a UI topic is not automatically a visual question. "What does personality mean in this context?" is a conceptual question — use the terminal. "Which wizard layout works better?" is a visual question — use the browser.

If they agree to the companion, read the detailed guide before proceeding:
`skills/brainstorming/visual-companion.md`
```

---

### spec-document-reviewer-prompt.md

```
# Spec Document Reviewer Prompt Template

Use this template when dispatching a spec document reviewer subagent.

**Purpose:** Verify the spec is complete, consistent, and ready for implementation planning.

**Dispatch after:** Spec document is written to docs/superpowers/specs/

Task tool (general-purpose):
  description: "Review spec document"
  prompt: |
    You are a spec document reviewer. Verify this spec is complete and ready for planning.

    **Spec to review:** [SPEC_FILE_PATH]

    ## What to Check

    | Category | What to Look For |
    |----------|------------------|
    | Completeness | TODOs, placeholders, "TBD", incomplete sections |
    | Consistency | Internal contradictions, conflicting requirements |
    | Clarity | Requirements ambiguous enough to cause someone to build the wrong thing |
    | Scope | Focused enough for a single plan — not covering multiple independent subsystems |
    | YAGNI | Unrequested features, over-engineering |

    ## Calibration

    **Only flag issues that would cause real problems during implementation planning.**
    A missing section, a contradiction, or a requirement so ambiguous it could be
    interpreted two different ways — those are issues. Minor wording improvements,
    stylistic preferences, and "sections less detailed than others" are not.

    Approve unless there are serious gaps that would lead to a flawed plan.

    ## Output Format

    ## Spec Review

    **Status:** Approved | Issues Found

    **Issues (if any):**
    - [Section X]: [specific issue] - [why it matters for planning]

    **Recommendations (advisory, do not block approval):**
    - [suggestions for improvement]

**Reviewer returns:** Status, Issues (if any), Recommendations
```

---

### visual-companion.md

```
# Visual Companion Guide

Browser-based visual brainstorming companion for showing mockups, diagrams, and options.

## When to Use

Decide per-question, not per-session. The test: **would the user understand this better by seeing it than reading it?**

**Use the browser** when the content itself is visual:

- **UI mockups** — wireframes, layouts, navigation structures, component designs
- **Architecture diagrams** — system components, data flow, relationship maps
- **Side-by-side visual comparisons** — comparing two layouts, two color schemes, two design directions
- **Design polish** — when the question is about look and feel, spacing, visual hierarchy
- **Spatial relationships** — state machines, flowcharts, entity relationships rendered as diagrams

**Use the terminal** when the content is text or tabular:

- **Requirements and scope questions** — "what does X mean?", "which features are in scope?"
- **Conceptual A/B/C choices** — picking between approaches described in words
- **Tradeoff lists** — pros/cons, comparison tables
- **Technical decisions** — API design, data modeling, architectural approach selection
- **Clarifying questions** — anything where the answer is words, not a visual preference

A question *about* a UI topic is not automatically a visual question. "What kind of wizard do you want?" is conceptual — use the terminal. "Which of these wizard layouts feels right?" is visual — use the browser.

## How It Works

The server watches a directory for HTML files and serves the newest one to the browser. You write HTML content, the user sees it in their browser and can click to select options. Selections are recorded to a `.events` file that you read on your next turn.

**Content fragments vs full documents:** If your HTML file starts with `<!DOCTYPE` or `<html`, the server serves it as-is (just injects the helper script). Otherwise, the server automatically wraps your content in the frame template — adding the header, CSS theme, selection indicator, and all interactive infrastructure. **Write content fragments by default.** Only write full documents when you need complete control over the page.

## Starting a Session

```bash
# Start server with persistence (mockups saved to project)
scripts/start-server.sh --project-dir /path/to/project

# Returns: {"type":"server-started","port":52341,"url":"http://localhost:52341",
#           "screen_dir":"/path/to/project/.superpowers/brainstorm/12345-1706000000"}
```

Save `screen_dir` from the response. Tell user to open the URL.

**Finding connection info:** The server writes its startup JSON to `$SCREEN_DIR/.server-info`. If you launched the server in the background and didn't capture stdout, read that file to get the URL and port. When using `--project-dir`, check `<project>/.superpowers/brainstorm/` for the session directory.

**Note:** Pass the project root as `--project-dir` so mockups persist in `.superpowers/brainstorm/` and survive server restarts. Without it, files go to `/tmp` and get cleaned up. Remind the user to add `.superpowers/` to `.gitignore` if it's not already there.

**Launching the server by platform:**

**Claude Code (macOS / Linux):**
```bash
# Default mode works — the script backgrounds the server itself
scripts/start-server.sh --project-dir /path/to/project
```

**Claude Code (Windows):**
```bash
# Windows auto-detects and uses foreground mode, which blocks the tool call.
# Use run_in_background: true on the Bash tool call so the server survives
# across conversation turns.
scripts/start-server.sh --project-dir /path/to/project
```
When calling this via the Bash tool, set `run_in_background: true`. Then read `$SCREEN_DIR/.server-info` on the next turn to get the URL and port.

**Codex:**
```bash
# Codex reaps background processes. The script auto-detects CODEX_CI and
# switches to foreground mode. Run it normally — no extra flags needed.
scripts/start-server.sh --project-dir /path/to/project
```

**Gemini CLI:**
```bash
# Use --foreground and set is_background: true on your shell tool call
# so the process survives across turns
scripts/start-server.sh --project-dir /path/to/project --foreground
```

**Other environments:** The server must keep running in the background across conversation turns. If your environment reaps detached processes, use `--foreground` and launch the command with your platform's background execution mechanism.

If the URL is unreachable from your browser (common in remote/containerized setups), bind a non-loopback host:

```bash
scripts/start-server.sh \
  --project-dir /path/to/project \
  --host 0.0.0.0 \
  --url-host localhost
```

Use `--url-host` to control what hostname is printed in the returned URL JSON.

## The Loop

1. **Check server is alive**, then **write HTML** to a new file in `screen_dir`:
   - Before each write, check that `$SCREEN_DIR/.server-info` exists. If it doesn't (or `.server-stopped` exists), the server has shut down — restart it with `start-server.sh` before continuing. The server auto-exits after 30 minutes of inactivity.
   - Use semantic filenames: `platform.html`, `visual-style.html`, `layout.html`
   - **Never reuse filenames** — each screen gets a fresh file
   - Use Write tool — **never use cat/heredoc** (dumps noise into terminal)
   - Server automatically serves the newest file

2. **Tell user what to expect and end your turn:**
   - Remind them of the URL (every step, not just first)
   - Give a brief text summary of what's on screen (e.g., "Showing 3 layout options for the homepage")
   - Ask them to respond in the terminal: "Take a look and let me know what you think. Click to select an option if you'd like."

3. **On your next turn** — after the user responds in the terminal:
   - Read `$SCREEN_DIR/.events` if it exists — this contains the user's browser interactions (clicks, selections) as JSON lines
   - Merge with the user's terminal text to get the full picture
   - The terminal message is the primary feedback; `.events` provides structured interaction data

4. **Iterate or advance** — if feedback changes current screen, write a new file (e.g., `layout-v2.html`). Only move to the next question when the current step is validated.

5. **Unload when returning to terminal** — when the next step doesn't need the browser (e.g., a clarifying question, a tradeoff discussion), push a waiting screen to clear the stale content:

   ```html
   <!-- filename: waiting.html (or waiting-2.html, etc.) -->
   <div style="display:flex;align-items:center;justify-content:center;min-height:60vh">
     <p class="subtitle">Continuing in terminal...</p>
   </div>
   ```

   This prevents the user from staring at a resolved choice while the conversation has moved on. When the next visual question comes up, push a new content file as usual.

6. Repeat until done.

## Writing Content Fragments

Write just the content that goes inside the page. The server wraps it in the frame template automatically (header, theme CSS, selection indicator, and all interactive infrastructure).

**Minimal example:**

```html
<h2>Which layout works better?</h2>
<p class="subtitle">Consider readability and visual hierarchy</p>

<div class="options">
  <div class="option" data-choice="a" onclick="toggleSelect(this)">
    <div class="letter">A</div>
    <div class="content">
      <h3>Single Column</h3>
      <p>Clean, focused reading experience</p>
    </div>
  </div>
  <div class="option" data-choice="b" onclick="toggleSelect(this)">
    <div class="letter">B</div>
    <div class="content">
      <h3>Two Column</h3>
      <p>Sidebar navigation with main content</p>
    </div>
  </div>
</div>
```

That's it. No `<html>`, no CSS, no `<script>` tags needed. The server provides all of that.

## CSS Classes Available

The frame template provides these CSS classes for your content:

### Options (A/B/C choices)

```html
<div class="options">
  <div class="option" data-choice="a" onclick="toggleSelect(this)">
    <div class="letter">A</div>
    <div class="content">
      <h3>Title</h3>
      <p>Description</p>
    </div>
  </div>
</div>
```

**Multi-select:** Add `data-multiselect` to the container to let users select multiple options. Each click toggles the item. The indicator bar shows the count.

```html
<div class="options" data-multiselect>
  <!-- same option markup — users can select/deselect multiple -->
</div>
```

### Cards (visual designs)

```html
<div class="cards">
  <div class="card" data-choice="design1" onclick="toggleSelect(this)">
    <div class="card-image"><!-- mockup content --></div>
    <div class="card-body">
      <h3>Name</h3>
      <p>Description</p>
    </div>
  </div>
</div>
```

### Mockup container

```html
<div class="mockup">
  <div class="mockup-header">Preview: Dashboard Layout</div>
  <div class="mockup-body"><!-- your mockup HTML --></div>
</div>
```

### Split view (side-by-side)

```html
<div class="split">
  <div class="mockup"><!-- left --></div>
  <div class="mockup"><!-- right --></div>
</div>
```

### Pros/Cons

```html
<div class="pros-cons">
  <div class="pros"><h4>Pros</h4><ul><li>Benefit</li></ul></div>
  <div class="cons"><h4>Cons</h4><ul><li>Drawback</li></ul></div>
</div>
```

### Mock elements (wireframe building blocks)

```html
<div class="mock-nav">Logo | Home | About | Contact</div>
<div style="display: flex;">
  <div class="mock-sidebar">Navigation</div>
  <div class="mock-content">Main content area</div>
</div>
<button class="mock-button">Action Button</button>
<input class="mock-input" placeholder="Input field">
<div class="placeholder">Placeholder area</div>
```

### Typography and sections

- `h2` — page title
- `h3` — section heading
- `.subtitle` — secondary text below title
- `.section` — content block with bottom margin
- `.label` — small uppercase label text

## Browser Events Format

When the user clicks options in the browser, their interactions are recorded to `$SCREEN_DIR/.events` (one JSON object per line). The file is cleared automatically when you push a new screen.

```jsonl
{"type":"click","choice":"a","text":"Option A - Simple Layout","timestamp":1706000101}
{"type":"click","choice":"c","text":"Option C - Complex Grid","timestamp":1706000108}
{"type":"click","choice":"b","text":"Option B - Hybrid","timestamp":1706000115}
```

The full event stream shows the user's exploration path — they may click multiple options before settling. The last `choice` event is typically the final selection, but the pattern of clicks can reveal hesitation or preferences worth asking about.

If `.events` doesn't exist, the user didn't interact with the browser — use only their terminal text.

## Design Tips

- **Scale fidelity to the question** — wireframes for layout, polish for polish questions
- **Explain the question on each page** — "Which layout feels more professional?" not just "Pick one"
- **Iterate before advancing** — if feedback changes current screen, write a new version
- **2-4 options max** per screen
- **Use real content when it matters** — for a photography portfolio, use actual images (Unsplash). Placeholder content obscures design issues.
- **Keep mockups simple** — focus on layout and structure, not pixel-perfect design

## File Naming

- Use semantic names: `platform.html`, `visual-style.html`, `layout.html`
- Never reuse filenames — each screen must be a new file
- For iterations: append version suffix like `layout-v2.html`, `layout-v3.html`
- Server serves newest file by modification time

## Cleaning Up

```bash
scripts/stop-server.sh $SCREEN_DIR
```

If the session used `--project-dir`, mockup files persist in `.superpowers/brainstorm/` for later reference. Only `/tmp` sessions get deleted on stop.

## Reference

- Frame template (CSS reference): `scripts/frame-template.html`
- Helper script (client-side): `scripts/helper.js`
```

---

## 二、中文翻译

### SKILL.md 翻译

**元数据**
- name: brainstorming
- description: 在任何创意性工作（新增功能、构建组件、添加功能、修改行为）之前必须调用此 skill。在实现前探索用户意图、需求与设计。

---

**# 将想法转化为设计**

通过自然协作对话，帮助将想法转化为完整的设计和规格文档。

首先了解当前项目背景，然后逐一提问来细化想法。理解清楚要构建什么之后，呈现设计方案并获取用户批准。

**[HARD-GATE]**
在呈现设计并获得用户批准之前，绝对不能调用任何实现 skill、编写任何代码、搭建任何项目或采取任何实现行动。无论项目看起来多简单，此规则适用于所有项目。

---

**## 反模式："这太简单了，不需要设计"**

每个项目都必须经历此流程。待办列表、单函数工具、配置变更——无一例外。"简单"项目恰恰是未经审视的假设造成最多无用功的地方。设计可以简短（真正简单的项目只需几句话），但必须呈现并获得批准。

---

**## 检查清单**

必须为以下每项创建任务并按顺序完成：

1. **探索项目上下文** — 检查文件、文档、最近的提交
2. **提供可视化伴侣**（如果话题涉及视觉问题）— 单独一条消息，不与澄清问题合并。参见下方"Visual Companion"章节。
3. **提出澄清问题** — 每次一个，理解目的/约束/成功标准
4. **提出 2-3 种方案** — 附带权衡分析和推荐意见
5. **呈现设计** — 按复杂度分章节呈现，每节后获取用户确认
6. **编写设计文档** — 保存至 `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md` 并提交
7. **Spec 审查循环** — dispatch spec-document-reviewer subagent（使用精心构造的审查上下文，绝不传入会话历史）；修复问题后重新 dispatch，直至通过（最多 3 次迭代，超出则上报给人工）
8. **用户审查已写入的 spec** — 请用户在继续前审查 spec 文件
9. **切换至实现阶段** — 调用 writing-plans skill 创建实现计划

---

**## 流程图**

（原文此处为 Graphviz dot 格式流程图，ASCII 重绘版本见下）

**终态是调用 writing-plans。** 不能调用 frontend-design、mcp-builder 或其他任何实现 skill。brainstorming 结束后唯一调用的 skill 是 writing-plans。

---

**## 流程详解**

**理解想法：**
- 先探查当前项目状态（文件、文档、最近提交）
- 提问细节之前先评估范围：若请求描述了多个独立子系统（如"构建一个包含聊天、文件存储、计费和分析的平台"），立即标记出来，不要在需要先分解的项目上浪费问题
- 若项目过大无法放入单一 spec，帮助用户分解为子项目：各部分是什么、相互关系如何、构建顺序如何？然后对第一个子项目走完正常设计流程。每个子项目有自己的 spec → plan → 实现周期
- 对范围合适的项目，每次提一个问题来细化想法
- 尽量使用选择题，开放性问题也可以
- 每条消息只问一个问题——如果某个话题需要更多探讨，拆分成多个问题
- 聚焦于理解：目的、约束、成功标准

**探索方案：**
- 提出 2-3 种不同方案及其权衡
- 以对话方式呈现选项，附带推荐意见和理由
- 先给出推荐选项并解释原因

**呈现设计：**
- 一旦理解了要构建的内容，就呈现设计
- 按复杂度缩放每个章节：简单的几句话，复杂的最多 200-300 字
- 每节结束后询问是否看起来正确
- 覆盖：架构、组件、数据流、错误处理、测试
- 如有不清楚的地方，随时回头澄清

**为隔离性和清晰性而设计：**
- 将系统拆解为职责单一、通过明确接口通信、可独立理解和测试的小单元
- 对每个单元，需要能回答：它做什么、如何使用、依赖什么？
- 不阅读内部实现就能理解一个单元的功能吗？修改内部实现而不破坏消费者吗？如果做不到，边界需要重新划定
- 更小、边界清晰的单元也更易于 AI 处理——在一次 context 中推理更准确，文件聚焦时编辑更可靠。文件变大往往是职责过多的信号

**在现有代码库中工作：**
- 提议变更前先探查现有结构，遵循现有模式
- 若现有代码存在影响当前工作的问题（如文件过大、边界不清、职责缠绕），将有针对性的改进纳入设计——像优秀开发者改善所工作代码那样
- 不要提议不相关的重构，专注于服务当前目标

---

**## 设计完成后**

**文档：**
- 将验证后的设计（spec）写入 `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md`（用户偏好的 spec 位置可覆盖此默认值）
- 如果有 elements-of-style:writing-clearly-and-concisely skill，可使用
- 将设计文档提交到 git

**Spec 审查循环：**
写完 spec 文档后：
1. dispatch spec-document-reviewer subagent（参见 spec-document-reviewer-prompt.md）
2. 若发现问题：修复、重新 dispatch，循环直至通过
3. 若循环超过 3 次，上报给人工指导

**用户审查门控：**
spec 审查通过后，请用户在继续前审查已写入的 spec：

> "Spec 已写入并提交至 `<path>`。请审查后告知是否需要修改，然后我们开始编写实现计划。"

等待用户回复。若有修改请求，执行后重新跑 spec 审查循环。仅在用户批准后才继续。

**实现：**
- 调用 writing-plans skill 创建详细实现计划
- 不能调用其他任何 skill，writing-plans 是下一步

---

**## 核心原则**

- **每次一个问题** — 不要用多个问题压垮用户
- **优先使用选择题** — 比开放性问题更容易回答
- **YAGNI 毫不留情** — 从所有设计中去除不必要的功能
- **探索替代方案** — 在确定方案前始终提出 2-3 种选择
- **增量验证** — 呈现设计，获批后再继续
- **保持灵活** — 有不清楚的地方就回头澄清

---

**## Visual Companion（可视化伴侣）**

一个浏览器端伴侣工具，用于在 brainstorming 过程中展示 mockup、图表和视觉选项。作为工具使用，不是一种模式。接受伴侣意味着它可用于需要视觉处理的问题，并不意味着每个问题都要通过浏览器。

**提供伴侣：** 当预计后续问题涉及视觉内容（mockup、布局、图表）时，一次性征求同意：
> "我们正在处理的某些内容，如果能在浏览器中展示给你可能会更容易理解。我可以在过程中制作 mockup、图表、对比图等视觉内容。此功能仍是新功能，可能比较消耗 token。是否想试试？（需要打开本地 URL）"

**此提议必须单独成一条消息。** 不得与澄清问题、上下文摘要或任何其他内容合并。消息仅包含上述提议，别无其他。等待用户回复后再继续。若用户拒绝，以纯文本方式继续 brainstorming。

**逐问题决策：** 即使用户接受，也要**对每个问题**单独决定是用浏览器还是终端。判断标准：**用户通过看到它是否比读到它更容易理解？**

- **使用浏览器**：内容本身是视觉的——mockup、线框图、布局对比、架构图、并排视觉设计
- **使用终端**：内容是文本——需求问题、概念选择、权衡列表、A/B/C/D 文字选项、范围决策

关于 UI 话题的问题不自动是视觉问题。"这个上下文中'个性'是什么意思？"是概念问题——用终端。"哪种向导布局更好？"是视觉问题——用浏览器。

如果用户同意使用伴侣，在继续前阅读详细指南：`skills/brainstorming/visual-companion.md`

---

### spec-document-reviewer-prompt.md 翻译

**# Spec 文档审查者提示模板**

在 dispatch spec-document-reviewer subagent 时使用此模板。

**用途：** 验证 spec 是否完整、一致，是否已准备好进入实现计划阶段。

**在以下情况后 dispatch：** Spec 文档已写入 `docs/superpowers/specs/`

**Task tool（通用）：**
- description: "审查 spec 文档"
- prompt 内容：
  - 你是一位 spec 文档审查者。验证此 spec 是否完整且已准备好进行规划。
  - **待审查的 spec：** [SPEC_FILE_PATH]

**检查项：**

| 类别 | 检查内容 |
|------|----------|
| Completeness（完整性） | TODOs、占位符、"TBD"、未完成章节 |
| Consistency（一致性） | 内部矛盾、冲突的需求 |
| Clarity（清晰度） | 模糊到可能导致构建出错误内容的需求 |
| Scope（范围） | 是否足够聚焦于单一计划——不覆盖多个独立子系统 |
| YAGNI | 未请求的功能、过度工程化 |

**校准原则：**

只标记会在实现计划阶段造成真实问题的 issue。缺少某节、矛盾、或可被以两种不同方式解读的模糊需求——这些是问题。轻微的措辞改进、风格偏好、"某些章节不如其他章节详细"——这些不是问题。

除非存在会导致计划有缺陷的严重空缺，否则批准通过。

**输出格式：**

```
## Spec Review

**Status:** Approved | Issues Found

**Issues (if any):**
- [Section X]: [specific issue] - [why it matters for planning]

**Recommendations (advisory, do not block approval):**
- [suggestions for improvement]
```

**审查者返回：** Status、Issues（如有）、Recommendations

---

### visual-companion.md 翻译

**# 可视化伴侣指南**

浏览器端可视化 brainstorming 伴侣，用于展示 mockup、图表和选项。

**## 何时使用**

逐问题决策，不是逐会话决策。判断标准：**用户通过看到它是否比读到它更容易理解？**

**使用浏览器** 当内容本身是视觉的：
- **UI mockup** — 线框图、布局、导航结构、组件设计
- **架构图** — 系统组件、数据流、关系图
- **并排视觉对比** — 对比两种布局、两种配色方案、两种设计方向
- **设计细化** — 当问题关于外观感受、间距、视觉层次
- **空间关系** — 以图表形式渲染的状态机、流程图、实体关系

**使用终端** 当内容是文本或表格：
- **需求与范围问题** — "X 是什么意思？"、"哪些功能在范围内？"
- **概念性 A/B/C 选择** — 用文字描述方案后的选择
- **权衡列表** — 优缺点、对比表格
- **技术决策** — API 设计、数据建模、架构方案选择
- **澄清问题** — 任何答案是文字而非视觉偏好的问题

关于 UI 话题的问题不自动是视觉问题。"你想要什么类型的向导？"是概念性的——用终端。"这些向导布局哪个感觉更好？"是视觉的——用浏览器。

**## 工作原理**

服务器监视一个目录中的 HTML 文件，并将最新的一个提供给浏览器。AI 写入 HTML 内容，用户在浏览器中看到并可点击选择选项。选择结果记录到 `.events` 文件，下一轮由 AI 读取。

**内容片段 vs 完整文档：** 若 HTML 文件以 `<!DOCTYPE` 或 `<html` 开头，服务器原样提供（仅注入辅助脚本）。否则服务器自动用框架模板包裹内容——添加标题、CSS 主题、选择指示器和所有交互基础设施。**默认写内容片段。** 仅在需要完全控制页面时写完整文档。

**## 启动会话**

```bash
# 带持久化启动服务器（mockup 保存到项目）
scripts/start-server.sh --project-dir /path/to/project

# 返回: {"type":"server-started","port":52341,"url":"http://localhost:52341",
#         "screen_dir":"/path/to/project/.superpowers/brainstorm/12345-1706000000"}
```

保存响应中的 `screen_dir`，告知用户打开该 URL。

**查找连接信息：** 服务器将启动 JSON 写入 `$SCREEN_DIR/.server-info`。若服务器在后台启动且未捕获 stdout，读取该文件获取 URL 和端口。使用 `--project-dir` 时，在 `<project>/.superpowers/brainstorm/` 中查找会话目录。

**注意：** 传入 `--project-dir` 让 mockup 持久化在 `.superpowers/brainstorm/` 中并在服务器重启后保留。不传则文件放在 `/tmp` 并在停止时清理。提醒用户若 `.superpowers/` 未在 `.gitignore` 中则添加。

**各平台启动方式：**
- **Claude Code (macOS/Linux)：** 默认模式正常工作，脚本自行后台化服务器
- **Claude Code (Windows)：** Windows 自动检测并使用前台模式，该模式会阻塞工具调用。在 Bash 工具调用时设置 `run_in_background: true`，然后下一轮读取 `$SCREEN_DIR/.server-info` 获取 URL 和端口
- **Codex：** 脚本自动检测 `CODEX_CI` 并切换到前台模式，正常运行即可
- **Gemini CLI：** 使用 `--foreground` 并在 shell 工具调用时设置 `is_background: true`
- **其他环境：** 服务器必须在对话轮次间持续在后台运行。若环境会回收分离的进程，使用 `--foreground` 并通过平台的后台执行机制启动

若 URL 从浏览器无法访问（常见于远程/容器化环境），绑定非回环主机：

```bash
scripts/start-server.sh \
  --project-dir /path/to/project \
  --host 0.0.0.0 \
  --url-host localhost
```

**## 交互循环**

1. **检查服务器在线**，然后**写 HTML** 到 `screen_dir` 中的新文件：
   - 每次写入前检查 `$SCREEN_DIR/.server-info` 是否存在。若不存在（或 `.server-stopped` 存在），服务器已关闭——用 `start-server.sh` 重启后再继续。服务器在无活动 30 分钟后自动退出
   - 使用语义化文件名：`platform.html`、`visual-style.html`、`layout.html`
   - **永不复用文件名** — 每个屏幕用新文件
   - 使用 Write 工具——**不要用 cat/heredoc**（会向终端输出噪音）
   - 服务器自动提供最新文件

2. **告诉用户预期内容并结束本轮：**
   - 提醒 URL（每步都提，不只是第一次）
   - 简短文字说明屏幕上的内容（如"正在显示首页的 3 种布局选项"）
   - 请用户在终端回复："看一看，告诉我你的想法。如果想选择某个选项可以点击。"

3. **下一轮**——用户在终端回复后：
   - 若 `$SCREEN_DIR/.events` 存在则读取——包含用户的浏览器交互（点击、选择）的 JSON 行
   - 与用户的终端文字合并获取完整图景
   - 终端消息是主要反馈，`.events` 提供结构化交互数据

4. **迭代或推进** — 若反馈改变当前屏幕，写新文件（如 `layout-v2.html`）。仅在当前步骤验证完成后才推进到下一个问题

5. **返回终端时卸载** — 当下一步不需要浏览器（如澄清问题、权衡讨论），推送等待屏幕清除过时内容：

   ```html
   <!-- filename: waiting.html (or waiting-2.html, etc.) -->
   <div style="display:flex;align-items:center;justify-content:center;min-height:60vh">
     <p class="subtitle">Continuing in terminal...</p>
   </div>
   ```

   防止用户盯着已解决的选择而对话已继续。当下一个视觉问题出现时，正常推送新内容文件

6. 重复直至完成

**## 编写内容片段**

只写放在页面内的内容。服务器自动用框架模板包裹（标题、主题 CSS、选择指示器和所有交互基础设施）。

**最简示例：** 使用 `.options` 容器 + `.option` 卡片，无需 `<html>`、CSS 或 `<script>` 标签。

**## 可用 CSS 类**

- `.options` — A/B/C 选项容器；加 `data-multiselect` 支持多选
- `.option[data-choice="a"]` — 单个选项
- `.cards` / `.card` — 视觉设计卡片
- `.mockup` / `.mockup-header` / `.mockup-body` — mockup 容器
- `.split` — 并排视图（两个 `.mockup`）
- `.pros-cons` / `.pros` / `.cons` — 优缺点布局
- `.mock-nav`、`.mock-sidebar`、`.mock-content`、`.mock-button`、`.mock-input`、`.placeholder` — 线框构建块
- `h2`（页面标题）、`h3`（节标题）、`.subtitle`、`.section`、`.label`

**## 浏览器事件格式**

用户在浏览器中点击选项时，交互记录到 `$SCREEN_DIR/.events`（每行一个 JSON 对象）。推送新屏幕时文件自动清空。

完整事件流展示用户的探索路径——可能点击多个选项后才确定。最后一个 `choice` 事件通常是最终选择，但点击模式可揭示犹豫或值得询问的偏好。

若 `.events` 不存在，用户未与浏览器交互——只使用终端文字。

**## 设计提示**

- **保真度匹配问题** — 布局问题用线框，细化问题用精细设计
- **每页说明问题** — "哪种布局更专业？"而非只是"选一个"
- **迭代后再推进** — 若反馈改变当前屏幕，写新版本
- **每屏最多 2-4 个选项**
- **需要时用真实内容** — 摄影作品集用真实图片（Unsplash）；占位内容会掩盖设计问题
- **保持 mockup 简洁** — 聚焦布局和结构，而非像素级精确设计

**## 文件命名**

- 使用语义化名称：`platform.html`、`visual-style.html`、`layout.html`
- 永不复用文件名——每个屏幕必须是新文件
- 迭代版本：追加版本后缀如 `layout-v2.html`、`layout-v3.html`
- 服务器按修改时间提供最新文件

**## 清理**

```bash
scripts/stop-server.sh $SCREEN_DIR
```

若会话使用了 `--project-dir`，mockup 文件持久化在 `.superpowers/brainstorm/` 中以供后续参考。只有 `/tmp` 会话在停止时删除。

---

## 三、剖析解读

### 3.1 功能与定位

brainstorming 的唯一职责是：**在任何实现动作之前，通过对话将模糊想法转化为经过验证的设计文档**。

它处于整个 superpowers 工作流的第二环——using-superpowers 判断需要创意性工作时 dispatch 它，它完成后唯一的出口是 writing-plans。

**HARD-GATE 是核心约束**。这不是软性建议，是硬性门控：未呈现设计并获用户明确批准前，绝对不能写代码、搭脚手架、调用任何实现 skill。注意其措辞——"regardless of perceived simplicity"，把"感觉很简单"这个最常见的逃脱借口也堵死了。

**"Anti-Pattern: This Is Too Simple To Need A Design"** 章节的存在意义：它专门对抗"这太小了不值得设计"的惰性思维。这是经验教训的沉淀——越是"简单"的任务，越容易因未经审视的假设积累返工。设计可以只有几句话，但过程不能省略。

这个 skill 的哲学核心是：**先理解，再构建**。花在澄清上的时间远比返工便宜。

---

### 3.2 使用场景与案例

**典型场景：为电商平台添加优惠券系统**

brainstorming 的完整流程如下：

```
用户说: "给电商平台加个优惠券功能"
         |
         v
1. 探索项目上下文
   → 查看现有订单模型、价格计算逻辑、数据库结构
         |
         v
2. 提供可视化伴侣（如涉及 UI 设计）
   → "要不要在浏览器里看 mockup？"（单独消息，等待回复）
         |
         v
3. 逐一澄清问题
   → "优惠券是一次性使用还是可多次使用的？"
   → "折扣类型：百分比还是固定金额？"
   → "有效期规则是什么？"
   → "是否可以与其他促销叠加？"
         |
         v
4. 提出 2-3 种方案
   → 方案 A：简单折扣码（百分比/固定金额）
   → 方案 B：规则引擎（支持多种条件组合）
   → 方案 C：分层优惠体系（与会员等级联动）
   推荐：方案 A（YAGNI——从最简开始）
         |
         v
5. 呈现设计各章节并逐节确认
   → 数据模型、API 设计、折扣计算逻辑、错误处理
         |
         v
6. 写入 spec 文档并提交
         |
         v
7. Spec 审查循环（subagent 独立验证）
         |
         v
8. 请用户审查 spec 文件
         |
         v
9. 调用 writing-plans
```

**正确 vs 错误对比：**

| | 行为 |
|---|---|
| 错误 | 用户说"加个优惠券功能"，Claude 立刻开始写 `Coupon` 模型和路由 |
| 正确 | Claude 调用 brainstorming，先问"优惠券是一次性使用还是可复用的？" |

**规模判断的关键：** 若用户描述的是"包含聊天、文件存储、计费、分析的平台"，第一步不是问细节，而是立即标记范围过大，帮助用户分解为子项目——每个子项目走一轮独立的 brainstorming → spec → writing-plans 周期。

---

### 3.3 Subagents / References 深度解读

**spec-document-reviewer-prompt.md — Spec 审查 Subagent**

这是 spec 写完后 dispatch 给审查 subagent 的提示词模板。关键设计决策：

- **Fresh context**：模板明确要求"never your session history"——用全新上下文的 agent 来审查，而非让写 spec 的同一个 agent 自我复查。设计意图：写 spec 的 agent 因"知道太多背景"而容易忽视对读者来说不清楚的地方；外部 reviewer 的视角能发现这些盲点
- **校准原则**：仅标记会在实现阶段造成真实问题的 issue（缺失、矛盾、模糊到可双重解读）。轻微措辞问题、风格偏好不阻塞批准——避免审查流程变成无谓的完美主义循环
- **最多 3 次迭代**：若 3 轮后仍未通过，上报人工，防止 AI 陷入无法自己解决的死循环
- **五维检查**：Completeness（完整性）、Consistency（一致性）、Clarity（清晰度）、Scope（范围）、YAGNI——覆盖了 spec 最常见的失效模式

**visual-companion.md — 可视化伴侣详细指南**

描述浏览器端可视化工具的完整操作协议：

- **架构**：本地 HTTP 服务器监视目录，将最新 HTML 文件推送给浏览器；用户点击选项写入 `.events` 文件，AI 下轮读取
- **内容片段模式**：AI 只写 `<div>` 内容，服务器自动注入框架（CSS、JS、交互基础设施）——关注内容而非样板
- **跨平台处理**：针对 macOS/Linux、Windows、Codex、Gemini CLI 分别说明后台进程处理方式——各平台进程管理行为不同，需要不同策略
- **"等待屏幕"模式**：对话从视觉问题切换回文字问题时，推送一个清空屏幕，防止用户盯着已过时的内容
- **逐问题决策原则**：接受伴侣不等于所有问题都走浏览器。核心判断：这个问题的答案是视觉偏好还是文字选择？UI 相关 ≠ 视觉问题

---

### 3.4 流程图 / 示意图

原文为 Graphviz dot 格式，以下为 ASCII 重绘版本：

```
+---------------------------+
| Explore project context   |
+---------------------------+
             |
             v
   +-----------------------+
   | Visual questions      |
   | ahead?                |
   +-----------------------+
        |           |
       yes          no
        |           |
        v           |
+------------------+|
| Offer Visual     ||
| Companion        ||
| (own msg only)   ||
+------------------+|
        |           |
        +-----------+
             |
             v
+---------------------------+
| Ask clarifying questions  |
| (one at a time)           |
+---------------------------+
             |
             v
+---------------------------+
| Propose 2-3 approaches    |
+---------------------------+
             |
             v
+---------------------------+
| Present design sections   |
+---------------------------+
             |
             v
   +---------------------+
   | User approves       |
   | design?             |
   +---------------------+
      |           |
    no,           yes
   revise          |
      |            v
      |   +------------------+
      +-->| Write design doc |
          +------------------+
                   |
                   v
          +------------------+
          | Spec review loop |
          | (subagent)       |
          +------------------+
                   |
                   v
       +------------------------+
       | Spec review passed?    |
       +------------------------+
           |           |
      issues       approved
      found,           |
      fix &            v
      re-dispatch  +------------------------+
           |       | User reviews spec?     |
           +-----> +------------------------+
                       |           |
                  changes       approved
                  requested         |
                       |            v
                       |   +=========================+
                       |   || Invoke writing-plans  ||
                       |   || (terminal state)      ||
                       |   +=========================+
                       |
                       v
              (back to Write design doc)
```

**关键约束可视化：**

```
brainstorming
     |
     | 唯一合法出口
     v
writing-plans

[BLOCKED] frontend-design
[BLOCKED] mcp-builder
[BLOCKED] 任何其他实现 skill
```

---

### 3.5 与其他 Skills 的协作关系

| 协作 Skill | 关系类型 | 说明 |
|------------|----------|------|
| using-superpowers | 上游调用者 | 入口 skill，判断需要创意性工作时 dispatch brainstorming |
| writing-plans | 唯一下游 | brainstorming 完成后的唯一合法下一步；SKILL.md 明确禁止调用其他实现 skill |
| visual-companion | 内嵌工具 | brainstorming 过程中按需使用的浏览器可视化辅助；不是独立 skill，是 brainstorming 的子工具 |
| spec-document-reviewer | Subagent | brainstorming 内部 dispatch 的审查 subagent；以 fresh context 独立验证 spec 质量 |
| elements-of-style:writing-clearly-and-concisely | 可选辅助 | 若存在，可用于提升 spec 文档的写作质量（非强制） |
