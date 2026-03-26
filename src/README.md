# Coding Agents Playbook

This playbook is the practical guide for using AI coding agents—such as Claude, Copilot, or other LLM-based tools—as effective contributors to the Asterinas project.

Asterinas is a systems-level Rust project. Its codebase demands precision: memory safety, formal invariants, and strict coding conventions. Coding agents are powerful accelerators, but they require deliberate setup and oversight to produce contributions that meet our bar.

This playbook covers four areas:

| Chapter | What you'll learn |
|---|---|
| [Agent Workflow & Task Decomposition](workflow.md) | How to scope tasks so an agent can execute them reliably |
| [Writing Effective Prompts](prompts.md) | Prompt patterns that produce idiomatic, convention-following Rust |
| [Code Review & Verification](review.md) | How to review and validate agent-generated code before submission |
| [CI/CD Integration](cicd.md) | Running agents inside our CI pipeline and surfacing results |

## Philosophy

We treat coding agents as **junior contributors**, not oracles. Every line they produce goes through the same review process as human-authored code. The playbook is designed around that principle:

- Agents handle well-scoped, bounded tasks.
- Humans define the problem, review the output, and own the final result.
- The CI pipeline is the authority on correctness—agent output that doesn't pass CI doesn't ship.

## Prerequisites

Before using an agent on Asterinas contributions, you should be familiar with:

- The [Coding Guidelines](../coding-guidelines.md) — agents are prompted to follow these; reviewers check against them.
- Basic familiarity with the Asterinas architecture (kernel subsystems, the `ostd` crate, and the component model).
- Access to a working Asterinas development environment.
