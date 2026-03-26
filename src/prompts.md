# Writing Effective Prompts

This chapter documents prompt patterns that reliably produce idiomatic, convention-following contributions to the Asterinas codebase.

## Core Principles

**Be explicit about the environment.** Agents default to `std` assumptions. Asterinas runs in a `no_std` kernel context. Always state this.

**Paste the relevant code.** Agents hallucinate types and method names. Grounding them in real source eliminates most of that.

**State what you don't want.** Constraints on what the agent must *not* do are as important as the goal itself.

**Ask for a plan first.** For tasks longer than ~30 lines, ask the agent to describe its approach before writing code. Redirect early rather than after a large diff.

---

## Prompt Templates

### Template 1 — Implementing a Trait

```
You are contributing to Asterinas, a Rust-based OS kernel.
The codebase uses `#![no_std]` throughout. Do not use `std::`.

Implement `{TraitName}` for `{TypeName}`.

The type is defined here:
---
{paste type definition}
---

The trait is defined here:
---
{paste trait definition}
---

Conventions to follow:
- Use `core::fmt` not `std::fmt`
- Error messages: lowercase, no trailing period
- No `unwrap()` or `expect()` in library code; propagate errors with `?`

Do not change the type definition or any existing impls.
Output only the new impl block.
```

### Template 2 — Writing Tests

```
You are contributing to Asterinas, a Rust-based OS kernel.

Write unit tests for the following function:
---
{paste function}
---

Requirements:
- Use `#[test]` in a `#[cfg(test)]` module at the bottom of the file
- Cover: happy path, empty/zero input, boundary values, and one error case
- Tests must compile with `cargo test -p {crate_name}`
- Do not use any test helpers that aren't already imported in the file

Existing tests for context:
---
{paste existing test module if any}
---
```

### Template 3 — Writing Doc Comments

```
Write Rust doc comments for the following items.

Codebase: Asterinas kernel (no_std Rust)
Audience: contributors who are familiar with OS concepts but new to this module

Items to document:
---
{paste structs, enums, functions}
---

Style rules:
- First line: short imperative sentence ("Returns the…", "Represents a…")
- Use `# Errors` section for functions that return `Result`
- Use `# Panics` section if a panic is possible
- Use `# Safety` section for `unsafe fn`
- Do not add `# Examples` unless the usage is non-obvious
- Do not restate the type name in the first line
```

### Template 4 — Fixing a Failing Test

```
A test in Asterinas is failing. Help me fix the implementation, not the test.

Failing test:
---
{paste test}
---

Current implementation:
---
{paste function or impl}
---

Error output:
---
{paste cargo test output}
---

Constraints:
- Do not modify the test
- Do not change the public API (function signature, type names)
- The fix must not introduce new `unsafe` blocks
```

---

## Anti-Patterns to Avoid

**Don't ask for "best practices" without grounding.** The agent will produce generic Rust, not Asterinas-style Rust.

```
# Bad
Write a best-practices implementation of a slab allocator in Rust.

# Good
Implement a slab allocator following the pattern established in
ostd/src/mm/heap_allocator.rs (pasted below). The new allocator lives
in ostd/src/mm/slab.rs.
```

**Don't omit the `no_std` constraint.** The agent will silently use `Vec`, `Box`, `println!`, or `std::sync`.

**Don't ask for large refactors in one prompt.** Break them into a sequence of smaller prompts and verify each step.

**Don't accept "I'll assume…" without checking.** If the agent states an assumption, verify it against the actual source before continuing.

---

## Iterating on Output

When the first response isn't right, correct it directly rather than starting over:

```
The function you wrote uses `std::collections::HashMap`. This is a no_std
codebase. Replace it with the `hashbrown::HashMap` that is already imported
at the top of the file. Keep everything else the same.
```

Targeted corrections produce better results than broad re-prompting.
