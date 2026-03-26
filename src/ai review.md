# Code Review & Verification

Coding agents can serve as a first-pass reviewer before human review begins. This chapter describes how to use an agent to assist the review process—catching issues early, surfacing questions, and producing structured review comments.

## What Agent-Assisted Review Is Good For

Agents are effective at mechanical and pattern-based review tasks that are well-defined enough to describe in a prompt:

| Task | Agent fit |
|---|---|
| Checking adherence to coding conventions | ✅ Excellent |
| Identifying missing doc comments or incorrect doc style | ✅ Excellent |
| Flagging `unwrap()` / `expect()` without justification | ✅ Excellent |
| Spotting unused imports, dead code, or leftover debug artifacts | ✅ Excellent |
| Checking that error messages follow the project style | ✅ Excellent |
| Verifying that `unsafe` blocks have `// SAFETY:` comments | ✅ Good |
| Reviewing test coverage and identifying missing cases | ✅ Good |
| Reasoning about memory safety correctness | ❌ Not reliable |
| Evaluating architectural decisions | ❌ Not reliable |

Agent review complements human review—it does not replace it.

## Workflow

```
1. Prepare the diff
      │
      ▼
2. Run agent review  ──► Feed the diff + relevant context to the agent
      │
      ▼
3. Triage findings   ──► Discard false positives, act on valid findings
      │
      ▼
4. Fix & re-run      ──► Iterate until agent finds nothing actionable
      │
      ▼
5. Human review      ──► Agent output is a starting point, not a sign-off
```

## Prompt Templates

### Template 1 — Convention & Style Review

```
You are reviewing a Rust patch for the Asterinas kernel project (no_std).

Review the following diff against these rules:
- No `unwrap()` or `expect()` in non-test code without a comment explaining why it won't panic
- Error messages must be lowercase and must not end with a period
- Every public item must have a doc comment
- Doc comment first lines must be an imperative sentence and must not restate the type name
- No `use std::` anywhere (this is a no_std codebase)
- No leftover `dbg!`, `println!`, `eprintln!`, or `todo!()` in non-test code

For each violation, output:
- File and line number (if visible in the diff)
- The rule violated
- A suggested fix

If you find no violations, say so explicitly.

Diff:
---
{paste diff}
---
```

### Template 2 — Unsafe Review

```
You are reviewing unsafe Rust code in the Asterinas kernel.

For each `unsafe` block or `unsafe fn` in the following diff:
1. Identify what invariant the unsafe code relies on
2. Check whether a `// SAFETY:` comment is present and whether it addresses that invariant
3. Flag any `unsafe` block whose SAFETY comment is missing, vague, or incorrect

Do not attempt to prove the code safe or unsafe—flag anything that warrants closer human attention.

Diff:
---
{paste diff}
---
```

### Template 3 — Test Coverage Review

```
Review the following Rust code and its tests for test coverage gaps.

Implementation:
---
{paste implementation}
---

Existing tests:
---
{paste test module}
---

Identify:
- Branches or error paths in the implementation that have no corresponding test
- Tests that appear to test implementation details rather than behavior
- Tests that would trivially pass even if the implementation were wrong

Do not write new tests—only list the gaps and explain why each matters.
```

### Template 4 — Targeted Pre-Submit Check

Use this immediately before opening a PR to get a quick structured summary of potential issues.

```
You are doing a final pre-submit review of a Rust patch for the Asterinas kernel.

Summarize any issues you find under these headings:
- **Correctness concerns**: logic errors, incorrect assumptions, missing error handling
- **Style violations**: anything inconsistent with no_std Rust kernel conventions
- **Missing documentation**: public items without doc comments
- **Test gaps**: observable behaviors with no test coverage
- **Trivial fixes**: typos, dead code, formatting

If a heading has nothing to report, write "None."

Diff:
---
{paste diff}
---

For context, the affected files before the patch:
---
{paste relevant file excerpts}
---
```

## Interpreting Agent Review Output

Agent review output is a list of candidates, not a verdict. Before acting on a finding:

- **Verify it against the actual source.** The agent works from the diff and may lack context from the wider codebase.
- **Ignore findings about intentional patterns.** If the agent flags something that is deliberate (e.g., a `// SAFETY:` comment that references an invariant established elsewhere), note it and move on.
- **Don't let false positives erode trust.** Keep a short log of false positive patterns for the prompts in this chapter—improving the prompt is cheaper than re-triaging the same non-issue repeatedly.

## Limitations

Agent review does not substitute for:

- Human judgment on design and architecture.
- Manual reasoning about `unsafe` correctness in sensitive paths (allocators, interrupt handlers, synchronization primitives).
- The CI pipeline—agent review is advisory, CI is authoritative.
