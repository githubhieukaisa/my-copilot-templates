# Memory Palace Index

## Purpose

Store long-lived technical decisions so they survive chat/session context limits.

## Main Hall

- `architecture_decisions.md` - ADR log for major architecture and tooling decisions.

## What Belongs Here

- Durable decisions that affect gameplay architecture, performance patterns, and production workflows.
- Trade-offs that future work must respect.
- Explicit constraints for frame budget, scene safety, and cross-platform runtime behavior.

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

- Input System default policy and action map conventions.
- Dependency management policy for DI containers and ScriptableObject configuration.
- HFSM conventions for player, enemy, and boss entity behavior.
- Object pooling policy for projectiles, VFX, enemies, and pickups.
- Scene loading strategy and additive scene composition constraints.

## Update Procedure

1. Add ADR entry in `architecture_decisions.md`.
2. Mark old ADR as `superseded` when replaced.
3. Add direct cross-reference in related rule/agent file when behavior changes.
