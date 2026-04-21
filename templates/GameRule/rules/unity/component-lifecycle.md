---
paths:
  - "**/*.cs"
  - "**/*.csx"
---

# Unity Component Lifecycle Rules (Strict)

These are definitive lifecycle rules for generated Unity C# code.
Every `MonoBehaviour` must follow this responsibility split.

## 1. `Awake()` Rules

`Awake()` is for local initialization and caching only.

Allowed in `Awake()`:

- Cache `Transform`, `Rigidbody2D`, `Collider2D`, and required local components.
- Initialize internal data structures and state defaults.
- Validate serialized fields and required dependencies with explicit null checks.
- Prepare object-pool references.

Not allowed in `Awake()`:

- Gameplay logic that depends on other objects finishing initialization.
- Input polling.
- Physics stepping.

## 2. `Start()` Rules

`Start()` is for inter-object dependencies and runtime wiring.

Allowed in `Start()`:

- Resolve references that depend on other objects' `Awake()` completion.
- Subscribe to events/signals from other systems.
- Kick off non-physics startup flows (UI sync, state bootstrap, scripted intros).

Not allowed in `Start()`:

- Repeated per-frame logic.
- Heavy runtime searching patterns when references can be serialized/injected.

## 3. `Update()` Rules

`Update()` is for frame-based, non-physics runtime logic.

Allowed in `Update()`:

- Player input polling.
- Timers, cooldown counters, and state checks.
- Animation parameter updates not tied to physics stepping.

Not allowed in `Update()`:

- Physics movement and force application.
- `GameObject.Find()` calls.
- `GetComponent()` calls.
- Managed allocations in hot paths.

## 4. `FixedUpdate()` Rules

`FixedUpdate()` is exclusively for physics logic.

Allowed in `FixedUpdate()`:

- `Rigidbody2D` velocity updates.
- `Rigidbody2D` force/impulse application.
- Physics queries synchronized with fixed timestep.

Not allowed in `FixedUpdate()`:

- UI updates.
- General input polling (capture input in `Update()`, consume in `FixedUpdate()`).
- `GameObject.Find()` calls.
- `GetComponent()` calls.

## 5. Required Pattern

Generated Unity scripts must follow this flow:

1. Cache + validate in `Awake()`.
2. Wire external dependencies in `Start()`.
3. Read input and timers in `Update()`.
4. Apply physics in `FixedUpdate()`.

## 6. Example Lifecycle Template

```csharp
using UnityEngine;

public sealed class PlayerController2D : MonoBehaviour
{
    [SerializeField] private float _moveSpeed = 6f;

    private Transform _cachedTransform;
    private Rigidbody2D _rigidbody2D;
    private float _horizontalInput;

    private void Awake()
    {
        _cachedTransform = transform;
        _rigidbody2D = GetComponent<Rigidbody2D>();

        Debug.Assert(_rigidbody2D != null, "Rigidbody2D is required.", this);
    }

    private void Start()
    {
        // Inter-object wiring only (events/services/managers).
    }

    private void Update()
    {
        _horizontalInput = Input.GetAxisRaw("Horizontal");
    }

    private void FixedUpdate()
    {
        Vector2 velocity = _rigidbody2D.linearVelocity;
        velocity.x = _horizontalInput * _moveSpeed;
        _rigidbody2D.linearVelocity = velocity;
    }
}
```
