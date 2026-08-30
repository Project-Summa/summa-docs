# Phase 1 Milestones

> Defining the milestones, checklists and exit criteria for the Phase 1 Local First MVP.

---

# Table of Contents

- Purpose
- Phase 1 Objective
- Milestone Overview
- Milestone 1.1 — Project Foundation
- Milestone 1.2 — Data Layer
- Milestone 1.3 — Profile and Category Management
- Milestone 1.4 — Transaction Management
- Milestone 1.5 — Dashboard and Statistics
- Milestone 1.6 — Data Management
- Milestone 1.7 — Security and Settings
- Milestone 1.8 — Quality and Release
- Open Decisions
- Phase 1 Deliverables
- Phase 1 Exit Criteria
- Phase 2 Handoff
- Definition of Done

---

# Purpose

This document converts Phase 1 implementation into concrete milestones and completion criteria.

Phase 1 is not complete when features are implemented.

It is complete when:

- All features work correctly on both platforms
- Tests cover critical flows
- The application is stable enough for daily use
- The release pipeline is operational
- No known data loss scenarios exist

---

# Phase 1 Objective

The objective of Phase 1 is:

```text
Build a complete personal finance application that works entirely offline,
replacing spreadsheets and manual tracking for everyday financial management.
```

---

# Milestone Overview

| Milestone | Name | Primary Result |
|---|---|---|
| 1.1 | Project Foundation | Working Flutter project with infrastructure |
| 1.2 | Data Layer | Database, DAOs and repositories operational |
| 1.3 | Profile and Category Management | Users can manage profiles and categories |
| 1.4 | Transaction Management | Users can create, edit and delete transactions |
| 1.5 | Dashboard and Statistics | Users can view balance and spending insights |
| 1.6 | Data Management | Users can export and backup their data |
| 1.7 | Security and Settings | Users can secure and customize the app |
| 1.8 | Quality and Release | Application is tested, stable and released |

---

# Milestone 1.1 — Project Foundation

## Objective

Establish the Flutter project with all infrastructure in place.

---

## Required Work

```text
Flutter project creation and configuration
Riverpod setup and provider infrastructure
GoRouter navigation skeleton
drift database setup
Design token implementation
Material 3 theme configuration
CI/CD pipeline setup
Linting and formatting configuration
```

---

## Checklist

- [ ] Flutter project created in `summa-mobile`
- [ ] Application ID configured (Android and iOS)
- [ ] Minimum SDK versions set (Android API 26, iOS 16)
- [ ] Riverpod configured with provider structure
- [ ] GoRouter configured with route definitions
- [ ] drift database class created
- [ ] Design tokens implemented (colors, typography, spacing, radius)
- [ ] Material 3 light theme configured
- [ ] Material 3 dark theme configured
- [ ] Core component library started (button, input, card, list item)
- [ ] Screen state components created (loading, empty, error)
- [ ] CI pipeline runs dart format, dart analyze and flutter test
- [ ] Linting rules configured in analysis_options.yaml
- [ ] Project builds successfully on Android
- [ ] Project builds successfully on iOS
- [ ] Basic navigation works between placeholder screens

---

## Exit Criteria

Milestone 1.1 is complete when a developer can run the application on both platforms, navigate between screens and see themed UI components.

---

# Milestone 1.2 — Data Layer

## Objective

Implement the complete data layer with database, DAOs and repositories.

---

## Required Work

```text
drift table definitions for all Phase 1 entities
DAO implementations for each entity
Repository interfaces in domain layer
Repository implementations in data layer
Database migrations setup
Seed data for default categories
In-memory database for testing
```

---

## Checklist

- [ ] Workspace table defined in drift
- [ ] Profile table defined in drift
- [ ] Category table defined in drift
- [ ] Transaction table defined in drift
- [ ] TransactionSplit table defined in drift
- [ ] Attachment table defined in drift
- [ ] Common sync fields present on all tables
- [ ] Workspace DAO implemented
- [ ] Profile DAO implemented
- [ ] Category DAO implemented
- [ ] Transaction DAO implemented
- [ ] TransactionSplit DAO implemented
- [ ] Attachment DAO implemented
- [ ] Repository interfaces defined in domain layer
- [ ] Repository implementations in data layer
- [ ] Default categories seeded on first launch
- [ ] Auto-created workspace on first launch
- [ ] Database migrations configured
- [ ] Indexes created for common queries
- [ ] In-memory database available for tests
- [ ] All DAOs have unit tests

---

## Exit Criteria

Milestone 1.2 is complete when all entities can be created, read, updated and soft-deleted through the repository layer.

---

# Milestone 1.3 — Profile and Category Management

## Objective

Allow users to create and manage profiles and categories.

---

## Required Work

```text
Profile creation, editing and selection
Category creation, editing and archiving
Default category seeding
Icon and color selection for categories
Profile type selection (personal/household)
```

---

## Checklist

### Profiles

- [ ] Create profile screen
- [ ] Edit profile screen
- [ ] Profile selection (switch active profile)
- [ ] Profile type: personal
- [ ] Profile type: household
- [ ] Default profile on first launch
- [ ] Profile currency selection
- [ ] Profile name validation
- [ ] ViewModel handles all profile events
- [ ] Unit tests for profile use cases
- [ ] Widget tests for profile screens

### Categories

- [ ] Category list screen
- [ ] Create category screen
- [ ] Edit category screen
- [ ] Archive category
- [ ] Category type: expense
- [ ] Category type: income
- [ ] Icon selection
- [ ] Color selection
- [ ] Default categories seeded
- [ ] Category name validation
- [ ] ViewModel handles all category events
- [ ] Unit tests for category use cases
- [ ] Widget tests for category screens

---

## Exit Criteria

Milestone 1.3 is complete when a user can create multiple profiles, switch between them, create custom categories and organize them with icons and colors.

---

# Milestone 1.4 — Transaction Management

## Objective

Allow users to create, edit, delete and view transactions.

---

## Required Work

```text
Transaction creation (expense, income, transfer)
Transaction editing
Transaction deletion (soft delete)
Transaction list with filtering
Transaction details view
Category assignment
Profile assignment
Date selection
Amount validation
Notes and merchant fields
```

---

## Checklist

- [ ] Create expense transaction
- [ ] Create income transaction
- [ ] Create transfer transaction
- [ ] Edit transaction
- [ ] Delete transaction (soft delete)
- [ ] Transaction list screen
- [ ] Filter by category
- [ ] Filter by date range
- [ ] Filter by transaction type
- [ ] Transaction details screen
- [ ] Category selection
- [ ] Profile selection
- [ ] Date picker
- [ ] Amount input with validation
- [ ] Note field
- [ ] Merchant field
- [ ] Amount stored as integer minor units
- [ ] Currency from active profile
- [ ] Transaction list grouped by date
- [ ] ViewModel handles all transaction events
- [ ] Unit tests for transaction use cases
- [ ] Widget tests for transaction screens
- [ ] Integration test: create → view → edit → delete

---

## Exit Criteria

Milestone 1.4 is complete when a user can record their daily financial activity with expenses, income and transfers, organized by categories and profiles.

---

# Milestone 1.5 — Dashboard and Statistics

## Objective

Provide users with financial insights through dashboard and statistics.

---

## Required Work

```text
Dashboard with balance, recent transactions and monthly summary
Statistics with monthly totals, category breakdown and trends
Empty states for new users
Loading and error states
```

---

## Checklist

### Dashboard

- [ ] Current balance display
- [ ] Monthly income total
- [ ] Monthly expense total
- [ ] Recent transactions list (last 5-10)
- [ ] Category summary for current month
- [ ] Empty state when no transactions
- [ ] Loading state on initial load
- [ ] Error state with retry
- [ ] Pull to refresh
- [ ] Navigate to transaction details
- [ ] Navigate to create transaction
- [ ] ViewModel handles dashboard events
- [ ] Unit tests for balance calculation
- [ ] Widget tests for dashboard screen

### Statistics

- [ ] Monthly totals (income vs expense)
- [ ] Category breakdown
- [ ] Date range selector (month/year)
- [ ] Spending by category chart
- [ ] Income vs expense comparison
- [ ] Empty state when no data
- [ ] Accessible chart summaries (text alternatives)
- [ ] ViewModel handles statistics events
- [ ] Unit tests for statistics calculations
- [ ] Widget tests for statistics screen

---

## Exit Criteria

Milestone 1.5 is complete when a user can open the app and immediately see their financial position, recent activity and spending patterns.

---

# Milestone 1.6 — Data Management

## Objective

Allow users to export and backup their financial data.

---

## Required Work

```text
JSON export
CSV export
Local backup creation
Backup restore
Backup validation
User confirmation flows
```

---

## Checklist

### Export

- [ ] JSON export with all data
- [ ] CSV export for transactions
- [ ] Export includes all profiles
- [ ] Export includes all categories
- [ ] Export includes all transactions
- [ ] Export file naming convention
- [ ] Share exported file
- [ ] Unit tests for export format

### Backup and Restore

- [ ] Create local backup
- [ ] Backup includes database
- [ ] Backup includes attachments
- [ ] Backup file naming convention
- [ ] List available backups
- [ ] Restore from backup
- [ ] Backup validation (check integrity)
- [ ] User confirmation before restore
- [ ] Warning about data replacement
- [ ] Restore failure recovery
- [ ] Unit tests for backup format
- [ ] Integration test: backup → delete → restore

---

## Exit Criteria

Milestone 1.6 is complete when a user can export their data in standard formats and create/restore backups to protect against data loss.

---

# Milestone 1.7 — Security and Settings

## Objective

Allow users to secure the application and customize their experience.

---

## Required Work

```text
Application lock (PIN)
Biometric authentication
Theme selection (light/dark)
Currency settings
Language settings (if applicable)
Data deletion
```

---

## Checklist

### Application Lock

- [ ] PIN setup flow
- [ ] PIN change flow
- [ ] PIN disable flow
- [ ] PIN verification on app launch
- [ ] PIN verification on resume (after timeout)
- [ ] Lockout after failed attempts
- [ ] Biometric enable flow
- [ ] Biometric authentication
- [ ] Biometric fallback to PIN
- [ ] Unit tests for PIN validation
- [ ] Unit tests for lockout policy
- [ ] Integration test: setup → lock → unlock

### Settings

- [ ] Settings screen
- [ ] Theme selection (light/dark/system)
- [ ] Currency selection
- [ ] Language selection (if applicable)
- [ ] Security settings section
- [ ] Data section (export, backup, delete)
- [ ] About section (version, links)
- [ ] Data deletion with confirmation
- [ ] Data deletion requires PIN verification
- [ ] Widget tests for settings screen

---

## Exit Criteria

Milestone 1.7 is complete when a user can secure the app with PIN or biometrics, customize the theme and manage their data.

---

# Milestone 1.8 — Quality and Release

## Objective

Ensure the application is stable, tested and ready for release.

---

## Required Work

```text
Integration tests for critical flows
Accessibility review
Performance optimization
Security review
Store assets preparation
Beta testing
Crash stabilization
Release notes
```

---

## Checklist

### Testing

- [ ] All unit tests pass
- [ ] All widget tests pass
- [ ] All integration tests pass
- [ ] Coverage meets threshold (80% overall)
- [ ] No flaky tests
- [ ] Tested on Android 8 (API 26)
- [ ] Tested on latest Android
- [ ] Tested on iOS 16
- [ ] Tested on latest iOS
- [ ] Tested on small screen (iPhone SE)
- [ ] Tested on large screen (tablet)

### Quality

- [ ] No crash bugs
- [ ] No data loss scenarios
- [ ] Performance acceptable on mid-range devices
- [ ] App launches in under 2 seconds
- [ ] Scrolling is smooth (60fps)
- [ ] No memory leaks
- [ ] Accessibility review passed
- [ ] Security review passed

### Release

- [ ] App icon finalized
- [ ] Screenshots captured (4-8 per platform)
- [ ] Store description written
- [ ] Privacy policy published
- [ ] Google Play listing created
- [ ] App Store listing created
- [ ] Beta testing completed
- [ ] Crash reporting active
- [ ] Release notes written
- [ ] Version 1.0.0 tagged

---

## Exit Criteria

Milestone 1.8 is complete when the application is available on both stores, crash reporting is active and no critical bugs are known.

---

# Open Decisions

The following decisions should be resolved before their respective milestones.

| Decision | Status | Deadline | Milestone |
|---|---|---|---|
| Flutter application ID | Open | Before project creation | 1.1 |
| Default categories list | Open | Before seed data | 1.2 |
| Transaction splits in MVP | Review | Before transaction schema | 1.4 |
| Budget feature in MVP | Review | Before Phase 1 backlog freeze | 1.4 |
| Supported languages | Open | Before settings implementation | 1.7 |
| Local database encryption | Review | Before public beta | 1.7 |
| Beta tester recruitment | Open | Before closed beta | 1.8 |
| Store developer accounts | Open | Before store submission | 1.8 |

---

# Phase 1 Deliverables

At completion, the repository should contain:

```text
summa-mobile/
├── lib/
│   ├── app/
│   ├── core/
│   │   ├── design/
│   │   ├── database/
│   │   ├── navigation/
│   │   └── providers/
│   ├── features/
│   │   ├── dashboard/
│   │   ├── transactions/
│   │   ├── categories/
│   │   ├── profiles/
│   │   ├── statistics/
│   │   ├── export/
│   │   ├── backup/
│   │   ├── settings/
│   │   └── security/
│   ├── data/
│   ├── domain/
│   └── main.dart
├── test/
├── test_widget/
├── integration_test/
├── android/
├── ios/
├── pubspec.yaml
├── analysis_options.yaml
└── README.md
```

---

# Phase 1 Exit Criteria

Phase 1 is complete only when:

- [ ] All Phase 1 documents exist and are reviewed
- [ ] All P0 features are implemented and tested
- [ ] Application works on Android 8+ and iOS 16+
- [ ] No crash bugs exist
- [ ] No data loss scenarios exist
- [ ] Application lock works reliably
- [ ] Backup and restore works reliably
- [ ] Export produces valid JSON and CSV
- [ ] Test coverage meets threshold
- [ ] CI pipeline passes
- [ ] Beta testing is completed
- [ ] Store listings are live
- [ ] Crash reporting is active
- [ ] Release notes are published

---

# Phase 2 Handoff

Phase 2 begins with:

```text
Smart Features
```

Recommended implementation order:

```text
1. Receipt OCR (Google ML Kit)
2. IPS QR scanner
3. Bank statement import (PDF, CSV, Excel)
4. Smart categorization
5. Recurring transactions
6. Reminders
```

Phase 2 depends on:

- Stable transaction data model
- Working category system
- Reliable local database
- Established testing patterns

---

# Definition of Done

Phase 1 is officially complete when a user can:

1. Open the app for the first time
2. Create a profile
3. Record daily expenses and income
4. Organize transactions with categories
5. View their balance and spending trends
6. Export their data
7. Create and restore backups
8. Secure the app with PIN or biometrics
9. Use the app daily without issues

All of this — without an internet connection, without an account, without their data leaving the device.
