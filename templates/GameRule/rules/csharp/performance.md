---
paths:
  - "**/*.cs"
  - "**/*.csx"
---

# C# Performance For Unity 2D (Memory and GC)

## Primary Goal

Avoid garbage-collection spikes and frame-time jitter during gameplay.

## Allocation Rules

- No managed allocations in `Update()`, `FixedUpdate()`, or `LateUpdate()`.
- Avoid `new` in per-frame loops unless there is a documented exception.
- Reuse collections (`List<T>`, `Dictionary<TKey, TValue>`) and call `Clear()`.
- Avoid boxing (especially via non-generic collections or interface casts in hot paths).
- Avoid LINQ in hot paths due to iterator allocations.

## String and Logging Rules

- Do not use string concatenation inside loops or frame-loop methods.
- Prefer cached format strings or `StringBuilder` for repeated string construction.
- Reduce `Debug.Log` calls in runtime loops.

```csharp
private readonly StringBuilder _statusBuilder = new StringBuilder(64);

private string BuildStatusText(int score, int lives)
{
    _statusBuilder.Clear();
    _statusBuilder.Append("Score: ").Append(score);
    _statusBuilder.Append("  Lives: ").Append(lives);
    return _statusBuilder.ToString();
}
```

## Caching Rules

- Cache `Transform` and required components in `Awake()` or `Start()`.
- Never call `GetComponent()` inside frame-loop methods.
- Never call `GameObject.Find()` inside frame-loop methods.

```csharp
public sealed class EnemyChaser : MonoBehaviour
{
    [SerializeField] private Transform _target;

    private Transform _cachedTransform;
    private Rigidbody2D _rigidbody2D;

    private void Awake()
    {
        _cachedTransform = transform;
        _rigidbody2D = GetComponent<Rigidbody2D>();

        Debug.Assert(_target != null, "Target must be assigned.", this);
        Debug.Assert(_rigidbody2D != null, "Rigidbody2D is required.", this);
    }

    private void FixedUpdate()
    {
        Vector2 direction = (_target.position - _cachedTransform.position).normalized;
        _rigidbody2D.linearVelocity = direction * 4f;
    }
}
```

## Object Pooling (Mandatory)

- Use object pooling for projectiles, hit effects, enemies, pickups, and repeated UI/world indicators.
- Avoid repeated `Instantiate` and `Destroy` during gameplay loops.
- Pre-warm pools during scene load where possible.

## Physics and Query Rules

- Prefer non-alloc physics APIs where available.
- Reuse hit buffers for overlap/raycast queries.
- Keep query frequency bounded and avoid redundant checks.

## Coroutine and Timing Rules

- Cache common yield instructions when reused frequently.
- Avoid creating many short-lived coroutines each frame.

## Profiling and Validation

- Use Unity Profiler and verify `GC Alloc` stays at 0 B in hot gameplay loops.
- Validate memory behavior on target hardware, not only in Editor.
- Profile after every gameplay feature that adds spawn, UI updates, or frequent physics checks.
