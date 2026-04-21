# Architecture Decision Log

This file is append-only.
Do not rewrite old ADR entries.
When direction changes, add a new ADR and mark old ones as superseded.

## ADR-001: State Management Strategy

Status: accepted
Date: 2026-04-21

### Context

This codebase targets Flutter Desktop (Windows, macOS, Linux) and must keep UI rendering fast and predictable.
The team needs a default state management approach with strong compile-time safety, low boilerplate, and clean separation between UI and business logic.

### Decision

Use Riverpod as the default state management solution.
Use BLoC only for highly complex, event-driven stream scenarios, such as real-time game loops or socket synchronization pipelines.

### Consequences

- Positive:
  - Better compile-time safety and dependency clarity.
  - Lower boilerplate than BLoC in normal feature flows.
  - Easier alignment with const-first widget patterns for desktop performance.
- Trade-off:
  - Team must enforce provider boundaries to avoid oversized providers.
  - Some stream-heavy workflows may still be clearer in BLoC.
- Follow-up:
  - Keep BLoC usage as explicit exception with documented justification.

## ADR-002: Local Storage Strategy

Status: accepted
Date: 2026-04-21

### Context

Desktop apps require reliable local persistence and offline-first behavior.
Storage must be type-safe, testable, and stable across Windows, macOS, and Linux.

### Decision

Use drift with SQLite as the local storage standard for offline-first data.

### Consequences

- Positive:
  - Type-safe SQL and schema-aware migrations.
  - Strong stability across desktop targets.
  - Built-in patterns that work well with background isolate usage.
- Trade-off:
  - Requires migration discipline and schema version management.
  - SQL schema design and query optimization remain team responsibilities.
- Follow-up:
  - Add migration tests for every schema change.
  - Keep repository boundaries strict so UI never depends on raw SQL.

## ADR-003: Desktop Window Management

Status: accepted
Date: 2026-04-21

### Context

The application must behave consistently across native desktop platforms.
The team needs one window control API for resizing, centering, tray behavior, and custom title bar workflows.

### Decision

Use the window_manager package as the default desktop window management layer.

### Consequences

- Positive:
  - Consistent API across Windows, macOS, and Linux.
  - Supports sizing, positioning, visibility control, and custom window UX patterns.
  - Enables tray and title bar related workflows in a unified way.
- Trade-off:
  - Some OS-specific behavior still needs targeted validation per platform.
  - Packaging and permissions must be verified in release builds.
- Follow-up:
  - Maintain platform test checklist for window behavior before release.
  - Document any OS-specific exceptions in future ADRs.
