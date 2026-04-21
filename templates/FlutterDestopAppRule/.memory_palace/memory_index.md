# Memory Palace Index

## Purpose

Store long-lived technical decisions so they survive chat/session context limits.

## Main Hall & Context Splitting Rule (Room & Drawer Principle)

To prevent context bloat, **NEVER dump everything into a single monolithic file.** The Memory Palace is divided into specific "Rooms". As a Copilot Agent, you MUST only read the specific room relevant to the user's current prompt.

- `architecture_decisions.md` - Core ADRs for system-wide tooling and high-level architecture only.
- _(Add new files here as the project grows, e.g., `ui_decisions.md`, `state_management_decisions.md`, etc.)_

**The Splitting Rule:** If any file in the `.memory_palace/` directory exceeds 150 lines or covers more than one specific domain, YOU MUST SPLIT IT.

1. Create a new, highly specific Markdown file (a new "Room").
2. Move the relevant content to the new file.
3. Update this `memory_index.md` file with a link and a 1-sentence description of the new Room.
4. When answering future queries, ONLY load the specific Room needed for the task. DO NOT load the entire Memory Palace.

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
