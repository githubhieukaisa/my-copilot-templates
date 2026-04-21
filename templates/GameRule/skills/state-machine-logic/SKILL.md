---
name: state-machine-logic
description: Use this skill when implementing or refactoring Unity 2D Enemy NPC AI with finite state machines, deterministic transitions, and anti-flicker state guards.
origin: GameRule
---

# Enemy NPC FSM For Unity C#

## When to use this skill

- Building Enemy NPC behavior with discrete states such as Idle, Patrol, Chase, Attack, Jump, and WallSlide.
- Refactoring monolithic `Update()` logic into deterministic state-driven architecture.
- Fixing unstable transitions where states rapidly toggle (flicker), especially around wall and ground contact.
- Enforcing strict transition contracts for maintainable AI code.

## Architecture Pattern

### 1. Core Structure

- `EnemyStateMachine` owns the current state, previous state, and transition gate.
- `IEnemyState` defines `Enter`, `Exit`, `Tick`, and `FixedTick`.
- `EnemySensors2D` provides debounced/stable physics contact signals.
- `EnemyContext` stores shared references and runtime values.

### 2. Strict Transition Rules

- Evaluate transitions in one place only (a single transition evaluator).
- Evaluate transition conditions in `FixedUpdate()` when they depend on physics contact.
- Allow at most one transition per fixed step.
- Block self-transitions (`current == target`).
- Enforce a minimum dwell time before leaving a newly entered state.
- Enforce transition cooldown between opposite states.
- Never transition from raw contact flags; use debounced/stable contact values.

### 3. Anti-Flicker Rules (Mandatory)

When an entity continuously touches wall/ground edges, prevent `Idle <-> WallSlide` thrashing with all of the following:

1. Contact debouncing: require contact stability for a short confirmation window (example: 0.05s to enter, 0.08s to exit).
2. State dwell time: after entering a state, keep it for at least a minimum duration (example: 0.10s).
3. Transition cooldown: after any transition, block additional transitions briefly (example: 0.06s).
4. Priority order: evaluate higher-priority states first (e.g., Stunned > Attack > WallSlide > Idle).
5. Signed velocity guard: enter `WallSlide` only when vertical velocity is downward below a threshold.

### 4. Lifecycle Responsibility Split

- `Awake()`: cache components, initialize state machine and transition gates.
- `Update()`: non-physics timers and animation parameter syncing.
- `FixedUpdate()`: contact probes, transition evaluation, and physics-related state ticks.

## Code Examples

### Example 1: FSM Contracts and Transition Gate

```csharp
using System.Collections.Generic;
using UnityEngine;

public interface IEnemyState
{
	string Id { get; }
	void Enter(EnemyController2D owner);
	void Exit(EnemyController2D owner);
	void Tick(EnemyController2D owner, float deltaTime);
	void FixedTick(EnemyController2D owner, float fixedDeltaTime);
}

public sealed class TransitionGate
{
	private readonly float _minStateDuration;
	private readonly float _transitionCooldown;
	private float _lastTransitionTime = float.NegativeInfinity;

	public TransitionGate(float minStateDuration, float transitionCooldown)
	{
		_minStateDuration = minStateDuration;
		_transitionCooldown = transitionCooldown;
	}

	public bool CanTransition(string current, string target, float stateEnteredAt, float now)
	{
		if (current == target) return false;
		if (now - stateEnteredAt < _minStateDuration) return false;
		if (now - _lastTransitionTime < _transitionCooldown) return false;
		return true;
	}

	public void MarkTransition(float now)
	{
		_lastTransitionTime = now;
	}
}

public sealed class EnemyStateMachine
{
	private readonly Dictionary<string, IEnemyState> _states = new Dictionary<string, IEnemyState>();
	private readonly TransitionGate _gate;
	private readonly EnemyController2D _owner;

	private IEnemyState _current;
	private float _stateEnteredAt;

	public string CurrentId => _current != null ? _current.Id : string.Empty;

	public EnemyStateMachine(EnemyController2D owner, TransitionGate gate)
	{
		_owner = owner;
		_gate = gate;
	}

	public void Register(IEnemyState state)
	{
		_states[state.Id] = state;
	}

	public void SetInitial(string id, float now)
	{
		_current = _states[id];
		_stateEnteredAt = now;
		_current.Enter(_owner);
	}

	public bool TryTransition(string targetId, float now)
	{
		if (_current == null || !_states.ContainsKey(targetId)) return false;
		if (!_gate.CanTransition(_current.Id, targetId, _stateEnteredAt, now)) return false;

		_current.Exit(_owner);
		_current = _states[targetId];
		_stateEnteredAt = now;
		_gate.MarkTransition(now);
		_current.Enter(_owner);
		return true;
	}

	public void Tick(float deltaTime)
	{
		_current?.Tick(_owner, deltaTime);
	}

	public void FixedTick(float fixedDeltaTime)
	{
		_current?.FixedTick(_owner, fixedDeltaTime);
	}
}
```

### Example 2: Debounced Contact Signals

```csharp
using UnityEngine;

public sealed class DebouncedBool
{
	private readonly float _onDelay;
	private readonly float _offDelay;
	private bool _stableValue;
	private bool _lastRaw;
	private float _lastRawChangeAt;

	public DebouncedBool(float onDelay, float offDelay, bool initialValue = false)
	{
		_onDelay = onDelay;
		_offDelay = offDelay;
		_stableValue = initialValue;
		_lastRaw = initialValue;
		_lastRawChangeAt = float.NegativeInfinity;
	}

	public bool Update(bool rawValue, float now)
	{
		if (rawValue != _lastRaw)
		{
			_lastRaw = rawValue;
			_lastRawChangeAt = now;
		}

		float required = rawValue ? _onDelay : _offDelay;
		if (now - _lastRawChangeAt >= required)
		{
			_stableValue = rawValue;
		}

		return _stableValue;
	}
}
```

### Example 3: Anti-Flicker Idle/WallSlide Transition Evaluator

```csharp
using UnityEngine;

public sealed class EnemyController2D : MonoBehaviour
{
	private const string IdleState = "Idle";
	private const string WallSlideState = "WallSlide";

	[SerializeField] private float _wallSlideEnterDelay = 0.05f;
	[SerializeField] private float _wallSlideExitDelay = 0.08f;
	[SerializeField] private float _wallSlideMinDownSpeed = -0.05f;

	private Rigidbody2D _rigidbody2D;
	private EnemyStateMachine _fsm;

	private DebouncedBool _groundStable;
	private DebouncedBool _wallStable;

	private float _wallSlideEligibleSince = float.NegativeInfinity;
	private float _idleEligibleSince = float.NegativeInfinity;

	private bool _isGroundedRaw;
	private bool _isTouchingWallRaw;

	private void Awake()
	{
		_rigidbody2D = GetComponent<Rigidbody2D>();
		Debug.Assert(_rigidbody2D != null, "Rigidbody2D is required.", this);

		TransitionGate gate = new TransitionGate(minStateDuration: 0.10f, transitionCooldown: 0.06f);
		_fsm = new EnemyStateMachine(this, gate);

		_groundStable = new DebouncedBool(onDelay: 0.03f, offDelay: 0.07f);
		_wallStable = new DebouncedBool(onDelay: 0.03f, offDelay: 0.07f);

		// Register states here, then:
		// _fsm.SetInitial(IdleState, Time.time);
	}

	private void FixedUpdate()
	{
		float now = Time.time;
		bool isGrounded = _groundStable.Update(_isGroundedRaw, now);
		bool isTouchingWall = _wallStable.Update(_isTouchingWallRaw, now);
		float vy = _rigidbody2D.linearVelocity.y;

		bool wallSlideCandidate = !isGrounded && isTouchingWall && vy < _wallSlideMinDownSpeed;
		if (wallSlideCandidate)
		{
			if (float.IsNegativeInfinity(_wallSlideEligibleSince)) _wallSlideEligibleSince = now;
		}
		else
		{
			_wallSlideEligibleSince = float.NegativeInfinity;
		}

		bool idleCandidate = isGrounded || !isTouchingWall || vy >= _wallSlideMinDownSpeed;
		if (idleCandidate)
		{
			if (float.IsNegativeInfinity(_idleEligibleSince)) _idleEligibleSince = now;
		}
		else
		{
			_idleEligibleSince = float.NegativeInfinity;
		}

		bool wallSlideConfirmed = !float.IsNegativeInfinity(_wallSlideEligibleSince)
			&& now - _wallSlideEligibleSince >= _wallSlideEnterDelay;
		bool idleConfirmed = !float.IsNegativeInfinity(_idleEligibleSince)
			&& now - _idleEligibleSince >= _wallSlideExitDelay;

		if (_fsm.CurrentId == IdleState && wallSlideConfirmed)
		{
			_fsm.TryTransition(WallSlideState, now);
		}
		else if (_fsm.CurrentId == WallSlideState && idleConfirmed)
		{
			_fsm.TryTransition(IdleState, now);
		}

		_fsm.FixedTick(Time.fixedDeltaTime);
	}

	private void Update()
	{
		_fsm.Tick(Time.deltaTime);
	}
}
```
