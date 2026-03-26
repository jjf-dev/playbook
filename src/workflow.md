# Agent Workflow & Task Decomposition

Coding agents perform best on tasks with clear boundaries, deterministic success criteria, and limited cross-cutting concerns. This chapter describes how to decompose work before handing it to an agent.

## The Golden Rule: One Logical Change Per Session

An agent session should produce **one reviewable diff**. If you can't summarize the change in a single sentence, the task is too large. Split it.

| Too large | Well-scoped |
|---|---|
| "Implement virtio-net support" | "Add the `VirtioNetHeader` struct and its `Pod` impl in `virtio/net/header.rs`" |
| "Fix all clippy warnings" | "Fix the `needless_pass_by_value` warnings in `ostd/src/mm/`" |
| "Refactor the page allocator" | "Extract `BuddyAllocator::split` into a standalone function with doc comments" |

## Recommended Workflow

```
1. Define the task
      │
      ▼
2. Write a task brief  ──► Include: file paths, relevant types, expected behavior,
      │                           links to related issues or docs
      ▼
3. Run the agent
      │
      ▼
4. Inspect raw output ──► Check for hallucinated APIs, unsafe blocks, ignored invariants
      │
      ▼
5. Run CI locally     ──► `make check` must pass before you proceed
      │
      ▼
6. Open a PR          ──► Label with `agent-assisted`
```

## Writing a Task Brief

A task brief is a short document you pass to the agent as context. It doesn't need to be long—a few paragraphs plus relevant code snippets is enough. A good brief answers:

- **What** is the goal? (one sentence)
- **Where** does the change live? (file paths, module names)
- **What** already exists that the agent should build on? (paste the relevant type definitions or function signatures)
- **What** must stay the same? (invariants, existing public API surface, unsafe contracts)
- **How** do we know it's done? (test name, CI job, observable behavior)

### Example Task Brief

```
Goal: Add `Display` impl for `FrameAllocError` in ostd/src/mm/frame/alloc.rs

Context:
  - `FrameAllocError` is defined at line 42 of that file (pasted below)
  - We use `core::fmt`, not `std`
  - The message should be lowercase and not end with a period, per our conventions

Invariants:
  - Do not change the enum variants or their fields
  - Do not add any new dependencies

Done when: `cargo test -p ostd` passes and the new impl appears in rustdoc
```

## Task Categories and Agent Fit

| Category | Agent fit | Notes |
|---|---|---|
| Boilerplate impls (`Display`, `Debug`, `From`) | ✅ Excellent | Low risk, easy to verify |
| Doc comment writing | ✅ Excellent | Always review for accuracy |
| Test case generation | ✅ Good | Check edge cases manually |
| Bug fixes with a clear repro | ✅ Good | Provide the failing test |
| New data structures | ⚠️ Moderate | Review ownership and lifetime choices carefully |
| `unsafe` code | ❌ Not recommended | Agents cannot reason reliably about safety invariants |
| Cross-subsystem refactors | ❌ Not recommended | Too many implicit dependencies |

## After the Agent Runs

Before opening a PR:

1. **Read every line** of the diff. Do not rely on the agent's own summary.
2. Run `cargo clippy --all-targets` and fix any new warnings.
3. Run `cargo test` in the affected crate.
4. If the change touches `unsafe`, have a second human reviewer regardless of CI status.
