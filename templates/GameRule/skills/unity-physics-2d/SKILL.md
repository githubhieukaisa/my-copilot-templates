---
name: unity-physics-2d
description: Use this skill when writing or reviewing Unity 2D gameplay physics, Rigidbody2D movement, and Raycast2D-based sensing with deterministic FixedUpdate behavior.
origin: GameRule
---

# Unity Physics 2D Best Practices

## When to use this skill

- Implementing player or enemy movement that depends on `Rigidbody2D`.
- Building ground checks, wall checks, ledge checks, and line-of-sight with `Raycast2D`.
- Choosing the correct body type (`Dynamic`, `Kinematic`, `Static`) for gameplay objects.
- Fixing jitter, tunneling, or inconsistent collisions caused by physics logic in `Update()`.

## Architecture Pattern

### 1. Body Type Decision Rules

- `Dynamic`:
  - Use for actors affected by gravity, forces, and collision response.
  - Examples: player characters, physically reactive enemies, moving crates.
- `Kinematic`:
  - Use for scripted movers not driven by forces.
  - Move with `Rigidbody2D.MovePosition`/`MoveRotation` in `FixedUpdate()`.
  - Enable full kinematic contacts only when collision reporting requirements demand it.
- `Static`:
  - Use for non-moving level geometry and permanent colliders.
  - Do not move static bodies at runtime.

### 2. Raycast Rules (Strict)

- Always provide an explicit `LayerMask`; never rely on default layers for gameplay checks.
- Keep ray origin, direction, and max distance explicit and serialized where appropriate.
- Ignore triggers when using solid-world probes unless trigger interaction is intentional.
- Use non-allocating raycast patterns in repeated checks.

### 3. Fixed Timestep Rules (Mandatory)

- Physics mutations must only happen in `FixedUpdate()`:
  - velocity writes
  - force and impulse application
  - `MovePosition`/`MoveRotation`
- `Update()` may read input and timers only; it must not mutate physics state.
- Reason: `FixedUpdate()` runs on a fixed timestep synchronized with physics simulation, preventing frame-rate dependent behavior, jitter, and nondeterministic collisions.

### 4. System Split

- `Awake()`: cache `Rigidbody2D`, configure contact filters, allocate hit buffers.
- `Update()`: capture input intent only.
- `FixedUpdate()`: perform physics probes, resolve movement, apply velocities/forces.

## Code Examples

### Example 1: Body Type Setup Guidance

```csharp
using UnityEngine;

public sealed class BodyTypeExample : MonoBehaviour
{
	[SerializeField] private Rigidbody2D _rb;

	private void Awake()
	{
		Debug.Assert(_rb != null, "Rigidbody2D is required.", this);

		// Player-like actor: responds to gravity and collisions.
		_rb.bodyType = RigidbodyType2D.Dynamic;

		// Scripted platform: uncomment when needed.
		// _rb.bodyType = RigidbodyType2D.Kinematic;

		// Level geometry should usually be Static and not moved at runtime.
		// _rb.bodyType = RigidbodyType2D.Static;
	}
}
```

### Example 2: Raycast2D With Layer Mask (Single Hit)

```csharp
using UnityEngine;

public sealed class GroundProbe : MonoBehaviour
{
	[SerializeField] private Transform _origin;
	[SerializeField] private float _distance = 0.2f;
	[SerializeField] private LayerMask _groundMask;

	public bool IsGrounded()
	{
		RaycastHit2D hit = Physics2D.Raycast(
			_origin.position,
			Vector2.down,
			_distance,
			_groundMask);

		return hit.collider != null;
	}
}
```

### Example 3: Non-Allocating Raycast Pattern

```csharp
using UnityEngine;

public sealed class NonAllocProbe : MonoBehaviour
{
	[SerializeField] private Transform _origin;
	[SerializeField] private float _distance = 0.5f;
	[SerializeField] private LayerMask _worldMask;

	private readonly RaycastHit2D[] _hits = new RaycastHit2D[8];
	private ContactFilter2D _filter;

	private void Awake()
	{
		_filter = new ContactFilter2D
		{
			useLayerMask = true,
			layerMask = _worldMask,
			useTriggers = false
		};
	}

	public int ProbeForwardNonAlloc(Vector2 direction)
	{
		return Physics2D.Raycast(
			_origin.position,
			direction.normalized,
			_filter,
			_hits,
			_distance);
	}
}
```

### Example 4: Input In Update, Physics In FixedUpdate

```csharp
using UnityEngine;

public sealed class PlayerMotor2D : MonoBehaviour
{
	[SerializeField] private float _moveSpeed = 6f;

	private Rigidbody2D _rb;
	private float _horizontalInput;

	private void Awake()
	{
		_rb = GetComponent<Rigidbody2D>();
		Debug.Assert(_rb != null, "Rigidbody2D is required.", this);
	}

	private void Update()
	{
		_horizontalInput = Input.GetAxisRaw("Horizontal");
	}

	private void FixedUpdate()
	{
		Vector2 velocity = _rb.linearVelocity;
		velocity.x = _horizontalInput * _moveSpeed;
		_rb.linearVelocity = velocity;
	}
}
```
