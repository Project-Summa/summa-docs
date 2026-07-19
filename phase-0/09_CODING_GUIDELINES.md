# Coding Guidelines

> Defining shared engineering standards for the Summa codebase.

---

## Table of Contents

- [Purpose](#purpose)
- [General Principles](#general-principles)
- [Architecture Rules](#architecture-rules)
- [Naming](#naming)
- [Code Organization](#code-organization)
- [Functions and Classes](#functions-and-classes)
- [Error Handling](#error-handling)
- [Logging](#logging)
- [Validation](#validation)
- [Money and Currency](#money-and-currency)
- [Date and Time](#date-and-time)
- [Nullability](#nullability)
- [Comments and Documentation](#comments-and-documentation)
- [Testing](#testing)
- [Android Guidelines](#android-guidelines)
- [Jetpack Compose Guidelines](#jetpack-compose-guidelines)
- [iOS Guidelines](#ios-guidelines)
- [SwiftUI Guidelines](#swiftui-guidelines)
- [Backend Guidelines](#backend-guidelines)
- [Database Guidelines](#database-guidelines)
- [API Guidelines](#api-guidelines)
- [Performance](#performance)
- [Security and Privacy](#security-and-privacy)
- [Dependencies](#dependencies)
- [Formatting and Linting](#formatting-and-linting)
- [AI-Generated Code](#ai-generated-code)
- [Code Review Checklist](#code-review-checklist)
- [Definition of Done](#definition-of-done)
- [Guiding Principle](#guiding-principle)

---

## Purpose

This document defines coding standards across:

- Android
- iOS
- Backend
- Shared documentation
- Scripts
- Tests

The goal is not to enforce personal style preferences.

The goal is to make the codebase predictable, safe and maintainable.

---

## General Principles

### Prefer Clarity

Readable code is preferred over clever code.

A contributor should understand normal control flow without mentally decoding abstractions.

---

### Keep Changes Focused

A code change should solve one defined problem.

Avoid unrelated refactoring inside feature pull requests.

---

### Make Invalid States Difficult

Use types, validation and domain models to prevent invalid data.

---

### Avoid Premature Abstraction

Do not create abstractions only because they may be useful someday.

Introduce an abstraction when:

- Multiple implementations exist
- Testing requires replacement
- A clear architectural boundary exists
- Duplication represents the same concept

---

### Be Explicit

Prefer explicit dependencies, parameters and state transitions.

Avoid hidden global state.

---

### Fail Predictably

Errors should be modeled, logged safely and presented clearly.

---

## Architecture Rules

Allowed dependency direction:

```text
Presentation
    ↓
Domain
    ↓
Data interfaces
    ↓
Infrastructure implementations
```

The exact module structure may vary by platform, but the dependency direction must remain clear.

---

## Forbidden Dependencies

Avoid:

```text
UI → DAO
UI → HTTP client
ViewModel → Room DAO
Domain → Android framework
Domain → SwiftUI
Domain → PostgreSQL driver
Business logic → Logging implementation
```

---

## Domain Independence

Domain code should not depend on:

- Android SDK
- iOS frameworks
- HTTP
- Database drivers
- UI components
- Serialization formats where avoidable

Domain models represent product concepts, not transport or storage details.

---

## Naming

Names should describe purpose.

Good:

```text
CreateTransactionUseCase
TransactionRepository
calculateMonthlyBalance
workspaceId
```

Avoid:

```text
Manager
Helper
Utils
DataThing
process
handleStuff
temp
```

Generic names are acceptable only when the scope makes the meaning obvious.

---

## Boolean Naming

Boolean names should read like conditions.

Examples:

```text
isLoading
hasMore
canEdit
shouldRefresh
wasDeleted
```

Avoid:

```text
loadingFlag
editValue
check
```

---

## Collection Naming

Collections should use plural names.

Examples:

```text
transactions
categories
workspaceMembers
```

---

## Identifier Naming

Use:

```text
transactionId
workspaceId
profileId
```

Avoid generic:

```text
id
```

when multiple identifiers appear in the same scope.

---

## Code Organization

Files should have one primary responsibility.

A file may contain closely related private helpers, but unrelated public types should be separated.

Avoid files such as:

```text
Utils.kt
Helpers.swift
common.go
misc.ts
```

Create purpose-specific files instead.

---

## Functions and Classes

Functions should:

- Perform one coherent operation
- Have clear input and output
- Avoid hidden side effects
- Return modeled errors
- Remain short enough to understand

Large functions should be split by behavior, not arbitrary line count.

---

## Parameter Count

When a function requires many related parameters, consider a value object.

Example:

```kotlin
data class CreateTransactionCommand(
    val profileId: ProfileId,
    val categoryId: CategoryId,
    val amount: Money,
    val occurredAt: Instant,
    val note: String?
)
```

Do not use a parameter object only to hide poor design.

---

## Immutability

Prefer immutable values and state.

Examples:

- Kotlin `val`
- Swift `let`
- Immutable UI state
- Copy-based state updates
- Domain models without uncontrolled mutation

Mutation should remain localized and intentional.

---

## Error Handling

Errors should be modeled by category.

Possible categories:

```text
Validation
NotFound
Conflict
PermissionDenied
Authentication
Storage
Network
Unknown
```

Do not use exceptions for normal expected control flow when a typed result is clearer.

---

## Error Boundaries

Infrastructure errors should be translated before reaching presentation.

Example:

```text
SQLite exception
    ↓
Repository error
    ↓
Domain or application error
    ↓
User-friendly UI message
```

UI should not display raw exception messages.

---

## Exception Handling

Never silently swallow exceptions.

Avoid:

```kotlin
try {
    operation()
} catch (e: Exception) {
}
```

At minimum:

- Handle expected error
- Preserve useful context
- Log safely
- Return a modeled failure

---

## Logging

Logs should explain operational behavior without exposing user data.

Allowed examples:

```text
Database migration started
Sync batch failed with conflict
Attachment upload exceeded configured size
```

Avoid:

```text
User spent 4599 RSD at Merchant X
Transaction note: ...
Access token: ...
```

---

## Log Levels

```text
Debug    Development detail
Info     Important lifecycle event
Warning  Recoverable abnormal condition
Error    Failed operation requiring attention
```

Verbose debug logging should not be enabled in production by default.

---

## Validation

Validation belongs at multiple boundaries.

### UI Validation

Immediate feedback for:

- Empty required values
- Invalid local format
- Impossible amount
- Missing category

### Domain Validation

Business rules such as:

- Amount must be valid
- Currency must be supported
- Profile must belong to workspace
- Split totals must match transaction amount

### API Validation

All remote input must be validated again.

Never trust client validation.

---

## Money and Currency

Never use floating-point values for financial amounts.

Use integer minor units.

Example:

```text
125050 minor units = 1,250.50
```

Domain representation should combine:

```text
amountMinor
currency
```

Prefer a dedicated type.

Example:

```kotlin
data class Money(
    val amountMinor: Long,
    val currency: CurrencyCode
)
```

---

## Money Operations

Money operations must:

- Require compatible currency
- Define rounding explicitly
- Avoid implicit conversion
- Detect overflow where relevant
- Preserve exact split totals

---

## Expense Splitting

When a value cannot be divided evenly, the remainder policy must be deterministic.

Example:

```text
100 minor units split among 3 members
34 + 33 + 33
```

The allocation order must be stable and documented.

---

## Date and Time

Use distinct types for:

- Instant in time
- Local calendar date
- Month or accounting period
- Timezone

Do not represent all temporal values as untyped strings.

---

## Timestamp Rules

Stored and transmitted timestamps use UTC.

Display uses the user's timezone.

Examples:

```text
createdAt       Instant
updatedAt       Instant
occurredAt      Instant or explicit local financial date
billingMonth    YearMonth-like type
```

The exact transaction-date semantics must be consistent across platforms.

---

## Nullability

Null should represent a meaningful absence.

Avoid nullable values when a default or separate state is clearer.

Example:

```text
note = null
```

is reasonable.

Example:

```text
amount = null
```

inside a valid persisted transaction is not reasonable.

---

## Optional Updates

PATCH-like updates must distinguish:

- Value unchanged
- Value assigned
- Value cleared

A normal nullable type may not be sufficient for this distinction.

---

## Comments and Documentation

Comments should explain:

- Why
- Constraint
- Non-obvious tradeoff
- Security implication
- Compatibility requirement

Comments should not repeat code.

Avoid:

```kotlin
// Increase count by one
count++
```

Useful:

```kotlin
// Preserve stable allocation order so split results remain identical
// across Android, iOS and the backend.
```

---

## Public API Documentation

Public interfaces should document:

- Responsibility
- Parameters where non-obvious
- Return behavior
- Errors
- Threading or concurrency expectations
- Security implications
- Ownership expectations

---

## TODO Comments

TODO comments must include context.

Preferred:

```text
TODO(#142): Replace temporary parser after bank format samples are approved.
```

Avoid:

```text
TODO fix later
```

---

## Testing

Tests should validate behavior rather than implementation details.

A refactor that preserves behavior should not require rewriting most tests.

---

## Test Structure

Preferred pattern:

```text
Given
When
Then
```

Test names should describe behavior.

Example:

```text
creating expense with zero amount returns validation error
```

---

## Required Test Categories

### Unit Tests

- Domain rules
- Use cases
- View models
- Formatters
- Parsers
- Conflict rules

### Integration Tests

- Database
- Repositories
- Migrations
- API
- Storage
- Synchronization

### UI Tests

- Critical flows
- Validation
- Navigation
- Accessibility
- State restoration

### End-to-End Tests

- Backup and restore
- Multi-device sync
- Account lifecycle
- Workspace invitation

---

## Test Data

Test data must be fictional.

Never use:

- Real bank statements
- Real user emails
- Production database dumps
- Real access tokens
- Personal transaction history

---

## Android Guidelines

Android code uses:

- Kotlin
- Jetpack Compose
- Coroutines
- Flow
- Room
- Hilt
- DataStore

Follow official Kotlin style unless this document defines a stricter project rule.

---

## Kotlin Naming

```text
Class and interface    PascalCase
Function and property  camelCase
Constant               UPPER_SNAKE_CASE
Package                lowercase
```

---

## Kotlin Coroutines

Use structured concurrency.

Avoid:

```text
GlobalScope
unmanaged background jobs
blocking main thread
```

View models should launch work through their owned scope.

Repositories should expose suspend functions or Flow as appropriate.

---

## Kotlin Flow

Use Flow for observable streams.

Avoid collecting the same database stream repeatedly without reason.

Use `StateFlow` for UI state.

UI should collect state in a lifecycle-aware manner.

---

## Jetpack Compose Guidelines

Composable functions should:

- Be stateless where practical
- Receive state and event callbacks
- Avoid direct repository access
- Avoid starting uncontrolled work during composition
- Use stable keys in lazy collections
- Use design-system components
- Support previews where useful

---

## Compose Screen Pattern

Preferred:

```text
Route
    ↓
ViewModel
    ↓
Screen
    ↓
Content
    ↓
Reusable components
```

Example:

```kotlin
@Composable
fun DashboardRoute(
    viewModel: DashboardViewModel
) {
    val state by viewModel.state.collectAsStateWithLifecycle()

    DashboardScreen(
        state = state,
        onEvent = viewModel::onEvent
    )
}
```

---

## Compose State

Avoid passing the full ViewModel deep into the component tree.

Pass:

- State
- Specific callbacks
- Stable value objects

---

## Compose Side Effects

Use appropriate APIs:

```text
LaunchedEffect
DisposableEffect
SideEffect
rememberUpdatedState
```

Side effects must not run accidentally on every recomposition.

---

## Compose Preview

Reusable visual components should include previews for:

- Light theme
- Dark theme
- Empty state
- Long text
- Large font where practical

Preview data must remain fictional.

---

## iOS Guidelines

iOS uses:

- Swift
- SwiftUI
- Swift Concurrency
- SwiftData or approved persistence layer
- Swift Package Manager

Follow standard Swift naming conventions.

---

## Swift Concurrency

Prefer:

```text
async/await
Task
actors where shared mutable state requires isolation
```

Avoid blocking the main actor.

UI state changes must occur on the appropriate actor.

---

## SwiftUI Guidelines

Views should:

- Remain small and composable
- Receive state rather than fetch data directly
- Use observable view models
- Avoid business logic in `body`
- Use design tokens
- Support previews
- Respect Dynamic Type

---

## Backend Guidelines

Backend code uses Go.

Standard project rules:

- Use `gofmt`
- Return errors explicitly
- Wrap errors with context
- Keep packages focused
- Avoid global mutable state
- Pass `context.Context`
- Use dependency injection through constructors
- Keep HTTP transport separate from domain logic

---

## Go Error Handling

Errors should be checked immediately.

Avoid panic for normal runtime errors.

Panic may be acceptable only for unrecoverable startup configuration failures where continuing would be unsafe.

---

## Go Interfaces

Define interfaces near the consumer where practical.

Avoid large general-purpose interfaces.

Prefer:

```go
type TransactionReader interface {
    FindByID(ctx context.Context, id string) (Transaction, error)
}
```

over broad repositories containing unrelated operations.

---

## HTTP Handlers

Handlers should:

- Parse request
- Validate transport format
- Authenticate principal
- Call application service
- Map result to HTTP response

Handlers should not contain database logic.

---

## Database Guidelines

Database rules:

- Use migrations
- Never use destructive fallback in production
- Use transactions for multi-step writes
- Add foreign keys
- Add indexes based on query requirements
- Avoid `SELECT *` in long-lived backend queries
- Use explicit column lists
- Validate migration rollback or recovery plan

---

## SQL Naming

Use:

```text
snake_case
plural table names
_id suffix for foreign keys
_at suffix for timestamps
```

Examples:

```text
workspace_members
transaction_id
created_at
deleted_at
```

---

## API Guidelines

API transport models must remain separate from domain models where semantics differ.

Do not expose database rows directly as API responses.

API changes must update:

- OpenAPI
- Tests
- Documentation
- Compatibility notes

---

## Performance

Performance optimization should follow measurement.

However, obvious inefficiencies should be avoided.

Examples:

- Loading all transactions into memory
- Recalculating full history for every UI update
- Performing database work on the main thread
- Repeated network calls during recomposition
- Unbounded API responses
- N+1 database queries

---

## Pagination

Large datasets must use pagination or streaming.

Transaction history must not assume a small dataset.

---

## Security and Privacy

Code must not:

- Log financial content
- Store plaintext passwords
- Disable TLS validation in production
- Hardcode secrets
- Expose another workspace's data
- Trust client-provided authorization
- Use unvalidated file paths
- Include real data in fixtures

---

## Dependencies

Before adding a dependency, evaluate:

- Is it actively maintained?
- Is its license compatible?
- Is it necessary?
- Can the platform standard library solve the problem?
- What is its security history?
- Does it significantly increase application size?
- Can it be replaced?

Every dependency increases maintenance responsibility.

---

## Dependency Updates

Updates should:

- Be reviewed
- Pass tests
- Include migration notes where needed
- Avoid bundling unrelated major upgrades
- Be tested on supported platforms

Automated update pull requests must not be merged blindly.

---

## Formatting and Linting

### Android

```text
ktlint
Detekt
Android Lint
```

### iOS

```text
SwiftFormat or project-approved formatter
SwiftLint
```

### Backend

```text
gofmt
go vet
staticcheck
```

### Documentation

```text
Markdown lint
Link checker
```

Formatting should be automated in CI.

---

## AI-Generated Code

AI-generated code is allowed only when reviewed and understood.

The author must verify:

- Correctness
- Architecture
- Security
- Licensing
- Tests
- Error behavior
- Edge cases
- Performance

AI output should be treated like untrusted external code.

---

## AI Prompt Context

AI tools should receive relevant documentation.

Example:

```text
Implement the transaction creation use case according to:

docs/phase-0/01_ARCHITECTURE.md
docs/phase-0/02_DATABASE.md
docs/phase-0/03_MOBILE_ARCHITECTURE.md
docs/phase-0/09_CODING_GUIDELINES.md
```

Do not provide production secrets or personal financial data.

---

## Code Review Checklist

Before merge, verify:

- Solves the linked issue
- Follows architecture
- Contains no unrelated changes
- Handles errors
- Validates input
- Contains tests
- Uses design tokens
- Avoids sensitive logging
- Preserves compatibility
- Updates documentation
- Uses correct money representation
- Uses correct date representation
- Passes lint and formatting
- Contains no secrets
- Can be explained by the author

---

## Definition of Done

Coding guidelines are ready when:

- General principles are approved
- Architecture dependency rules are defined
- Naming standards are defined
- Money rules are defined
- Date and time rules are defined
- Error and logging rules are defined
- Android rules are defined
- iOS rules are defined
- Backend rules are defined
- Database rules are defined
- Test expectations are defined
- Linting tools are selected
- AI-generated code rules are defined

---

## Guiding Principle

The best Summa code should be straightforward to read, difficult to misuse and easy to test.

Consistency and correctness are more important than cleverness.