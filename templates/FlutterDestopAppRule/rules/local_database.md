# Local Database Rules (Offline-First)

## Objective

Define strict rules for reliable local persistence using Drift/SQLite in Flutter Desktop.

## Default Policy

- **MUST** design data flow as offline-first.
- **MUST** use repository abstraction over Drift/SQLite.
- **MUST** keep UI independent from raw SQL and table adapters.
- **MUST** version schema and manage migrations explicitly.

## Storage Stack

- Preferred: Drift over SQLite for typed queries and migration safety.
- Acceptable: direct SQLite access only if Drift is not viable for a constrained case.
- All database access flows through data source + repository layers.

## Schema and Migration Rules

- Define primary keys explicitly.
- Add indexes for frequent filter/sort columns.
- Avoid destructive migration without rollback plan.
- Every schema change must include migration and tests.

```dart
@DriftDatabase(tables: [Documents])
class AppDatabase extends _$AppDatabase {
	AppDatabase(QueryExecutor e) : super(e);

	@override
	int get schemaVersion => 2;
}
```

## Repository Contract Rules

- Repositories return domain models, not raw row maps.
- Repository APIs are intention-based (`saveDocument`, `watchDocuments`, `deleteById`).
- Use transactions for multi-step consistency updates.

```dart
Future<void> replaceAll(List<DocumentCompanion> rows) {
	return transaction(() async {
		await delete(documents).go();
		await batch((b) => b.insertAll(documents, rows));
	});
}
```

## Performance Rules

- Batch writes where possible.
- Stream only data required by current UI scope.
- Offload heavy mapping/transformation from large result sets to isolate when needed.
- Avoid repeated full-table reads for small UI updates.

## Sync and Conflict Rules

- Keep local database as source of truth for UI rendering.
- Record sync metadata (`updatedAt`, `syncState`, `version` where applicable).
- Define deterministic conflict strategy (last-write-wins or domain-merge rule).

## Reliability Rules

- Handle database open failures with user-safe fallback.
- Guard against partial writes via transactions.
- Keep migration code idempotent and testable.

## Testing Rules

- Unit tests for repository contract and query behavior.
- Migration tests from previous schema versions.
- Integration tests for write-read consistency and transaction rollback.

## Forbidden Anti-Patterns

- Raw SQL in UI/widget classes.
- Schema changes without migration tests.
- Blocking large query post-processing on UI isolate.
- Treating remote API as source of truth while local cache is stale.

## Review Checklist

- Is repository boundary respected?
- Are migrations versioned and tested?
- Are transactions used for multi-step writes?
- Is local DB the immediate source for UI?
- Are heavy transforms isolated from UI thread?
