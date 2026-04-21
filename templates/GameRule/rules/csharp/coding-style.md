---
paths:
  - "**/*.cs"
  - "**/*.csx"
---

# C# Coding Style For Unity 2D

## Naming Conventions

- Use `PascalCase` for classes, structs, enums, interfaces, methods, properties, events, and public fields.
- Use `camelCase` for method parameters and local variables.
- Use `_camelCase` for private fields.
- Prefix interfaces with `I` (example: `IDamageable`).
- Use clear gameplay names (`MoveSpeed`, `JumpForce`, `GroundCheckRadius`) instead of abbreviations.

## Fields and Serialization

- Prefer `[SerializeField] private` fields over public mutable fields.
- Expose read-only access through properties when needed.
- Use `[Header]`, `[Tooltip]`, `[Range]`, and `[Min]` to make Inspector data safer.
- Keep serialized fields grouped at the top of each `MonoBehaviour`.
- Do not serialize transient runtime caches.

```csharp
public sealed class PlayerMotor : MonoBehaviour
{
    [Header("Movement")]
    [SerializeField, Min(0f)] private float _moveSpeed = 6f;
    [SerializeField, Min(0f)] private float _jumpForce = 12f;

    private Rigidbody2D _rigidbody2D;

    public float MoveSpeed => _moveSpeed;

    private void Awake()
    {
        _rigidbody2D = GetComponent<Rigidbody2D>();
        Debug.Assert(_rigidbody2D != null, "Rigidbody2D is required.", this);
    }
}
```

## Class Layout

- Keep one top-level type per file and match file name to type name.
- Recommended order:
  1. Constants and static fields
  2. Serialized fields
  3. Private runtime fields/cache
  4. Properties
  5. Unity message methods (`Awake`, `OnEnable`, `Start`, `Update`, etc.)
  6. Public methods
  7. Private helper methods

## Method Style

- Keep `Update`, `FixedUpdate`, and `LateUpdate` short and allocation-free.
- Prefer early returns to reduce nested conditionals.
- Extract repeated gameplay logic into private helper methods.
- Avoid magic numbers; use serialized fields or named constants.

## Null Safety Rules

- Validate required references in `Awake()` and `OnValidate()`.
- Guard all optional dependencies before use.
- Never assume Inspector wiring is correct without checks.
- Fail fast with clear messages when required dependencies are missing.

## Unity-Specific Constraints

- Never call `GameObject.Find()` in frame-loop methods.
- Never call `GetComponent()` in frame-loop methods.
- Cache `Transform` and required components during initialization.
- Use object pooling for repeated runtime spawn/despawn flows.
