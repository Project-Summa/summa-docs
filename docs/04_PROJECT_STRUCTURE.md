# Project Structure

> Repository organization, directory layout and development workflow.

---

# Table of Contents

- Purpose
- Repository Strategy
- Multi-Repo Structure
- Directory Responsibilities
- Naming Conventions
- Branch Strategy
- Commit Convention
- Pull Requests
- Versioning
- Development Workflow
- Future Expansion

---

# Purpose

A consistent project structure makes development easier, improves onboarding for new contributors and keeps the codebase maintainable over many years.

Every file should have an obvious location.

Developers should never have to guess where new code belongs.

---

# Repository Strategy

Summa uses a multi-repo strategy.

Each major component lives in its own Git repository under the Project Summa organization.

Reasons:

- Independent release cycles per platform
- Clear ownership boundaries
- Smaller, focused repositories
- Platform-specific CI/CD without cross-contamination
- Independent contributor access control
- Reduced clone size for single-platform contributors

Shared concerns (documentation, branding, GitHub configuration) are maintained in the central `summa` repository.

---

# Repository Structure

```

summa/

├── .github/
│
├── mobile/
│
├── backend/
│
├── docs/
│
├── design/
│
├── branding/
│
├── scripts/
│
├── website/
│
├── LICENSE
├── README.md
└── CONTRIBUTING.md

```

---

# Directory Responsibilities

## .github/

Contains:

- GitHub Actions
- Issue templates
- Pull Request templates
- Discussion templates

---

## mobile/

Cross-platform mobile application built with Flutter.

Technology:

- Dart
- Flutter
- drift

Contains:

- lib/ (application source code)
- test/ (unit and widget tests)
- integration_test/ (integration tests)
- assets/ (fonts, images, animations)

---

## backend/

Self-host synchronization server.

Contains:

- REST API
- Authentication
- Sync Engine
- Database
- Docker configuration

---

## docs/

Project documentation.

Includes:

- Architecture
- Roadmap
- Security
- API
- ADRs
- Design System

---

## design/

Design assets.

Examples:

- Figma exports
- Icons
- Illustrations
- Mockups

---

## branding/

Brand identity.

Includes:

- Logo
- Color palette
- Typography
- Brand guidelines

---

## scripts/

Development utilities.

Examples:

- Build scripts
- Release scripts
- Formatting scripts

---

## website/

Landing page.

Documentation website (future).

---

# Mobile Structure

```

mobile/

lib/

app/

core/

data/

domain/

features/

navigation/

ui/

main.dart

```

---

# Mobile Modules

## core

Shared application code.

Examples:

- Theme
- Utilities
- Extensions
- Constants

---

## data

Responsible for data access.

Contains:

- drift database
- DAOs
- Repository implementations
- Local storage
- Future networking

---

## domain

Business logic.

Contains:

- Models
- Use Cases
- Repository interfaces

Pure Dart.

No Flutter framework dependencies.

---

## features

Each feature has its own module.

Example:

```

features/

dashboard/

transactions/

categories/

statistics/

settings/

scanner/

```

Each feature contains:

- UI (widgets)
- ViewModel / State Notifier
- State
- Events

---

## navigation

Navigation configuration.

Route definitions.

Deep links.

---

## ui

Reusable UI components.

Examples:

- Buttons
- Cards
- Dialogs
- Bottom Navigation
- Charts

---

# Backend Structure

```

backend/

cmd/

internal/

pkg/

api/

config/

database/

docker/

migrations/

tests/

```

---

# Documentation Structure

```

docs/

00_PROJECT_OVERVIEW.md

01_VISION_AND_PRINCIPLES.md

02_ROADMAP.md

03_TECH_STACK.md

04_PROJECT_STRUCTURE.md

adr/

phase-0/

phase-1/

phase-2/

phase-3/

phase-4/

```

---

# Naming Conventions

Folders

lowercase

Examples

```

dashboard

transactions

statistics

```

---

Classes

PascalCase

Examples

```

TransactionRepository

DashboardViewModel

ExpenseCard

```

---

Functions

camelCase

Examples

```

loadTransactions()

calculateBalance()

createProfile()

```

---

Constants

UPPER_SNAKE_CASE

Example

```

DEFAULT_CURRENCY

MAX_PROFILES

```

---

Files

Use descriptive names in snake_case (Dart convention).

Good

```

dashboard_screen.dart

transaction_repository.dart

statistics_view_model.dart

```

Avoid

```

screen.dart

utils.dart

helper.dart

```

---

## Repository Inventory

| Repository | Description |
|---|---|
| `summa-docs` | Documentation, architecture decisions, design system, branding guidelines and project governance |
| `summa-mobile` | Cross-platform mobile application (Flutter, Dart, drift) |
| `summa` | Open-source synchronization backend (Go, PostgreSQL, Docker) |
| `summa-website` | Official website and documentation portal |
| `summa-cloud` | Private infrastructure and operational tooling for Summa Cloud |

# Branch Strategy

Main branches

```

main

develop

```

Feature branches

```

feature/dashboard

feature/statistics

feature/self-host

```

Bug fixes

```

fix/export-crash

fix/dashboard-loading

```

Documentation

```

docs/roadmap

docs/api

```

---

# Commit Convention

Format

```

type(scope): description

```

Examples

```

feat(dashboard): add balance card

fix(export): resolve CSV formatting

docs(roadmap): update phase 2

refactor(repository): simplify data layer

test(database): add migration tests

```

Commit Types

- feat
- fix
- docs
- refactor
- test
- perf
- ci
- chore

---

# Pull Requests

Every Pull Request should include:

- Summary
- Motivation
- Screenshots (if UI changes)
- Testing notes
- Linked issue

---

# Versioning

Semantic Versioning

```

Major.Minor.Patch

```

Examples

```

1.0.0

1.2.0

2.0.0

```

---

# Development Workflow

1. Create Issue

↓

2. Create Feature Branch

↓

3. Implement

↓

4. Test

↓

5. Open Pull Request

↓

6. Review

↓

7. Merge into develop

↓

8. Release to main

---

# Future Expansion

The repository structure should support future additions such as:

- Desktop applications
- Browser extension
- Plugin SDK
- CLI tools
- Public API SDKs

without requiring major restructuring.

Flutter's built-in desktop support simplifies future expansion to Windows, macOS and Linux.

---

# Guiding Principle

The project structure should remain predictable.

Consistency is more valuable than clever organization.

Every contributor should immediately understand where new code belongs.
