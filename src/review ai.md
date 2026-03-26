# Code Review & Verification

Agent-generated code goes through the same review process as human-authored code. This chapter describes what to check before submitting and what reviewers look for when a PR is labeled `agent-assisted`.

## Author Checklist

Run through this list before opening a PR. If any item fails, fix it before requesting review.

### Correctness

- [ ] Every file in the diff has been read line by line.
- [ ] All referenced types, methods, and constants actually exist in the codebase (no hallucinated APIs).
- [ ] The logic matches the task brief—the agent didn't silently scope-creep or simplify.
- [ ] `cargo build --all-targets` passes.
- [ ] `cargo test -p <affected-crate>` passes.
- [ ] `cargo clippy --all-targets -- -D warnings` passes with no new warnings.

### Style & Conventions

- [ ] Follows the [Coding Guidelines](../coding-guidelines.md).
- [ ] Doc comments are accurate (not just plausible-sounding).
- [ ] Error messages are lowercase and do not end with a period.
- [ ] No `unwrap()` or `expect()` in non-test code without an accompanying `// SAFETY:` or `// PANIC:` comment explaining why it's acceptable.

### Safety

- [ ] No new `unsafe` blocks were introduced unless the task explicitly required them.
- [ ] Every `unsafe` block has a `// SAFETY:` comment explaining the invariant being upheld.
- [ ] If `unsafe` code is present, a second human reviewer is assigned before merge.

### Scope

- [ ] The diff does not include changes outside the agreed task scope.
- [ ] No debug artifacts, commented-out code, or `todo!()` macros remain.

---

## What Reviewers Check

Reviewers of `agent-assisted` PRs apply standard review criteria plus a few extra checks.

### Hallucinated API Detection

Agents sometimes invent plausible-but-nonexistent method names or trait impls. Reviewers verify that:

- All called methods exist on the type at the referenced call site.
- All trait bounds are actually implemented for the types in question.
- Module paths are valid (`use` statements resolve).

### Semantic Drift

Agents sometimes produce code that *compiles* but doesn't do what the task required. Reviewers ask: does this code do what the PR description says, or does it do something subtly different?

### Over-Engineering

Agents tend toward generic, over-abstracted solutions. Reviewers check that the implementation isn't more complex than the problem requires.

### Test Quality

Agent-generated tests sometimes:

- Test the implementation rather than the behavior (change the impl, the test still passes).
- Omit error cases or boundary conditions.
- Use `assert!(true)` or otherwise trivially pass.

Reviewers read test bodies, not just test names.

---

## Local Verification Commands

```sh
# Build everything including tests and examples
cargo build --all-targets

# Run tests for a specific crate
cargo test -p ostd

# Run the full test suite
cargo test --workspace

# Lint with warnings as errors
cargo clippy --all-targets -- -D warnings

# Check formatting
cargo fmt --all -- --check

# Run the full pre-submit check (mirrors CI)
make check
```

---

## Escalation

If during review you find that an agent has introduced code you cannot fully reason about—particularly in `unsafe` contexts or core allocator paths—escalate to a maintainer rather than approving with a comment. Uncertainty is a blocker, not a footnote.
