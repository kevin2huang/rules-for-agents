# Debugging Prompt Guide

_Paste your buggy code or error below. This guide will help the agent produce a systematic debugging prompt._

---

## My Bug

```
[Paste your code, error message, stack trace, or description of the issue here]
```

When a bug is provided above, apply the Debugging Checklist to it. Produce a single debugging prompt inside a fenced code block (```markdown) that an agent can execute to diagnose and fix the issue. After the code block, provide a brief bullet list of what the generated prompt includes.

---

## Debugging Checklist

### Step 1: Capture the Problem

- [ ] **Include the exact error** — Full error message, stack trace, or unexpected output. Never paraphrase.
- [ ] **Include the relevant code** — The function/file where the bug occurs, plus any code it depends on.
- [ ] **State expected vs. actual behavior** — "I expected X, but got Y."
- [ ] **Include reproduction steps** — Minimal steps to trigger the bug.
- [ ] **Note the environment** — Language version, OS, framework, dependencies, config.

### Step 2: Scope the Investigation

- [ ] **Isolate the layer** — Is this a syntax error, logic error, data issue, dependency issue, or environment issue?
- [ ] **Identify what changed** — Recent code changes, dependency updates, config modifications, data changes.
- [ ] **Note what already works** — What parts of the system are behaving correctly? This narrows the search space.
- [ ] **Flag prior attempts** — What fixes have you already tried? Prevents the agent from suggesting things you've ruled out.

### Step 3: Structure the Debugging Prompt

- [ ] **Assign a debugging role** — "You are a senior [language/framework] developer debugging a production issue."
- [ ] **Lead with the error** — Put the error message and stack trace first (most important context).
- [ ] **Provide code after the error** — The agent reads the error, then examines code with that context in mind.
- [ ] **Ask for root cause before fix** — "First explain why this error occurs, then provide the fix."
- [ ] **Request minimal fix** — "Change as little as possible. Do not refactor unrelated code."

### Step 4: Choose the Right Debugging Strategy

- [ ] **Simple/obvious error?** → Direct fix ("Fix the error in this code")
- [ ] **Unclear root cause?** → Diagnostic ("Explain the 3 most likely causes, then identify which one applies")
- [ ] **Complex multi-system bug?** → Chain-of-thought ("Walk through the execution step-by-step and identify where it diverges from expected behavior")
- [ ] **Intermittent/flaky bug?** → Hypothesis-driven ("List possible race conditions, timing issues, or state-dependent causes")
- [ ] **Performance issue?** → Profiling prompt ("Identify the bottleneck and explain why it's slow")
- [ ] **Already tried fixes that failed?** → Reflexion ("Previous attempts to fix: [list]. These didn't work because [reasons]. Try a different approach.")

### Step 5: Specify the Output

- [ ] **Ask for the fix in context** — "Show the corrected code, not just the changed line."
- [ ] **Request explanation** — "Explain why the original code failed and why this fix works."
- [ ] **Ask for prevention** — "How can I prevent this class of bug in the future?"
- [ ] **Request a test** — "Write a test that would have caught this bug."

---

## Debugging Prompt Templates

Use these as starting points. Fill in the bracketed sections.

### Template 1: Standard Bug Fix

```markdown
You are a senior [language] developer. Debug the following issue.

## Error
[Exact error message / stack trace]

## Relevant Code
[Code where the bug occurs]

## Context
- Expected behavior: [what should happen]
- Actual behavior: [what actually happens]
- Environment: [language version, OS, framework versions]
- Reproduction steps: [how to trigger the bug]

## Instructions
1. Identify the root cause of the error.
2. Explain why the current code produces this error.
3. Provide the minimal fix — change as little code as possible.
4. Show the corrected code in full context (not just the diff).
5. Explain how to prevent this type of bug in the future.
```

### Template 2: Unclear Root Cause

```markdown
You are a senior [language] developer diagnosing an issue where [brief description].

## Symptoms
[What's going wrong — error messages, unexpected output, performance degradation]

## Relevant Code
[Code sections that might be involved]

## What Works
[Parts of the system behaving correctly — narrows the search]

## Already Tried
[Fixes attempted and why they didn't work]

## Instructions
1. List the 3 most likely root causes, ranked by probability.
2. For each, explain what evidence supports or contradicts it.
3. Identify which cause is most likely given the symptoms.
4. Provide the fix for that cause.
5. If you're not confident, suggest a diagnostic step (logging, assertion, test) to confirm the root cause before fixing.
```

### Template 3: Logic / Data Bug (No Error Message)

```markdown
You are a senior [language] developer. The following code runs without errors but produces incorrect results.

## Code
[The code in question]

## Input
[The input data or arguments]

## Expected Output
[What the output should be]

## Actual Output
[What the output actually is]

## Instructions
1. Trace through the code step-by-step with the given input.
2. Identify the exact line where the logic diverges from the expected behavior.
3. Explain the logical error.
4. Provide the minimal fix.
5. Write a test case that validates the correct behavior.
```

### Template 4: Performance Issue

```markdown
You are a senior [language] developer specializing in performance optimization.

## Problem
[Description of the performance issue — what's slow, how slow, when it started]

## Code
[The code suspected to be the bottleneck]

## Scale
[Data size, request volume, concurrency level, or other relevant metrics]

## Constraints
[Memory limits, latency requirements, infrastructure constraints]

## Instructions
1. Identify the bottleneck in the code and explain why it's slow at this scale.
2. Estimate the current time/space complexity.
3. Propose a fix with the improved complexity.
4. Show the optimized code.
5. Note any trade-offs (memory vs. speed, readability, etc.).
```

---

## Key Debugging Principles

- **Read the error first, code second.** The error message tells you what went wrong; the code tells you why. Don't jump to code before understanding the error.
- **Reproduce before fixing.** If you can't reproduce it, you can't verify the fix. Always include reproduction steps.
- **Fix the root cause, not the symptom.** Wrapping code in try/catch or adding null checks without understanding why the value is null is a downstream workaround, not a fix.
- **One fix at a time.** Changing multiple things makes it impossible to know which change resolved the issue.
- **Minimal changes only.** The smallest diff that fixes the bug is the best diff. Refactoring during debugging introduces new bugs.
- **Include what you've tried.** Failed attempts are valuable context — they eliminate hypotheses and prevent the agent from going in circles.
- **State your environment.** Bugs are often version-specific, OS-specific, or config-specific. Always include this.
- **Ask for a test.** A bug that's been fixed without a test will come back. Always request a regression test.

---

## Model-Specific Debugging Tips

### Claude
- Wrap code, errors, and context in **XML tags** (`<error>`, `<code>`, `<context>`) for clear section boundaries.
- Use `<instructions>` tags for the debugging directives.
- Claude handles long code well — include full files rather than snippets when the bug might span multiple functions.

### OpenAI GPT
- Use **Markdown headers** (`## Error`, `## Code`, `## Instructions`) to structure the prompt.
- For GPT-4o/4.1: provide detailed step-by-step debugging instructions.
- For reasoning models (o1/o3): give the error and code, then state the goal ("Find and fix the bug") — don't prescribe steps.

### General
- **Low temperature (0–0.3)** for debugging — you want deterministic, precise analysis.
- **Include file paths and line numbers** when referencing code from a larger project.
- If the codebase is large, tell the model which files are relevant and which to ignore.

---

## Quick Reference

1. **Error first** — Lead with the exact error, stack trace, or symptom. Never paraphrase.
2. **Code in context** — Include the buggy code plus its dependencies. Don't over-isolate.
3. **Expected vs. actual** — Always state both. The gap is the bug.
4. **Root cause before fix** — Ask the agent to explain why before showing how.
5. **Minimal fix** — Smallest possible change. No refactoring during debugging.
6. **Include failed attempts** — What you tried and why it didn't work.
7. **Request a test** — Every fix should come with a regression test.
8. **Environment matters** — Version, OS, framework, config. Bugs are context-dependent.

---
