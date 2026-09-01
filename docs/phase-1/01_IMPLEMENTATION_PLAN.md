# Implementation Plan

> Defining the implementation strategy, order and key decisions for Phase 1.

---

# Table of Contents

- Purpose
- Implementation Philosophy
- Implementation Order
- Dependency Map
- Key Decisions
- Feature Priorities
- Risk Management
- Development Workflow
- Definition of Done
- Guiding Principle

---

# Purpose

This document defines how Phase 1 features will be implemented.

It establishes the implementation order, identifies dependencies between features and records key decisions that affect the build sequence.

The goal is to ensure that every feature builds on a stable foundation and that no feature blocks another unnecessarily.

---

# Implementation Philosophy

## Build Bottom-Up

Start with the data layer. Build up to the UI.

Every feature should be fully functional at the data and domain layers before the UI is implemented.

This ensures:

- Testability from the start
- Clean separation of concerns
- UI independence from data sources
- Easier future synchronization integration

---

## One Feature at a Time

Complete one feature fully before starting the next.

A "complete" feature includes:

- Domain models
- Use cases
- Repository implementation
- drift DAOs
- ViewModel
- UI screens
- Unit tests
- Widget tests

Partial features create integration risk and complicate testing.

---

## Vertical Slices

Each feature is a vertical slice through all layers.

```

UI → ViewModel → Use Case → Repository → DAO → Database

```

Implementing vertical slices ensures every layer works together before moving to the next feature.

---

# Implementation Order

The recommended implementation order is based on dependency analysis.

## Phase 1A — Foundation

These must be implemented first. Everything else depends on them.

```text
1. Flutter project setup and configuration
2. Design token implementation
3. Database foundation (drift setup, migrations)
4. Core domain models
5. Navigation skeleton (GoRouter)
6. Riverpod provider infrastructure
```

---

## Phase 1B — Core Entities

These are the building blocks of all financial features.

```text
7. Workspace (auto-created, invisible to user)
8. Profile management (create, edit, select)
9. Category management (seed defaults, create, edit, archive)
```

---

## Phase 1C — Financial Features

These depend on profiles and categories being complete.

```text
10. Transaction creation (expense, income)
11. Transaction editing and deletion
12. Transaction listing and filtering
13. Transaction details view
```

---

## Phase 1D — Insights

These depend on transaction data being available.

```text
14. Dashboard (balance, recent transactions, monthly summary)
15. Statistics (monthly totals, category breakdown, trends)
```

---

## Phase 1E — Data Management

These depend on a stable database with real data.

```text
16. Export (JSON, CSV)
17. Backup and restore
```

---

## Phase 1F — Security and Settings

These can be implemented in parallel with other features but should be complete before release.

```text
18. Application lock (PIN)
19. Biometric authentication
20. Settings (theme, currency, language)
21. Data deletion
```

---

## Phase 1G — Quality and Release

These happen after all features are complete.

```text
22. Integration tests for critical flows
23. Accessibility review
24. Performance optimization
25. Security review
26. Store assets and listing
27. Beta testing
28. Crash stabilization
```

---

# Dependency Map

```text
Flutter Project Setup
    │
    ├── Design Tokens
    │       │
    │       └── All UI Components
    │
    ├── Database Foundation
    │       │
    │       ├── Core Domain Models
    │       │       │
    │       │       ├── Workspace
    │       │       │       │
    │       │       │       ├── Profiles
    │       │       │       │       │
    │       │       │       │       ├── Categories
    │       │       │       │       │       │
    │       │       │       │       │       ├── Transactions
    │       │       │       │       │       │       │
    │       │       │       │       │       │       ├── Dashboard
    │       │       │       │       │       │       │
    │       │       │       │       │       │       ├── Statistics
    │       │       │       │       │       │       │
    │       │       │       │       │       │       ├── Export
    │       │       │       │       │       │       │
    │       │       │       │       │       │       └── Backup
    │       │       │       │       │       │
    │       │       │       │       │       └── Transaction Splits
    │       │       │       │       │
    │       │       │       │       └── Settings
    │       │       │       │
    │       │       │       └── Application Lock
    │       │       │
    │       │       └── Navigation
    │       │
    │       └── Riverpod Providers
    │
    └── CI/CD Pipeline
```

---

# Key Decisions

## Decision 1 — Application ID

| Option | Value |
|---|---|
| Android | `com.projectsumma.summa` |
| iOS | `com.projectsumma.summa` |

Status: Open (pending confirmation)

---

## Decision 2 — Minimum SDK Versions

| Platform | Version | Reason |
|---|---|---|
| Android | API 26 (Android 8) | Covers 95%+ of active devices |
| iOS | 16.0 | Covers current and recent devices |

Status: Resolved (documented in Phase 0)

---

## Decision 3 — State Management

| Option | Selected |
|---|---|
| Riverpod | Yes |
| BLoC | No |

Reason: Riverpod provides better testability, simpler provider definitions and compile-time safety.

Status: Resolved (documented in Phase 0)

---

## Decision 4 — Navigation

| Option | Selected |
|---|---|
| GoRouter | Yes |
| Navigator 2.0 | No |
| auto_route | No |

Reason: GoRouter is the recommended Flutter routing solution with declarative routing and deep link support.

Status: Resolved (documented in Phase 0)

---

## Decision 5 — Database

| Option | Selected |
|---|---|
| drift (SQLite) | Yes |
| sqflite | No |
| floor | No |
| isar | No |

Reason: drift provides type-safe queries, migration support and code generation.

Status: Resolved (documented in Phase 0)

---

## Decision 6 — Immutable State

| Option | Selected |
|---|---|
| freezed | Yes |
| built_value | No |
| Manual | No |

Reason: freezed provides concise immutable class definitions with copyWith, JSON serialization and union types.

Status: Resolved

---

## Decision 7 — Budget Feature in MVP

| Option | Selected |
|---|---|
| Include budgets | Review |
| Defer to Phase 2 | Review |

Reason: Budgets add complexity but are a core finance feature. Decision depends on Phase 1 timeline.

Status: Open (decision before Phase 1 backlog freeze)

---

## Decision 8 — Transaction Splits in MVP

| Option | Selected |
|---|---|
| Include splits | Review |
| Defer to Phase 2 | Review |

Reason: Splits are important for household mode but add significant UI complexity.

Status: Open (decision before transaction schema implementation)

---

## Decision 9 — Local Database Encryption

| Option | Selected |
|---|---|
| Encrypt in Phase 1 | Review |
| Defer to Phase 2 | Review |

Reason: Encryption adds security but complicates backup and restore. May be deferred if it risks the MVP timeline.

Status: Open (decision before public beta)

---

## Decision 10 — Supported Currencies

| Option | Selected |
|---|---|
| All ISO 4217 | Yes |
| Popular subset | No |

Reason: Supporting all currencies is straightforward with proper formatting libraries.

Status: Resolved

---

## Decision 11 — Supported Languages

| Option | Selected |
|---|---|
| English only (MVP) | Yes |
| English + Serbian | Review |
| Full i18n | No |

Reason: Localization infrastructure should exist but full translation can be deferred.

Status: Open (decision before Phase 1 backlog freeze)

---

## Decision 12 — Backup Format

| Option | Selected |
|---|---|
| JSON archive | Yes |
| SQLite file copy | Yes |
| Encrypted archive | Review |

Reason: JSON is human-readable and portable. SQLite copy is fast and lossless. Encryption depends on Decision 9.

Status: Resolved (encryption status open)

---

# Feature Priorities

## Must Have (P0)

These features define the MVP. Without them, Phase 1 is not complete.

- Profile creation and selection
- Category management
- Transaction creation (expense, income)
- Transaction editing and deletion
- Dashboard with balance
- Monthly statistics
- JSON export
- Local backup and restore
- Application lock

---

## Should Have (P1)

These features significantly improve the experience but are not strictly required for the MVP.

- Transfer transactions
- Transaction notes and merchant fields
- Category icons and colors
- Yearly statistics
- CSV export
- Dark theme
- Biometric authentication
- Spending trends

---

## Nice to Have (P2)

These features can be deferred to a later release if the timeline is tight.

- Transaction splits (shared expenses)
- Budget tracking
- Dynamic colors (Android 12+)
- Excel export
- Advanced filtering
- Accessibility enhancements beyond basics

---

# Risk Management

## Risk 1 — Architecture Doesn't Hold

Probability: Low

Impact: High

Mitigation: Implement the simplest feature (profiles) first to validate the full vertical slice before committing to all features.

---

## Risk 2 — drift Complexity

Probability: Medium

Impact: Medium

Mitigation: Start with simple queries. Add complexity gradually. Keep DAOs focused on single entities.

---

## Risk 3 — Design System Gaps

Probability: Medium

Impact: Low

Mitigation: Implement design tokens first. Fill component gaps as features require them. Don't block feature work on design perfection.

---

## Risk 4 — Scope Creep

Probability: High

Impact: High

Mitigation: Strict adherence to P0/P1/P2 priorities. Defer P2 features without guilt. Ship the MVP, iterate later.

---

## Risk 5 — Testing Bottleneck

Probability: Medium

Impact: Medium

Mitigation: Write tests alongside implementation, not after. Use the testing strategy defined in 07_TESTING_STRATEGY.md from the start.

---

# Development Workflow

## Daily Workflow

```text
1. Pick next feature from implementation order
2. Create feature branch from develop
3. Implement domain models and use cases
4. Write unit tests
5. Implement repository and DAO
6. Write integration tests
7. Implement ViewModel and state
8. Implement UI
9. Write widget tests
10. Submit PR for review
11. Merge after approval and CI passes
```

---

## Branch Strategy

```text
master        → stable releases
develop       → integration branch
feature/*     → individual features
fix/*         → bug fixes
hotfix/*      → critical production fixes
```

---

## Commit Convention

```text
feat(transactions): add transaction creation screen
fix(database): correct migration for version 2
test(categories): add unit tests for category repository
docs(phase-1): update implementation plan
```

---

# Definition of Done

A feature is done when:

- Domain models are defined and tested
- Use cases are implemented and tested
- Repository interface exists in domain layer
- Repository implementation exists in data layer
- drift DAOs are implemented and tested
- ViewModel handles all events correctly
- UI renders all states (loading, content, empty, error)
- Widget tests cover the screen
- Code follows coding guidelines
- No linting warnings exist
- PR is reviewed and approved

---

# Guiding Principle

The implementation plan is a guide, not a contract.

If implementation reveals that the order should change, update this document and explain why.

The goal is a working application, not adherence to a plan.
