# Mobile Architecture

> Defining the architecture, structure and development guidelines for the Summa mobile application.

---

# Table of Contents

- Purpose
- Goals
- Supported Platforms
- Architecture Overview
- Design Principles
- Application Layers
- Feature-Based Structure
- Data Flow
- State Management
- Dependency Injection
- Navigation
- Error Handling
- Offline Strategy
- Performance
- Testing
- Future Expansion

---

# Purpose

This document defines the architecture used by the mobile application.

The architecture should remain:

- Predictable
- Testable
- Scalable
- Easy to understand
- Easy to maintain

The Flutter application follows a single codebase approach, sharing logic and UI across Android and iOS while respecting platform conventions where appropriate.

---

# Goals

The architecture should provide:

- Clear separation of responsibilities
- Easy feature development
- Modularity
- Offline-first behavior
- Testability
- Reusable UI components
- Future synchronization support

---

# Supported Platforms

## Android

Language

Dart

UI

Flutter Widgets

Architecture

MVVM

Minimum Version

Android 8 (API 26)

---

## iOS

Language

Dart

UI

Flutter Widgets

Architecture

MVVM

Minimum Version

iOS 16

---

# Architecture Overview

```

User

↓

Flutter Widget (Screen)

↓

ViewModel / StateNotifier

↓

Use Case

↓

Repository

↓

Local Database

↓

Repository

↓

ViewModel / StateNotifier

↓

Flutter Widget (Screen)

```

Every screen follows the exact same architecture.

---

# Design Principles

## Feature First

Features are isolated.

Each feature owns:

- UI (widgets)
- ViewModel / State Notifier
- Navigation
- State
- Events

---

## Unidirectional Data Flow

Data always flows in one direction.

```

User

↓

Event

↓

ViewModel

↓

State

↓

UI

```

UI never modifies state directly.

---

## Single Source of Truth

Only one source owns application data.

During Phase 1:

drift Database

Future:

drift + Sync Engine

---

## Immutable State

UI State should always be immutable.

State changes create a new state object.

Never mutate existing state.

Use freezed for immutable state classes.

---

# Application Layers

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

## Presentation

Contains:

- Flutter Widgets (Screens)
- ViewModels / State Notifiers
- Navigation
- UI State
- Events

Presentation never accesses the database directly.

---

## Domain

Contains:

- Models
- Use Cases
- Repository Interfaces

Pure Dart.

No Flutter framework dependencies.

---

## Data

Contains:

- Repository implementations
- drift DAOs
- Local Data Source
- Future Remote Data Source

Responsible for retrieving and saving data.

---

## Infrastructure

Contains:

- SQLite (via drift)
- flutter_secure_storage
- shared_preferences
- Networking
- Encryption
- Notifications

Implementation details live here.

---

# Feature Structure

Every feature follows the same structure.

```

features/

transactions/

ui/

components/

screens/

transaction_screen.dart

viewmodel/

transaction_view_model.dart

state/

transaction_state.dart

events/

transaction_event.dart

navigation/

transaction_navigation.dart

domain/

usecases/

repository/

data/

```

Each feature should remain independent.

---

# UI Architecture

Flutter widgets should remain stateless whenever possible.

Preferred structure:

```

Screen

↓

Content

↓

Section

↓

Component

```

Example:

DashboardScreen

↓

DashboardContent

↓

BalanceSection

↓

BalanceCard

This improves reuse and testing.

---

# ViewModels

Responsibilities:

- Receive UI events
- Execute Use Cases
- Update UI State
- Handle errors

ViewModels must not:

- Access DAOs directly
- Perform SQL queries
- Know UI implementation details

---

# Use Cases

Every business action becomes a Use Case.

Examples:

CreateTransactionUseCase

DeleteTransactionUseCase

UpdateTransactionUseCase

CalculateBalanceUseCase

ExportDataUseCase

The UI communicates only with Use Cases.

---

# Repository Pattern

Repositories hide implementation details.

Presentation never knows whether data comes from:

- drift
- REST API
- Cache
- Sync Engine

Future synchronization becomes transparent.

---

# State Management

Every screen exposes:

```

UiState

```

Example:

DashboardState

TransactionState

StatisticsState

SettingsState

UiState should contain:

- Loading
- Success
- Empty
- Error

State management uses Riverpod (preferred) or BLoC.

---

# Events

Every user interaction becomes an Event.

Example

```

DashboardEvent

```

Possible events:

Refresh

AddTransaction

DeleteTransaction

FilterChanged

Retry

Events are received only by ViewModels.

---

# Dependency Injection

Framework

Riverpod

Injection should be constructor-based whenever possible.

Avoid Service Locator patterns.

---

# Navigation

Navigation uses GoRouter.

Each feature owns its destination.

```

Dashboard

↓

Transactions

↓

Transaction Details

↓

Edit Transaction

```

Navigation logic should remain separate from UI logic.

---

# Error Handling

Errors should be categorized.

Examples:

Validation Error

Database Error

Unknown Error

Network Error (Future)

Errors should always provide meaningful messages.

The application should recover whenever possible.

---

# Offline Strategy

Offline is the default mode.

Every feature must work without internet.

Future synchronization should happen silently in the background.

Loss of connectivity should never interrupt normal usage.

---

# Performance

Guidelines:

Avoid unnecessary rebuilds.

Prefer ListView.builder for long lists.

Avoid blocking the main isolate.

Use const widgets where possible.

Move heavy work to compute() or background isolates.

Use immutable collections.

---

# Accessibility

Every screen should support:

- Dynamic font sizes
- Screen readers (Semantics)
- Sufficient contrast
- Large touch targets

Accessibility is a requirement, not an enhancement.

---

# Animations

Animations should be subtle.

Preferred:

- Fade
- Slide
- Scale

Avoid excessive motion.

Animations should communicate state changes.

---

# Theming

Material 3

Supports:

- Light Theme
- Dark Theme
- Dynamic Colors (Android 12+)

All colors come from the Design System.

Hardcoded colors are forbidden.

---

# Logging

Logging should be centralized.

Development:

Verbose logging.

Production:

Only warnings and errors.

Sensitive information must never be logged.

---

# Testing

Every layer should be testable.

Presentation

Widget Tests

Domain

Unit Tests

Repositories

Integration Tests

---

# Future Expansion

The architecture should support:

- OCR

- QR Scanner

- Bank Import

- Sync Engine

- Cloud

without restructuring existing features.

---

# Guiding Principle

The mobile architecture should remain boring.

Developers should immediately understand how every screen works without reading additional documentation.

Consistency is more valuable than cleverness.
