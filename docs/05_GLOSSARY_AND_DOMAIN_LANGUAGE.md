# Glossary and Domain Language

> Defining shared terminology used throughout the Summa project.

---

## Table of Contents

- [Purpose](#purpose)
- [Core Domain Terms](#core-domain-terms)
- [Financial Terms](#financial-terms)
- [Technical Terms](#technical-terms)
- [Synchronization Terms](#synchronization-terms)
- [Security Terms](#security-terms)
- [Deployment Terms](#deployment-terms)
- [Phase-Specific Terms](#phase-specific-terms)

---

## Purpose

This document defines the shared vocabulary used across Summa documentation, code, design and communication.

Every contributor should use these terms consistently.

When a term has multiple interpretations, the Summa-specific meaning takes precedence.

---

## Core Domain Terms

### Workspace

A Workspace is the top-level organizational unit in Summa. Every financial entity (Transaction, Category, Budget, Attachment) belongs to exactly one Workspace.

In local-only mode (Phase 1), the application creates a single default Workspace automatically. The user does not interact with the Workspace concept directly.

When synchronization is introduced (Phase 3), a Workspace can be shared across multiple devices and among multiple users.

**Scope boundary:** All data isolation happens at the Workspace level. One Workspace cannot access data from another Workspace.

---

### Profile

A Profile represents one member identity within a Workspace.

In local-only mode, the user creates Profiles to represent themselves, their family members or their household. A Profile has a display name, a type (personal or household) and a preferred currency.

When synchronization is enabled, a Profile is associated with a server-side Account. Each Account may be a member of multiple Workspaces, with a separate Profile in each.

**Relationship:** A Workspace contains one or more Profiles. A Profile belongs to exactly one Workspace.

---

### Account

An Account represents a server-side user identity. Accounts exist only when synchronization is enabled (Phase 3+).

An Account is identified by email and authenticated with a password or passkey. An Account can be a member of multiple Workspaces.

**Relationship:** An Account has many Workspace memberships. Each membership has one Profile.

---

### Category

A Category classifies a Transaction as either `expense` or `income`.

Categories are scoped to a Workspace. Default categories are seeded when a Workspace is created. Users can create custom categories with a name, icon and color.

Categories can be filtered by type and sorted by user-defined order.

---

### Transaction

A Transaction is the central financial record in Summa. It represents a single financial event: an expense, an income or a transfer.

Every Transaction has an amount (stored as integer minor units), a currency, a type, a date and an optional category and note.

---

### Transaction Split

A Transaction Split represents one participant's share of a shared expense.

When a Transaction is split among multiple Profiles, each split record stores the portion allocated to one participant. The sum of all splits must equal the Transaction amount.

---

### Budget

A Budget defines a spending limit for a Category or for all spending within a Profile. Budgets have a period (weekly, monthly or yearly) and an amount in minor units.

---

### Attachment

An Attachment is a file linked to a Transaction. Common examples include receipt images, invoices and PDFs.

Attachments are stored locally and optionally synchronized with the server in later phases.

---

## Financial Terms

### Minor Units

The smallest unit of a currency, used to avoid floating-point arithmetic. For example, `125050` minor units of USD represent `$1,250.50`. All monetary amounts in Summa are stored and transmitted as integer minor units.

---

### Currency Code

An ISO 4217 three-letter code representing a currency. Examples: `USD`, `EUR`, `RSD`, `GBP`.

---

### Balance

The sum of all Transaction amounts for a given period and scope. Balance is always calculated, never stored directly.

---

### Income

A Transaction of type `income`. Represents money received.

---

### Expense

A Transaction of type `expense`. Represents money spent.

---

### Transfer

A Transaction of type `transfer`. Represents money moved between accounts or categories. Transfers are neutral to the overall balance calculation.

---

## Technical Terms

### Soft Delete

A deletion strategy where records are never permanently removed immediately. Instead, the `deleted_at` column receives a timestamp. The record is excluded from normal queries but remains in the database for synchronization and recovery purposes.

---

### Optimistic Concurrency

A conflict prevention strategy where each record has a `version` number. When updating a record, the client sends the version it read. If the server's version differs, the update is rejected, signaling a conflict.

---

### Design Token

A named, reusable visual value (color, spacing, typography, radius, elevation) defined in the design system. Design tokens ensure consistency across platforms and between design and implementation.

---

### Common Fields

The set of fields present on every entity: `id`, `created_at`, `updated_at`, `deleted_at`, `version`, `sync_status`, `device_id`. These fields enable soft deletion, optimistic concurrency and future synchronization.

---

## Synchronization Terms

### Sync Engine

The component responsible for exchanging data changes between the local database and the server. The Sync Engine handles push (uploading local changes) and pull (downloading remote changes), conflict detection and resolution.

---

### Sync Status

A field on every entity indicating its synchronization state:

| Value | Meaning |
|---|---|
| `local` | Exists only on this device, not yet queued for sync |
| `pending` | Modified locally, waiting to be pushed to the server |
| `synced` | Successfully synchronized with the server |
| `conflict` | A conflict was detected and requires resolution |

---

### Tombstone

A soft-deleted record retained specifically for synchronization purposes. When a record is deleted on one device, the tombstone propagates to other devices so they know to remove the record locally.

---

### Cursor

A server-provided opaque token used to paginate through sync results. A cursor marks the client's position in the change stream, enabling incremental synchronization.

---

### Conflict

A situation where the same record was modified on two different devices between synchronization cycles. Conflicts must be detected and resolved by the Sync Engine or presented to the user for manual resolution.

---

### Idempotency

The property that repeating the same request produces the same result. All Summa API operations must be idempotent to safely handle network retries without creating duplicate records.

---

### Idempotency Key

A client-generated unique identifier attached to a write request. The server uses this key to detect and reject duplicate requests.

---

## Security Terms

### Application Lock

A local security feature that requires the user to authenticate (PIN, pattern, password or biometric) before accessing the application.

---

### Biometric Prompt

A system-provided authentication dialog using fingerprint, face recognition or other biometric methods.

---

### Session

A server-side representation of an authenticated device session. A session has an access token, a refresh token, an expiration and a device identifier.

---

### Access Token

A short-lived JWT used to authenticate API requests. Included in the `Authorization` header.

---

### Refresh Token

A long-lived, opaque token used to obtain a new access token without re-authentication. Refresh tokens are rotated on each use and bound to a specific device.

---

### Device Revocation

The process of invalidating all tokens associated with a device, immediately ending its session.

---

### Workspace Member

A user who has been granted access to a Workspace. Each member has a Role that defines their permissions.

---

### Role

A permission set assigned to a Workspace Member. Roles include `owner`, `admin`, `editor` and `viewer`.

---

## Deployment Terms

### Local-Only Mode

The default deployment mode in Phase 1. All data is stored on the device. No server is required. No account is needed.

---

### Self-Hosted Mode

A deployment mode (Phase 3) where the user runs the [backend](https://github.com/Project-Summa/summa) (`summa`) on their own infrastructure. The user controls their data and server configuration.

---

### Summa Cloud

A managed deployment mode (Phase 4) where the Summa team operates the backend infrastructure. Users create accounts and subscribe for managed synchronization and backup.

---

### Attachment Root

The base directory on the server filesystem where user-uploaded files are stored. Every file path is prefixed with the workspace identifier to prevent cross-workspace access.

---

## Phase-Specific Terms

### Phase 1 — Local First MVP

The first implementation phase. Delivers a fully functional local-only Android application with profiles, categories, transactions, dashboard, statistics, export and settings. No backend or synchronization.

---

### Phase 2 — Intelligence

Adds smart features to the local application: OCR receipt scanning, bank statement import, recurring transactions and improved statistics.

---

### Phase 3 — Synchronization

Introduces the open-source backend ([`summa`](https://github.com/Project-Summa/summa)), user accounts, device registration, real-time synchronization, shared workspaces, push notifications and self-hosting support.

---

### Phase 4 — Summa Cloud

Introduces managed cloud hosting, subscriptions, multi-device management, admin console and enterprise features.

---

## Guiding Principle

Shared language reduces misunderstanding.

When a term is used in documentation, code or design, it should carry the same meaning everywhere.

When a new term is introduced, it should be added to this document.