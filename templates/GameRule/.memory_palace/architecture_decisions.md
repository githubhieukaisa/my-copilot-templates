# Architecture Decision Log

This file is append-only.
Do not rewrite old ADR entries.
When direction changes, add a new ADR and mark old ones as superseded.

## ADR-001: Input Handling

Status: accepted
Date: 2026-04-21

### Context

The project must scale across keyboard/mouse and controller input with consistent gameplay behavior.
The legacy Input Manager makes action rebinding, modular control schemes, and long-term platform scaling harder to maintain.

### Decision

Use the new Unity Input System as the default input layer.
Do not add new gameplay features on the legacy Input Manager.

### Consequences

- Positive:
  - Action maps and control schemes provide cleaner cross-platform input expansion.
  - Rebinding workflows are easier to implement and validate.
  - Input logic becomes more modular and testable.
- Trade-off:
  - Team must maintain consistent action naming and map boundaries.
  - Existing legacy bindings require migration effort.
- Follow-up:
  - Define project-wide action map conventions.
  - Add input regression checks for keyboard/mouse and controller paths.

## ADR-002: Dependency Management

Status: accepted
Date: 2026-04-21

### Context

Gameplay systems have growing service dependencies and configuration needs.
Uncontrolled singleton usage creates hidden couplings, brittle scene setup, and difficult testing.

### Decision

Use VContainer or Zenject for runtime dependency wiring.
Use strict ScriptableObject assets for configuration boundaries to prevent singleton spaghetti.

### Consequences

- Positive:
  - Dependency graphs become explicit and easier to reason about.
  - Test setup improves because systems can be composed with predictable lifetimes.
  - Configuration stays data-driven and editor-friendly through ScriptableObject assets.
- Trade-off:
  - Team must maintain container composition roots and registration discipline.
  - Legacy singleton-heavy features require staged refactors.
- Follow-up:
  - Define allowed lifetime patterns for gameplay services.
  - Document configuration ownership rules for ScriptableObject assets.

## ADR-003: Entity State Management

Status: accepted
Date: 2026-04-21

### Context

Player and enemy logic grows quickly in complexity as combat behaviors, transitions, and interrupts increase.
Large switch-based state handling becomes hard to scale, test, and debug over time.

### Decision

Use Hierarchical Finite State Machines (HFSM) for player and enemy behavior logic.
Avoid monolithic switch-statement state controllers for production gameplay systems.

### Consequences

- Positive:
  - Clear parent-child state composition improves extensibility and readability.
  - Transition rules become explicit and easier to test.
  - Shared behavior can be centralized without duplicating state logic.
- Trade-off:
  - Initial architecture setup is heavier than a single switch implementation.
  - Team must enforce consistent transition contracts to avoid state explosion.
- Follow-up:
  - Create a standard HFSM skeleton for player and enemy modules.
  - Add tests for critical transition paths and interruption scenarios.
