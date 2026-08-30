# Testing Strategy

> Defining how Phase 1 features are tested to ensure quality and reliability.

---

# Table of Contents

- Purpose
- Testing Philosophy
- Test Types
- Test Organization
- Unit Tests
- Widget Tests
- Integration Tests
- Test Coverage
- Test Data
- Mocking Strategy
- CI Integration
- Test Performance
- Definition of Done
- Guiding Principle

---

# Purpose

This document defines the testing strategy for Phase 1.

Every feature must be tested before it is considered complete. Testing is not optional — it is a core part of the implementation process.

The goal is to catch bugs before users do, enable confident refactoring and document expected behavior through executable tests.

---

# Testing Philosophy

## Test Behavior, Not Implementation

Tests should verify what the code does, not how it does it.

Internal implementation details can change without breaking tests. Public behavior and user-visible outcomes should remain stable.

---

## Test Early, Test Continuously

Write tests alongside implementation, not after.

A feature without tests is not complete. A PR without tests should not be merged.

---

## Test at the Right Level

| What to Test | Test Type |
|---|---|
| Business logic | Unit test |
| UI rendering | Widget test |
| Full user flow | Integration test |
| Database queries | Integration test |
| Navigation | Widget test |
| Error handling | Unit test |

---

## Keep Tests Fast

Unit tests should run in milliseconds. Widget tests in seconds. Integration tests in minutes.

If tests are slow, developers won't run them.

---

# Test Types

## Unit Tests

Test individual classes and functions in isolation.

- Use cases
- ViewModels
- Domain models
- Utility functions
- Validation logic
- Business rules

Dependencies are mocked.

---

## Widget Tests

Test Flutter widgets in isolation.

- Screen rendering
- User interactions
- State changes reflected in UI
- Error states displayed correctly
- Loading states displayed correctly
- Empty states displayed correctly

Dependencies are provided via Riverpod overrides.

---

## Integration Tests

Test complete flows across multiple layers.

- Create transaction end-to-end
- Backup and restore flow
- PIN setup and verification
- Profile switching
- Category filtering

Uses real database (in-memory for speed).

---

# Test Organization

## Directory Structure

```text
lib/
    features/
        transactions/
            domain/
                usecases/
                    create_transaction_use_case.dart
            data/
                repositories/
                    transaction_repository_impl.dart

test/
    features/
        transactions/
            domain/
                usecases/
                    create_transaction_use_case_test.dart
            data/
                repositories/
                    transaction_repository_impl_test.dart
            viewmodel/
                transaction_view_model_test.dart

test_widget/
    features/
        transactions/
            screens/
                transaction_screen_test.dart

integration_test/
    features/
        transactions/
            transaction_flow_test.dart
```

## Naming Convention

```text
<feature>_<what_is_tested>_test.dart
```

Examples:

```text
create_transaction_use_case_test.dart
transaction_view_model_test.dart
transaction_screen_test.dart
transaction_flow_test.dart
```

---

# Unit Tests

## What to Test

### Use Cases

```text
Given: Valid input parameters
When:  Use case is executed
Then:  Expected result is returned

Given: Invalid input parameters
When:  Use case is executed
Then:  Appropriate error is thrown

Given: Repository failure
When:  Use case is executed
Then:  Error is propagated correctly
```

### ViewModels

```text
Given: Initial state
When:  ViewModel is created
Then:  State is loading or empty

Given: Valid event
When:  Event is dispatched
Then:  State updates correctly

Given: Error during processing
When:  Event is dispatched
Then:  State contains error message
```

### Domain Models

```text
Given: Valid data
When:  Model is created
Then:  All fields are set correctly

Given: Invalid data
When:  Model is created
Then:  Validation error is thrown

Given: Model with copyWith
When:  Field is changed
Then:  Original is unchanged, copy has new value
```

## Example Unit Test

```dart
group('CreateTransactionUseCase', () {
  test('creates expense transaction with valid data', () async {
    // Given
    final repository = MockTransactionRepository();
    final useCase = CreateTransactionUseCase(repository);
    final request = CreateTransactionRequest(
      profileId: 'profile-1',
      categoryId: 'category-1',
      amountMinor: 125050,
      currency: 'USD',
      type: TransactionType.expense,
      occurredAt: DateTime.utc(2024, 1, 15),
    );

    // When
    final result = await useCase.execute(request);

    // Then
    expect(result.isSuccess, true);
    verify(repository.create(any)).called(1);
  });

  test('fails with zero amount', () async {
    // Given
    final repository = MockTransactionRepository();
    final useCase = CreateTransactionUseCase(repository);
    final request = CreateTransactionRequest(
      profileId: 'profile-1',
      amountMinor: 0,
      currency: 'USD',
      type: TransactionType.expense,
    );

    // When
    final result = await useCase.execute(request);

    // Then
    expect(result.isFailure, true);
    expect(result.error, isA<ValidationError>());
  });
});
```

---

# Widget Tests

## What to Test

- Screen renders correctly with data
- Screen shows loading state
- Screen shows empty state
- Screen shows error state
- User interactions trigger correct events
- Navigation occurs on user actions
- Form validation displays errors

## Example Widget Test

```dart
testWidgets('shows empty state when no transactions', (tester) async {
  // Given
  final container = ProviderContainer(
    overrides: [
      transactionListProvider.overrideWithValue(
        const AsyncValue.data([]),
      ),
    ],
  );

  // When
  await tester.pumpWidget(
    UncontrolledProviderScope(
      container: container,
      child: const MaterialApp(
        home: TransactionListScreen(),
      ),
    ),
  );

  // Then
  expect(find.text('No transactions yet'), findsOneWidget);
  expect(find.byIcon(Icons.receipt_long), findsOneWidget);
});
```

---

# Integration Tests

## What to Test

- Complete user flows across multiple screens
- Database operations with real data
- Navigation between features
- State persistence across screen changes
- Backup and restore with real data

## Critical Integration Tests

| Flow | Priority |
|---|---|
| Create profile → Add transaction → View on dashboard | P0 |
| Create category → Assign to transaction → Filter | P0 |
| Create backup → Delete data → Restore backup | P0 |
| Set up PIN → Lock app → Unlock with PIN | P0 |
| Create transaction → Edit → Delete | P0 |
| Switch profiles → Verify data isolation | P1 |
| Export data → Verify file contents | P1 |
| Enable biometric → Lock app → Unlock | P1 |

---

# Test Coverage

## Coverage Targets

| Layer | Target |
|---|---|
| Domain (models, use cases) | 90%+ |
| Data (repositories, DAOs) | 85%+ |
| ViewModels | 80%+ |
| UI (widgets) | 70%+ |
| Overall | 80%+ |

## Coverage Measurement

```bash
flutter test --coverage
genhtml coverage/lcov.info -o coverage/html
open coverage/html/index.html
```

## Coverage Exceptions

The following do not require coverage:

- Generated code (freezed, drift)
- Platform-specific code (native plugins)
- Debug-only code
- Main entry point

---

# Test Data

## Test Data Factory

Create factory functions for common test objects.

```dart
class TestData {
  static Workspace workspace({String? id}) => Workspace(
    id: id ?? 'workspace-1',
    name: 'Test Workspace',
    createdAt: DateTime.utc(2024, 1, 1),
  );

  static Profile profile({
    String? id,
    String? workspaceId,
    ProfileType? type,
  }) => Profile(
    id: id ?? 'profile-1',
    workspaceId: workspaceId ?? 'workspace-1',
    name: 'Personal',
    type: type ?? ProfileType.personal,
    currency: 'USD',
    isDefault: true,
    createdAt: DateTime.utc(2024, 1, 1),
  );

  static Transaction transaction({
    String? id,
    String? profileId,
    TransactionType? type,
    int? amountMinor,
  }) => Transaction(
    id: id ?? 'transaction-1',
    workspaceId: 'workspace-1',
    profileId: profileId ?? 'profile-1',
    amountMinor: amountMinor ?? 125050,
    currency: 'USD',
    transactionType: type ?? TransactionType.expense,
    occurredAt: DateTime.utc(2024, 1, 15),
    createdAt: DateTime.utc(2024, 1, 15),
  );
}
```

## In-Memory Database

For integration tests, use an in-memory drift database.

```dart
final db = AppDatabase.memory();
```

This provides:

- Fast test execution
- No file system side effects
- Clean state for each test
- Real SQL query execution

---

# Mocking Strategy

## What to Mock

| Dependency | Mock? | Reason |
|---|---|---|
| Repository interfaces | Yes | Isolate use cases from data layer |
| Use cases | Yes | Isolate ViewModels from business logic |
| Navigation | Yes | Verify navigation without real routing |
| Secure storage | Yes | Avoid real keychain access in tests |
| File system | Yes | Avoid real file operations |

## What NOT to Mock

| Dependency | Mock? | Reason |
|---|---|---|
| Domain models | No | Test real behavior |
| drift database | No (integration) | Test real queries |
| Riverpod providers | No | Test real dependency resolution |

## Mock Generation

Use mockito or mocktail for mock generation.

```dart
@GenerateMocks([TransactionRepository])
import 'create_transaction_use_case_test.mocks.dart';
```

---

# CI Integration

## Test Pipeline

```text
PR Created
    │
    ▼
Run dart format --set-exit-if-changed
    │
    ▼
Run dart analyze
    │
    ▼
Run flutter test (unit + widget)
    │
    ▼
Run flutter test --coverage
    │
    ▼
Check coverage threshold
    │
    ▼
Build debug APK
    │
    ▼
Build debug IPA (if macOS runner available)
    │
    ▼
Report results
```

## Failure Policy

- Any test failure blocks the PR
- Coverage below threshold blocks the PR
- Format violations block the PR
- Analysis warnings block the PR

---

# Test Performance

## Targets

| Test Type | Target Time |
|---|---|
| Single unit test | < 100ms |
| Full unit test suite | < 30s |
| Single widget test | < 1s |
| Full widget test suite | < 60s |
| Integration test | < 30s each |
| Full test suite | < 5 minutes |

## Optimization Tips

- Use in-memory database for integration tests
- Avoid real timers and delays
- Use `pumpAndSettle` instead of arbitrary waits
- Share expensive setup across tests in a group
- Run tests in parallel where possible

---

# Definition of Done

Testing is complete when:

- All unit tests pass
- All widget tests pass
- All integration tests pass
- Coverage meets threshold (80% overall)
- CI pipeline passes
- No flaky tests exist
- Test data factories cover all entities
- Critical flows have integration tests

---

# Guiding Principle

Tests are not a burden — they are a safety net.

Every test written today prevents a bug tomorrow. Every test that passes today enables a confident refactor next month.

Write tests that you would want to read six months from now. Clear, focused and descriptive.

The goal is not 100% coverage. The goal is confidence that the application works as expected.
