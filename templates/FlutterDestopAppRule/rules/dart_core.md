# Dart Core Rules (Desktop)

## Objective

Define non-negotiable Dart coding rules for performance, safety, and maintainability in Flutter Desktop apps.

## Non-Negotiable

- **MUST** use `const` constructors and `const` widgets whenever inputs are compile-time constants.
- **MUST** keep models immutable (`final` fields, copy semantics, no mutable shared state).
- **MUST** keep code null-safe and explicit about optional values.
- **MUST** move expensive CPU tasks off the UI isolate.
- **MUST NOT** perform heavy work inside widget `build()`.

## Const-First Policy

- Use `const` for:
  - Stateless widget constructors.
  - Literal collections that do not change.
  - Reusable style objects and constant intents.
- If `const` is not possible, document why in code review notes.

```dart
class SaveButton extends StatelessWidget {
	const SaveButton({super.key, required this.onPressed});

	final VoidCallback onPressed;

	@override
	Widget build(BuildContext context) {
		return ElevatedButton(
			onPressed: onPressed,
			child: const Text('Save'),
		);
	}
}
```

## Isolate Policy

- Use `Isolate.run()` or `compute` for:
  - Large JSON parse/serialization.
  - Encryption, compression, diffing, indexing, sorting large sets.
  - Any synchronous operation likely to exceed a frame budget.
- Keep isolate payloads small and serializable.
- Return typed results and map failures to domain errors.

```dart
Future<List<Record>> parseRecords(String rawJson) async {
	return Isolate.run(() => decodeRecords(rawJson));
}
```

## Async and Error Discipline

- Wrap async boundaries with typed error handling.
- Never swallow exceptions silently.
- Convert infrastructure errors to user-safe domain failures.
- Keep retry logic explicit and bounded.

## Collection and Allocation Discipline

- Prefer lazy iterables when full materialization is not needed.
- Avoid repeated conversions in hot paths (`toList`, `map`, `where` chains in loops).
- Cache expensive derived values when inputs are stable.

## File and Function Boundaries

- Keep functions focused and short.
- Extract pure helper functions for transformation logic.
- Keep platform-specific code isolated behind adapters.

## Forbidden Anti-Patterns

- Mutable global state without ownership boundaries.
- CPU-heavy synchronous loops on UI isolate.
- Blocking file/database operations from UI rendering flow.
- Hidden side effects in getters.

## Review Checklist

- Are `const` opportunities fully used?
- Are heavy tasks offloaded to isolate?
- Are models immutable and null-safe?
- Is error handling explicit and user-safe?
- Are hot paths allocation-aware?
