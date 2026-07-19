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

Android (Kotlin)
│
├── UI (Jetpack Compose)
├── ViewModels
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

## Android

Language

- Kotlin

UI

- Jetpack Compose
- Material 3

Architecture

- MVVM

State Management

- StateFlow
- Flow
- Coroutines

Navigation

- Navigation Compose

Dependency Injection

- Hilt

Database

- Room

Serialization

- kotlinx.serialization

Networking (Future)

- Ktor Client

Image Loading

- Coil

Preferences

- DataStore

Logging

- Timber

Minimum SDK

Android 8.0 (API 26)

---

## iOS

Language

- Swift

UI

- SwiftUI

Architecture

- MVVM

Persistence

- SwiftData (or Core Data if required)

Networking

- URLSession

Dependency Management

- Swift Package Manager

Minimum Version

iOS 17 (subject to change)

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

Decision pending.

Candidates

- Gin
- Fiber
- Chi

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

Room

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

Android

EncryptedSharedPreferences

iOS

Keychain

Transport

HTTPS only.

Passwords

Argon2id or bcrypt.

Never store plaintext passwords.

---

# OCR

Phase 2

Possible providers

Google ML Kit

Advantages

- Offline
- Free
- Fast

Future

Optional AI-powered OCR improvements.

---

# QR Scanner

Library

ZXing

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

Android Studio

Xcode

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

JUnit

Mocking

MockK

UI Tests

Compose Testing

Backend

Go testing

Future

Integration tests

End-to-end tests

Performance benchmarks

---

# Code Quality

Formatting

ktlint

Static Analysis

Detekt

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

Kubernetes support is not planned.

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

Compose Multiplatform

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

- Why Kotlin?
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