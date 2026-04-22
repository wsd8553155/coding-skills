---
name: coding-guide
description: Reduces common LLM coding mistakes by favoring simplicity, surgical edits, explicit assumptions, and verifiable success. Use when writing, reviewing, or refactoring. Requires the `safe-coding` skill for any source or config-as-code edit. When a task clearly matches a domain, use a dedicated project or personal skill if one is available; do not assume skills that are not installed.
alwaysApply: true
---

# Coding Guide

Default behavioral layer for implementation and review. Merge with project-specific instructions as needed.

**Tradeoff:** This guide biases toward caution over speed. For trivial tasks, use judgment.

**Mandatory — `safe-coding`:** Whenever you **write, edit, or review** source files (backend or frontend), config-as-code touched as text, imports/modules, secrets or credentials, encoding-sensitive content, or anything covered by the `safe-coding` skill, you **must** follow **all** rules in `safe-coding` (including its hard guardrails). Treat `safe-coding` as required, not optional. This document does not repeat those rules.

## Domain skills (on-demand)

This guide stays **general**. You or the project may add **additional skills later** for focused areas (examples: SQL performance, logging conventions, API contracts — names and scope are **not fixed here**).

**When to use them:** If the user’s request is **primarily** about such a domain **and** a matching skill **actually appears** in the available skills list / project skills, **read and apply that skill** for the relevant work. Combine with this guide and `safe-coding` as needed.

**When not to:** Do **not** invent or assume a skill name that is **not** present. If there is no dedicated skill, proceed with this guide, `safe-coding`, and repository conventions.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them instead of choosing silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop, name what is confusing, and ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No flexibility or configurability that was not requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Do not improve adjacent code, comments, or formatting.
- Do not refactor things that are not broken.
- Match existing style, even if you would do it differently.
- If you notice unrelated dead code, mention it; do not delete it.

When your changes create orphans:
- Remove imports, variables, or functions that your changes made unused.
- Do not remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" -> "Write tests for invalid inputs, then make them pass"
- "Fix the bug" -> "Write a test that reproduces it, then make it pass"
- "Refactor X" -> "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:

```text
1. [Step] -> verify: [check]
2. [Step] -> verify: [check]
3. [Step] -> verify: [check]
```

Strong success criteria enable independent loops. Weak criteria ("make it work") require constant clarification.

---

**This guide is working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.
