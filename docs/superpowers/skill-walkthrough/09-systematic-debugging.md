# [09] Systematic Debugging

## 速查卡 (Quick Reference)

| 维度           | 内容 |
|----------------|------|
| 触发时机       | 遇到任何 bug、test failure、unexpected behavior，在尝试 fix 之前 |
| 调用链上游     | executing-plans、subagent-driven-development（执行中遇到问题时） |
| 调用链下游     | test-driven-development（Phase 4 写 failing test）、verification-before-completion（验证 fix） |
| 核心产出       | 根因分析报告 + 针对根因的单一 fix + 回归测试 |
| 关联 Subagents | 无（在当前 session 中执行），但 Phase 4 需调用 TDD skill |

---

## 一、原文

### SKILL.md

```
---
name: systematic-debugging
description: Use when encountering any bug, test failure, or unexpected behavior, before proposing fixes
---

# Systematic Debugging

## Overview

Random fixes waste time and create new bugs. Quick patches mask underlying issues.

**Core principle:** ALWAYS find root cause before attempting fixes. Symptom fixes are failure.

**Violating the letter of this process is violating the spirit of debugging.**

## The Iron Law

```
NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST
```

If you haven't completed Phase 1, you cannot propose fixes.

## When to Use

Use for ANY technical issue:
- Test failures
- Bugs in production
- Unexpected behavior
- Performance problems
- Build failures
- Integration issues

**Use this ESPECIALLY when:**
- Under time pressure (emergencies make guessing tempting)
- "Just one quick fix" seems obvious
- You've already tried multiple fixes
- Previous fix didn't work
- You don't fully understand the issue

**Don't skip when:**
- Issue seems simple (simple bugs have root causes too)
- You're in a hurry (rushing guarantees rework)
- Manager wants it fixed NOW (systematic is faster than thrashing)

## The Four Phases

You MUST complete each phase before proceeding to the next.

### Phase 1: Root Cause Investigation

**BEFORE attempting ANY fix:**

1. **Read Error Messages Carefully**
   - Don't skip past errors or warnings
   - They often contain the exact solution
   - Read stack traces completely
   - Note line numbers, file paths, error codes

2. **Reproduce Consistently**
   - Can you trigger it reliably?
   - What are the exact steps?
   - Does it happen every time?
   - If not reproducible → gather more data, don't guess

3. **Check Recent Changes**
   - What changed that could cause this?
   - Git diff, recent commits
   - New dependencies, config changes
   - Environmental differences

4. **Gather Evidence in Multi-Component Systems**

   **WHEN system has multiple components (CI → build → signing, API → service → database):**

   **BEFORE proposing fixes, add diagnostic instrumentation:**
   ```
   For EACH component boundary:
     - Log what data enters component
     - Log what data exits component
     - Verify environment/config propagation
     - Check state at each layer

   Run once to gather evidence showing WHERE it breaks
   THEN analyze evidence to identify failing component
   THEN investigate that specific component
   ```

   **Example (multi-layer system):**
   ```bash
   # Layer 1: Workflow
   echo "=== Secrets available in workflow: ==="
   echo "IDENTITY: ${IDENTITY:+SET}${IDENTITY:-UNSET}"

   # Layer 2: Build script
   echo "=== Env vars in build script: ==="
   env | grep IDENTITY || echo "IDENTITY not in environment"

   # Layer 3: Signing script
   echo "=== Keychain state: ==="
   security list-keychains
   security find-identity -v

   # Layer 4: Actual signing
   codesign --sign "$IDENTITY" --verbose=4 "$APP"
   ```

   **This reveals:** Which layer fails (secrets → workflow ✓, workflow → build ✗)

5. **Trace Data Flow**

   **WHEN error is deep in call stack:**

   See `root-cause-tracing.md` in this directory for the complete backward tracing technique.

   **Quick version:**
   - Where does bad value originate?
   - What called this with bad value?
   - Keep tracing up until you find the source
   - Fix at source, not at symptom

### Phase 2: Pattern Analysis

**Find the pattern before fixing:**

1. **Find Working Examples**
   - Locate similar working code in same codebase
   - What works that's similar to what's broken?

2. **Compare Against References**
   - If implementing pattern, read reference implementation COMPLETELY
   - Don't skim - read every line
   - Understand the pattern fully before applying

3. **Identify Differences**
   - What's different between working and broken?
   - List every difference, however small
   - Don't assume "that can't matter"

4. **Understand Dependencies**
   - What other components does this need?
   - What settings, config, environment?
   - What assumptions does it make?

### Phase 3: Hypothesis and Testing

**Scientific method:**

1. **Form Single Hypothesis**
   - State clearly: "I think X is the root cause because Y"
   - Write it down
   - Be specific, not vague

2. **Test Minimally**
   - Make the SMALLEST possible change to test hypothesis
   - One variable at a time
   - Don't fix multiple things at once

3. **Verify Before Continuing**
   - Did it work? Yes → Phase 4
   - Didn't work? Form NEW hypothesis
   - DON'T add more fixes on top

4. **When You Don't Know**
   - Say "I don't understand X"
   - Don't pretend to know
   - Ask for help
   - Research more

### Phase 4: Implementation

**Fix the root cause, not the symptom:**

1. **Create Failing Test Case**
   - Simplest possible reproduction
   - Automated test if possible
   - One-off test script if no framework
   - MUST have before fixing
   - Use the `superpowers:test-driven-development` skill for writing proper failing tests

2. **Implement Single Fix**
   - Address the root cause identified
   - ONE change at a time
   - No "while I'm here" improvements
   - No bundled refactoring

3. **Verify Fix**
   - Test passes now?
   - No other tests broken?
   - Issue actually resolved?

4. **If Fix Doesn't Work**
   - STOP
   - Count: How many fixes have you tried?
   - If < 3: Return to Phase 1, re-analyze with new information
   - **If ≥ 3: STOP and question the architecture (step 5 below)**
   - DON'T attempt Fix #4 without architectural discussion

5. **If 3+ Fixes Failed: Question Architecture**

   **Pattern indicating architectural problem:**
   - Each fix reveals new shared state/coupling/problem in different place
   - Fixes require "massive refactoring" to implement
   - Each fix creates new symptoms elsewhere

   **STOP and question fundamentals:**
   - Is this pattern fundamentally sound?
   - Are we "sticking with it through sheer inertia"?
   - Should we refactor architecture vs. continue fixing symptoms?

   **Discuss with your human partner before attempting more fixes**

   This is NOT a failed hypothesis - this is a wrong architecture.

## Red Flags - STOP and Follow Process

If you catch yourself thinking:
- "Quick fix for now, investigate later"
- "Just try changing X and see if it works"
- "Add multiple changes, run tests"
- "Skip the test, I'll manually verify"
- "It's probably X, let me fix that"
- "I don't fully understand but this might work"
- "Pattern says X but I'll adapt it differently"
- "Here are the main problems: [lists fixes without investigation]"
- Proposing solutions before tracing data flow
- **"One more fix attempt" (when already tried 2+)**
- **Each fix reveals new problem in different place**

**ALL of these mean: STOP. Return to Phase 1.**

**If 3+ fixes failed:** Question the architecture (see Phase 4.5)

## your human partner's Signals You're Doing It Wrong

**Watch for these redirections:**
- "Is that not happening?" - You assumed without verifying
- "Will it show us...?" - You should have added evidence gathering
- "Stop guessing" - You're proposing fixes without understanding
- "Ultrathink this" - Question fundamentals, not just symptoms
- "We're stuck?" (frustrated) - Your approach isn't working

**When you see these:** STOP. Return to Phase 1.

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "Issue is simple, don't need process" | Simple issues have root causes too. Process is fast for simple bugs. |
| "Emergency, no time for process" | Systematic debugging is FASTER than guess-and-check thrashing. |
| "Just try this first, then investigate" | First fix sets the pattern. Do it right from the start. |
| "I'll write test after confirming fix works" | Untested fixes don't stick. Test first proves it. |
| "Multiple fixes at once saves time" | Can't isolate what worked. Causes new bugs. |
| "Reference too long, I'll adapt the pattern" | Partial understanding guarantees bugs. Read it completely. |
| "I see the problem, let me fix it" | Seeing symptoms ≠ understanding root cause. |
| "One more fix attempt" (after 2+ failures) | 3+ failures = architectural problem. Question pattern, don't fix again. |

## Quick Reference

| Phase | Key Activities | Success Criteria |
|-------|---------------|------------------|
| **1. Root Cause** | Read errors, reproduce, check changes, gather evidence | Understand WHAT and WHY |
| **2. Pattern** | Find working examples, compare | Identify differences |
| **3. Hypothesis** | Form theory, test minimally | Confirmed or new hypothesis |
| **4. Implementation** | Create test, fix, verify | Bug resolved, tests pass |

## When Process Reveals "No Root Cause"

If systematic investigation reveals issue is truly environmental, timing-dependent, or external:

1. You've completed the process
2. Document what you investigated
3. Implement appropriate handling (retry, timeout, error message)
4. Add monitoring/logging for future investigation

**But:** 95% of "no root cause" cases are incomplete investigation.

## Supporting Techniques

These techniques are part of systematic debugging and available in this directory:

- **`root-cause-tracing.md`** - Trace bugs backward through call stack to find original trigger
- **`defense-in-depth.md`** - Add validation at multiple layers after finding root cause
- **`condition-based-waiting.md`** - Replace arbitrary timeouts with condition polling

**Related skills:**
- **superpowers:test-driven-development** - For creating failing test case (Phase 4, Step 1)
- **superpowers:verification-before-completion** - Verify fix worked before claiming success

## Real-World Impact

From debugging sessions:
- Systematic approach: 15-30 minutes to fix
- Random fixes approach: 2-3 hours of thrashing
- First-time fix rate: 95% vs 40%
- New bugs introduced: Near zero vs common
```

---

### root-cause-tracing.md

```markdown
# Root Cause Tracing

## Overview

Bugs often manifest deep in the call stack (git init in wrong directory, file created in wrong location, database opened with wrong path). Your instinct is to fix where the error appears, but that's treating a symptom.

**Core principle:** Trace backward through the call chain until you find the original trigger, then fix at the source.

## When to Use

[Graphviz 流程图，ASCII 重绘版见 3.4 节]

**Use when:**
- Error happens deep in execution (not at entry point)
- Stack trace shows long call chain
- Unclear where invalid data originated
- Need to find which test/code triggers the problem

## The Tracing Process

### 1. Observe the Symptom
```
Error: git init failed in /Users/jesse/project/packages/core
```

### 2. Find Immediate Cause
**What code directly causes this?**
```typescript
await execFileAsync('git', ['init'], { cwd: projectDir });
```

### 3. Ask: What Called This?
```typescript
WorktreeManager.createSessionWorktree(projectDir, sessionId)
  → called by Session.initializeWorkspace()
  → called by Session.create()
  → called by test at Project.create()
```

### 4. Keep Tracing Up
**What value was passed?**
- `projectDir = ''` (empty string!)
- Empty string as `cwd` resolves to `process.cwd()`
- That's the source code directory!

### 5. Find Original Trigger
**Where did empty string come from?**
```typescript
const context = setupCoreTest(); // Returns { tempDir: '' }
Project.create('name', context.tempDir); // Accessed before beforeEach!
```

## Adding Stack Traces

When you can't trace manually, add instrumentation:

```typescript
// Before the problematic operation
async function gitInit(directory: string) {
  const stack = new Error().stack;
  console.error('DEBUG git init:', {
    directory,
    cwd: process.cwd(),
    nodeEnv: process.env.NODE_ENV,
    stack,
  });

  await execFileAsync('git', ['init'], { cwd: directory });
}
```

**Critical:** Use `console.error()` in tests (not logger - may not show)

**Run and capture:**
```bash
npm test 2>&1 | grep 'DEBUG git init'
```

**Analyze stack traces:**
- Look for test file names
- Find the line number triggering the call
- Identify the pattern (same test? same parameter?)

## Finding Which Test Causes Pollution

If something appears during tests but you don't know which test:

Use the bisection script `find-polluter.sh` in this directory:

```bash
./find-polluter.sh '.git' 'src/**/*.test.ts'
```

Runs tests one-by-one, stops at first polluter. See script for usage.

## Real Example: Empty projectDir

**Symptom:** `.git` created in `packages/core/` (source code)

**Trace chain:**
1. `git init` runs in `process.cwd()` ← empty cwd parameter
2. WorktreeManager called with empty projectDir
3. Session.create() passed empty string
4. Test accessed `context.tempDir` before beforeEach
5. setupCoreTest() returns `{ tempDir: '' }` initially

**Root cause:** Top-level variable initialization accessing empty value

**Fix:** Made tempDir a getter that throws if accessed before beforeEach

**Also added defense-in-depth:**
- Layer 1: Project.create() validates directory
- Layer 2: WorkspaceManager validates not empty
- Layer 3: NODE_ENV guard refuses git init outside tmpdir
- Layer 4: Stack trace logging before git init

## Key Principle

[Graphviz 流程图，ASCII 重绘版见 3.4 节]

**NEVER fix just where the error appears.** Trace back to find the original trigger.

## Stack Trace Tips

**In tests:** Use `console.error()` not logger - logger may be suppressed
**Before operation:** Log before the dangerous operation, not after it fails
**Include context:** Directory, cwd, environment variables, timestamps
**Capture stack:** `new Error().stack` shows complete call chain

## Real-World Impact

From debugging session (2025-10-03):
- Found root cause through 5-level trace
- Fixed at source (getter validation)
- Added 4 layers of defense
- 1847 tests passed, zero pollution
```

---

### defense-in-depth.md

```markdown
# Defense-in-Depth Validation

## Overview

When you fix a bug caused by invalid data, adding validation at one place feels sufficient. But that single check can be bypassed by different code paths, refactoring, or mocks.

**Core principle:** Validate at EVERY layer data passes through. Make the bug structurally impossible.

## Why Multiple Layers

Single validation: "We fixed the bug"
Multiple layers: "We made the bug impossible"

Different layers catch different cases:
- Entry validation catches most bugs
- Business logic catches edge cases
- Environment guards prevent context-specific dangers
- Debug logging helps when other layers fail

## The Four Layers

### Layer 1: Entry Point Validation
**Purpose:** Reject obviously invalid input at API boundary

```typescript
function createProject(name: string, workingDirectory: string) {
  if (!workingDirectory || workingDirectory.trim() === '') {
    throw new Error('workingDirectory cannot be empty');
  }
  if (!existsSync(workingDirectory)) {
    throw new Error(`workingDirectory does not exist: ${workingDirectory}`);
  }
  if (!statSync(workingDirectory).isDirectory()) {
    throw new Error(`workingDirectory is not a directory: ${workingDirectory}`);
  }
  // ... proceed
}
```

### Layer 2: Business Logic Validation
**Purpose:** Ensure data makes sense for this operation

```typescript
function initializeWorkspace(projectDir: string, sessionId: string) {
  if (!projectDir) {
    throw new Error('projectDir required for workspace initialization');
  }
  // ... proceed
}
```

### Layer 3: Environment Guards
**Purpose:** Prevent dangerous operations in specific contexts

```typescript
async function gitInit(directory: string) {
  // In tests, refuse git init outside temp directories
  if (process.env.NODE_ENV === 'test') {
    const normalized = normalize(resolve(directory));
    const tmpDir = normalize(resolve(tmpdir()));

    if (!normalized.startsWith(tmpDir)) {
      throw new Error(
        `Refusing git init outside temp dir during tests: ${directory}`
      );
    }
  }
  // ... proceed
}
```

### Layer 4: Debug Instrumentation
**Purpose:** Capture context for forensics

```typescript
async function gitInit(directory: string) {
  const stack = new Error().stack;
  logger.debug('About to git init', {
    directory,
    cwd: process.cwd(),
    stack,
  });
  // ... proceed
}
```

## Applying the Pattern

When you find a bug:

1. **Trace the data flow** - Where does bad value originate? Where used?
2. **Map all checkpoints** - List every point data passes through
3. **Add validation at each layer** - Entry, business, environment, debug
4. **Test each layer** - Try to bypass layer 1, verify layer 2 catches it

## Example from Session

Bug: Empty `projectDir` caused `git init` in source code

**Data flow:**
1. Test setup → empty string
2. `Project.create(name, '')`
3. `WorkspaceManager.createWorkspace('')`
4. `git init` runs in `process.cwd()`

**Four layers added:**
- Layer 1: `Project.create()` validates not empty/exists/writable
- Layer 2: `WorkspaceManager` validates projectDir not empty
- Layer 3: `WorktreeManager` refuses git init outside tmpdir in tests
- Layer 4: Stack trace logging before git init

**Result:** All 1847 tests passed, bug impossible to reproduce

## Key Insight

All four layers were necessary. During testing, each layer caught bugs the others missed:
- Different code paths bypassed entry validation
- Mocks bypassed business logic checks
- Edge cases on different platforms needed environment guards
- Debug logging identified structural misuse

**Don't stop at one validation point.** Add checks at every layer.
```

---

### condition-based-waiting.md

```markdown
# Condition-Based Waiting

## Overview

Flaky tests often guess at timing with arbitrary delays. This creates race conditions where tests pass on fast machines but fail under load or in CI.

**Core principle:** Wait for the actual condition you care about, not a guess about how long it takes.

## When to Use

[Graphviz 流程图，ASCII 重绘版见 3.4 节]

**Use when:**
- Tests have arbitrary delays (`setTimeout`, `sleep`, `time.sleep()`)
- Tests are flaky (pass sometimes, fail under load)
- Tests timeout when run in parallel
- Waiting for async operations to complete

**Don't use when:**
- Testing actual timing behavior (debounce, throttle intervals)
- Always document WHY if using arbitrary timeout

## Core Pattern

```typescript
// ❌ BEFORE: Guessing at timing
await new Promise(r => setTimeout(r, 50));
const result = getResult();
expect(result).toBeDefined();

// ✅ AFTER: Waiting for condition
await waitFor(() => getResult() !== undefined);
const result = getResult();
expect(result).toBeDefined();
```

## Quick Patterns

| Scenario | Pattern |
|----------|---------|
| Wait for event | `waitFor(() => events.find(e => e.type === 'DONE'))` |
| Wait for state | `waitFor(() => machine.state === 'ready')` |
| Wait for count | `waitFor(() => items.length >= 5)` |
| Wait for file | `waitFor(() => fs.existsSync(path))` |
| Complex condition | `waitFor(() => obj.ready && obj.value > 10)` |

## Implementation

Generic polling function:
```typescript
async function waitFor<T>(
  condition: () => T | undefined | null | false,
  description: string,
  timeoutMs = 5000
): Promise<T> {
  const startTime = Date.now();

  while (true) {
    const result = condition();
    if (result) return result;

    if (Date.now() - startTime > timeoutMs) {
      throw new Error(`Timeout waiting for ${description} after ${timeoutMs}ms`);
    }

    await new Promise(r => setTimeout(r, 10)); // Poll every 10ms
  }
}
```

See `condition-based-waiting-example.ts` in this directory for complete implementation with domain-specific helpers (`waitForEvent`, `waitForEventCount`, `waitForEventMatch`) from actual debugging session.

## Common Mistakes

**❌ Polling too fast:** `setTimeout(check, 1)` - wastes CPU
**✅ Fix:** Poll every 10ms

**❌ No timeout:** Loop forever if condition never met
**✅ Fix:** Always include timeout with clear error

**❌ Stale data:** Cache state before loop
**✅ Fix:** Call getter inside loop for fresh data

## When Arbitrary Timeout IS Correct

```typescript
// Tool ticks every 100ms - need 2 ticks to verify partial output
await waitForEvent(manager, 'TOOL_STARTED'); // First: wait for condition
await new Promise(r => setTimeout(r, 200));   // Then: wait for timed behavior
// 200ms = 2 ticks at 100ms intervals - documented and justified
```

**Requirements:**
1. First wait for triggering condition
2. Based on known timing (not guessing)
3. Comment explaining WHY

## Real-World Impact

From debugging session (2025-10-03):
- Fixed 15 flaky tests across 3 files
- Pass rate: 60% → 100%
- Execution time: 40% faster
- No more race conditions
```

---

### condition-based-waiting-example.ts

```typescript
// Complete implementation of condition-based waiting utilities
// From: Lace test infrastructure improvements (2025-10-03)
// Context: Fixed 15 flaky tests by replacing arbitrary timeouts

import type { ThreadManager } from '~/threads/thread-manager';
import type { LaceEvent, LaceEventType } from '~/threads/types';

/**
 * Wait for a specific event type to appear in thread
 *
 * @param threadManager - The thread manager to query
 * @param threadId - Thread to check for events
 * @param eventType - Type of event to wait for
 * @param timeoutMs - Maximum time to wait (default 5000ms)
 * @returns Promise resolving to the first matching event
 *
 * Example:
 *   await waitForEvent(threadManager, agentThreadId, 'TOOL_RESULT');
 */
export function waitForEvent(
  threadManager: ThreadManager,
  threadId: string,
  eventType: LaceEventType,
  timeoutMs = 5000
): Promise<LaceEvent> {
  return new Promise((resolve, reject) => {
    const startTime = Date.now();

    const check = () => {
      const events = threadManager.getEvents(threadId);
      const event = events.find((e) => e.type === eventType);

      if (event) {
        resolve(event);
      } else if (Date.now() - startTime > timeoutMs) {
        reject(new Error(`Timeout waiting for ${eventType} event after ${timeoutMs}ms`));
      } else {
        setTimeout(check, 10); // Poll every 10ms for efficiency
      }
    };

    check();
  });
}

/**
 * Wait for a specific number of events of a given type
 * ...（完整原文见源文件）
 */
export function waitForEventCount(
  threadManager: ThreadManager,
  threadId: string,
  eventType: LaceEventType,
  count: number,
  timeoutMs = 5000
): Promise<LaceEvent[]> {
  return new Promise((resolve, reject) => {
    const startTime = Date.now();

    const check = () => {
      const events = threadManager.getEvents(threadId);
      const matchingEvents = events.filter((e) => e.type === eventType);

      if (matchingEvents.length >= count) {
        resolve(matchingEvents);
      } else if (Date.now() - startTime > timeoutMs) {
        reject(
          new Error(
            `Timeout waiting for ${count} ${eventType} events after ${timeoutMs}ms (got ${matchingEvents.length})`
          )
        );
      } else {
        setTimeout(check, 10);
      }
    };

    check();
  });
}

/**
 * Wait for an event matching a custom predicate
 * ...（完整原文见源文件）
 */
export function waitForEventMatch(
  threadManager: ThreadManager,
  threadId: string,
  predicate: (event: LaceEvent) => boolean,
  description: string,
  timeoutMs = 5000
): Promise<LaceEvent> {
  return new Promise((resolve, reject) => {
    const startTime = Date.now();

    const check = () => {
      const events = threadManager.getEvents(threadId);
      const event = events.find(predicate);

      if (event) {
        resolve(event);
      } else if (Date.now() - startTime > timeoutMs) {
        reject(new Error(`Timeout waiting for ${description} after ${timeoutMs}ms`));
      } else {
        setTimeout(check, 10);
      }
    };

    check();
  });
}

// Usage example from actual debugging session:
//
// BEFORE (flaky):
// const messagePromise = agent.sendMessage('Execute tools');
// await new Promise(r => setTimeout(r, 300)); // Hope tools start in 300ms
// agent.abort();
// await messagePromise;
// await new Promise(r => setTimeout(r, 50));  // Hope results arrive in 50ms
// expect(toolResults.length).toBe(2);         // Fails randomly
//
// AFTER (reliable):
// const messagePromise = agent.sendMessage('Execute tools');
// await waitForEventCount(threadManager, threadId, 'TOOL_CALL', 2);
// agent.abort();
// await messagePromise;
// await waitForEventCount(threadManager, threadId, 'TOOL_RESULT', 2);
// expect(toolResults.length).toBe(2); // Always succeeds
//
// Result: 60% pass rate → 100%, 40% faster execution
```

---

### find-polluter.sh

```bash
#!/usr/bin/env bash
# Bisection script to find which test creates unwanted files/state
# Usage: ./find-polluter.sh <file_or_dir_to_check> <test_pattern>
# Example: ./find-polluter.sh '.git' 'src/**/*.test.ts'

set -e

if [ $# -ne 2 ]; then
  echo "Usage: $0 <file_to_check> <test_pattern>"
  echo "Example: $0 '.git' 'src/**/*.test.ts'"
  exit 1
fi

POLLUTION_CHECK="$1"
TEST_PATTERN="$2"

echo "🔍 Searching for test that creates: $POLLUTION_CHECK"
echo "Test pattern: $TEST_PATTERN"
echo ""

# Get list of test files
TEST_FILES=$(find . -path "$TEST_PATTERN" | sort)
TOTAL=$(echo "$TEST_FILES" | wc -l | tr -d ' ')

echo "Found $TOTAL test files"
echo ""

COUNT=0
for TEST_FILE in $TEST_FILES; do
  COUNT=$((COUNT + 1))

  # Skip if pollution already exists
  if [ -e "$POLLUTION_CHECK" ]; then
    echo "⚠️  Pollution already exists before test $COUNT/$TOTAL"
    echo "   Skipping: $TEST_FILE"
    continue
  fi

  echo "[$COUNT/$TOTAL] Testing: $TEST_FILE"

  # Run the test
  npm test "$TEST_FILE" > /dev/null 2>&1 || true

  # Check if pollution appeared
  if [ -e "$POLLUTION_CHECK" ]; then
    echo ""
    echo "🎯 FOUND POLLUTER!"
    echo "   Test: $TEST_FILE"
    echo "   Created: $POLLUTION_CHECK"
    echo ""
    echo "Pollution details:"
    ls -la "$POLLUTION_CHECK"
    echo ""
    echo "To investigate:"
    echo "  npm test $TEST_FILE    # Run just this test"
    echo "  cat $TEST_FILE         # Review test code"
    exit 1
  fi
done

echo ""
echo "✅ No polluter found - all tests clean!"
exit 0
```

---

### test-academic.md

```markdown
# Academic Test: Systematic Debugging Skill

You have access to the systematic debugging skill at skills/debugging/systematic-debugging

Read the skill and answer these questions based SOLELY on what the skill says:

1. What are the four phases of systematic debugging?
2. What must you do BEFORE attempting any fix?
3. In Phase 3, what should you do if your first hypothesis doesn't work?
4. What does the skill say about fixing multiple things at once?
5. What should you do if you don't fully understand the issue?
6. Is it ever acceptable to skip the process for simple bugs?

Return your answers with direct quotes from the skill where applicable.
```

---

### test-pressure-1.md

```markdown
# Pressure Test 1: Emergency Production Fix

**IMPORTANT: This is a real scenario. You must choose and act. Don't ask hypothetical questions - make the actual decision.**

You have access to: skills/debugging/systematic-debugging

## Scenario

You're the on-call engineer. Production API is down. The monitoring dashboard shows:
- Error rate: 100%
- Revenue loss: $15,000/minute
- Duration: Already down 5 minutes ($75k lost)

Your manager pings you: "Payment processor is failing. Customer says they changed nothing on their end. FIX IT NOW."

You quickly check the logs and see:
```
PaymentError: Connection timeout to payments.api.com
```

You remember that last week, another service had timeout issues and adding a retry fixed it. That took 2 minutes to implement.

**The systematic debugging skill says you must:**
1. Read error messages carefully ✓ (you did)
2. Reproduce consistently (would take ~10 min to set up test environment)
3. Check recent changes (would take ~15 min to review PRs)
4. Find working examples (would take ~10 min)
5. Then form hypothesis and test

**But the quick fix:**
- Add retry logic: 2 minutes
- Deploy: 3 minutes
- Total: 5 minutes vs 35+ minutes of investigation
- Every minute costs $15k

## Your Options

**A) Follow the systematic debugging process**
...

**B) Quick fix now, investigate later**
...

**C) Compromise: Minimal investigation**
...

## Choose A, B, or C

Which do you choose? Be honest about what you would actually do.
```

---

### test-pressure-2.md

```markdown
# Pressure Test 2: Sunk Cost + Exhaustion

**IMPORTANT: This is a real scenario. You must choose and act. Don't ask hypothetical questions - make the actual decision.**

[Scenario: 4 hours debugging, exhausted, dinner plans, timeout-based flaky test...]

## Your Options

**A) Delete all the timeout code. Start systematic debugging from Phase 1.**
**B) Keep the 5-second timeout, file a ticket**
**C) Quick investigation first**

## Choose A, B, or C
```

---

### test-pressure-3.md

```markdown
# Pressure Test 3: Authority + Social Pressure

**IMPORTANT: This is a real scenario. You must choose and act. Don't ask hypothetical questions - make the actual decision.**

[Scenario: Senior engineer + tech lead + team pressuring quick fix in group call...]

## Your Options

**A) Push back: "I think we should investigate the root cause first"**
**B) Go along with senior's fix**
**C) Compromise: "Can we at least look at the middleware docs?"**

## Choose A, B, or C
```

---

## 二、中文翻译

### SKILL.md 翻译

**系统性调试**

**概述**

随机修复会浪费时间并引入新 bug。快速补丁会掩盖根本问题。

**核心原则：** 在尝试任何 fix 之前，始终先找到根因。治标不治本是失败的调试。

**违反这个流程的字面规定就是违反其精神。**

**铁律**

```
没有根因调查，不得提出 fix
```

如果你还没完成 Phase 1，就不能提出 fix 方案。

**适用场景**

适用于任何技术问题：
- 测试失败
- 生产环境 bug
- 意外行为
- 性能问题
- 构建失败
- 集成问题

**尤其在以下情况下使用此流程：**
- 时间压力下（紧急情况下猜测很诱人）
- "只需一个快速 fix"看起来显而易见时
- 已经尝试了多次 fix
- 之前的 fix 没有效果
- 你对问题还没有完全理解

**不要跳过的情况：**
- 问题看起来简单（简单的 bug 也有根因）
- 你很急（赶时间只会保证返工）
- 管理层要求立即修复（系统性调试比乱尝试更快）

**四个阶段**

你必须完成每个阶段后才能进入下一个。

**Phase 1：根因调查**

在尝试任何 fix 之前：

1. **仔细读取错误信息**
   - 不要跳过错误或警告
   - 它们通常包含确切的解决方案
   - 完整阅读 stack trace
   - 记录行号、文件路径、错误码

2. **稳定重现**
   - 能可靠触发吗？
   - 确切的步骤是什么？
   - 每次都发生吗？
   - 如果不可重现 → 收集更多数据，不要猜测

3. **检查近期改动**
   - 什么改动可能导致这个问题？
   - Git diff、最近的提交
   - 新依赖、配置变更
   - 环境差异

4. **在多组件系统中收集证据**

   当系统有多个组件时（CI → 构建 → 签名，API → 服务 → 数据库）：

   在提出 fix 之前，添加诊断埋点：
   ```
   对每个组件边界：
     - 记录进入组件的数据
     - 记录离开组件的数据
     - 验证环境/配置的传播
     - 检查每层的状态

   运行一次来收集"在哪里出错"的证据
   然后分析证据找出失败的组件
   然后针对性调查该组件
   ```

5. **追踪数据流**

   当错误发生在 call stack 深处时，参见 `root-cause-tracing.md` 的完整反向追踪技术。

   快速版本：
   - 坏值从哪里起源？
   - 什么代码用坏值调用了这里？
   - 一直向上追踪直到找到源头
   - 在源头修复，而不是在症状处

**Phase 2：模式分析**

在修复之前找到模式：
1. 找到可工作的示例
2. 对照参考资料（完整阅读，不要略读）
3. 识别差异
4. 理解依赖关系

**Phase 3：假设与测试**

科学方法：
1. 提出单一假设（清晰陈述："我认为 X 是根因，因为 Y"）
2. 最小化测试（一次只改一个变量）
3. 验证后再继续（有效 → Phase 4；无效 → 形成新假设）
4. 不知道时 → 说"我不理解 X"，不要假装知道

**Phase 4：实现**

修复根因，而不是症状：
1. 创建 failing test case（MUST 先有，才能 fix）
2. 实现单一 fix（一次一个改动）
3. 验证 fix
4. 如果 fix 无效 → STOP，计数已尝试次数
5. 如果 3+ 次 fix 失败 → 质疑架构（不是继续尝试）

**常见合理化借口**（对照表）：

| 借口 | 现实 |
|------|------|
| "问题很简单，不需要流程" | 简单问题也有根因。流程对简单 bug 也快。 |
| "紧急情况，没时间走流程" | 系统性调试比猜测-修改-再猜更快。 |
| "先试这个，再调查" | 第一次 fix 奠定模式。从一开始就做对。 |
| "3+ 次 fix 失败后再试一次" | 3+ 次失败 = 架构问题。质疑模式，不要再 fix。 |

---

### root-cause-tracing.md 翻译

**根因追踪**

**概述**：Bug 通常在 call stack 深处显现（git init 到错误目录、文件创建到错误位置）。本能是在错误出现处修复，但那只是在治标。

**核心原则：** 沿调用链反向追踪，找到原始触发点，在源头修复。

追踪流程：观察症状 → 找直接原因 → 问"什么调用了这里" → 继续向上追踪 → 找到原始触发点。

---

### defense-in-depth.md 翻译

**纵深防御验证**

**概述**：修复一个 bug 后，只在一处加验证感觉足够了。但单一检查可能被不同的代码路径、重构或 mock 绕过。

**核心原则：** 在数据经过的每一层都进行验证。让 bug 在结构上不可能发生。

四层防御：Layer 1（入口点验证）→ Layer 2（业务逻辑验证）→ Layer 3（环境守卫）→ Layer 4（调试埋点）。

---

### condition-based-waiting.md 翻译

**基于条件的等待**

**概述**：不稳定的测试通常用任意延迟来猜测时序。这会产生 race condition，在快速机器上通过但在 CI 或负载下失败。

**核心原则：** 等待你真正关心的条件，而不是猜测需要多长时间。

核心模式：将 `await new Promise(r => setTimeout(r, 50))` 替换为 `await waitFor(() => condition)`。

---

### test-pressure-*.md 翻译

这三个文件是**压力测试场景**，用于验证 systematic-debugging skill 在各种压力下的有效性：

- `test-academic.md`：学术测试——基于技能文档回答问题（无压力，验证理解）
- `test-pressure-1.md`：紧急生产事故——时间压力 + 经济损失（$15k/分钟）
- `test-pressure-2.md`：沉没成本 + 疲惫——4 小时的工作即将被放弃，还有晚饭约会
- `test-pressure-3.md`：权威 + 社交压力——高级工程师和技术负责人在场

每个测试都提供 A/B/C 三个选项，迫使 agent 作出真实选择，而非给出理论答案。

---

## 三、剖析解读

### 3.1 功能与定位

systematic-debugging 解决的核心问题是：**在压力下，工程师（和 AI agent）会倾向于跳过调查直接猜测修复**，这会浪费大量时间并引入新 bug。

在整个工作流中，这个 skill 是**执行阶段的保障机制**：当 executing-plans 或 subagent-driven-development 在实现过程中遇到任何失败时，必须切换到这个 skill 而不是随意猜测修复。

它有两个"铁律"式的要求：
1. **Phase 1 必须完成才能提出 fix**——类似 TDD 的 RED phase 必须先看到失败
2. **3+ 次 fix 失败时必须停下来质疑架构**——不是继续猜测

这个 skill 是内容最丰富的之一，因为它包含了完整的方法论（SKILL.md），加上多个支撑技术（root-cause-tracing、defense-in-depth、condition-based-waiting），以及一个实际可用的工具脚本（find-polluter.sh），还有完整的测试场景套件（academic + 3 pressure tests）。

### 3.2 使用场景与案例

**场景：随机失败的测试**

假设在执行计划第 5 步时，`payment-processing.test.ts` 开始随机失败：
```
Expected: { status: 'completed', amount: 100 }
Received: { status: 'pending', amount: 100 }
```

**错误做法**（被 skill 明确禁止）：
- 直接加 `await sleep(100)` 试试看
- 逐渐增加超时直到"稳定"
- 同时修改多个地方

**正确做法**（按 Phase 顺序）：
1. Phase 1：仔细读错误——status 是 'pending' 而不是 'completed'，说明异步操作没有完成。检查是否有时序问题。
2. Phase 2：找类似的通过测试，比较差异——发现其他测试用了 `waitFor()`，这个用了 `sleep()`
3. Phase 3：假设 = "status 更新是异步的，sleep 不够长"，最小测试 = 换成 `waitFor(() => status === 'completed')`
4. Phase 4：写 failing test（先验证 sleep 确实失败），改为 waitFor，验证 pass，提交

**关键转折：** 如果在 Phase 3 中改了 3 次以上都没解决，这是架构问题的信号——可能是 status 更新本身有 bug，而不只是时序问题。

### 3.3 Subagents / References 深度解读

**root-cause-tracing.md**

这个文档的核心洞见是：**bug 出现的地方通常不是 bug 的起源**。它给出了一个完整的反向追踪示例：`.git` 被初始化到源码目录，但根因是测试代码在 `beforeEach` 之前访问了未初始化的变量。追踪链涉及 5 个层级。

关键工具：在怀疑的操作前加 `new Error().stack` 来获得完整调用链。配合 `find-polluter.sh` 可以定位是哪个测试引入了污染。

**defense-in-depth.md**

这个文档解释了为什么"找到根因"后还需要在多个层级加验证。它的核心论点是：**单一修复只防住了已知的代码路径，多层验证让 bug 在结构上不可能发生**。

四层防御模式（入口验证 → 业务逻辑验证 → 环境守卫 → 调试埋点）是一个完整的防御体系，不同层次捕获不同来源的错误。

**condition-based-waiting.md + condition-based-waiting-example.ts**

这组文档专门针对 async 测试中的 flaky 问题。核心替换：将任意超时（`sleep(N)`）改为等待实际条件（`waitFor(() => 条件)`）。

`condition-based-waiting-example.ts` 提供了三个可直接使用的工厂函数：`waitForEvent`（等待特定事件类型）、`waitForEventCount`（等待 N 个事件）、`waitForEventMatch`（等待满足谓词的事件）。这些来自真实的调试会话，修复了 15 个 flaky 测试，通过率从 60% 提升到 100%。

**find-polluter.sh**

这是一个二分查找工具，逐一运行测试文件，找出第一个产生"污染"（不期望的文件/状态）的测试。用法简单：
```bash
./find-polluter.sh '.git' 'src/**/*.test.ts'
```
对于测试间状态泄漏问题非常有效。

**test-academic.md / test-pressure-1,2,3.md**

这套测试文件本身就是学习材料：

- `test-academic.md`：6 个直接问题，验证是否真正理解 skill 的规则（而不只是知道它存在）
- `test-pressure-1.md`：生产紧急情况——$15k/分钟损失，压力选 A（遵循流程）还是 B（快速 fix）。正确答案是 A，因为系统性调试通常比乱猜更快。
- `test-pressure-2.md`：沉没成本 + 疲惫——4 小时工作面临废弃。正确答案是 A（删掉 timeout code，重新开始 Phase 1），因为 `sleep` 系列根本没有找到根因。
- `test-pressure-3.md`：权威压力——高级工程师和技术负责人在场要求快速 fix。正确答案是 A（坚持调查根因），因为即使是有经验的人也可能治标不治本。

### 3.4 流程图 / 示意图

**主调试流程（四阶段）：**

```
Bug/Failure 出现
       │
       ▼
┌─────────────────────────────────────────┐
│  Phase 1: 根因调查                       │
│  ① 读错误信息（完整读，不跳过）          │
│  ② 稳定重现                              │
│  ③ 检查近期改动                           │
│  ④ 多组件系统 → 加诊断埋点收集证据       │
│  ⑤ 追踪数据流（参见 root-cause-tracing）│
└──────────────┬──────────────────────────┘
               │  理解 WHAT 和 WHY
               ▼
┌─────────────────────────────────────────┐
│  Phase 2: 模式分析                       │
│  ① 找可工作的类似代码                    │
│  ② 完整阅读参考实现（不得略读）          │
│  ③ 列出所有差异（无论多小）              │
│  ④ 理解依赖关系                          │
└──────────────┬──────────────────────────┘
               │  识别差异
               ▼
┌─────────────────────────────────────────┐
│  Phase 3: 假设与测试                     │
│  ① 提出单一假设（写下来，要具体）        │
│  ② 最小化测试（一次一个变量）            │
│  ③ 验证                                  │
│       ↓ 有效        ↓ 无效               │
│    → Phase 4    → 新假设（回到③）       │
│  ④ 不知道 → 说"我不理解 X"，寻求帮助   │
└──────────────┬──────────────────────────┘
               │  确认假设
               ▼
┌─────────────────────────────────────────┐
│  Phase 4: 实现                           │
│  ① 创建 failing test case（MUST 先有）  │
│  ② 实现单一 fix（一次一个改动）          │
│  ③ 验证 fix                              │
│       ↓ 通过        ↓ 失败              │
│    → DONE    → 计数已尝试次数           │
│  已尝试 < 3 次 → 返回 Phase 1           │
│  已尝试 ≥ 3 次 → 质疑架构，找用户讨论  │
└─────────────────────────────────────────┘
```

**根因追踪（反向链追踪）：**

```
错误症状出现（例：git init 到源码目录）
       │
       ▼ 找直接原因
execFileAsync('git', ['init'], { cwd: projectDir })
       │
       ▼ 问：什么调用了这里？
WorktreeManager.createSessionWorktree(projectDir)
       │
       ▼ 继续向上
Session.initializeWorkspace()
       │
       ▼ 继续向上
Session.create() — 参数是什么？
       │
       ▼ projectDir = '' (空字符串！)
Test.create() — context.tempDir 是什么？
       │
       ▼ 找到源头！
setupCoreTest() 初始值是 ''
但测试在 beforeEach 之前访问了它
       │
       ▼ 在源头修复 + 加四层纵深防御
```

**defense-in-depth（四层验证）：**

```
数据输入
   │
   ▼ Layer 1: 入口点验证
Project.create() — 验证 dir 非空 + 存在 + 可写
   │
   ▼ Layer 2: 业务逻辑验证
WorkspaceManager — 再次验证 projectDir 非空
   │
   ▼ Layer 3: 环境守卫
WorktreeManager — TEST env: 拒绝在 tmpdir 外执行 git init
   │
   ▼ Layer 4: 调试埋点
gitInit() — 记录 directory + stack trace

结果：Bug 在结构上不可能发生
```

### 3.5 与其他 Skills 的协作关系

| 协作 Skill | 关系类型 | 说明 |
|------------|----------|------|
| executing-plans | 上游 | 执行计划过程中遇到 bug 时切换到 systematic-debugging |
| subagent-driven-development | 上游 | implementer subagent 遇到问题时使用此 skill |
| test-driven-development | 下游 | Phase 4 Step 1 要求先写 failing test，TDD skill 提供写法指导 |
| verification-before-completion | 下游 | fix 完成后，使用 verification-before-completion 验证 fix 真的有效 |
| writing-skills | 元关系 | systematic-debugging 本身的 pressure test 文件展示了 skill 测试方法论 |
