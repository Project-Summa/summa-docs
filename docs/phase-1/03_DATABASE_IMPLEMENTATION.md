# Database Implementation

> Implementing the local SQLite database layer using drift for the Summa mobile application.

---

# Table of Contents

- Purpose
- drift Setup
- Database Class
- Table Definitions
- DAOs
- Common Fields
- Migrations
- Query Patterns
- Transactions
- Indexes
- Seed Data
- Backup and Restore
- Testing
- Performance
- Future Expansion

---

# Purpose

This document defines the concrete database implementation for Phase 1 of Summa.

The Phase 0 database design document (`02_DATABASE.md`) establishes the schema philosophy, entity definitions and naming conventions. This document translates those decisions into working drift code.

Every table, column, query and migration described here implements the Phase 0 specification. Where implementation details diverge from the design, this document explains why.

The database is the single source of truth for all application data.

---

# drift Setup

## Dependencies

The following packages are required in `pubspec.yaml`.

```yaml
dependencies:
  drift: ^2.16.0
  sqlite3_flutter_libs: ^0.5.0
  path_provider: ^2.1.0
  path: ^1.9.0
  uuid: ^4.0.0

dev_dependencies:
  drift_dev: ^2.16.0
  build_runner: ^2.4.0
```

Package responsibilities:

| Package | Purpose |
|---------|---------|
| drift | Type-safe SQLite ORM for Dart |
| sqlite3_flutter_libs | Bundles native SQLite3 libraries for each platform |
| path_provider | Resolves platform-specific file system paths |
| path | Cross-platform path manipulation |
| uuid | Client-side UUID generation |
| drift_dev | Code generation for drift tables and DAOs |
| build_runner | Runs code generation tasks |

---

## Code Generation

drift uses code generation to produce type-safe query classes from table definitions.

Run the generator with:

```bash
dart run build_runner build --delete-conflicting-outputs
```

During active development, use watch mode:

```bash
dart run build_runner watch --delete-conflicting-outputs
```

Generated files follow the pattern `*.g.dart` and `*.drift.dart`. These files are committed to version control to avoid requiring code generation on every build.

---

## Configuration

Create a `drift.yaml` configuration file in the project root.

```yaml
targets:
  $default:
    builders:
      drift_dev:
        options:
          generate_connect_constructor: true
          override_hash_and_equals: true
          sql:
            dialect: sqlite
            options:
              version: 1
              modules: []
```

Key options:

- `generate_connect_constructor` — enables the `connect()` constructor for dependency injection and testing
- `override_hash_and_equals` — generates proper equality for data classes
- `dialect: sqlite` — targets SQLite specifically (not PostgreSQL)

---

# Database Class

## AppDatabase Definition

The `AppDatabase` class is the central entry point for all database operations.

```dart
@DriftDatabase(tables: [
  Workspaces,
  Profiles,
  Categories,
  Transactions,
  TransactionSplits,
  Budgets,
  Attachments,
], daos: [
  WorkspaceDao,
  ProfileDao,
  CategoryDao,
  TransactionDao,
  TransactionSplitDao,
  BudgetDao,
  AttachmentDao,
])
class AppDatabase extends _$AppDatabase {
  AppDatabase() : super(_openConnection());
  AppDatabase.forTesting(super.e);

  @override
  int get schemaVersion => 1;

  @override
  MigrationStrategy get migration => MigrationStrategy(
        onCreate: (Migrator m) async {
          await m.createAll();
        },
        onUpgrade: (Migrator m, int from, int to) async {
          // Migration logic defined in Migrations section
        },
      );
}
```

---

## Singleton Pattern

A single database instance is shared across the application.

```dart
class DatabaseProvider {
  static AppDatabase? _instance;

  static AppDatabase get instance {
    _instance ??= AppDatabase();
    return _instance!;
  }

  static Future<void> close() async {
    await _instance?.close();
    _instance = null;
  }
}
```

The singleton is registered as a Riverpod provider.

```dart
final databaseProvider = Provider<AppDatabase>((ref) {
  final db = DatabaseProvider.instance;
  ref.onDispose(() => db.close());
  return db;
});
```

---

## Connection Factory

The connection factory opens the SQLite file in the application documents directory.

```dart
LazyDatabase _openConnection() {
  return LazyDatabase(() async {
    final dbFolder = await getApplicationDocumentsDirectory();
    final file = File(p.join(dbFolder.path, 'summa.sqlite'));

    return NativeDatabase.createInBackground(file);
  });
}
```

The database file is named `summa.sqlite` and stored in the platform-specific documents directory. This ensures the file is included in device backups and persists across application updates.

---

# Table Definitions

Every table extends `Table` from drift and follows the naming conventions defined in Phase 0.

All table names are plural and snake_case. All column names are snake_case. All primary keys are UUIDs.

---

## Workspaces Table

```dart
class Workspaces extends Table {
  TextColumn get id => text().clientDefault(() => const Uuid().v4())();
  TextColumn get name => text().withLength(min: 1, max: 100)();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
  DateTimeColumn get updatedAt => dateTime().withDefault(currentDateAndTime)();
  DateTimeColumn get deletedAt => dateTime().nullable()();
  IntColumn get version => integer().withDefault(const Constant(1))();
  TextColumn get syncStatus => text().withDefault(const Constant('local'))();
  TextColumn get deviceId => text()();

  @override
  Set<Column> get primaryKey => {id};
}
```

---

## Profiles Table

```dart
class Profiles extends Table {
  TextColumn get id => text().clientDefault(() => const Uuid().v4())();
  TextColumn get workspaceId => text().references(Workspaces, #id)();
  TextColumn get name => text().withLength(min: 1, max: 50)();
  TextColumn get type => text().withDefault(const Constant('personal'))();
  TextColumn get currency => text().withLength(min: 3, max: 3)();
  BoolColumn get isDefault => boolean().withDefault(const Constant(false))();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
  DateTimeColumn get updatedAt => dateTime().withDefault(currentDateAndTime)();
  DateTimeColumn get deletedAt => dateTime().nullable()();
  IntColumn get version => integer().withDefault(const Constant(1))();
  TextColumn get syncStatus => text().withDefault(const Constant('local'))();
  TextColumn get deviceId => text()();

  @override
  Set<Column> get primaryKey => {id};
}
```

---

## Categories Table

```dart
class Categories extends Table {
  TextColumn get id => text().clientDefault(() => const Uuid().v4())();
  TextColumn get workspaceId => text().references(Workspaces, #id)();
  TextColumn get profileId => text().references(Profiles, #id).nullable()();
  TextColumn get name => text().withLength(min: 1, max: 50)();
  TextColumn get icon => text().withLength(min: 1, max: 50)();
  TextColumn get color => text().withLength(min: 4, max: 9)();
  TextColumn get type => text()();
  BoolColumn get isDefault => boolean().withDefault(const Constant(false))();
  IntColumn get sortOrder => integer().withDefault(const Constant(0))();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
  DateTimeColumn get updatedAt => dateTime().withDefault(currentDateAndTime)();
  DateTimeColumn get deletedAt => dateTime().nullable()();
  IntColumn get version => integer().withDefault(const Constant(1))();
  TextColumn get syncStatus => text().withDefault(const Constant('local'))();
  TextColumn get deviceId => text()();

  @override
  Set<Column> get primaryKey => {id};
}
```

---

## Transactions Table

```dart
class Transactions extends Table {
  TextColumn get id => text().clientDefault(() => const Uuid().v4())();
  TextColumn get workspaceId => text().references(Workspaces, #id)();
  TextColumn get profileId => text().references(Profiles, #id)();
  TextColumn get categoryId => text().references(Categories, #id).nullable()();
  IntColumn get amountMinor => integer()();
  TextColumn get currency => text().withLength(min: 3, max: 3)();
  TextColumn get transactionType => text()();
  TextColumn get note => text().nullable()();
  TextColumn get merchant => text().nullable()();
  DateTimeColumn get occurredAt => dateTime()();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
  DateTimeColumn get updatedAt => dateTime().withDefault(currentDateAndTime)();
  DateTimeColumn get deletedAt => dateTime().nullable()();
  IntColumn get version => integer().withDefault(const Constant(1))();
  TextColumn get syncStatus => text().withDefault(const Constant('local'))();
  TextColumn get deviceId => text()();

  @override
  Set<Column> get primaryKey => {id};
}
```

---

## TransactionSplits Table

```dart
class TransactionSplits extends Table {
  TextColumn get id => text().clientDefault(() => const Uuid().v4())();
  TextColumn get workspaceId => text().references(Workspaces, #id)();
  TextColumn get transactionId => text().references(Transactions, #id)();
  TextColumn get profileId => text().references(Profiles, #id)();
  IntColumn get amountMinor => integer()();
  BoolColumn get isSettled => boolean().withDefault(const Constant(false))();
  DateTimeColumn get settledAt => dateTime().nullable()();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
  DateTimeColumn get updatedAt => dateTime().withDefault(currentDateAndTime)();
  DateTimeColumn get deletedAt => dateTime().nullable()();
  IntColumn get version => integer().withDefault(const Constant(1))();
  TextColumn get syncStatus => text().withDefault(const Constant('local'))();
  TextColumn get deviceId => text()();

  @override
  Set<Column> get primaryKey => {id};
}
```

---

## Budgets Table

```dart
class Budgets extends Table {
  TextColumn get id => text().clientDefault(() => const Uuid().v4())();
  TextColumn get workspaceId => text().references(Workspaces, #id)();
  TextColumn get profileId => text().references(Profiles, #id)();
  TextColumn get categoryId => text().references(Categories, #id).nullable()();
  IntColumn get amountMinor => integer()();
  TextColumn get currency => text().withLength(min: 3, max: 3)();
  TextColumn get period => text()();
  DateTimeColumn get startDate => dateTime()();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
  DateTimeColumn get updatedAt => dateTime().withDefault(currentDateAndTime)();
  DateTimeColumn get deletedAt => dateTime().nullable()();
  IntColumn get version => integer().withDefault(const Constant(1))();
  TextColumn get syncStatus => text().withDefault(const Constant('local'))();
  TextColumn get deviceId => text()();

  @override
  Set<Column> get primaryKey => {id};
}
```

---

## Attachments Table

```dart
class Attachments extends Table {
  TextColumn get id => text().clientDefault(() => const Uuid().v4())();
  TextColumn get workspaceId => text().references(Workspaces, #id)();
  TextColumn get transactionId => text().references(Transactions, #id)();
  TextColumn get fileName => text()();
  TextColumn get mimeType => text()();
  TextColumn get localPath => text()();
  IntColumn get fileSize => integer()();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
  DateTimeColumn get updatedAt => dateTime().withDefault(currentDateAndTime)();
  DateTimeColumn get deletedAt => dateTime().nullable()();
  IntColumn get version => integer().withDefault(const Constant(1))();
  TextColumn get syncStatus => text().withDefault(const Constant('local'))();
  TextColumn get deviceId => text()();

  @override
  Set<Column> get primaryKey => {id};
}
```

---

# DAOs

Data Access Objects encapsulate all database queries for a single entity.

Each DAO is a class annotated with `@DriftDao`. DAOs are injected into repositories through Riverpod and never accessed directly from the presentation layer.

---

## WorkspaceDao

```dart
@DriftDao(tables: [Workspaces])
class WorkspaceDao extends DatabaseAccessor<AppDatabase>
    with _$WorkspaceDaoMixin {
  WorkspaceDao(AppDatabase db) : super(db);

  /// Returns the single local workspace.
  /// Creates a default workspace if none exists.
  Future<Workspace> getOrCreateDefault() async {
    final existing = await (select(workspaces)
          ..where((w) => w.deletedAt.isNull())
          ..limit(1))
        .getSingleOrNull();

    if (existing != null) return existing;

    final id = const Uuid().v4();
    await into(workspaces).insert(
      WorkspacesCompanion.insert(
        id: Value(id),
        name: 'My Workspace',
        deviceId: await _getDeviceId(),
      ),
    );
    return (await (select(workspaces)..where((w) => w.id.equals(id))))
        .single;
  }

  /// Returns all active (non-deleted) workspaces.
  Future<List<Workspace>> getAll() {
    return (select(workspaces)
          ..where((w) => w.deletedAt.isNull())
          ..orderBy([(w) => OrderingTerm.asc(w.createdAt)]))
        .get();
  }

  /// Soft-deletes a workspace by setting deleted_at.
  Future<void> softDelete(String id) {
    return (update(workspaces)..where((w) => w.id.equals(id))).write(
      WorkspacesCompanion(
        deletedAt: Value(DateTime.now().toUtc()),
        updatedAt: Value(DateTime.now().toUtc()),
        version: const Value.absent(), // TODO: increment in Phase 3
      ),
    );
  }
}
```

---

## ProfileDao

```dart
@DriftDao(tables: [Profiles])
class ProfileDao extends DatabaseAccessor<AppDatabase>
    with _$ProfileDaoMixin {
  ProfileDao(AppDatabase db) : super(db);

  /// Returns all active profiles for a workspace.
  Future<List<Profile>> getByWorkspace(String workspaceId) {
    return (select(profiles)
          ..where((p) =>
              p.workspaceId.equals(workspaceId) & p.deletedAt.isNull())
          ..orderBy([(p) => OrderingTerm.asc(p.name)]))
        .get();
  }

  /// Returns the default profile for a workspace.
  Future<Profile?> getDefault(String workspaceId) {
    return (select(profiles)
          ..where((p) =>
              p.workspaceId.equals(workspaceId) &
              p.isDefault.equals(true) &
              p.deletedAt.isNull()))
        .getSingleOrNull();
  }

  /// Watches all active profiles for a workspace.
  Stream<List<Profile>> watchByWorkspace(String workspaceId) {
    return (select(profiles)
          ..where((p) =>
              p.workspaceId.equals(workspaceId) & p.deletedAt.isNull())
          ..orderBy([(p) => OrderingTerm.asc(p.name)]))
        .watch();
  }

  /// Inserts a new profile.
  Future<void> create(ProfilesCompanion entry) {
    return into(profiles).insert(entry);
  }

  /// Updates an existing profile.
  Future<void> updateProfile(ProfilesCompanion entry) {
    return (update(profiles)..where((p) => p.id.equals(entry.id.value)))
        .write(entry.copyWith(
      updatedAt: Value(DateTime.now().toUtc()),
    ));
  }

  /// Soft-deletes a profile.
  Future<void> softDelete(String id) {
    return (update(profiles)..where((p) => p.id.equals(id))).write(
      ProfilesCompanion(
        deletedAt: Value(DateTime.now().toUtc()),
        updatedAt: Value(DateTime.now().toUtc()),
      ),
    );
  }
}
```

---

## CategoryDao

```dart
@DriftDao(tables: [Categories])
class CategoryDao extends DatabaseAccessor<AppDatabase>
    with _$CategoryDaoMixin {
  CategoryDao(AppDatabase db) : super(db);

  /// Returns all active categories for a workspace.
  Future<List<Category>> getByWorkspace(String workspaceId) {
    return (select(categories)
          ..where((c) =>
              c.workspaceId.equals(workspaceId) & c.deletedAt.isNull())
          ..orderBy([
            (c) => OrderingTerm.asc(c.sortOrder),
            (c) => OrderingTerm.asc(c.name),
          ]))
        .get();
  }

  /// Returns categories filtered by type (expense or income).
  Future<List<Category>> getByType(
      String workspaceId, String type) {
    return (select(categories)
          ..where((c) =>
              c.workspaceId.equals(workspaceId) &
              c.type.equals(type) &
              c.deletedAt.isNull())
          ..orderBy([(c) => OrderingTerm.asc(c.sortOrder)]))
        .get();
  }

  /// Watches all active categories for a workspace.
  Stream<List<Category>> watchByWorkspace(String workspaceId) {
    return (select(categories)
          ..where((c) =>
              c.workspaceId.equals(workspaceId) & c.deletedAt.isNull())
          ..orderBy([
            (c) => OrderingTerm.asc(c.sortOrder),
            (c) => OrderingTerm.asc(c.name),
          ]))
        .watch();
  }

  /// Inserts a new category.
  Future<void> create(CategoriesCompanion entry) {
    return into(categories).insert(entry);
  }

  /// Updates an existing category.
  Future<void> updateCategory(CategoriesCompanion entry) {
    return (update(categories)
          ..where((c) => c.id.equals(entry.id.value)))
        .write(entry.copyWith(
      updatedAt: Value(DateTime.now().toUtc()),
    ));
  }

  /// Soft-deletes a category.
  Future<void> softDelete(String id) {
    return (update(categories)..where((c) => c.id.equals(id))).write(
      CategoriesCompanion(
        deletedAt: Value(DateTime.now().toUtc()),
        updatedAt: Value(DateTime.now().toUtc()),
      ),
    );
  }
}
```

---

## TransactionDao

```dart
@DriftDao(tables: [Transactions, Categories])
class TransactionDao extends DatabaseAccessor<AppDatabase>
    with _$TransactionDaoMixin {
  TransactionDao(AppDatabase db) : super(db);

  /// Returns transactions for a profile with pagination.
  Future<List<Transaction>> getByProfile(
    String profileId, {
    int limit = 20,
    int offset = 0,
  }) {
    return (select(transactions)
          ..where((t) =>
              t.profileId.equals(profileId) & t.deletedAt.isNull())
          ..orderBy([(t) => OrderingTerm.desc(t.occurredAt)])
          ..limit(limit, offset: offset))
        .get();
  }

  /// Watches transactions for a profile.
  Stream<List<Transaction>> watchByProfile(String profileId) {
    return (select(transactions)
          ..where((t) =>
              t.profileId.equals(profileId) & t.deletedAt.isNull())
          ..orderBy([(t) => OrderingTerm.desc(t.occurredAt)]))
        .watch();
  }

  /// Returns a single transaction by ID.
  Future<Transaction?> getById(String id) {
    return (select(transactions)
          ..where((t) => t.id.equals(id) & t.deletedAt.isNull()))
        .getSingleOrNull();
  }

  /// Returns transactions within a date range.
  Future<List<Transaction>> getByDateRange(
    String profileId,
    DateTime from,
    DateTime to,
  ) {
    return (select(transactions)
          ..where((t) =>
              t.profileId.equals(profileId) &
              t.deletedAt.isNull() &
              t.occurredAt.isBiggerOrEqualValue(from) &
              t.occurredAt.isSmallerOrEqualValue(to))
          ..orderBy([(t) => OrderingTerm.desc(t.occurredAt)]))
        .get();
  }

  /// Returns transactions filtered by category.
  Future<List<Transaction>> getByCategory(
    String profileId,
    String categoryId,
  ) {
    return (select(transactions)
          ..where((t) =>
              t.profileId.equals(profileId) &
              t.categoryId.equals(categoryId) &
              t.deletedAt.isNull())
          ..orderBy([(t) => OrderingTerm.desc(t.occurredAt)]))
        .get();
  }

  /// Returns transactions filtered by type.
  Future<List<Transaction>> getByType(
    String profileId,
    String transactionType,
  ) {
    return (select(transactions)
          ..where((t) =>
              t.profileId.equals(profileId) &
              t.transactionType.equals(transactionType) &
              t.deletedAt.isNull())
          ..orderBy([(t) => OrderingTerm.desc(t.occurredAt)]))
        .get();
  }

  /// Inserts a new transaction.
  Future<void> create(TransactionsCompanion entry) {
    return into(transactions).insert(entry);
  }

  /// Updates an existing transaction.
  Future<void> updateTransaction(TransactionsCompanion entry) {
    return (update(transactions)
          ..where((t) => t.id.equals(entry.id.value)))
        .write(entry.copyWith(
      updatedAt: Value(DateTime.now().toUtc()),
    ));
  }

  /// Soft-deletes a transaction.
  Future<void> softDelete(String id) {
    return (update(transactions)..where((t) => t.id.equals(id))).write(
      TransactionsCompanion(
        deletedAt: Value(DateTime.now().toUtc()),
        updatedAt: Value(DateTime.now().toUtc()),
      ),
    );
  }

  /// Calculates the total balance for a profile.
  Future<int> getBalance(String profileId) async {
    final expenses = await customSelect(
      'SELECT COALESCE(SUM(amount_minor), 0) as total '
      'FROM transactions '
      'WHERE profile_id = ? AND transaction_type = ? AND deleted_at IS NULL',
      variables: [
        Variable.withString(profileId),
        Variable.withString('expense'),
      ],
    ).getSingle();

    final income = await customSelect(
      'SELECT COALESCE(SUM(amount_minor), 0) as total '
      'FROM transactions '
      'WHERE profile_id = ? AND transaction_type = ? AND deleted_at IS NULL',
      variables: [
        Variable.withString(profileId),
        Variable.withString('income'),
      ],
    ).getSingle();

    return (income.data['total'] as int) -
        (expenses.data['total'] as int);
  }
}
```

---

## TransactionSplitDao

```dart
@DriftDao(tables: [TransactionSplits])
class TransactionSplitDao extends DatabaseAccessor<AppDatabase>
    with _$TransactionSplitDaoMixin {
  TransactionSplitDao(AppDatabase db) : super(db);

  /// Returns all splits for a transaction.
  Future<List<TransactionSplit>> getByTransaction(
      String transactionId) {
    return (select(transactionSplits)
          ..where((s) =>
              s.transactionId.equals(transactionId) &
              s.deletedAt.isNull()))
        .get();
  }

  /// Returns unsettled splits for a profile.
  Future<List<TransactionSplit>> getUnsettled(String profileId) {
    return (select(transactionSplits)
          ..where((s) =>
              s.profileId.equals(profileId) &
              s.isSettled.equals(false) &
              s.deletedAt.isNull()))
        .get();
  }

  /// Inserts a new split.
  Future<void> create(TransactionSplitsCompanion entry) {
    return into(transactionSplits).insert(entry);
  }

  /// Marks a split as settled.
  Future<void> settle(String id) {
    return (update(transactionSplits)
          ..where((s) => s.id.equals(id)))
        .write(
      TransactionSplitsCompanion(
        isSettled: const Value(true),
        settledAt: Value(DateTime.now().toUtc()),
        updatedAt: Value(DateTime.now().toUtc()),
      ),
    );
  }
}
```

---

## BudgetDao

```dart
@DriftDao(tables: [Budgets, Transactions])
class BudgetDao extends DatabaseAccessor<AppDatabase>
    with _$BudgetDaoMixin {
  BudgetDao(AppDatabase db) : super(db);

  /// Returns all active budgets for a profile.
  Future<List<Budget>> getByProfile(String profileId) {
    return (select(budgets)
          ..where((b) =>
              b.profileId.equals(profileId) & b.deletedAt.isNull())
          ..orderBy([(b) => OrderingTerm.asc(b.startDate)]))
        .get();
  }

  /// Returns the budget for a specific category.
  Future<Budget?> getByCategory(
      String profileId, String categoryId) {
    return (select(budgets)
          ..where((b) =>
              b.profileId.equals(profileId) &
              b.categoryId.equals(categoryId) &
              b.deletedAt.isNull()))
        .getSingleOrNull();
  }

  /// Inserts a new budget.
  Future<void> create(BudgetsCompanion entry) {
    return into(budgets).insert(entry);
  }

  /// Updates an existing budget.
  Future<void> updateBudget(BudgetsCompanion entry) {
    return (update(budgets)
          ..where((b) => b.id.equals(entry.id.value)))
        .write(entry.copyWith(
      updatedAt: Value(DateTime.now().toUtc()),
    ));
  }

  /// Soft-deletes a budget.
  Future<void> softDelete(String id) {
    return (update(budgets)..where((b) => b.id.equals(id))).write(
      BudgetsCompanion(
        deletedAt: Value(DateTime.now().toUtc()),
        updatedAt: Value(DateTime.now().toUtc()),
      ),
    );
  }
}
```

---

## AttachmentDao

```dart
@DriftDao(tables: [Attachments])
class AttachmentDao extends DatabaseAccessor<AppDatabase>
    with _$AttachmentDaoMixin {
  AttachmentDao(AppDatabase db) : super(db);

  /// Returns all attachments for a transaction.
  Future<List<Attachment>> getByTransaction(String transactionId) {
    return (select(attachments)
          ..where((a) =>
              a.transactionId.equals(transactionId) &
              a.deletedAt.isNull())
          ..orderBy([(a) => OrderingTerm.asc(a.createdAt)]))
        .get();
  }

  /// Inserts a new attachment.
  Future<void> create(AttachmentsCompanion entry) {
    return into(attachments).insert(entry);
  }

  /// Soft-deletes an attachment.
  Future<void> softDelete(String id) {
    return (update(attachments)..where((a) => a.id.equals(id))).write(
      AttachmentsCompanion(
        deletedAt: Value(DateTime.now().toUtc()),
        updatedAt: Value(DateTime.now().toUtc()),
      ),
    );
  }
}
```

---

# Common Fields

## Sync Metadata Fields

Every table in Phase 1 includes the following synchronization metadata fields as defined in Phase 0.

| Field | drift Type | Default | Phase 1 Behavior |
|-------|-----------|---------|-----------------|
| id | TextColumn | UUID v4 (client-generated) | Active. Used as primary key. |
| created_at | DateTimeColumn | Current UTC time | Active. Set on insert. |
| updated_at | DateTimeColumn | Current UTC time | Active. Updated on every modification. |
| deleted_at | DateTimeColumn | null | Active. Set on soft delete. |
| version | IntegerColumn | 1 | Present but not incremented. Reserved for Phase 3. |
| sync_status | TextColumn | 'local' | Present but not actively used. Always 'local'. |
| device_id | TextColumn | (required) | Set on creation. Not used for sync logic. |

---

## How Sync Fields Are Handled

In Phase 1, the sync fields exist in the schema to ensure forward compatibility. Their behavior is intentionally minimal.

**id** — Generated client-side using `uuid` package. Every record gets a globally unique identifier before it reaches the database. This eliminates the need for auto-increment integers and simplifies future synchronization.

**created_at** — Set once at insert time using `currentDateAndTime` default. Never modified after creation.

**updated_at** — Set at insert time and updated on every write operation. DAOs are responsible for setting this field explicitly on updates.

**deleted_at** — Null for active records. Set to the current UTC timestamp when a record is soft-deleted. All queries filter by `deleted_at IS NULL` to exclude deleted records.

**version** — Defaults to 1. Not incremented during Phase 1 because there is no conflict resolution logic. In Phase 3, every modification will increment this field and the Sync Engine will use it for optimistic concurrency.

**sync_status** — Defaults to `'local'`. Possible values are `local`, `pending`, `synced`, `conflict`. In Phase 1, all records remain `'local'`. The Sync Engine in Phase 3 will transition records through the other states.

**device_id** — Set to the current device's UUID at creation time. In Phase 1, this is informational only. In Phase 3, it identifies which device last modified a record.

---

## Helper: Device ID

The device ID is resolved once and cached.

```dart
class DeviceIdProvider {
  static String? _cachedId;

  static Future<String> get() async {
    if (_cachedId != null) return _cachedId!;

    const storage = FlutterSecureStorage();
    var id = await storage.read(key: 'device_id');

    if (id == null) {
      id = const Uuid().v4();
      await storage.write(key: 'device_id', value: id);
    }

    _cachedId = id;
    return id;
  }
}
```

---

# Migrations

## Strategy

drift provides a declarative migration system. Every schema change increments the `schemaVersion` in `AppDatabase`.

The migration strategy follows these rules:

1. `onCreate` creates all tables for a fresh install
2. `onUpgrade` handles transitions between schema versions
3. Destructive migrations are forbidden in production
4. Every migration is tested

---

## Version Management

```dart
@override
MigrationStrategy get migration => MigrationStrategy(
      onCreate: (Migrator m) async {
        await m.createAll();
        await _seedDefaultData();
      },
      onUpgrade: (Migrator m, int from, int to) async {
        if (from < 2) {
          // Example: add a column in schema version 2
          // await m.addColumn(transactions, transactions.newColumn);
        }
        if (from < 3) {
          // Example: add a new table in schema version 3
          // await m.create(recurringTransactions);
        }
      },
      beforeOpen: (details) async {
        await customStatement('PRAGMA foreign_keys = ON');
      },
    );
```

---

## Migration Testing

Every migration must be tested using drift's `SchemaVerifier`.

```dart
void main() {
  test('upgrade from v1 to v2', () async {
    final verifier = SchemaVerifier(GeneratedHelper());
    final schema = await verifier.schemaAt(1);

    final db = AppDatabase.forTesting(schema.newConnection());
    await db.customStatement('PRAGMA foreign_keys = ON');

    // Verify the database opens and functions correctly
    // after migration from version 1 to version 2
  });
}
```

---

## Adding a New Column

When a new column is added to an existing table:

1. Increment `schemaVersion`
2. Add the column to the drift table definition
3. Add a migration step in `onUpgrade`
4. Provide a default value if the column is non-nullable
5. Run `build_runner` to regenerate code
6. Write a migration test

Example:

```dart
// In schema version 2, add 'tags' to transactions
if (from < 2) {
  await m.addColumn(transactions, transactions.tags);
}
```

---

## Adding a New Table

When a new table is added:

1. Increment `schemaVersion`
2. Define the new table class
3. Add it to the `@DriftDatabase(tables: [...])` annotation
4. Add a migration step: `await m.create(newTable);`
5. Create a DAO for the new table
6. Run `build_runner`
7. Write a migration test

---

# Query Patterns

## Filtering

All queries filter out soft-deleted records by default.

```dart
// Standard filter pattern
..where((t) => t.deletedAt.isNull())
```

Combined filters use the `&` operator.

```dart
..where((t) =>
    t.profileId.equals(profileId) &
    t.transactionType.equals('expense') &
    t.deletedAt.isNull())
```

---

## Sorting

Sorting uses `orderBy` with `OrderingTerm`.

```dart
// Most recent first (default for transaction lists)
..orderBy([(t) => OrderingTerm.desc(t.occurredAt)])

// Alphabetical (for categories)
..orderBy([(c) => OrderingTerm.asc(c.name)])

// Custom order (for user-defined category ordering)
..orderBy([(c) => OrderingTerm.asc(c.sortOrder)])
```

---

## Pagination

Pagination uses `limit` and `offset`.

```dart
Future<List<Transaction>> getPage(
  String profileId, {
  int page = 0,
  int pageSize = 20,
}) {
  return (select(transactions)
        ..where((t) =>
            t.profileId.equals(profileId) & t.deletedAt.isNull())
        ..orderBy([(t) => OrderingTerm.desc(t.occurredAt)])
        ..limit(pageSize, offset: page * pageSize))
      .get();
}
```

---

## Watching (Reactive Queries)

Stream-based queries automatically emit new values when the underlying data changes.

```dart
Stream<List<Transaction>> watchRecent(String profileId, {int limit = 10}) {
  return (select(transactions)
        ..where((t) =>
            t.profileId.equals(profileId) & t.deletedAt.isNull())
        ..orderBy([(t) => OrderingTerm.desc(t.occurredAt)])
        ..limit(limit))
      .watch();
}
```

Streams are consumed by ViewModels through Riverpod and automatically update the UI.

---

## Aggregations

Aggregations use `customSelect` for complex calculations.

```dart
/// Monthly spending total for a profile.
Future<int> getMonthlySpending(
  String profileId,
  int year,
  int month,
) async {
  final result = await customSelect(
    'SELECT COALESCE(SUM(amount_minor), 0) as total '
    'FROM transactions '
    'WHERE profile_id = ? '
    'AND transaction_type = ? '
    'AND deleted_at IS NULL '
    'AND strftime("%Y", occurred_at) = ? '
    'AND strftime("%m", occurred_at) = ?',
    variables: [
      Variable.withString(profileId),
      Variable.withString('expense'),
      Variable.withString(year.toString()),
      Variable.withString(month.toString().padLeft(2, '0')),
    ],
  ).getSingle();

  return result.data['total'] as int;
}
```

---

## Joins

drift supports typed joins between related tables.

```dart
/// Returns transactions with their category name.
Future<List<TransactionWithCategory>> getWithCategories(
    String profileId) async {
  final query = select(transactions).join([
    leftOuterJoin(
      categories,
      categories.id.equalsExp(transactions.categoryId),
    ),
  ])
    ..where(
        transactions.profileId.equals(profileId) & transactions.deletedAt.isNull())
    ..orderBy([OrderingTerm.desc(transactions.occurredAt)]);

  final rows = await query.get();
  return rows.map((row) {
    return TransactionWithCategory(
      transaction: row.readTable(transactions),
      categoryName: row.readTableOrNull(categories)?.name,
    );
  }).toList();
}
```

---

# Transactions

## Multi-Table Operations

Some operations modify multiple tables atomically. drift's `transaction` method ensures all operations succeed or all are rolled back.

---

## Creating a Transaction with Splits

```dart
Future<void> createWithSplits(
  TransactionsCompanion transaction,
  List<TransactionSplitsCompanion> splits,
) async {
  await database.transaction(() async {
    await into(transactions).insert(transaction);

    for (final split in splits) {
      await into(transactionSplits).insert(split);
    }
  });
}
```

---

## Deleting a Transaction and Its Dependencies

```dart
Future<void> deleteWithDependencies(String transactionId) async {
  await database.transaction(() async {
    final now = DateTime.now().toUtc();

    // Soft-delete all splits
    await (update(transactionSplits)
          ..where((s) => s.transactionId.equals(transactionId)))
        .write(TransactionSplitsCompanion(
      deletedAt: Value(now),
      updatedAt: Value(now),
    ));

    // Soft-delete all attachments
    await (update(attachments)
          ..where((a) => a.transactionId.equals(transactionId)))
        .write(AttachmentsCompanion(
      deletedAt: Value(now),
      updatedAt: Value(now),
    ));

    // Soft-delete the transaction
    await (update(transactions)
          ..where((t) => t.id.equals(transactionId)))
        .write(TransactionsCompanion(
      deletedAt: Value(now),
      updatedAt: Value(now),
    ));
  });
}
```

---

## Validating Split Totals

When creating splits, the sum of all split amounts must equal the transaction amount.

```dart
Future<void> createWithValidatedSplits(
  TransactionsCompanion transaction,
  List<TransactionSplitsCompanion> splits,
) async {
  final splitTotal = splits.fold<int>(
    0,
    (sum, s) => sum + (s.amountMinor.value),
  );

  if (splitTotal != transaction.amountMinor.value) {
    throw SplitAmountMismatchException(
      expected: transaction.amountMinor.value,
      actual: splitTotal,
    );
  }

  await createWithSplits(transaction, splits);
}
```

---

# Indexes

## Required Indexes

Indexes are created to optimize the most frequent query patterns.

```dart
@override
MigrationStrategy get migration => MigrationStrategy(
      onCreate: (Migrator m) async {
        await m.createAll();
        await _createIndexes();
      },
      // ...
    );

Future<void> _createIndexes() async {
  // Transaction lookups by profile (most common query)
  await customStatement(
    'CREATE INDEX idx_transactions_profile_id '
    'ON transactions (profile_id) WHERE deleted_at IS NULL',
  );

  // Transaction lookups by category
  await customStatement(
    'CREATE INDEX idx_transactions_category_id '
    'ON transactions (category_id) WHERE deleted_at IS NULL',
  );

  // Transaction sorting by date
  await customStatement(
    'CREATE INDEX idx_transactions_occurred_at '
    'ON transactions (occurred_at DESC)',
  );

  // Transaction filtering by type
  await customStatement(
    'CREATE INDEX idx_transactions_type '
    'ON transactions (transaction_type) WHERE deleted_at IS NULL',
  );

  // Composite index for dashboard queries
  await customStatement(
    'CREATE INDEX idx_transactions_profile_type_date '
    'ON transactions (profile_id, transaction_type, occurred_at) '
    'WHERE deleted_at IS NULL',
  );

  // Category lookups by workspace
  await customStatement(
    'CREATE INDEX idx_categories_workspace_id '
    'ON categories (workspace_id) WHERE deleted_at IS NULL',
  );

  // Profile lookups by workspace
  await customStatement(
    'CREATE INDEX idx_profiles_workspace_id '
    'ON profiles (workspace_id) WHERE deleted_at IS NULL',
  );

  // Split lookups by transaction
  await customStatement(
    'CREATE INDEX idx_splits_transaction_id '
    'ON transaction_splits (transaction_id) WHERE deleted_at IS NULL',
  );

  // Budget lookups by profile
  await customStatement(
    'CREATE INDEX idx_budgets_profile_id '
    'ON budgets (profile_id) WHERE deleted_at IS NULL',
  );

  // Attachment lookups by transaction
  await customStatement(
    'CREATE INDEX idx_attachments_transaction_id '
    'ON attachments (transaction_id) WHERE deleted_at IS NULL',
  );
}
```

---

## Index Rationale

| Index | Query Pattern | Frequency |
|-------|--------------|-----------|
| transactions.profile_id | List transactions for a profile | Every screen load |
| transactions.category_id | Filter by category | Category detail view |
| transactions.occurred_at | Sort by date, date range queries | Every transaction list |
| transactions.transaction_type | Filter expense/income | Dashboard, statistics |
| transactions (composite) | Dashboard balance + recent | Every dashboard load |
| categories.workspace_id | List categories for workspace | Category picker |
| profiles.workspace_id | List profiles for workspace | Profile switcher |
| transaction_splits.transaction_id | Load splits for transaction | Transaction detail |
| budgets.profile_id | Load budgets for profile | Budget screen |
| attachments.transaction_id | Load attachments for transaction | Transaction detail |

Partial indexes (with `WHERE deleted_at IS NULL`) keep the index small by excluding soft-deleted records.

---

# Seed Data

## Default Categories

On first launch, the application seeds default categories into the auto-created workspace.

```dart
Future<void> _seedDefaultData() async {
  final workspace = await workspaceDao.getOrCreateDefault();
  final deviceId = await DeviceIdProvider.get();

  final defaultCategories = [
    // Expense categories
    _category(workspace.id, deviceId, 'Food & Dining', 'restaurant', '#FF5722', 'expense', 0),
    _category(workspace.id, deviceId, 'Transport', 'directions_car', '#2196F3', 'expense', 1),
    _category(workspace.id, deviceId, 'Shopping', 'shopping_bag', '#E91E63', 'expense', 2),
    _category(workspace.id, deviceId, 'Bills & Utilities', 'receipt', '#FF9800', 'expense', 3),
    _category(workspace.id, deviceId, 'Entertainment', 'movie', '#9C27B0', 'expense', 4),
    _category(workspace.id, deviceId, 'Health', 'local_hospital', '#4CAF50', 'expense', 5),
    _category(workspace.id, deviceId, 'Education', 'school', '#00BCD4', 'expense', 6),
    _category(workspace.id, deviceId, 'Other Expense', 'more_horiz', '#607D8B', 'expense', 7),

    // Income categories
    _category(workspace.id, deviceId, 'Salary', 'work', '#4CAF50', 'income', 0),
    _category(workspace.id, deviceId, 'Freelance', 'laptop', '#2196F3', 'income', 1),
    _category(workspace.id, deviceId, 'Investment', 'trending_up', '#FF9800', 'income', 2),
    _category(workspace.id, deviceId, 'Other Income', 'more_horiz', '#607D8B', 'income', 3),
  ];

  for (final category in defaultCategories) {
    await into(categories).insert(category);
  }
}

CategoriesCompanion _category(
  String workspaceId,
  String deviceId,
  String name,
  String icon,
  String color,
  String type,
  int sortOrder,
) {
  return CategoriesCompanion.insert(
    id: Value(const Uuid().v4()),
    workspaceId: workspaceId,
    name: name,
    icon: icon,
    color: color,
    type: type,
    isDefault: const Value(true),
    sortOrder: Value(sortOrder),
    deviceId: deviceId,
  );
}
```

---

## Default Workspace

A single workspace is created automatically on first launch. The user never interacts with the workspace concept directly.

The workspace is created by `WorkspaceDao.getOrCreateDefault()` during the `onCreate` migration.

---

## Default Profile

A default personal profile is created alongside the workspace.

```dart
Future<void> _seedDefaultProfile(String workspaceId, String deviceId) async {
  await into(profiles).insert(
    ProfilesCompanion.insert(
      id: Value(const Uuid().v4()),
      workspaceId: workspaceId,
      name: 'Personal',
      type: const Value('personal'),
      currency: const Value('USD'),
      isDefault: const Value(true),
      deviceId: deviceId,
    ),
  );
}
```

The user can change the currency and profile name after first launch.

---

# Backup and Restore

## Export Formats

Phase 1 supports two export formats.

| Format | Use Case | Content |
|--------|----------|---------|
| JSON | Full data export, human-readable | All entities with relationships |
| SQLite copy | Fast backup, lossless | Raw database file |

---

## JSON Export

```dart
class BackupService {
  final AppDatabase database;

  BackupService(this.database);

  Future<Map<String, dynamic>> exportToJson() async {
    final workspaces = await database.select(database.workspaces).get();
    final profiles = await database.select(database.profiles).get();
    final categories = await database.select(database.categories).get();
    final transactions = await database.select(database.transactions).get();
    final splits =
        await database.select(database.transactionSplits).get();
    final budgets = await database.select(database.budgets).get();
    final attachments =
        await database.select(database.attachments).get();

    return {
      'version': 1,
      'exported_at': DateTime.now().toUtc().toIso8601String(),
      'workspaces': workspaces.map((w) => w.toJson()).toList(),
      'profiles': profiles.map((p) => p.toJson()).toList(),
      'categories': categories.map((c) => c.toJson()).toList(),
      'transactions': transactions.map((t) => t.toJson()).toList(),
      'transaction_splits': splits.map((s) => s.toJson()).toList(),
      'budgets': budgets.map((b) => b.toJson()).toList(),
      'attachments': attachments.map((a) => a.toJson()).toList(),
    };
  }
}
```

---

## JSON Import

```dart
Future<void> restoreFromJson(Map<String, dynamic> data) async {
  await database.transaction(() async {
    // Clear existing data
    await database.delete(database.attachments).go();
    await database.delete(database.transactionSplits).go();
    await database.delete(database.transactions).go();
    await database.delete(database.budgets).go();
    await database.delete(database.categories).go();
    await database.delete(database.profiles).go();
    await database.delete(database.workspaces).go();

    // Insert in dependency order
    for (final w in data['workspaces']) {
      await into(database.workspaces).insert(
        Workspace.fromJson(w as Map<String, dynamic>),
      );
    }
    // ... repeat for each entity in dependency order
  });
}
```

---

## SQLite File Copy

For fast, lossless backup, the database file is copied directly.

```dart
Future<void> backupToFile(String backupPath) async {
  final dbFolder = await getApplicationDocumentsDirectory();
  final source = File(p.join(dbFolder.path, 'summa.sqlite'));
  await source.copy(backupPath);
}

Future<void> restoreFromFile(String backupPath) async {
  await DatabaseProvider.close();

  final dbFolder = await getApplicationDocumentsDirectory();
  final target = File(p.join(dbFolder.path, 'summa.sqlite'));
  await File(backupPath).copy(target.path);

  // Re-open the database
  DatabaseProvider.instance;
}
```

SQLite copy is faster than JSON for large datasets and preserves all data exactly. However, it is not portable across schema versions.

---

# Testing

## In-Memory Database

All DAO and repository tests use an in-memory SQLite database.

```dart
AppDatabase createTestDatabase() {
  return AppDatabase.forTesting(
    DatabaseConnection.delayed(
      Future(() async {
        return NativeDatabase.memory();
      }),
    ),
  );
}
```

---

## Test Setup

```dart
late AppDatabase database;

setUp(() async {
  database = createTestDatabase();
  await database.customStatement('PRAGMA foreign_keys = ON');
});

tearDown(() async {
  await database.close();
});
```

---

## DAO Test Example

```dart
group('TransactionDao', () {
  test('creates and retrieves a transaction', () async {
    final dao = TransactionDao(database);

    await dao.create(TransactionsCompanion.insert(
      workspaceId: 'workspace-1',
      profileId: 'profile-1',
      amountMinor: 125050,
      currency: 'USD',
      transactionType: 'expense',
      occurredAt: DateTime.utc(2025, 1, 15),
      deviceId: 'device-1',
    ));

    final results = await dao.getByProfile('profile-1');
    expect(results, hasLength(1));
    expect(results.first.amountMinor, equals(125050));
    expect(results.first.currency, equals('USD'));
  });

  test('soft-deleted transactions are excluded', () async {
    final dao = TransactionDao(database);

    await dao.create(TransactionsCompanion.insert(
      workspaceId: 'workspace-1',
      profileId: 'profile-1',
      amountMinor: 5000,
      currency: 'USD',
      transactionType: 'expense',
      occurredAt: DateTime.utc(2025, 1, 15),
      deviceId: 'device-1',
    ));

    final all = await dao.getByProfile('profile-1');
    await dao.softDelete(all.first.id);

    final remaining = await dao.getByProfile('profile-1');
    expect(remaining, isEmpty);
  });

  test('balance calculation is correct', () async {
    final dao = TransactionDao(database);

    await dao.create(TransactionsCompanion.insert(
      workspaceId: 'workspace-1',
      profileId: 'profile-1',
      amountMinor: 100000,
      currency: 'USD',
      transactionType: 'income',
      occurredAt: DateTime.utc(2025, 1, 15),
      deviceId: 'device-1',
    ));

    await dao.create(TransactionsCompanion.insert(
      workspaceId: 'workspace-1',
      profileId: 'profile-1',
      amountMinor: 35000,
      currency: 'USD',
      transactionType: 'expense',
      occurredAt: DateTime.utc(2025, 1, 16),
      deviceId: 'device-1',
    ));

    final balance = await dao.getBalance('profile-1');
    expect(balance, equals(65000));
  });
});
```

---

## Migration Test Example

```dart
group('Migrations', () {
  test('database creates successfully on version 1', () async {
    final db = createTestDatabase();

    // Verify all tables exist
    final tables = await db.customSelect(
      "SELECT name FROM sqlite_master WHERE type='table' ORDER BY name",
    ).get();

    final tableNames = tables.map((t) => t.data['name'] as String).toSet();
    expect(tableNames, contains('workspaces'));
    expect(tableNames, contains('profiles'));
    expect(tableNames, contains('categories'));
    expect(tableNames, contains('transactions'));
    expect(tableNames, contains('transaction_splits'));
    expect(tableNames, contains('budgets'));
    expect(tableNames, contains('attachments'));

    await db.close();
  });
});
```

---

## Test Coverage Targets

| Layer | Target | Tool |
|-------|--------|------|
| DAOs | 90%+ | Unit tests with in-memory database |
| Repositories | 85%+ | Unit tests with mock DAOs |
| Use Cases | 90%+ | Unit tests with mock repositories |
| Migrations | 100% | Schema verifier tests |

---

# Performance

## Query Optimization

Guidelines for writing performant queries.

**Use partial indexes** — Indexes with `WHERE deleted_at IS NULL` exclude soft-deleted records and stay small.

**Avoid N+1 queries** — Use joins or batch loading instead of querying inside loops.

**Limit result sets** — Always use `limit` for list queries. Never load all records into memory.

**Use streams for UI** — `watch()` returns a stream that only emits when data changes. This avoids unnecessary rebuilds.

---

## Batch Operations

For bulk inserts, use `batch` instead of individual inserts.

```dart
Future<void> importTransactions(List<TransactionsCompanion> items) async {
  await database.batch((batch) {
    batch.insertAll(transactions, items);
  });
}
```

Batch operations are significantly faster than individual inserts because they execute within a single transaction and reduce I/O overhead.

---

## WAL Mode

SQLite Write-Ahead Logging is enabled by default by drift. WAL mode allows concurrent reads during writes and improves performance for the typical mobile usage pattern.

No additional configuration is required.

---

## Lazy Loading

The database connection is created lazily using `LazyDatabase`. The SQLite file is not opened until the first query is executed. This keeps application startup fast.

---

## Memory Management

Close the database when the application terminates.

```dart
class AppLifecycleObserver extends WidgetsBindingObserver {
  @override
  void didChangeAppLifecycleState(AppLifecycleState state) {
    if (state == AppLifecycleState.detached) {
      DatabaseProvider.close();
    }
  }
}
```

---

## Query Performance Benchmarks

Target performance for common operations on a mid-range device with 10,000 transactions.

| Operation | Target |
|-----------|--------|
| Load transaction list (20 items) | < 50ms |
| Dashboard balance calculation | < 100ms |
| Monthly statistics aggregation | < 200ms |
| Category breakdown | < 150ms |
| Full-text search (if added) | < 300ms |
| JSON export (10,000 transactions) | < 2s |
| SQLite backup copy | < 500ms |

---

# Future Expansion

## Phase 2 — Smart Features

Phase 2 adds entities that build on the Phase 1 schema.

**RecurringTransaction** — A new table that references `transactions` as a template. The schema expands with a `recurring_transactions` table and a foreign key from `transactions` to identify the source recurring rule.

**BankStatement** — Imported bank statement metadata. A new table with a foreign key to `workspaces`.

**OCRDocument** — OCR scan results linked to transactions. A new table with a foreign key to `transactions`.

All Phase 2 tables include the same common fields (id, created_at, updated_at, deleted_at, version, sync_status, device_id).

---

## Phase 3 — Synchronization

The sync fields that are dormant in Phase 1 become active.

**version** — Every modification increments the version number. The Sync Engine uses version numbers for optimistic concurrency control. If two devices modify the same record, the one with the higher version wins.

**sync_status** — Records transition through states:

```
local → pending → synced
                → conflict
```

When a local change is made, `sync_status` becomes `pending`. After successful upload, it becomes `synced`. If a conflict is detected, it becomes `conflict` and requires resolution.

**device_id** — The Sync Engine uses `device_id` to determine which device last modified a record. This is essential for conflict detection and resolution.

**deleted_at** — Soft-deleted records are synchronized as deletion events. The Sync Engine propagates deletions to all devices.

---

## Phase 3 — Sync Engine Integration

The DAOs will be extended with sync-aware methods.

```dart
// Future Phase 3 additions to TransactionDao

/// Returns all records with pending sync status.
Future<List<Transaction>> getPendingSync() {
  return (select(transactions)
        ..where((t) => t.syncStatus.equals('pending')))
      .get();
}

/// Marks a record as synced after successful upload.
Future<void> markSynced(String id, int serverVersion) {
  return (update(transactions)..where((t) => t.id.equals(id))).write(
    TransactionsCompanion(
      syncStatus: const Value('synced'),
      version: Value(serverVersion),
    ),
  );
}

/// Marks a record as conflicted.
Future<void> markConflict(String id) {
  return (update(transactions)..where((t) => t.id.equals(id))).write(
    const TransactionsCompanion(
      syncStatus: Value('conflict'),
    ),
  );
}
```

---

## Phase 3 — Schema Compatibility

The Phase 1 schema is designed to be forward-compatible with Phase 3 synchronization. No breaking changes to existing tables are expected.

New tables introduced in Phase 3 (User, Device, Invitation, Notification, SyncEvent, AuditLog) will be added through the standard migration process.

---

## Phase 4 — Cloud

Phase 4 may introduce cloud backup. The local database remains the source of truth. Cloud storage is an additional backup destination, not a replacement for local storage.

---

# Guiding Principle

The database layer should be boring.

Every query should be predictable. Every migration should be reversible. Every test should be fast.

The database is the foundation. If the foundation is solid, everything built on top of it can move quickly.

Invest in correctness now. Speed comes from confidence, not shortcuts.
