# Summa

> A local-first, open-source personal finance platform built around privacy, ownership and flexibility.

---

# Table of Contents

- Introduction
- Why Summa?
- Vision
- Project Goals
- Core Features
- Development Philosophy
- Roadmap
- Target Audience
- Open Source
- Documentation Structure

---

# Introduction

Summa is an open-source, local-first personal finance platform designed to help individuals, families and small groups manage their finances while maintaining complete ownership over their data.

Unlike traditional finance applications, Summa does not require users to create an online account or upload sensitive financial information to third-party servers.

The application is fully functional offline and stores data locally by default. Users may optionally synchronize their data using either their own self-hosted server or the official Summa Cloud service.

The project focuses on three fundamental values:

- Privacy
- Simplicity
- Ownership

---

# Why Summa?

Modern finance applications usually require users to:

- create an account
- upload financial information
- trust proprietary cloud services
- pay subscriptions for essential functionality

Summa aims to solve these issues by offering a privacy-first alternative.

The goal is to give users complete control over:

- where their data is stored
- how their data is synchronized
- who has access to their finances

---

# Vision

Our vision is to build the most transparent and privacy-respecting personal finance platform available.

Every user should be able to choose how they use Summa:

- Completely offline
- Self-hosted
- Official Summa Cloud

without sacrificing features or usability.

---

# Project Goals

The primary goals of Summa are:

- Local-first architecture
- Offline functionality
- Beautiful native mobile applications
- Cross-device synchronization
- Open-source transparency
- Self-hosted infrastructure
- Optional cloud services
- Long-term maintainability

---

# Core Features

## Finance Tracking

- Income management
- Expense management
- Categories
- Budgets
- Statistics
- Monthly overview
- Yearly overview

---

## Household Support

Multiple local profiles

Examples:

- Personal
- Family
- Roommates
- Friends

Users can split expenses using multiple methods.

---

## Smart Features

Future versions will include:

- Receipt OCR
- IPS QR Scanner
- Bank statement import
- Automatic categorization
- Smart reminders
- Recurring transactions

---

## Data Ownership

Users always own their data.

Supported export formats include:

- JSON
- CSV
- Excel

Future versions may also support encrypted backups.

---

# Development Philosophy

Summa follows several architectural principles.

## Local First

The application must remain fully usable without internet access.

Internet should enhance the experience, never be required.

---

## User Owns the Data

Financial information belongs exclusively to the user.

Users must always be able to export their data.

---

## Privacy by Default

No advertisements.

No analytics that inspect financial information.

No hidden tracking.

---

## Open Source

Every line of code is publicly available.

Community contributions are encouraged.

---

## Cloud is Optional

Cloud synchronization is an optional convenience.

It is never a requirement.

---

# Roadmap

Development is divided into five phases.

## Phase 0

Architecture & Foundation

- Project planning
- Documentation
- Design System
- Database Design
- Security Model

---

## Phase 1

Local First MVP

- Profiles
- Categories
- Transactions
- Dashboard
- Statistics
- Local Backup

---

## Phase 2

Smart Features

- OCR
- IPS QR
- Bank Import
- Recurring Transactions
- Smart Categorization

---

## Phase 3

Self Hosted Platform

- Backend
- Synchronization
- Authentication
- Docker
- Admin Panel

---

## Phase 4

Summa Cloud

- Hosted synchronization
- Subscription
- Cloud Backup
- Push Notifications

---

# Target Audience

Summa is intended for:

- Individuals
- Families
- Couples
- Students
- Small households
- Privacy-conscious users
- Self-hosting enthusiasts

---

# Open Source

Summa is developed as an open-source project.

The community is encouraged to:

- report issues
- submit pull requests
- improve documentation
- suggest features
- contribute code

---

# Documentation

The documentation is organized into multiple sections.

```

docs/
│
├── 00_PROJECT_OVERVIEW.md
├── 01_VISION_AND_PRINCIPLES.md
├── 02_ROADMAP.md
├── 03_TECH_STACK.md
├── 04_PROJECT_STRUCTURE.md
│
├── phase-0/
├── phase-1/
├── phase-2/
├── phase-3/
└── phase-4/

```

Each phase contains detailed documentation covering architecture, implementation and development milestones.

---

# Current Status

🚧 Project Status

Planning & Architecture

The project is currently in the architecture and design phase.

Implementation has not yet started.

Our current priority is creating a solid foundation that will support long-term development.