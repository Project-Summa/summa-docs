# Phase 1 - Local First MVP

> Building the first usable Summa application.

---

# Table of Contents

- Introduction
- Purpose
- Why Phase 1 Matters
- Objectives
- Scope
- Deliverables
- Phase Structure
- Exit Criteria
- Success Metrics
- Dependencies
- Guiding Principle
- Next Phase

---

# Introduction

Phase 1 transforms the architectural foundation defined in Phase 0 into a working application.

For the first time, Summa becomes something a user can install, open and use to manage their personal finances — entirely offline, entirely on their device.

No internet connection is required. No account is needed. No data leaves the device.

This phase produces the Minimum Viable Product that validates Summa's core value proposition: a privacy-first finance tracker that works without cloud dependency.

---

# Purpose

The purpose of Phase 1 is to prove that the architecture works in practice.

Phase 0 defined the blueprint. Phase 1 builds the house.

Every architectural decision — the layered structure, the repository pattern, the drift database, the MVVM approach, the design system — gets validated through real implementation. If something doesn't work, it gets fixed here, before later phases build on top of it.

---

# Why Phase 1 Matters

Phase 1 is the first phase that produces user-facing value.

Everything before this was preparation. Everything after this depends on what gets built here.

The quality of Phase 1 determines:

- Whether the architecture holds under real implementation pressure
- Whether the codebase remains maintainable as features grow
- Whether the design system produces a coherent user experience
- Whether the database schema supports real financial workflows
- Whether the testing strategy catches real bugs
- Whether future phases can build confidently on this foundation

---

# Objectives

The primary objectives of Phase 1 are:

- Build a complete offline finance tracking application
- Validate the Phase 0 architecture through implementation
- Implement all core financial entities (profiles, categories, transactions)
- Create a functional dashboard with balance and summaries
- Provide basic statistics and reporting
- Implement data export and local backup
- Add application security (lock screen, biometrics)
- Establish the testing and release pipeline
- Prepare for Phase 2 smart features

---

# Scope

Phase 1 focuses exclusively on local-only functionality.

Included:

- Flutter application for Android and iOS
- Local SQLite database via drift
- Profile and workspace management
- Category management (built-in and custom)
- Transaction tracking (expense, income, transfer)
- Dashboard with balance and recent activity
- Monthly and yearly statistics
- Data export (JSON, CSV)
- Local backup and restore
- Application lock (PIN and biometrics)
- Theme selection (light and dark)
- Currency and language settings
- Design system implementation
- Unit, widget and integration tests
- CI/CD pipeline
- Store preparation

Excluded:

- Internet connectivity
- User accounts
- Synchronization
- Backend
- OCR and receipt scanning
- Bank statement import
- Smart categorization
- Recurring transactions
- Reminders
- Push notifications
- Cloud backup
- Multi-device support
- Desktop applications

---

# Deliverables

At the end of Phase 1 the following should exist.

## Application

- Installable Flutter application
- Working on Android 8+ (API 26)
- Working on iOS 16+
- All core features functional
- No internet permission required

---

## Code

- Flutter project in `summa-mobile`
- Complete feature implementations
- Unit tests for all use cases
- Widget tests for all screens
- Integration tests for critical flows
- Linting and formatting configuration

---

## Documentation

- Updated architecture docs if implementation reveals changes
- API documentation for local data layer
- User-facing feature documentation
- Contributing guide updates

---

## Infrastructure

- CI/CD pipeline for mobile
- Automated testing on every PR
- Debug and release build configurations
- Store listing preparation

---

# Phase Structure

Phase 1 consists of the following documents.

```

phase-1/

00_README.md

01_IMPLEMENTATION_PLAN.md

02_MOBILE_IMPLEMENTATION.md

03_DATABASE_IMPLEMENTATION.md

04_FEATURE_SPECIFICATIONS.md

05_SECURITY_IMPLEMENTATION.md

06_DESIGN_IMPLEMENTATION.md

07_TESTING_STRATEGY.md

08_RELEASE_PLAN.md

09_MILESTONES.md

```

Each document focuses on one aspect of the Phase 1 implementation.

---

# Exit Criteria

Phase 1 is complete when:

- A user can create and manage multiple profiles
- A user can create, edit and delete transactions
- A user can organize transactions with categories
- A user can view their balance and recent activity on a dashboard
- A user can view monthly and yearly spending statistics
- A user can export their data as JSON or CSV
- A user can create and restore local backups
- A user can secure the app with PIN or biometrics
- A user can switch between light and dark themes
- All critical flows have automated tests
- The application builds and passes CI on both platforms
- No known crash bugs exist
- The application works entirely offline

---

# Success Metrics

Phase 1 is considered successful if:

- A user can completely replace a spreadsheet with Summa
- The application feels native on both Android and iOS
- The codebase remains navigable after all features are implemented
- New contributors can understand the feature structure within one hour
- Test coverage exceeds 80% for domain and data layers
- The application launches in under 2 seconds on mid-range devices
- No financial data is lost under any normal usage scenario

---

# Dependencies

Phase 1 depends on:

- Phase 0 documentation being complete and reviewed
- Flutter development environment being configured
- Design system tokens being finalized
- CI/CD pipeline being operational
- Both Android and iOS build targets being validated

No external services or APIs are required.

---

# Guiding Principle

Phase 1 is the foundation that every future feature builds upon.

Every shortcut taken here becomes technical debt that compounds across every subsequent phase.

Invest in correctness, testability and clarity — even when it feels slower.

The application should work perfectly offline before we ever think about going online.

---

# Next Phase

After Phase 1 has been completed, development continues with:

Phase 2 — Smart Features

The smart features phase adds OCR, bank import, smart categorization and recurring transactions — reducing manual data entry while maintaining the local-first architecture.
