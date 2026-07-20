# Phase 0 Milestones

> Defining the work required to complete the Summa foundation and begin implementation safely.

---

## Table of Contents

- [Purpose](#purpose)
- [Phase 0 Objective](#phase-0-objective)
- [Milestone Overview](#milestone-overview)
- [Milestone 0.1 — Product Foundation](#milestone-01--product-foundation)
- [Milestone 0.2 — System Architecture](#milestone-02--system-architecture)
- [Milestone 0.3 — Data and API Contracts](#milestone-03--data-and-api-contracts)
- [Milestone 0.4 — Security and Privacy](#milestone-04--security-and-privacy)
- [Milestone 0.5 — Design and Development Standards](#milestone-05--design-and-development-standards)
- [Milestone 0.6 — Repository Preparation](#milestone-06--repository-preparation)
- [Milestone 0.7 — Implementation Readiness](#milestone-07--implementation-readiness)
- [Open Decisions](#open-decisions)
- [Phase 0 Deliverables](#phase-0-deliverables)
- [Phase 0 Exit Criteria](#phase-0-exit-criteria)
- [Phase 1 Handoff](#phase-1-handoff)
- [Initial Phase 1 Backlog](#initial-phase-1-backlog)
- [Definition of Done](#definition-of-done)

---

## Purpose

This document converts Phase 0 documentation into concrete milestones and completion criteria.

Phase 0 is not complete only because documents exist.

It is complete when the project team has:

- Reviewed the documents
- Resolved critical contradictions
- Recorded unresolved decisions
- Prepared the repository
- Created the implementation backlog
- Confirmed that Phase 1 can begin without major architectural uncertainty

---

## Phase 0 Objective

The objective of Phase 0 is:

```text
Prepare Summa for implementation without leaving foundational decisions to be improvised inside feature code.
```

Phase 0 should produce a stable baseline, not a permanently frozen architecture.

Future changes remain possible, but they must be deliberate and documented.

---

## Milestone Overview

| Milestone | Name | Primary Result |
|---|---|---|
| 0.1 | Product Foundation | Shared vision and scope |
| 0.2 | System Architecture | Defined system boundaries |
| 0.3 | Data and API Contracts | Stable data model and API conventions |
| 0.4 | Security and Privacy | Documented trust and protection model |
| 0.5 | Design and Development Standards | Consistent UI and coding rules |
| 0.6 | Repository Preparation | Ready repositories and workflows |
| 0.7 | Implementation Readiness | Approved Phase 1 starting point |

---

# Milestone 0.1 — Product Foundation

## Objective

Define why Summa exists and what it is intended to become.

---

## Required Documents

```text
docs/00_PROJECT_OVERVIEW.md
docs/01_VISION_AND_PRINCIPLES.md
docs/02_ROADMAP.md
docs/03_TECH_STACK.md
docs/04_PROJECT_STRUCTURE.md
docs/05_GLOSSARY_AND_DOMAIN_LANGUAGE.md
docs/phase-0/00_README.md
```

---

## Checklist

- [ ] Project purpose is clearly stated
- [ ] Target users are identified
- [ ] Local-first principle is documented
- [ ] User ownership principle is documented
- [ ] Cloud is explicitly optional
- [ ] Open-source direction is documented
- [ ] Non-goals are documented
- [ ] Phase 0 through Phase 4 are defined
- [ ] Major platforms are identified
- [ ] Project name is consistently written as `Summa`
- [ ] Confirm that `summa-backend` is the shared backend for self-hosted and managed deployments
- [ ] Keep Summa Cloud infrastructure separate from backend application code
- [ ] Ensure local-only mobile usage never depends on backend availability
- [ ] Glossary and domain language document exists and defines all key terms
- [ ] Core domain terms (Workspace, Profile, Account) are defined consistently across all docs

---

## Exit Criteria

Milestone 0.1 is complete when all team members agree on:

- Core purpose
- Product philosophy
- Main development phases
- Local-first requirement
- Optional cloud model

---

# Milestone 0.2 — System Architecture

## Objective

Define the boundaries and responsibilities of all major system components.

---

## Required Documents

```text
docs/phase-0/01_ARCHITECTURE.md
docs/phase-0/03_MOBILE_ARCHITECTURE.md
docs/phase-0/04_BACKEND_ARCHITECTURE.md
```

---

## Checklist

- [ ] High-level system architecture is documented
- [ ] Mobile layers are documented
- [ ] Backend responsibilities are documented
- [ ] Local-only mode is documented
- [ ] Self-hosted mode is documented
- [ ] Summa Cloud mode is documented
- [ ] Repository pattern is defined
- [ ] Use-case layer is defined
- [ ] Presentation boundaries are defined
- [ ] Sync engine is conceptually isolated
- [ ] Modular monolith is selected for backend
- [ ] Direct UI-to-database access is forbidden
- [ ] Future expansion does not require replacing the local-first core

---

## Architecture Review Questions

Before approval, answer:

- Can the Android application work without a backend?
- Can synchronization be added without replacing local repositories?
- Is business logic separate from UI?
- Is backend deployment simple enough for self-hosting?
- Are cloud-only concerns isolated?
- Are module responsibilities understandable?

---

## Exit Criteria

Milestone 0.2 is complete when a developer can explain the full request and data flow without relying on undocumented assumptions.

---

# Milestone 0.3 — Data and API Contracts

## Objective

Define stable data representation before feature implementation.

---

## Required Documents

```text
docs/phase-0/02_DATABASE.md
docs/phase-0/05_API_DESIGN.md
backend/api/openapi.yaml
```

The OpenAPI file may initially contain only foundational schemas and placeholder endpoint groups.

---

## Checklist

- [ ] Core entities are listed
- [ ] Entity relationships are documented
- [ ] UUID strategy is defined
- [ ] Money uses integer minor units consistently across all docs
- [ ] Currency representation is defined (ISO 4217)
- [ ] Date and timestamp rules are defined
- [ ] Soft deletion strategy is defined
- [ ] Migration policy is defined
- [ ] Synchronization metadata is defined conceptually
- [ ] All entities include common sync fields (id, created_at, updated_at, deleted_at, version, sync_status, device_id)
- [ ] Workspace entity is defined as top-level organizational unit
- [ ] Profile↔Workspace relationship is documented
- [ ] TransactionSplit entity is defined
- [ ] API base path is defined
- [ ] API versioning is defined
- [ ] JSON naming is defined
- [ ] Error format is defined
- [ ] Pagination format is defined
- [ ] Idempotency requirements are defined
- [ ] Optimistic concurrency is defined
- [ ] Sync push and pull contracts are drafted
- [ ] Attachment flow is drafted

---

## Required Data Model Review

The team must review at minimum:

```text
Profile
Workspace
WorkspaceMember
Category
Transaction
TransactionSplit
Budget
Attachment
Device
Sync metadata
```

Entities excluded from Phase 1 should still be clearly marked as future scope.

---

## OpenAPI Baseline

The initial OpenAPI file should define:

- Common error schema
- Money schema
- Pagination metadata
- Authentication header
- Workspace identifier
- Transaction schema draft
- Category schema draft
- Profile schema draft
- Server information endpoint
- Health endpoints

---

## Exit Criteria

Milestone 0.3 is complete when Android, iOS and backend developers can independently implement compatible data structures from the documentation.

---

# Milestone 0.4 — Security and Privacy

## Objective

Define how sensitive financial data is protected in every deployment mode.

---

## Required Documents

```text
docs/phase-0/06_SECURITY_MODEL.md
SECURITY.md
```

---

## Checklist

- [ ] Threat model is documented
- [ ] Data classification is documented
- [ ] Local storage protection is documented
- [ ] Application lock behavior is defined
- [ ] Biometric integration is planned
- [ ] Secure key storage is defined
- [ ] Password hashing is defined
- [ ] Access token lifecycle is defined
- [ ] Refresh token rotation is defined
- [ ] Device revocation is defined
- [ ] Workspace authorization is defined
- [ ] HTTPS requirement is defined
- [ ] Attachment protection is defined
- [ ] Backup protection is defined
- [ ] Sensitive logging is forbidden
- [ ] Secrets management is documented
- [ ] Dependency scanning is planned
- [ ] Vulnerability reporting process is documented
- [ ] Incident response outline exists

---

## Security Validation Questions

- What happens if a phone is stolen?
- What happens if a refresh token is stolen?
- What happens if a workspace member is removed?
- Can one workspace access another?
- Can a self-hosted administrator read server data?
- Are backups encrypted?
- What sensitive information may appear in logs?
- How are production secrets stored?
- How does a user revoke a device?

Unanswered critical questions block completion of this milestone.

---

## Exit Criteria

Milestone 0.4 is complete when security requirements can be converted into implementation tasks and automated tests.

---

# Milestone 0.5 — Design and Development Standards

## Objective

Establish consistent rules for interface design and code implementation.

---

## Required Documents

```text
docs/phase-0/07_DESIGN_SYSTEM.md
docs/phase-0/08_GIT_WORKFLOW.md
docs/phase-0/09_CODING_GUIDELINES.md
```

---

## Design Checklist

- [ ] Primary color is finalized
- [ ] Neutral palette is finalized
- [ ] Semantic colors are defined
- [ ] Light theme is defined
- [ ] Dark theme tokens are defined
- [ ] Typography scale is defined
- [ ] Spacing scale is defined
- [ ] Mobile grid is defined
- [ ] Radius system is defined
- [ ] Bottom navigation is approved
- [ ] Header pattern is approved
- [ ] Balance card is approved
- [ ] Transaction row is approved
- [ ] Input states are defined
- [ ] Empty and error states are defined
- [ ] Accessibility requirements are documented

---

## Figma Deliverables

Recommended minimum:

```text
Foundations page
Color variables
Typography styles
Spacing documentation
Button component
Input component
Card component
Transaction row component
Bottom navigation component
Top app bar component
Main dashboard wireframe
Transaction form wireframe
Light and dark examples
```

---

## Development Standards Checklist

- [ ] Branch naming is defined
- [ ] Commit format is defined
- [ ] Pull request template is defined
- [ ] Review requirements are defined
- [ ] Merge strategy is defined
- [ ] Release strategy is defined
- [ ] Kotlin conventions are defined
- [ ] Compose conventions are defined
- [ ] Swift conventions are defined
- [ ] SwiftUI conventions are defined
- [ ] Go conventions are defined
- [ ] Logging rules are defined
- [ ] Error handling rules are defined
- [ ] Testing requirements are defined
- [ ] AI-generated code rules are defined

---

## Exit Criteria

Milestone 0.5 is complete when separate contributors can produce visually and technically consistent work without inventing new conventions.

---

# Milestone 0.6 — Repository Preparation

## Objective

Create the initial repository structures and development automation across all repositories.

---

## Repository Structure

The `summa` central repository:

```text
summa/
├── .github/
├── docs/
├── design/
├── branding/
├── scripts/
├── website/
├── README.md
├── CONTRIBUTING.md
├── SECURITY.md
├── CODE_OF_CONDUCT.md
├── LICENSE
└── CHANGELOG.md
```

Platform repositories:

```text
summa-android/
summa-ios/
summa-backend/
```

Each platform repository maintains its own CI workflows, issue tracking and release cycle.

---

## Checklist

- [ ] GitHub repository exists
- [ ] Repository visibility is selected
- [ ] License is selected
- [ ] Root README exists
- [ ] CONTRIBUTING exists
- [ ] SECURITY exists
- [ ] CODE_OF_CONDUCT exists
- [ ] CHANGELOG exists
- [ ] `.gitignore` exists
- [ ] `.editorconfig` exists
- [ ] `.gitattributes` exists
- [ ] Documentation folders exist
- [ ] Android folder exists
- [ ] iOS folder exists
- [ ] Backend folder exists
- [ ] Branch protection is enabled
- [ ] Issue templates exist
- [ ] Pull request template exists
- [ ] CODEOWNERS exists
- [ ] Secret scanning is enabled where available

---

## Initial CI Workflows

Create at minimum:

```text
.github/workflows/docs.yml
.github/workflows/android.yml
```

The iOS and backend workflows may begin as placeholders until their skeleton projects exist.

---

## Documentation CI

Should validate:

- Markdown style
- Broken internal links
- File naming
- Mermaid syntax where practical

---

## Android CI

After Android skeleton exists:

- Build debug application
- Run unit tests
- Run ktlint
- Run Detekt
- Run Android Lint

---

## Exit Criteria

Milestone 0.6 is complete when a new contributor can clone the repository, understand its structure and run the available checks locally.

---

# Milestone 0.7 — Implementation Readiness

## Objective

Confirm that Phase 1 can begin with clear tasks and no unresolved foundational blockers.

---

## Required Decisions

Before starting Phase 1, finalize:

- Android application ID
- Minimum Android SDK
- Android module strategy
- Room database version 1 scope
- Dependency injection framework
- Navigation approach
- Design token implementation approach
- Initial supported currency behavior
- Initial supported languages
- Backup format scope
- Whether local database encryption enters the first public release
- Whether budgets enter the MVP
- Exact shared-expense scope in the MVP

---

## Android Skeleton Requirements

The Android project should contain:

```text
Application class
MainActivity
Compose theme
Navigation skeleton
Dependency injection setup
Database module placeholder
Core domain module or package
Feature package structure
Unit test setup
Linting setup
CI build
```

No full production feature is required during Phase 0.

---

## Optional iOS Skeleton

The iOS project may contain:

```text
Application entry point
Theme tokens
Navigation placeholder
Persistence placeholder
Test target
```

Android remains the first implementation priority.

---

## Optional Backend Skeleton

The backend may contain:

```text
Go module
Application entry point
Configuration loader
Health endpoint
Database connection placeholder
Dockerfile
Development compose file
Test setup
```

Backend feature implementation remains Phase 3 work.

The skeleton exists only to validate structure and tooling.

---

## Phase 1 Backlog Requirements

Every Phase 1 epic should contain:

- Goal
- User value
- Scope
- Out-of-scope
- Acceptance criteria
- Design reference
- Technical reference
- Test requirements
- Dependencies

---

## Exit Review

Conduct a Phase 0 review meeting.

Suggested agenda:

```text
1. Product principles
2. Architecture
3. Database
4. Mobile architecture
5. Backend direction
6. API
7. Security
8. Design system
9. Git workflow
10. Coding standards
11. Open decisions
12. Phase 1 backlog
```

Record the outcome of every unresolved item.

---

## Open Decisions

The following decisions should be resolved or explicitly deferred.

| Decision | Status | Deadline |
|---|---|---|
| Final open-source license | Open | Before public repository launch |
| Android application ID | Open | Before Android project creation |
| Local database encryption release target | Open | Before public beta |
| Exact profile and workspace relationship | Review | Before database implementation |
| Transaction split entity in MVP | Review | Before transaction schema |
| Budget feature in MVP | Review | Before Phase 1 backlog freeze |
| Inter versus native system font | Review | Before design implementation |
| Backend Go framework | Deferred | Before Phase 3 |
| PostgreSQL migration tool | Deferred | Before Phase 3 |
| Self-hosted email requirement | Deferred | Before Phase 3 |
| Cloud billing provider | Deferred | Before Phase 4 |

Deferred decisions must have a defined phase in which they become blocking.

---

## Phase 0 Deliverables

At completion, the repository should contain:

```text
docs/
├── 00_PROJECT_OVERVIEW.md
├── 01_VISION_AND_PRINCIPLES.md
├── 02_ROADMAP.md
├── 03_TECH_STACK.md
├── 04_PROJECT_STRUCTURE.md
├── 05_GLOSSARY_AND_DOMAIN_LANGUAGE.md
└── phase-0/
    ├── 00_README.md
    ├── 01_ARCHITECTURE.md
    ├── 02_DATABASE.md
    ├── 03_MOBILE_ARCHITECTURE.md
    ├── 04_BACKEND_ARCHITECTURE.md
    ├── 05_API_DESIGN.md
    ├── 06_SECURITY_MODEL.md
    ├── 07_DESIGN_SYSTEM.md
    ├── 08_GIT_WORKFLOW.md
    ├── 09_CODING_GUIDELINES.md
    └── 10_MILESTONES.md
```

Additional root documents:

```text
README.md
CONTRIBUTING.md
SECURITY.md
CODE_OF_CONDUCT.md
LICENSE
CHANGELOG.md
```

---

## Phase 0 Exit Criteria

Phase 0 is complete only when:

- [ ] All Phase 0 documents exist
- [ ] Glossary and domain language document exists
- [ ] Documents have been reviewed by the team
- [ ] Contradictions are resolved
- [ ] Entity definitions include all common sync fields
- [ ] Money representation uses integer minor units consistently
- [ ] Workspace and Profile relationship is defined
- [ ] Critical open decisions are closed
- [ ] Deferred decisions have assigned phases
- [ ] Repository structure exists
- [ ] Git workflow is active
- [ ] Initial CI is active
- [ ] Figma foundations exist
- [ ] Android skeleton builds successfully
- [ ] Phase 1 epics exist
- [ ] First Phase 1 milestone is approved
- [ ] No unresolved issue blocks the transaction data model

---

## Phase 1 Handoff

Phase 1 begins with:

```text
Local First MVP
```

Recommended implementation order:

```text
1. Android project foundation
2. Design token implementation
3. Local database
4. Profiles and workspaces
5. Categories
6. Transactions
7. Dashboard
8. Statistics
9. Export and backup
10. Settings and application lock
11. Testing and stabilization
12. First public release
```

---

## Initial Phase 1 Backlog

### Epic 1 — Android Foundation

- Create Android project
- Configure Compose
- Configure Hilt
- Configure Room
- Configure navigation
- Configure linting
- Configure tests
- Configure CI
- Implement design tokens

---

### Epic 2 — Profiles and Workspaces

- Create local workspace
- Create profile
- Edit profile
- Select active profile
- Define personal and household mode
- Persist active selection

---

### Epic 3 — Categories

- Seed default categories
- Create custom category
- Edit category
- Archive category
- Select icon
- Select color
- Filter by category type

---

### Epic 4 — Transactions

- Create expense
- Create income
- Edit transaction
- Delete transaction
- View transaction details
- Add note
- Select date
- Select category
- Select profile
- Validate amount

---

### Epic 5 — Dashboard

- Current balance
- Monthly income
- Monthly expenses
- Recent transactions
- Category summary
- Empty state
- Loading state
- Error state

---

### Epic 6 — Statistics

- Monthly totals
- Category totals
- Date filters
- Basic spending trend
- Accessible chart summaries

---

### Epic 7 — Export and Backup

- JSON export
- CSV export
- Local backup
- Restore
- Backup validation
- User confirmation
- Failure recovery

---

### Epic 8 — Settings and Security

- Currency
- Language
- Light and dark theme
- Application lock
- Biometrics
- Privacy settings
- Data deletion

---

### Epic 9 — Quality and Release

- Migration tests
- Accessibility review
- Performance review
- Security review
- Store assets
- Release notes
- Beta testing
- Crash stabilization

---

## Definition of Done

Phase 0 is officially complete when the team can begin the first Phase 1 issue without inventing new foundational architecture inside that issue.

The remaining decisions should be feature-level decisions, not unresolved system identity, storage, security or workflow questions.