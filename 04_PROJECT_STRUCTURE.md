# Project Structure

> Repository organization, directory layout and development workflow.

---

# Table of Contents

- Purpose
- Repository Strategy
- Monorepo Structure
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

Summa uses a monorepo.

All applications and services live inside the same Git repository.

Reasons:

- Easier dependency management
- Shared documentation
- Shared branding
- Shared issue tracking
- Easier project navigation
- Better CI/CD
- Single source of truth

---

# Repository Structure

```

summa/

├── .github/
│
├── android/
│
├── ios/
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

## android/

Native Android application.

Technology:

- Kotlin
- Jetpack Compose
- Room

Contains:

- UI
- ViewModels
- Domain
- Data Layer
- Resources
- Tests

---

## ios/

Native iOS application.

Technology:

- Swift
- SwiftUI

Contains:

- Views
- Models
- Persistence
- Resources
- Tests

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

# Android Structure

```

android/

app/

src/

main/

kotlin/

com/

summa/

core/

data/

domain/

features/

navigation/

ui/

MainActivity.kt

```

---

# Android Modules

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

- Room
- DAO
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

The domain layer must not depend on Android.

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

- UI
- ViewModel
- State
- Events

---

## navigation

Navigation graph.

Destination definitions.

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

Use descriptive names.

Good

```

DashboardScreen.kt

TransactionRepository.kt

StatisticsViewModel.kt

```

Avoid

```

Screen.kt

Utils.kt

Helper.kt

```

---

## Repository Structure

- `summa` — central documentation, architecture and governance
- `summa-android` — native Android application
- `summa-ios` — native iOS application
- `summa-backend` — optional open-source synchronization backend
- `summa-website` — official website and documentation portal
- `.github` — organization-wide GitHub configuration
- `summa-cloud` — private infrastructure and operations for Summa Cloud

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

---

# Guiding Principle

The project structure should remain predictable.

Consistency is more valuable than clever organization.

Every contributor should immediately understand where new code belongs.