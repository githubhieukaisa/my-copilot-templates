# State Management Rules

## Objective

Enforce strict separation of concerns and predictable state flow for Flutter Desktop.

## Default Policy

- **MUST** use Riverpod as the default state management solution.
- **MAY** use BLoC only when event-stream orchestration is complex and explicitly justified.
- **MUST NOT** place business logic in widgets.

## Layer Boundaries

- Presentation layer:
  - Renders UI.
  - Dispatches user intents.
  - Contains no domain/business decisions.
- Domain layer:
  - Use cases and business rules.
  - Validation and decision logic.
- Data layer:
  - Repository interfaces and implementations.
  - Local storage and sync orchestration.

## Riverpod Rules

- Keep providers small and purpose-specific.
- Prefer immutable state objects.
- Use `AsyncValue` consistently for loading/error/data branches.
- Side effects (save, delete, sync) belong in notifier/controller, never in widget build.

```dart
final documentsProvider =
		AsyncNotifierProvider<DocumentsController, List<Document>>(DocumentsController.new);

class DocumentsController extends AsyncNotifier<List<Document>> {
	@override
	Future<List<Document>> build() => ref.read(documentRepoProvider).fetchAll();

	Future<void> refresh() async {
		state = const AsyncLoading();
		state = await AsyncValue.guard(() => ref.read(documentRepoProvider).fetchAll());
	}
}
```

## UI Rules

- Widgets subscribe and render; they do not orchestrate persistence workflows.
- Keep widgets dumb and composable.
- Use callbacks/intents to request actions from providers/controllers.

## BLoC Exception Criteria

Use BLoC only if all are true:

1. Complex event sequencing is required.
2. Multiple async streams must be merged/coordinated.
3. Riverpod notifier patterns become less clear than explicit events.

## Error and State Transition Rules

- Use explicit states and deterministic transitions.
- Do not mix stale and fresh data silently.
- Surface user-safe failures; keep technical detail in logs.

## Testing Rules

- Unit test notifier/controller behavior for:
  - Initial state.
  - Success transitions.
  - Failure transitions.
  - Retry behavior.
- Widget tests verify correct rendering for loading/error/data.

## Forbidden Anti-Patterns

- `setState` as cross-screen state transport.
- Repository calls from widget tree build methods.
- Monolithic provider handling unrelated feature concerns.
- Mutable shared state exposed directly to UI.

## Review Checklist

- Is Riverpod used by default?
- Is business logic outside widgets?
- Are state transitions explicit and test-covered?
- Are side effects isolated in controllers/notifiers?
- Are BLoC usages justified by exception criteria?
