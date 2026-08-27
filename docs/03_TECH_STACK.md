# Technology Stack

> Technology choices, architecture decisions and development standards used throughout the Summa project.

---

# Table of Contents

- Purpose
- Design Principles
- High-Level Architecture
- Mobile Stack
- Backend Stack
- Database
- Synchronization
- Authentication
- OCR & Smart Features
- Development Tools
- Testing
- CI/CD
- Documentation
- Future Technologies
- Technology Decision Records

---

# Purpose

This document defines the technologies used across the Summa ecosystem.

The goal is to establish a stable and maintainable stack that prioritizes:

- Long-term support
- Maintainability
- Performance
- Developer experience
- Privacy
- Simplicity

Technology choices should not follow trends.

They should solve problems.

---

# Design Principles

Before selecting any technology, the following questions should always be answered.

- Does it improve maintainability?
- Does it improve performance?
- Does it reduce complexity?
- Is it actively maintained?
- Is it widely adopted?
- Can it be replaced in the future?

If the answer to multiple questions is "No", reconsider the choice.

---

# High-Level Architecture

```

Mobile (Flutter / Dart)
│
├── UI (Widgets)
├── State Management
├── Domain
├── Repository
├── Local Database
└── Sync Engine (Future)
│
REST API (Future)
│
Backend
│
PostgreSQL

```

---

# Mobile Stack

## Flutter

Language

- Dart

UI

- Flutter Widgets
- Material 3

Architecture

- MVVM (or BLoC / Riverpod, to be finalized)

State Management

- Riverpod (preferred)
- Alternatively BLoC

Navigation

- GoRouter

Dependency Injection

- Riverpod (built-in)
- Alternatively get_it

Database

- drift (type-safe SQLite for Dart)

Serialization

- json_serializable
- freezed

Networking (Future)

- dio

Image Loading

- cached_network_image

Preferences

- shared_preferences
- flutter_secure_storage (for sensitive data)

Logging

- logger
- talker

Minimum SDK

- Android 8.0 (API 26)
- iOS 16.0

---

# Backend Stack (Phase 3)

Status

Not implemented during MVP.

---

Preferred Language

Go

Reasons

- Excellent performance
- Small memory footprint
- Simple deployment
- Strong concurrency model
- Great Docker support
- Easy cross-compilation

Alternative Candidates

- Rust
- ASP.NET Core
- Node.js

---

Framework

Chi

Selected for its idiomatic Go design, lightweight middleware approach, excellent compatibility with `net/http` and strong community adoption.

Other candidates evaluated:

- Gin — rejected due to heavier abstraction over `net/http`
- Fiber — rejected due to `fasthttp` dependency limiting `net/http` ecosystem compatibility

---

API

REST API

Future consideration

gRPC for internal services if needed.

---

# Database

## Mobile

SQLite

Accessed using:

drift

---

## Server

PostgreSQL

Reasons

- Reliable
- Mature
- Excellent indexing
- JSON support
- Strong ecosystem

---

Migration Tool

To be decided.

Possible options

- Goose
- Atlas
- Flyway

---

# Synchronization

Current Status

Not available during Phase 1.

Future

Dedicated Sync Engine.

Responsibilities

- Conflict detection
- Conflict resolution
- Incremental sync
- Offline queue
- Retry logic

---

# Authentication

Phase 1

No authentication required.

Everything is local.

---

Phase 3

Authentication server.

Possible methods

- Email + Password
- Passkeys
- OAuth (future)

Session Tokens

JWT

Refresh Tokens

Supported.

---

# Security

Local Database

Encrypted (future)

Secure Storage

flutter_secure_storage

Uses platform-native secure storage under the hood:

- Android Keystore
- iOS Keychain

Transport

HTTPS only.

Passwords

Argon2id (preferred).

bcrypt remains acceptable where deployment constraints exist.

Never store plaintext passwords.

---

# OCR

Phase 2

Possible providers

Google ML Kit (via google_mlkit_text_recognition)

Advantages

- Offline
- Free
- Fast

Future

Optional AI-powered OCR improvements.

---

# QR Scanner

Library

mobile_scanner

Future

Native implementations if required.

---

# Bank Import

Supported Formats

- CSV
- Excel
- PDF

Implementation

Custom parser per supported bank.

---

# Development Tools

IDE

Android Studio (for emulator and platform tooling)

Code Editor

Visual Studio Code

Version Control

Git

Hosting

GitHub

Project Management

GitHub Projects

Issue Tracking

GitHub Issues

Design

Figma

Documentation

Markdown

---

# Testing

Unit Tests

flutter_test

Mocking

mockito

UI Tests

Flutter Widget Tests

Integration Tests

integration_test

Backend

Go testing

Future

End-to-end tests

Performance benchmarks

---

# Code Quality

Formatting

dart format

Static Analysis

dart analyze
custom analysis_options.yaml

Dependency Updates

Renovate (future)

---

# CI/CD

GitHub Actions

Pipeline

- Build
- Test
- Lint
- Static Analysis
- Release

Future

Automatic beta releases.

---

# Docker

Phase 3

Backend

Docker

Development

Docker Compose

Future

Kubernetes is not officially supported or required. Community self-managed Kubernetes deployments are not blocked, but no official Helm charts or manifests are provided.

---

# Documentation

Documentation is considered part of the product.

Documentation includes

- Architecture
- API
- Database
- Deployment
- Coding Guidelines
- ADRs
- Diagrams

---

# Future Technologies

Potential future additions

Desktop

Flutter Desktop (built-in cross-platform support)

Browser Extension

TBD

Plugin SDK

TBD

Public API

REST

GraphQL (under evaluation)

---

# Technology Decision Records

Every significant technology decision should be documented.

Examples

- Why Flutter?
- Why Go?
- Why PostgreSQL?
- Why REST?
- Why MVVM?
- Why Local First?

Future contributors should always understand the reasoning behind technical decisions.

---

# Guiding Principle

The technology stack should evolve carefully.

Changing technologies is acceptable.

Changing the project's philosophy is not.
