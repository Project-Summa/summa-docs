# Git Workflow

> Defining how contributors plan, implement, review and release changes in the Summa repository.

---

## Table of Contents

- [Purpose](#purpose)
- [Workflow Goals](#workflow-goals)
- [Repository Model](#repository-model)
- [Primary Branches](#primary-branches)
- [Supporting Branches](#supporting-branches)
- [Branch Naming](#branch-naming)
- [Issue Workflow](#issue-workflow)
- [Development Workflow](#development-workflow)
- [Commit Convention](#commit-convention)
- [Pull Requests](#pull-requests)
- [Code Review](#code-review)
- [Merge Strategy](#merge-strategy)
- [Branch Protection](#branch-protection)
- [Release Workflow](#release-workflow)
- [Hotfix Workflow](#hotfix-workflow)
- [Versioning](#versioning)
- [Changelog](#changelog)
- [Documentation Changes](#documentation-changes)
- [AI-Assisted Contributions](#ai-assisted-contributions)
- [Security Rules](#security-rules)
- [Repository Automation](#repository-automation)
- [Definition of Done](#definition-of-done)
- [Guiding Principle](#guiding-principle)

---

## Purpose

This document defines the Git and GitHub workflow used by Summa.

The workflow should make changes:

- Easy to understand
- Easy to review
- Easy to test
- Easy to revert
- Easy to release
- Traceable to an issue or decision

---

## Workflow Goals

The workflow should:

- Protect stable code
- Support multiple contributors
- Keep pull requests focused
- Maintain readable history
- Prevent accidental secret exposure
- Support mobile and backend development
- Work well with AI coding tools
- Avoid unnecessary process complexity

---

## Repository Model

Summa uses a multi-repo strategy.

```text
summa-docs         — central documentation and governance
summa-mobile       — cross-platform mobile application (Flutter)
summa              — synchronization backend
summa-website      — official website
.github            — organization-wide GitHub configuration
summa-cloud        — private cloud infrastructure
```

All repositories share the same organization-level issue tracker and pull request workflow conventions. Cross-repo references use `org/repo#issue` notation where needed.

---

## Primary Branches

### `master`

The `master` branch contains:

- Stable code
- Released code
- Approved documentation
- Production-ready configuration

Direct pushes to `master` are forbidden.

---

### `develop`

The `develop` branch contains:

- Integrated development work
- Features planned for the next release
- Changes that passed review and automated checks

Direct pushes to `develop` are forbidden.

---

## Supporting Branches

Supported branch types:

```text
feature/
fix/
hotfix/
refactor/
docs/
test/
chore/
release/
```

---

## Branch Naming

Format:

```text
type/short-description
```

Examples:

```text
feature/mobile-dashboard
feature/transaction-create-flow
fix/category-delete-crash
refactor/mobile-data-layer
docs/security-model
test/sync-conflict-cases
chore/update-kotlin
release/1.0.0
hotfix/1.0.1-export-crash
```

Use lowercase kebab case.

Avoid:

```text
new-feature
danilo-branch
test123
final-version
fix
```

---

## Issue Workflow

Every meaningful change should begin with a GitHub Issue.

Exceptions:

- Typographical corrections
- Extremely small documentation fixes
- Automated dependency updates
- Emergency security work

---

## Issue Types

Recommended issue templates:

```text
Bug Report
Feature Request
Documentation
Security Improvement
Technical Task
Design Task
Research Task
```

---

## Issue Content

A good issue should include:

- Problem
- Motivation
- Proposed outcome
- Scope
- Out-of-scope items
- Acceptance criteria
- Relevant documentation
- Design reference where applicable

---

## Issue Status

Recommended GitHub Project statuses:

```text
Backlog
Ready
In Progress
In Review
Blocked
Done
```

---

## Development Workflow

```text
Create or select issue
        ↓
Move issue to Ready
        ↓
Create branch from develop
        ↓
Implement focused change
        ↓
Add or update tests
        ↓
Run local checks
        ↓
Open pull request to develop
        ↓
Automated checks
        ↓
Code review
        ↓
Address feedback
        ↓
Merge
        ↓
Close issue
```

---

## Starting a Branch

Update local branches first:

```bash
git checkout develop
git pull origin develop
```

Create the branch:

```bash
git checkout -b feature/mobile-dashboard
```

---

## Keeping a Branch Updated

Before final review:

```bash
git fetch origin
git rebase origin/develop
```

A merge from `develop` may be used when rebasing would create unnecessary risk, but clean branch history is preferred.

Never rebase a shared protected branch.

---

## Commit Convention

Summa uses Conventional Commit-style messages.

Format:

```text
type(scope): description
```

Examples:

```text
feat(mobile): add transaction creation screen
fix(backend): reject expired refresh tokens
docs(security): document local backup encryption
refactor(database): extract transaction mapper
test(mobile): add dashboard view model tests
chore(deps): update flutter dependencies
```

---

## Commit Types

```text
feat      New functionality
fix       Bug fix
docs      Documentation only
refactor  Internal change without feature or bug behavior
test      Test changes
perf      Performance improvement
style     Formatting without logic changes
build     Build system changes
ci        CI workflow changes
chore     Maintenance task
revert    Revert previous change
security  Security improvement
```

---

## Commit Scope

Recommended scopes:

```text
mobile
backend
api
database
sync
security
design
docs
ci
release
```

Feature-specific scopes may be used:

```text
dashboard
transactions
categories
profiles
statistics
```

---

## Commit Message Rules

The subject should:

- Use imperative form
- Be concise
- Avoid ending with a period
- Explain the change rather than the activity

Good:

```text
fix(mobile): preserve transaction note after validation error
```

Avoid:

```text
fixed stuff
changes
work on dashboard
final commit
```

---

## Commit Size

Commits should be logically focused.

One commit should ideally represent one coherent change.

Avoid combining:

- Large formatting changes
- Dependency updates
- Feature implementation
- Unrelated bug fixes

in the same commit.

---

## Pull Requests

Every pull request should include:

```md
## Summary

## Motivation

## Changes

## Testing

## Screenshots

## Documentation

## Related Issues

Closes #
```

Sections that do not apply may be marked as not applicable.

---

## Pull Request Size

Preferred:

```text
Small to medium
Focused on one problem
Reviewable in one session
```

Large changes should be split by:

- Architecture
- Data layer
- Domain logic
- UI
- Tests
- Documentation

A pull request should not be split in a way that leaves the branch or target branch broken.

---

## Draft Pull Requests

Draft pull requests should be used for:

- Early architecture feedback
- Work in progress
- Complex design discussions
- CI validation before review

Draft status must be removed before requesting final approval.

---

## Pull Request Requirements

A pull request must:

- Reference an issue when applicable
- Pass automated checks
- Include tests
- Update documentation where needed
- Avoid unrelated changes
- Contain no secrets
- Contain no generated build artifacts
- Include screenshots for visible UI changes
- Explain migrations where applicable

---

## Code Review

Reviewers should evaluate:

- Correctness
- Architecture
- Security
- Privacy
- Tests
- Readability
- Accessibility
- Performance
- Documentation
- Backward compatibility

---

## Review Comments

Review comments should be:

- Specific
- Respectful
- Actionable
- Focused on code or design
- Clear about whether a change is required or optional

Suggested prefixes:

```text
blocking:
suggestion:
question:
nit:
security:
```

---

## Required Approvals

Initial project policy:

```text
At least one approval
```

Sensitive changes should require additional review.

Examples:

- Authentication
- Authorization
- Encryption
- Database migrations
- Sync protocol
- Billing
- Release workflows

---

## Author Review

Before requesting review, the author must inspect the complete diff.

The author should check:

- Accidental files
- Debug code
- Secrets
- Generated files
- Incomplete comments
- Missing tests
- Incorrect formatting
- Unrelated changes

---

## Merge Strategy

Preferred merge strategy:

```text
Squash and merge
```

Reasons:

- Clean `develop` history
- One logical commit per pull request
- Easier revert
- Consistent commit messages

The final squash commit should follow the commit convention.

---

## Merge Commit Exceptions

A merge commit may be used for:

- Release branch integration
- Complex coordinated branches
- Cases where preserving branch history has clear value

---

## Branch Protection

Protect:

```text
master
develop
```

Required rules:

- Pull request required
- Required review
- Required status checks
- Branch must be up to date
- Force push disabled
- Branch deletion disabled for primary branches
- Direct push disabled
- Conversation resolution required

---

## Required Checks

Depending on changed paths:

### Mobile

```text
Build
Unit tests
Widget tests where applicable
dart format
dart analyze
```

### Backend

```text
Build
Unit tests
Integration tests
Formatting
Static analysis
```

### Documentation

```text
Markdown lint
Broken link check
Mermaid syntax validation where available
```

---

## CODEOWNERS

Recommended ownership examples:

```text
/mobile/               @summa/mobile
/backend/              @summa/backend
/docs/                 @summa/maintainers
/.github/workflows/    @summa/maintainers
/SECURITY.md           @summa/security
```

The exact team names depend on repository organization.

---

## Release Workflow

Normal release flow:

```text
develop
   ↓
release/x.y.z
   ↓
Release testing and fixes
   ↓
master
   ↓
Git tag
   ↓
Release artifacts
   ↓
Merge release changes back to develop
```

---

## Release Branch

Example:

```bash
git checkout develop
git checkout -b release/1.0.0
```

Only release-related changes are allowed:

- Version numbers
- Changelog
- Release notes
- Final bug fixes
- Store metadata
- Documentation corrections

No new features should be added.

---

## Hotfix Workflow

Hotfixes begin from `master`.

```text
master
  ↓
hotfix/x.y.z-description
  ↓
Review and test
  ↓
master
  ↓
Tag
  ↓
Merge back to develop
```

Example:

```text
hotfix/1.0.1-backup-restore-crash
```

---

## Versioning

Summa follows Semantic Versioning.

```text
MAJOR.MINOR.PATCH
```

Example:

```text
1.4.2
```

### Major

Breaking compatibility or major product generation.

### Minor

Backward-compatible functionality.

### Patch

Backward-compatible fixes.

---

## Pre-Release Versions

Examples:

```text
1.0.0-alpha.1
1.0.0-beta.1
1.0.0-rc.1
```

---

## Git Tags

Release tags use:

```text
v1.0.0
v1.1.0-beta.1
```

Tags should be created only from `master`.

---

## Changelog

The root repository contains:

```text
CHANGELOG.md
```

Recommended sections:

```text
Added
Changed
Deprecated
Removed
Fixed
Security
```

Changelog entries should describe user-visible or contributor-relevant changes.

---

## Documentation Changes

Documentation should be updated in the same pull request as the related implementation.

Examples:

- New API endpoint
- Database migration
- New environment variable
- New component
- Changed security behavior
- Modified deployment process

Documentation must not knowingly describe behavior that no longer exists.

---

## AI-Assisted Contributions

AI tools may be used, but generated code must be reviewed by a human contributor.

The contributor remains responsible for:

- Correctness
- Licensing
- Security
- Tests
- Architecture
- Documentation
- Understanding the submitted code

---

## AI Contribution Rules

AI-generated changes must not be merged when the author cannot explain:

- What the code does
- Why it is needed
- How it follows project architecture
- How it was tested
- What security implications exist

AI tools must not receive:

- Production secrets
- Real user financial data
- Private signing keys
- Confidential vulnerability reports
- Unredacted bank statements

---

## Security Rules

Never commit:

```text
.env
local.properties
Signing keys
Private keys
Database dumps
Real backups
Access tokens
Refresh tokens
Production configuration
User financial data
```

Secret scanning should run in CI.

A discovered secret must be rotated, not only removed from Git history.

---

## Generated Files

Generated build output should not be committed unless explicitly required.

Examples to ignore:

```text
build/
.gradle/
DerivedData/
node_modules/
coverage/
dist/
```

Generated API clients may be committed only if the project explicitly adopts that strategy.

---

## Repository Automation

Recommended GitHub automation:

- CI builds
- Test execution
- Linting
- Dependency update pull requests
- Secret scanning
- Code scanning
- Release notes
- Stale issue management with caution
- Documentation link checking

Automation must not close legitimate issues solely because they are old.

---

## Definition of Done

The Git workflow is ready when:

- Branch strategy is approved
- Branch protections are enabled
- Commit format is documented
- Pull request template exists
- Issue templates exist
- Required CI checks are defined
- Merge strategy is approved
- Release flow is documented
- Hotfix flow is documented
- AI contribution rules are documented
- Secret handling rules are documented

---

## Guiding Principle

Every change should be understandable, reviewable and reversible.

The workflow exists to improve quality without creating unnecessary bureaucracy.