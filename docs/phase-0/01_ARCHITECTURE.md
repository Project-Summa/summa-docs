# System Architecture

> Defining the architectural foundation of the Summa ecosystem.

---

# Table of Contents

- Purpose
- Architectural Goals
- High-Level Overview
- Design Principles
- System Components
- Data Flow
- Application Layers
- Future Expansion
- Architecture Decisions
- Risks
- Guiding Principles

---

# Purpose

This document describes the overall architecture of the Summa platform.

Its purpose is to define how different parts of the project interact while remaining scalable, maintainable and easy to understand.

Every future implementation should follow the architecture described here.

---

# Architectural Goals

The architecture should achieve the following goals.

- Local First
- Offline Support
- Scalability
- Maintainability
- Testability
- Security
- Platform Independence
- Modular Development
- Easy Synchronization
- Future Cloud Compatibility

---

# High-Level Overview

During Phase 1 the architecture is intentionally simple.

```

+-----------------------+
|   Mobile App (Flutter)|
+-----------------------+
            │
            ▼
+-----------------------+
|      ViewModels       |
+-----------------------+
            │
            ▼
+-----------------------+
|      Repository       |
+-----------------------+
            │
            ▼
+-----------------------+
|     Local Storage     |
|    drift / SQLite     |
+-----------------------+

```

The application is completely self-contained.

No internet connection is required.

---

# Future Architecture

During later phases, synchronization is introduced.

```

            Mobile (Flutter)

                  │

                  ▼

            Repository

                  │

                  ▼

            Sync Engine

          ┌──────────────┐

          │              │

          ▼              ▼

     Local SQLite     REST API

                            │

                            ▼

                       Backend

                            │

                            ▼

                      PostgreSQL

```

The application should never depend on the server.

The server enhances the experience.

It never becomes a requirement.

---

# Core Design Principles

## Local First

Every feature should work without internet.

Synchronization is optional.

---

## Modular

Each feature should be isolated.

Features should communicate through well-defined interfaces.

---

## Dependency Inversion

Higher-level modules should never depend on implementation details.

Repositories expose interfaces.

Implementations remain replaceable.

---

## Single Responsibility

Every class should have one responsibility.

Every module should solve one problem.

---

## Separation of Concerns

Business logic should never be mixed with UI.

Networking should never leak into presentation.

Database logic should never appear inside screens.

---

# Data Ownership Model

## Local-Only Mode (Phase 1)

In local-only mode, the application creates a single **Workspace** automatically on first launch. The user creates one or more **Profiles** within that Workspace to represent themselves, their family or their household.

The user does not interact with the Workspace concept directly. All user-visible organizational boundaries are expressed through Profiles.

```
Device
  └── Workspace (auto-created, invisible to user)
        ├── Profile: Personal
        ├── Profile: Family
        └── Profile: Household
```

---

## Synchronization Mode (Phase 3+)

When synchronization is enabled, the existing local Workspace is registered with the server. It becomes a shared Workspace that can be accessed from multiple devices.

Each human user has one **Account** (identified by email). An Account can be a member of multiple Workspaces. Within each Workspace, the Account has a **Profile**.

```
Account: user@example.com
  ├── Workspace: Personal Finances
  │     └── Profile: Me
  ├── Workspace: Family Budget
  │     ├── Profile: Me
  │     └── Profile: Partner
  └── Workspace: Roommates
        ├── Profile: Me
        └── Profile: Roommate A
```

---

## Migration Path

When a user upgrades from local-only to sync-enabled:

1. The existing local Workspace is preserved unchanged.
2. A server-side Workspace is created to mirror the local one.
3. All existing data is pushed to the server as the initial sync.
4. No local data is lost or restructured.
5. Profiles remain as-is; they gain server-side Account associations.

This migration is non-destructive and reversible through local backup.

---

# System Components

The project consists of several independent components.

```

Mobile App (Flutter)

↓

Sync Engine

↓

Backend

↓

Database

↓

Cloud Infrastructure

```

Each component has a clear responsibility.

---

# Mobile Application

Responsibilities:

- UI
- Local storage
- Business logic
- Notifications
- Biometrics
- Offline mode

The mobile application is built with Flutter and targets both Android and iOS from a single codebase.

It should remain fully functional independently.

---

# Backend

Introduced in Phase 3.

Responsibilities include:

- Synchronization
- Authentication
- User Management
- Shared Workspaces
- Notifications
- Backups

The backend should never contain business logic that prevents offline usage.

---

# Database

Mobile

SQLite

Server

PostgreSQL

Both databases should use similar entity naming where practical.

---

# Sync Engine

The Sync Engine acts as an intermediary.

Responsibilities:

- Detect local changes
- Upload changes
- Download changes
- Resolve conflicts
- Retry failed requests
- Queue offline operations

The Sync Engine should never directly modify UI state.

---

# Data Flow

Every feature follows the same data flow.

```

User

↓

UI

↓

ViewModel

↓

Use Case

↓

Repository

↓

Local Database

↓

Repository

↓

ViewModel

↓

UI

```

Future synchronization happens independently.

```

Local Database

↓

Sync Queue

↓

REST API

↓

Backend

↓

Database

```

The UI never communicates directly with the backend.

---

# Layered Architecture

```

Presentation

↓

Domain

↓

Data

↓

Infrastructure

```

---

## Presentation Layer

Contains:

- Flutter Widgets
- ViewModels
- Navigation
- UI State

No database access.

No networking.

---

## Domain Layer

Contains:

- Business Models
- Use Cases
- Repository Interfaces

Pure Dart.

No Flutter framework dependencies.

---

## Data Layer

Contains:

- Repository Implementations
- drift DAOs
- Data Sources
- Mappers

Responsible for accessing data.

---

## Infrastructure Layer

Contains:

- Local Storage
- Networking
- Encryption
- Logging

Low-level implementation details.

---

# Feature-Based Structure

Every feature should remain self-contained.

Example

```

features/

dashboard/

transactions/

categories/

statistics/

settings/

scanner/

profiles/

```

Inside every feature:

```

ui/

viewmodel/

navigation/

components/

state/

events/

```

This keeps the project modular.

---

# Communication Rules

Allowed

```

Presentation

↓

Domain

↓

Data

↓

Infrastructure

```

Forbidden

```

UI

↓

Database

```

or

```

ViewModel

↓

drift DAO

```

Everything passes through repositories.

---

# Offline Strategy

Offline is the default operating mode.

Synchronization is treated as a background enhancement.

Loss of internet must never prevent users from:

- Viewing transactions
- Creating transactions
- Editing transactions
- Exporting data
- Viewing statistics

---

# Error Handling

Errors should never crash the application.

Instead:

Recover if possible.

Inform the user.

Log useful information.

Continue operating.

---

# Scalability

The architecture should support future additions.

Examples:

- Desktop App

- Web Dashboard

- Plugin System

- AI Features

- Public API

without major restructuring.

---

# Security Boundaries

Sensitive data should never leave the device unless synchronization is enabled.

The application should assume:

Local database is trusted.

Network is untrusted.

External services are optional.

---

# Future Expansion

The architecture should support:

Phase 2

- OCR
- Bank Import
- Smart Categorization

Phase 3

- Sync Engine
- Backend
- Docker

Phase 4

- Cloud Infrastructure
- Billing
- Hosted Services

without changing existing application layers.

---

# Architecture Decisions

Every significant architectural decision should be documented as an ADR.

Examples include:

- Why Flutter?
- Why MVVM?
- Why Repository Pattern?
- Why drift?
- Why SQLite?
- Why Go?
- Why REST?

---

# Risks

Potential risks include:

- Overengineering
- Excessive abstraction
- Tight coupling
- Premature optimization

The architecture should remain as simple as possible while supporting future growth.

---

# Guiding Principle

The architecture exists to support the product.

It should evolve carefully, prioritize clarity over complexity and always reinforce Summa's core principles:

- Local First
- User Owns the Data
- Privacy by Default
- Cloud is Optional
- Open Source