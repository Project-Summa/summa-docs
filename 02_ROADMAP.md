# Product Roadmap

> A long-term development roadmap for Summa.

---

# Table of Contents

- Purpose
- Roadmap Philosophy
- Development Timeline
- Phase Overview
- Phase 0 - Foundation
- Phase 1 - Local First MVP
- Phase 2 - Smart Features
- Phase 3 - Self Hosted Platform
- Phase 4 - Summa Cloud
- Future Ideas
- Release Strategy

---

# Purpose

This document defines the long-term development roadmap for Summa.

Instead of focusing on release dates, the roadmap is milestone-driven. Each phase represents a complete set of features and architectural improvements that build upon previous work.

A phase is considered complete only when all of its goals have been achieved and validated.

---

# Roadmap Philosophy

Summa follows several development principles.

- Build a strong foundation before adding features.
- Avoid premature optimization.
- Prioritize maintainability over speed.
- Every phase must result in a usable product.
- New features should never compromise existing functionality.
- Local-first remains the highest priority throughout development.

---

# Development Timeline

```
Phase 0
Architecture & Planning
        │
        ▼
Phase 1
Local First MVP
        │
        ▼
Phase 2
Smart Features
        │
        ▼
Phase 3
Self Hosted Platform
        │
        ▼
Phase 4
Summa Cloud
```

---

# Phase Overview

| Phase | Name | Goal |
|--------|------|------|
| 0 | Foundation | Define the architecture and standards |
| 1 | Local First MVP | Fully functional offline finance application |
| 2 | Smart Features | Automation and intelligent workflows |
| 3 | Self Hosted Platform | Multi-device synchronization and self-hosting |
| 4 | Summa Cloud | Hosted synchronization and subscription service |

---

# Phase 0 — Foundation

## Objective

Design the project before writing production code.

The goal of this phase is to establish a maintainable architecture that can support future growth.

---

## Deliverables

- Documentation
- Mobile architecture
- Backend architecture
- Database design
- API specification
- Security model
- Design system
- Branding
- Repository structure
- Git workflow
- Coding guidelines

---

## Definition of Done

Phase 0 is complete when:

- All architecture decisions are documented.
- Database schema is finalized.
- Design system is approved.
- Development workflow is defined.
- Repository structure is complete.
- Every future phase has a clear implementation plan.

---

## Success Criteria

Developers should be able to start implementing features without making major architectural decisions.

---

# Phase 1 — Local First MVP

## Objective

Build a complete personal finance application that works entirely offline.

No internet connection should be required.

---

## Features

### Profiles

- Multiple local profiles
- Personal mode
- Household mode
- Shared expenses

---

### Transactions

- Income
- Expenses
- Transfers
- Notes

---

### Categories

- Built-in categories
- Custom categories
- Icons
- Colors

---

### Dashboard

- Current balance
- Recent transactions
- Monthly summary

---

### Statistics

- Monthly reports
- Yearly reports
- Spending by category
- Spending trends

---

### Export

- JSON
- CSV
- Excel

---

### Backup

- Local backup
- Restore from backup

---

### Settings

- Theme
- Currency
- Language
- Biometrics

---

## Definition of Done

The application can be used daily without internet access.

---

## Success Criteria

A user can completely replace a spreadsheet with Summa.

---

# Phase 2 — Smart Features

## Objective

Reduce manual work through automation.

---

## Features

### Receipt OCR

- Scan receipts
- Detect merchant
- Detect total amount
- Detect VAT
- Detect date

---

### IPS QR Scanner

- Scan payment slips
- Create reminders
- Mark as paid

---

### Bank Statement Import

Supported formats:

- PDF
- CSV
- Excel

Supported banks will be added gradually.

---

### Smart Categorization

Automatically suggest categories based on previous transactions.

---

### Recurring Transactions

Support recurring:

- Salaries
- Rent
- Utilities
- Subscriptions

---

### Reminders

- Bills
- Subscriptions
- Loans
- Insurance

---

## Definition of Done

Users spend significantly less time entering financial data manually.

---

# Phase 3 — Self Hosted Platform

## Objective

Allow users to synchronize data while keeping full ownership.

---

## Features

### Backend

- REST API
- Synchronization engine
- Authentication
- User management

---

### Docker

Official Docker image.

Official Docker Compose configuration.

---

### Database

PostgreSQL

---

### Synchronization

- Android
- iOS
- Desktop (future)

---

### Shared Workspaces

- Families
- Couples
- Roommates

---

### Roles

- Owner
- Admin
- Member
- Guest

---

### Administration

Web administration panel

- Users
- Storage
- Logs
- Updates
- Backups

---

## Definition of Done

Users can synchronize multiple devices without relying on Summa Cloud.

---

# Phase 4 — Summa Cloud

## Objective

Provide a managed cloud experience using the exact same backend developed during Phase 3.

---

## Subscription

Subscription covers:

- Hosting
- Automatic backups
- Cloud synchronization
- Push notifications

Features remain available in self-hosted deployments.

---

## Features

- Hosted infrastructure
- Automatic updates
- Disaster recovery
- Monitoring
- Account management

---

## Definition of Done

Users can create an account and synchronize data without managing their own server.

---

# Future Ideas

These ideas are intentionally excluded from the current roadmap.

Possible future additions include:

- Desktop applications
- Browser extension
- Open Plugin API
- Budget forecasting
- AI-powered financial insights
- Investment tracking
- Cryptocurrency portfolio tracking
- Shared community templates

---

# Release Strategy

Development follows milestone-based releases.

Major releases:

- v1.x → Local First
- v2.x → Smart Features
- v3.x → Self Hosted Platform
- v4.x → Summa Cloud

Patch releases focus on:

- Bug fixes
- Performance
- Stability
- Security

---

# Roadmap Principles

The roadmap is a living document.

Features may move between phases if doing so improves the long-term quality of the project.

However, the following principles must never change:

- Local First
- Privacy by Default
- User Owns the Data
- Cloud is Optional
- Open Source