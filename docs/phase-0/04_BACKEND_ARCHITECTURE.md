# Backend Architecture

> Defining the architecture of the optional self-hosted synchronization server and the future Summa Cloud platform.

---

## Table of Contents

- [Purpose](#purpose)
- [Status](#status)
- [Backend Responsibilities](#backend-responsibilities)
- [Non-Responsibilities](#non-responsibilities)
- [Architectural Goals](#architectural-goals)
- [High-Level Architecture](#high-level-architecture)
- [Deployment Modes](#deployment-modes)
- [Architectural Style](#architectural-style)
- [Backend Components](#backend-components)
- [Request Flow](#request-flow)
- [Data Ownership Model](#data-ownership-model)
- [Workspace Model](#workspace-model)
- [Authentication and Authorization](#authentication-and-authorization)
- [Synchronization Architecture](#synchronization-architecture)
- [Conflict Resolution](#conflict-resolution)
- [Database Architecture](#database-architecture)
- [File and Attachment Storage](#file-and-attachment-storage)
- [Background Processing](#background-processing)
- [Notifications](#notifications)
- [Configuration](#configuration)
- [Self-Hosted Deployment](#self-hosted-deployment)
- [Summa Cloud Deployment](#summa-cloud-deployment)
- [Observability](#observability)
- [Security Boundaries](#security-boundaries)
- [Backup and Recovery](#backup-and-recovery)
- [API Versioning](#api-versioning)
- [Scalability](#scalability)
- [Testing Strategy](#testing-strategy)
- [Failure Handling](#failure-handling)
- [Development Environment](#development-environment)
- [Future Expansion](#future-expansion)
- [Risks](#risks)
- [Definition of Done](#definition-of-done)
- [Guiding Principle](#guiding-principle)

---

## Purpose

This document defines the architecture of the Summa backend.

The backend will not be required during the Local First MVP. It will be introduced during Phase 3 to provide optional synchronization between devices.

The same backend implementation should support two deployment modes:

1. Self-hosted Summa Server
2. Official Summa Cloud

The backend must remain optional.

A user who does not configure a server must still be able to use the mobile application locally without losing access to core functionality.

---

## Status

The backend is currently in the architecture and planning stage.

Implementation begins during:

```text
Phase 3 — Self Hosted Platform
```

Some backend-related concepts are documented during Phase 0 because the local database, entity identifiers, versioning strategy and synchronization fields must be designed in advance.

---

## Backend Responsibilities

The backend is responsible for:

- User authentication
- Device registration
- Multi-device synchronization
- Shared workspaces
- Workspace membership
- Invitation management
- Role-based authorization
- Synchronization conflict coordination
- Attachment synchronization
- Server-side backups
- Push notification coordination
- Audit events
- API access
- Managed cloud hosting in Phase 4

---

## Non-Responsibilities

The backend must not become responsible for functionality that should work locally.

The backend must not be required for:

- Creating transactions
- Editing transactions
- Deleting transactions
- Creating categories
- Viewing financial history
- Generating basic statistics
- Exporting local data
- Using the application offline

Core financial functionality remains inside the mobile application.

The server synchronizes data. It does not own the user experience.

---

## Architectural Goals

The backend architecture should be:

- Optional
- Self-hostable
- Secure
- Predictable
- Resource-efficient
- Easy to deploy
- Easy to upgrade
- Compatible with Summa Cloud
- Synchronization-friendly
- Observable without exposing financial data
- Maintainable by a small development team

---

## High-Level Architecture

```mermaid
flowchart TD
    A[Android Application] --> B[Local SQLite Database]
    C[iOS Application] --> D[Local SQLite or SwiftData Database]

    B --> E[Mobile Sync Engine]
    D --> F[Mobile Sync Engine]

    E --> G[Summa REST API]
    F --> G

    G --> H[Authentication Module]
    G --> I[Workspace Module]
    G --> J[Synchronization Module]
    G --> K[Attachment Module]
    G --> L[Notification Module]

    H --> M[(PostgreSQL)]
    I --> M
    J --> M
    K --> N[Object or Local File Storage]
    L --> O[Push Notification Providers]
```

The mobile database remains the primary source for the local user experience.

The server maintains a synchronized representation of data for users who enable synchronization.

---

## Deployment Model

The Summa backend is optional. Mobile applications remain fully usable
in local-only mode without an account or server connection.

The same `summa-backend` codebase powers both deployment models:

1. Self-hosted deployments operated by the user.
2. The managed Summa Cloud service operated by Project Summa.

Summa Cloud must not use a separate proprietary synchronization backend.
Differences between self-hosted and managed deployments are limited to
infrastructure, operations, scaling and managed services.

### Local-Only Mode

No backend connection exists.

```mermaid
flowchart LR
    A[Mobile Application] --> B[(Local Database)]
```

Characteristics:

- No account
- No internet requirement
- No remote backup
- No multi-device synchronization
- No server dependency

---

### Self-Hosted Mode

The user deploys a Summa server on their own infrastructure.

```mermaid
flowchart LR
    A[Mobile Application] --> B[Self-Hosted Summa Server]
    B --> C[(PostgreSQL)]
    B --> D[Attachment Storage]
```

Typical deployment targets:

- Home server
- NAS
- Virtual private server
- Docker-compatible hosting environment

---

### Summa Cloud Mode

The user connects to infrastructure managed by the Summa project.

```mermaid
flowchart LR
    A[Mobile Application] --> B[Summa Cloud API]
    B --> C[(Managed PostgreSQL)]
    B --> D[Managed Object Storage]
    B --> E[Backup Infrastructure]
```

The application should use the same API contract for self-hosted and cloud deployments.

---

## Architectural Style

The initial backend should use a **modular monolith**.

A modular monolith provides:

- One deployable backend application
- Clear internal module boundaries
- Simple local development
- Simple self-hosted installation
- Lower operational complexity
- Easier transactions across modules

Microservices are not planned for the initial backend.

They should only be considered when a verified operational requirement cannot be solved within the modular monolith.

---

## Why a Modular Monolith?

Summa will initially be maintained by a small team.

Introducing distributed services too early would create unnecessary complexity involving:

- Service discovery
- Distributed tracing
- Multiple deployment pipelines
- Cross-service authentication
- Network failure handling
- Distributed transactions
- More difficult self-hosting

The backend should remain one service until there is a demonstrated need for separation.

---

## Backend Technology

Preferred implementation language:

```text
Go
```

Preferred architecture:

```text
HTTP API
    ↓
Application Services
    ↓
Domain Modules
    ↓
Repository Interfaces
    ↓
PostgreSQL and Infrastructure
```

The final web framework, migration library and database access library will be selected before implementation.

Technology choices must not change the architectural boundaries described in this document.

---

## Backend Components

The backend consists of the following logical components.

```text
Backend
├── API Layer
├── Authentication
├── Users
├── Devices
├── Workspaces
├── Memberships
├── Invitations
├── Synchronization
├── Transactions
├── Categories
├── Profiles
├── Attachments
├── Notifications
├── Backups
├── Audit Events
├── Configuration
└── Observability
```

---

## API Layer

The API layer is responsible for:

- Receiving HTTP requests
- Parsing request data
- Validating transport-level input
- Authenticating sessions
- Calling application services
- Mapping errors to HTTP responses
- Serializing responses

The API layer must not contain business logic.

---

## Application Services

Application services coordinate use cases.

Examples:

- Create workspace
- Invite workspace member
- Register device
- Upload synchronization batch
- Download changes
- Refresh session
- Upload attachment
- Revoke device

Application services may coordinate multiple repositories and domain components.

---

## Domain Modules

Domain modules contain business rules.

Examples include:

- Workspace membership rules
- Permission validation
- Synchronization version rules
- Invitation expiration
- Conflict resolution policy
- Entity ownership rules

Domain code should not depend directly on:

- HTTP
- PostgreSQL drivers
- Docker
- Push providers
- File systems

---

## Repository Layer

Repository interfaces isolate domain and application logic from storage implementation.

Example:

```go
type TransactionRepository interface {
    FindChangesSince(
        ctx context.Context,
        workspaceID string,
        cursor string,
        limit int,
    ) ([]Transaction, string, error)

    UpsertBatch(
        ctx context.Context,
        transactions []Transaction,
    ) error
}
```

The exact interfaces will be defined during implementation.

---

## Infrastructure Layer

The infrastructure layer contains implementation details such as:

- PostgreSQL repositories
- File storage
- Email delivery
- Push notification providers
- Token signing
- Password hashing
- Logging
- Metrics
- Backup tooling

Infrastructure implementations must be replaceable where practical.

---

## Request Flow

A standard API request follows this flow:

```mermaid
sequenceDiagram
    participant Client as Mobile Client
    participant API as HTTP API
    participant Auth as Authentication
    participant Service as Application Service
    participant Domain as Domain Module
    participant Repo as Repository
    participant DB as PostgreSQL

    Client->>API: HTTP request
    API->>Auth: Validate session
    Auth-->>API: Authenticated principal
    API->>Service: Execute use case
    Service->>Domain: Validate business rules
    Domain-->>Service: Validation result
    Service->>Repo: Read or write data
    Repo->>DB: SQL operation
    DB-->>Repo: Result
    Repo-->>Service: Domain data
    Service-->>API: Use case result
    API-->>Client: HTTP response
```

---

## Data Ownership Model

Every synchronized financial entity must belong to a workspace.

Examples:

- Profile
- Category
- Transaction
- Budget
- Attachment
- Recurring transaction

The server must never rely only on a client-provided entity ID to determine access.

Every query must also validate:

- Authenticated user
- Workspace membership
- Required permission
- Entity workspace ownership

---

## Workspace Model

A workspace represents a synchronized financial context.

Examples:

- Personal finances
- Family
- Couple
- Roommates
- Shared travel budget

A user may belong to multiple workspaces.

A workspace may contain multiple users and devices.

```mermaid
erDiagram
    USER ||--o{ WORKSPACE_MEMBER : has
    WORKSPACE ||--o{ WORKSPACE_MEMBER : contains
    USER ||--o{ DEVICE : owns
    WORKSPACE ||--o{ PROFILE : contains
    WORKSPACE ||--o{ CATEGORY : contains
    WORKSPACE ||--o{ TRANSACTION : contains
```

---

## Personal Workspace

When a cloud or self-hosted account is created, the system may automatically create a personal workspace.

The personal workspace behaves like every other workspace but initially has one member.

This avoids maintaining separate synchronization logic for personal and shared data.

---

## Workspace Roles

Initial roles:

| Role | Description |
|---|---|
| Owner | Full control over workspace and membership |
| Admin | Can manage most workspace resources and members |
| Member | Can create and manage financial data |
| Viewer | Read-only access |

The earlier proposed `Guest` role should be treated as a future consideration unless a concrete use case is defined.

Permissions must be enforced on the server.

The mobile UI may hide unavailable actions, but UI restrictions are not security controls.

---

## Authentication and Authorization

Authentication is introduced only when remote synchronization is enabled.

Local-only users do not need an account.

Initial supported authentication method:

- Email and password

Planned future authentication methods:

- Passkeys
- OAuth providers
- Single sign-on for selected self-hosted environments

---

## Session Model

The preferred session model includes:

- Short-lived access token
- Rotating refresh token
- Device-specific sessions
- Session revocation
- Token expiration
- Refresh token reuse detection

Refresh tokens should be stored as hashed values on the server.

Plaintext refresh tokens must never be stored in the database.

---

## Password Storage

Passwords must be hashed using a modern password hashing algorithm.

Preferred option:

```text
Argon2id
```

Password hashes must include appropriate parameters and unique salts.

Passwords must never be:

- Logged
- Stored in plaintext
- Included in backups without hashing
- Returned through the API

---

## Authorization

Authorization must be workspace-aware.

Example authorization process:

```text
Request
    ↓
Authenticated User
    ↓
Workspace Membership
    ↓
Role and Permission Check
    ↓
Entity Ownership Check
    ↓
Operation Allowed or Rejected
```

Every protected operation must validate access server-side.

---

## Device Management

Every synchronized mobile installation should register as a device.

Device fields may include:

| Field | Purpose |
|---|---|
| `id` | Stable device identifier |
| `user_id` | Device owner |
| `name` | User-readable device name |
| `platform` | Android, iOS or future platform |
| `app_version` | Installed application version |
| `last_seen_at` | Last successful connection |
| `created_at` | Registration time |
| `revoked_at` | Revocation timestamp |

Users should be able to:

- View active devices
- Rename devices
- Revoke devices
- Sign out individual devices
- Sign out all other devices

---

## Synchronization Architecture

Synchronization should be incremental.

Clients should not upload and download the entire database during every synchronization cycle.

The sync protocol should support:

- Client-generated UUIDs
- Incremental change upload
- Incremental change download
- Soft deletion
- Idempotent writes
- Retry safety
- Conflict detection
- Per-workspace cursors
- Bounded batch sizes

---

## Synchronization Flow

```mermaid
sequenceDiagram
    participant App as Mobile Application
    participant Local as Local Database
    participant Queue as Local Sync Queue
    participant API as Summa API
    participant Server as Sync Service
    participant DB as PostgreSQL

    App->>Local: Create or edit entity
    Local->>Queue: Mark change as pending

    Queue->>API: Upload change batch
    API->>Server: Validate and process changes
    Server->>DB: Upsert accepted changes
    DB-->>Server: New server versions
    Server-->>API: Upload result
    API-->>Queue: Accepted, rejected or conflicted

    Queue->>API: Request changes after cursor
    API->>Server: Load incremental changes
    Server->>DB: Query change stream
    DB-->>Server: Changed entities
    Server-->>API: Changes and next cursor
    API-->>Queue: Download batch

    Queue->>Local: Apply remote changes
```

---

## Source of Truth

The local database is the source of truth for immediate user interaction.

When synchronization is enabled:

- Local writes happen first
- UI updates immediately
- Changes are queued
- Synchronization occurs asynchronously
- Server acknowledgment updates local sync metadata

The UI must not wait for the server before confirming a normal local edit.

---

## Idempotency

Synchronization requests must be safe to retry.

Every mutation should include a stable operation or entity identifier.

Repeated delivery of the same operation must not create duplicate data.

Idempotency is especially important for:

- Unstable networks
- Background synchronization
- Application restarts
- Server timeouts
- Mobile operating system interruptions

---

## Conflict Resolution

Conflicts occur when the same entity is modified independently on multiple devices.

The initial strategy should remain conservative and understandable.

Possible conflict inputs include:

- Entity UUID
- Client version
- Server version
- Updated timestamp
- Device identifier
- Changed fields
- Deletion state

---

## Initial Conflict Strategy

For simple scalar entities, the first implementation may use:

```text
Last accepted update wins
```

However, the server must preserve enough metadata to detect that a conflict occurred.

For sensitive or destructive conflicts, the system should avoid silently discarding information.

Examples:

- Simultaneous transaction edits
- Edit versus delete
- Shared category rename
- Workspace membership changes

Conflict rules must be documented per entity before sync implementation begins.

---

## Server Change Tracking

The backend should expose a monotonic synchronization cursor per workspace.

A cursor may represent:

- Change sequence
- Server revision
- Ordered change identifier

Clients request:

```text
all changes after cursor X
```

The server returns:

- Change batch
- Deletion tombstones
- Next cursor
- Whether more changes are available

Clients must treat cursors as opaque values.

---

## Database Architecture

Primary server database:

```text
PostgreSQL
```

The database should contain:

- Authentication data
- Users
- Devices
- Workspaces
- Memberships
- Invitations
- Financial entities
- Synchronization metadata
- Attachment metadata
- Audit events
- Service configuration where appropriate

Financial entities should use UUID primary keys generated by clients or the server, depending on the entity type.

---

## Database Transactions

Operations that modify multiple related records must use database transactions.

Examples:

- Creating a workspace and its owner membership
- Accepting an invitation
- Applying a synchronization batch
- Deleting a workspace
- Rotating refresh tokens
- Registering an attachment and storage reference

Partial application of these operations is not acceptable.

---

## Database Migrations

All schema changes must use version-controlled migrations.

Requirements:

- Migrations run in a deterministic order
- Migration history is stored in the database
- Production migrations must be backward-aware
- Destructive migrations require explicit documentation
- Migration failure must stop application startup safely
- Automatic data deletion is forbidden

Self-hosted administrators must receive clear upgrade instructions when manual intervention is required.

---

## Multi-Tenancy

The backend uses logical multi-tenancy based on workspaces.

Every synchronized business record contains a workspace identifier.

The application must prevent cross-workspace data access through:

- Query scoping
- Authorization checks
- Foreign key constraints
- Integration tests
- Optional database-level policies in the future

Summa Cloud and self-hosted deployments should use the same logical tenancy model.

---

## File and Attachment Storage

Attachments may include:

- Receipt images
- Invoice images
- PDF documents
- Imported statements
- Future encrypted backup archives

Metadata belongs in PostgreSQL.

Binary file content should not be stored directly in normal relational tables unless there is a justified small-file use case.

---

## Storage Providers

The attachment layer should support multiple implementations.

Initial self-hosted options:

- Local filesystem
- S3-compatible object storage

Summa Cloud:

- Managed object storage

The application should use a storage interface so the business layer does not depend on a provider.

---

## Attachment Security

Attachments must be treated as sensitive financial data.

Requirements:

- Authorization before upload and download
- Workspace ownership checks
- Content size limits
- MIME type validation
- Randomized storage keys
- No executable file serving
- Safe response headers
- Optional malware scanning in hosted environments
- No public buckets
- No predictable public URLs

---

## Background Processing

Some operations should run outside the HTTP request lifecycle.

Examples:

- Sending invitation emails
- Push notifications
- Backup creation
- Expired session cleanup
- Attachment cleanup
- Audit retention cleanup
- Future OCR tasks
- Future statement parsing

The initial implementation should avoid introducing a complex external queue unless required.

A PostgreSQL-backed job system or internal worker may be sufficient for the first backend version.

---

## Job Requirements

Background jobs should support:

- Retry count
- Exponential backoff
- Failure reason
- Scheduled execution
- Idempotency
- Dead-letter or failed state
- Operational visibility
- Safe restart behavior

Jobs must never contain plaintext secrets or unnecessary financial details in logs.

---

## Notifications

Push notifications are optional.

The notification infrastructure is built during Phase 3 (Synchronization) as part of the backend. However, user-facing push notifications are a Phase 4 (Summa Cloud) feature. Self-hosted deployments may configure push notifications if they provide their own FCM/APNs credentials, but this is not required for core synchronization functionality.

The backend may coordinate:

- Workspace invitations
- Shared transaction changes
- Reminder delivery
- Security alerts
- Device login alerts
- Backup failures

Financial values should not be included in push notification payloads by default.

Lock screen notifications may be visible to third parties.

Sensitive details should only appear after the user opens the application.

---

## Configuration

The backend must be configurable through environment variables and configuration files where appropriate.

Configuration categories include:

- HTTP server
- Database
- Authentication
- Token lifetimes
- File storage
- Email provider
- Push providers
- Logging
- Metrics
- Backup
- Rate limiting
- Allowed origins
- Public server URL

Secrets must not be committed to Git.

---

## Example Configuration

```env
SUMMA_ENV=production
SUMMA_HTTP_ADDRESS=0.0.0.0:8080
SUMMA_PUBLIC_URL=https://summa.example.com

SUMMA_DATABASE_URL=postgres://summa:password@database:5432/summa

SUMMA_STORAGE_DRIVER=filesystem
SUMMA_STORAGE_PATH=/var/lib/summa/files

SUMMA_ACCESS_TOKEN_TTL=15m
SUMMA_REFRESH_TOKEN_TTL=720h

SUMMA_LOG_LEVEL=info
```

The exact variable names may change during implementation.

A documented `.env.example` file must be provided.

---

## Self-Hosted Deployment

The official self-hosted installation should prioritize simplicity.

Target deployment:

```bash
docker compose up -d
```

Minimum required services:

```text
Summa Backend
PostgreSQL
```

Optional services:

```text
S3-compatible storage
Reverse proxy
SMTP server
Monitoring
External backup destination
```

---

## Self-Hosted Docker Architecture

```mermaid
flowchart TD
    A[Reverse Proxy] --> B[Summa Backend Container]
    B --> C[(PostgreSQL Container)]
    B --> D[Persistent Attachment Volume]
    C --> E[Persistent Database Volume]
```

The reverse proxy may be provided by the administrator.

Official examples may include:

- Caddy
- Traefik
- Nginx

TLS termination should occur before traffic reaches the public API.

---

## Self-Hosted Requirements

Official documentation must include:

- Installation guide
- Upgrade guide
- Backup guide
- Restore guide
- Environment variable reference
- Health check documentation
- Reverse proxy examples
- Storage requirements
- Security recommendations
- Troubleshooting section

The project should avoid requiring Kubernetes. Community self-managed Kubernetes deployments are not blocked, but no official Helm charts or manifests are provided.

---

## Summa Cloud Deployment

Summa Cloud uses the same backend code as self-hosted deployments.

Cloud-only infrastructure may include:

- Managed PostgreSQL
- Managed object storage
- Load balancers
- Secret management
- Centralized monitoring
- Automated backups
- Billing integration
- Email delivery
- Push notification services

Cloud-specific functionality must be isolated from the core backend.

---

## Cloud Feature Boundaries

The following may be cloud operational features:

- Subscription management
- Hosted account lifecycle
- Resource quotas
- Billing state
- Managed backup retention
- Service status
- Abuse prevention

Core financial features must remain available to self-hosted users.

---

## Subscription Enforcement

Subscription state should control managed hosting entitlement, not access to fundamental financial functionality.

A subscription may determine:

- Whether Summa hosts the account
- Storage quota
- Backup retention
- Managed service limits
- Support tier

It should not prevent a user from exporting or recovering their own data.

---

## Observability

The backend should provide:

- Structured logs
- Health endpoint
- Readiness endpoint
- Metrics
- Request correlation identifiers
- Error reporting
- Background job visibility

Observability must avoid exposing financial content.

---

## Logging

Logs may include:

- Request method
- Route template
- Status code
- Duration
- Correlation ID
- User ID where appropriate
- Workspace ID where appropriate
- Error category

Logs must not include:

- Passwords
- Tokens
- Transaction notes
- Merchant names
- Financial amounts
- Imported statement contents
- Attachment contents
- Full request bodies containing sensitive data

---

## Health Endpoints

Recommended endpoints:

```text
GET /health/live
GET /health/ready
```

`live` confirms the process is running.

`ready` confirms required dependencies are available.

Readiness may validate:

- Database connectivity
- Migration status
- Required storage availability

---

## Metrics

Useful service metrics include:

- Request count
- Error count
- Request latency
- Database latency
- Active background jobs
- Failed jobs
- Synchronization batch size
- Synchronization duration
- Attachment storage usage
- Backup success and failure

Metrics must not contain user financial content.

---

## Security Boundaries

The backend should assume:

- The public network is untrusted
- Client input is untrusted
- Uploaded files are untrusted
- Access tokens may be stolen
- Self-hosted deployments may be misconfigured
- Database backups contain sensitive data

All input must be validated.

All authorization must be enforced server-side.

---

## Rate Limiting

Rate limits should be applied to sensitive endpoints.

Examples:

- Login
- Password reset
- Email verification
- Invitation creation
- Token refresh
- Attachment upload
- Synchronization requests

Self-hosted administrators may configure rate limits within safe boundaries.

---

## Transport Security

Production deployments require HTTPS.

The backend should support trusted reverse proxy configuration.

Plain HTTP may be allowed only for:

- Local development
- Internal container networking
- Explicitly documented private environments

Mobile applications should reject insecure public endpoints by default.

---

## Backup and Recovery

Backups are essential because synchronized financial history is sensitive and difficult to recreate.

Server backups should include:

- PostgreSQL data
- Attachment storage
- Required encryption metadata
- Configuration reference
- Migration version

Secrets should be backed up through a separate secure process.

---

## Backup Requirements

Backups should be:

- Automated where possible
- Encrypted
- Versioned
- Tested
- Restorable
- Stored separately from the primary server
- Documented

A backup that has never been restored in testing must not be considered reliable.

---

## Self-Hosted Backup

The project should provide scripts or documented commands for:

- Database dump
- Attachment archive
- Full backup
- Restore
- Backup verification

The administrator remains responsible for backup destination and retention.

---

## Summa Cloud Recovery

Summa Cloud should eventually support:

- Automated scheduled backups
- Point-in-time database recovery
- Object storage versioning
- Documented recovery objectives
- Periodic restore testing
- Disaster recovery procedures

Specific recovery objectives will be defined during Phase 4.

---

## API Versioning

The public API should be versioned.

Initial format:

```text
/api/v1/
```

Examples:

```text
POST /api/v1/auth/login
GET  /api/v1/workspaces
POST /api/v1/sync/push
GET  /api/v1/sync/pull
```

Breaking changes require a new API version or a documented compatibility strategy.

---

## Compatibility

Self-hosted users may upgrade less frequently than cloud users.

The backend and mobile applications must support a documented compatibility window.

The API should return clear errors when:

- Client version is unsupported
- Server version is too old
- Required synchronization capability is unavailable

Upgrade requirements must not cause silent data loss.

---

## Scalability

The initial backend should scale vertically and support multiple application instances where practical.

Potential scaling path:

```text
Single Backend Instance
        ↓
Larger Instance
        ↓
Multiple Stateless API Instances
        ↓
Shared PostgreSQL and Object Storage
        ↓
Dedicated Worker Processes
```

The architecture should not assume that every deployment needs horizontal scaling.

Self-hosting simplicity remains a priority.

---

## Stateless API

API instances should remain stateless where practical.

Persistent state belongs in:

- PostgreSQL
- Object storage
- Cache only when introduced deliberately

In-memory state must not be required for correctness across requests.

---

## Caching

Caching should not be introduced until a measurable requirement exists.

Potential future caching targets:

- Session metadata
- Permission lookups
- Rate-limit state
- Frequently accessed configuration

Cache failure must not result in data loss.

---

## Testing Strategy

Backend testing includes:

### Unit Tests

Used for:

- Domain rules
- Permission logic
- Token validation
- Conflict resolution
- Input validation
- Cursor logic

### Integration Tests

Used for:

- PostgreSQL repositories
- Database migrations
- Authentication flows
- Synchronization batches
- Workspace isolation
- Attachment authorization

### API Tests

Used for:

- Request validation
- HTTP status codes
- Response schemas
- Authorization boundaries
- Pagination
- Version compatibility

### End-to-End Tests

Used for:

- Account creation
- Device registration
- Workspace invitation
- Synchronization between two clients
- Backup and restore
- Session revocation

---

## Critical Security Tests

The test suite must verify that:

- One workspace cannot access another workspace
- Revoked devices cannot refresh sessions
- Deleted members lose workspace access
- Duplicate sync operations remain idempotent
- Invalid cursors do not leak data
- Attachments cannot be downloaded without authorization
- Deleted entities synchronize correctly
- Refresh token reuse is detected

---

## Failure Handling

The backend must fail safely.

Examples:

### Database Unavailable

- Readiness endpoint fails
- Requests return controlled service errors
- Process does not silently discard writes

### Storage Unavailable

- Attachment operations fail explicitly
- Financial database operations remain available where safe

### Email Provider Unavailable

- Invitation email is retried
- Core API remains available

### Push Provider Unavailable

- Notification is retried or discarded according to policy
- Financial synchronization remains unaffected

### Migration Failure

- Application startup stops
- Error is clearly logged
- Existing data is not modified further

---

## Error Model

Internal errors should use stable categories.

Examples:

```text
validation_error
authentication_required
permission_denied
resource_not_found
conflict
rate_limited
unsupported_client
storage_unavailable
internal_error
```

Internal implementation details must not be exposed to clients.

---

## Development Environment

The backend development environment should be reproducible.

Recommended local services:

```text
Backend
PostgreSQL
Local attachment storage
Optional email testing service
```

Development startup should require minimal steps.

Example:

```bash
docker compose -f compose.dev.yml up -d
go run ./cmd/summa
```

The final command structure will be defined during implementation.

---

## Local Seed Data

Development environments may include optional seed data.

Seed data must:

- Use fictional users
- Use fictional transactions
- Contain no real financial information
- Be clearly separated from production migrations
- Be safe to delete and recreate

---

## Future Expansion

The backend architecture should allow future support for:

- Desktop synchronization
- Web application
- Public API
- Personal access tokens
- Webhooks
- Plugin system
- External import workers
- Advanced OCR processing
- Financial insight services
- Organization deployments
- End-to-end encrypted synchronization research

These features are not part of the initial backend scope.

---

## Explicitly Excluded from Initial Backend

The first self-hosted backend will not require:

- Kubernetes
- Microservices
- GraphQL
- Event streaming platforms
- Distributed databases
- Multi-region deployment
- Complex service mesh
- Mandatory Redis
- Mandatory external message broker
- Real-time collaborative editing
- Banking API credentials

These technologies may only be introduced after a verified requirement.

---

## Risks

### Synchronization Complexity

Synchronization is one of the most difficult parts of the platform.

Mitigation:

- Client-generated UUIDs
- Soft deletion
- Explicit versions
- Incremental batches
- Idempotent operations
- Per-entity conflict rules
- Extensive integration tests

---

### Self-Hosting Complexity

Too many required services may discourage users.

Mitigation:

- Modular monolith
- Minimal required containers
- Strong documentation
- Safe defaults
- Health checks
- Automated migrations

---

### Security Risk

The backend stores sensitive financial information.

Mitigation:

- Minimal data collection
- Strong authorization
- Secure password hashing
- Token rotation
- Encryption in transit
- Backup encryption
- Security testing
- Responsible disclosure process

---

### Cloud and Self-Hosted Divergence

Cloud-specific changes could make the self-hosted version inferior.

Mitigation:

- Shared backend codebase
- Core features remain deployment-neutral
- Cloud code remains isolated
- Feature decisions follow project principles

---

### Overengineering

Designing for future scale may slow implementation.

Mitigation:

- Modular monolith
- Minimal dependencies
- No speculative infrastructure
- Incremental implementation
- Requirements before abstractions

---

## Definition of Done

This architecture document is considered complete when:

- Backend responsibilities are agreed upon
- Deployment modes are clearly defined
- Modular monolith architecture is accepted
- Authentication boundaries are documented
- Workspace ownership is defined
- Synchronization principles are documented
- Storage architecture is defined
- Backup responsibilities are defined
- Self-hosted and cloud boundaries are clear
- Major backend risks are acknowledged

This document does not mean that every implementation detail is finalized.

Detailed endpoint contracts belong in:

```text
docs/phase-0/05_API_DESIGN.md
```

Detailed security requirements belong in:

```text
docs/phase-0/06_SECURITY_MODEL.md
```

---

## Guiding Principle

The Summa backend exists to synchronize and protect user-owned data.

It must never become a mandatory dependency for using the application.

The self-hosted server and Summa Cloud should provide convenience without weakening the local-first foundation of the project.