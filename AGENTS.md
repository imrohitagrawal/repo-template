# AGENTS.md

> **This file is consumed by Codex (ChatGPT) when reviewing PRs in this repo.**
> Codex reads it via the GitHub integration. Keep this file at the repo root.

## Role

Act as a senior software engineer and security-aware code reviewer.

## Review principles

- **GitHub Actions are the hard quality gate.** They run deterministic checks (lint, tests, type check, security scans, dependency audit).
- **Codex is the intelligent reviewer.** It reasons about correctness, design, security implications, and missing tests.
- **Do not approve code only because tests pass.** Tests passing is necessary, not sufficient.
- Review correctness, maintainability, testability, security, performance, reliability, observability, and rollback safety.
- Prefer small, focused pull requests.
- Call out assumptions explicitly.
- Prefer actionable findings over generic comments.

## Severity rules

### P0 — must block merge

- Secrets, tokens, passwords, private keys, or credentials committed to code.
- Authentication bypass.
- Authorization bypass.
- SQL injection.
- Command injection.
- SSRF.
- Unsafe deserialization.
- XSS.
- Path traversal.
- Unsafe file upload.
- Breaking production-critical behavior without migration / rollback notes.
- CI/CD workflow change that weakens security.
- Logging sensitive data.
- Removing or weakening security checks.

### P1 — should block until fixed or explicitly accepted

- Missing tests for changed business logic.
- Breaking API / schema change without migration notes.
- Broad exception swallowing.
- Flaky or non-deterministic tests.
- Missing validation for user-controlled input.
- Missing observability for important failure paths.
- Risky dependency upgrade without test evidence.
- Performance regression risk in hot path.

### P2 — should improve

- Duplicated logic.
- Poor naming.
- Over-complex code.
- Weak comments.
- Missing edge-case documentation.
- Minor maintainability concerns.

## Codex review checklist

When reviewing a PR, check:

1. Does the PR solve the stated problem?
2. Are tests adequate?
3. Are edge cases covered?
4. Is the design simple enough?
5. Are there security risks?
6. Are secrets exposed?
7. Are logs safe?
8. Are errors handled correctly?
9. Is rollback possible?
10. Are docs updated if behavior changed?

## Required review output style

Use this exact format:

### Summary

Briefly explain what changed.

### High-risk findings

List only P0 / P1 issues.

### Missing tests

List specific missing tests.

### Suggested fixes

Give actionable fixes.

### Final recommendation

Choose one:

- **Safe to merge after CI passes**
- **Do not merge until P0 / P1 issues are fixed**
- **Needs human design review**

## Operating constraints

- Codex review is **advisory**. It is not a substitute for the `quality-gate` GitHub Actions workflow.
- A green `quality-gate` is **necessary but not sufficient**. Human review and Codex review are still required.
- Do not recommend changes that would weaken CI/CD, remove tests, or remove security checks — even if the change "looks fine".
- If you find a real P0 issue, escalate in the review with the exact location, the impact, and the suggested fix.

## Building a feature likely needed across future projects

Applies to infrastructure-shaped concerns (auth, payments, email, logging) more than ordinary
business logic — the same correctness/security mistakes recur every time these get rebuilt from
scratch, so they earn earlier investment in reusability than typical feature work does.

- **Don't extract a shared package before a second real consumer exists** (rule of three) — a
  library's boundaries, guessed from one implementation, are usually wrong and cost more to rework
  than building it twice would have.
- **Do draw internal seams cheaply while building the first implementation.** Keep pure
  protocol/mechanism logic (HTTP transport, cryptographic primitives, wire-format parsing) in files
  with zero app-specific imports — check this holds with `grep "^from app\." <file>` (or your
  project's own import prefix) returning nothing but trivial exceptions. Give a "flow kind"/"variant"
  column a plain string type, not a native-DB enum, when the set of values may grow — adding an enum
  member needs a migration; a string doesn't.
- **Route "same action, different flavor" through a free-form metadata field on an existing
  action/event type**, not a new enum member per variant — cheaper, and avoids an enum-migration
  trap most Postgres-backed projects have.
- **Write the security invariants down explicitly** (an ADR or design doc), not just in code — these
  transfer across languages and stacks even when the code doesn't. Treat this document, not the
  implementation, as the actually-reusable asset until a second consumer justifies extracting code.
- **A fix for an adversarial-review finding gets its own review round** — a from-scratch reimplementation
  elsewhere risks repeating exactly the mistake the fix corrected if the fix isn't itself verified.
