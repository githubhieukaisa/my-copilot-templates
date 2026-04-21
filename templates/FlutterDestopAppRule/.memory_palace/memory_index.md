# Memory Palace Index

## Purpose

Store long-lived technical decisions so they survive chat/session context limits.

## Main Hall

- `architecture_decisions.md` - ADR log for major architecture and tooling decisions.

## What Belongs Here

- Durable decisions that affect implementation patterns.
- Trade-offs that future work must respect.
- Explicit constraints for cross-platform desktop behavior.

## What Does Not Belong Here

- Temporary tasks, daily notes, or debugging scratch.
- Repeated information already stated in rule files.
- Verbose narrative without concrete decision impact.

## ADR Template

```markdown
## ADR-00X: <Title>

Status: proposed | accepted | superseded
Date: YYYY-MM-DD

### Context

<What problem forced a decision?>

### Options

1. <Option A>
2. <Option B>

### Decision

<Chosen option and why>

### Consequences

- Positive:
- Trade-off:
- Follow-up:
```

## Baseline Topics To Capture

- Riverpod default policy and BLoC exception criteria.
- Drift/SQLite schema strategy and migration policy.
- Isolate policy for parse/index/encryption/compression workloads.
- Desktop UX baseline: hover, shortcuts/actions, context menus, window resize behavior.
- Packaging/signing/notarization strategy per OS.

## Update Procedure

1. Add ADR entry in `architecture_decisions.md`.
2. Mark old ADR as `superseded` when replaced.
3. Add direct cross-reference in related rule/agent file when behavior changes.
