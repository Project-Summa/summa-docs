# Security Implementation

> Implementing local security features for the Phase 1 MVP.

---

# Table of Contents

- Purpose
- Security Scope for Phase 1
- Application Lock
- PIN Authentication
- Biometric Authentication
- Secure Key Storage
- Local Database Protection
- Screen Protection
- Data Deletion
- Sensitive Data Handling
- Logging Restrictions
- Security Testing
- Future Security Phases
- Definition of Done
- Guiding Principle

---

# Purpose

This document defines how the security model from Phase 0 is implemented during Phase 1.

Phase 1 focuses exclusively on local security. No network communication exists, so transport security, API security and server-side concerns are deferred to Phase 3.

The goal is to protect user financial data against:

- Unauthorized physical access to the device
- Casual snooping by someone who borrows the device
- Data exposure through logs, screenshots or backups
- Accidental data loss

---

# Security Scope for Phase 1

## Included

- Application lock (PIN and biometrics)
- Secure key storage for PIN hash
- Screen protection (prevent screenshots in sensitive areas)
- Secure data deletion
- Sensitive data handling in logs
- Local backup integrity

## Excluded

- Network security (no network in Phase 1)
- Authentication tokens (no accounts in Phase 1)
- Workspace authorization (single user in Phase 1)
- Encryption at rest (deferred to Phase 2 review)
- Remote wipe (no server in Phase 1)

---

# Application Lock

The application lock prevents unauthorized access when the app is opened or resumed from background.

## Lock Triggers

The lock screen should appear when:

- The application is launched (cold start)
- The application is resumed from background after more than 60 seconds
- The user manually locks the app (if a lock shortcut exists)

## Lock Behavior

```text
App Launch / Resume
        │
        ▼
Is app lock enabled?
        │
    No ─┤── Yes
    │       │
    ▼       ▼
  Open   Is biometric enabled?
  App        │
         No ─┤── Yes
         │       │
         ▼       ▼
       Show    Show biometric prompt
       PIN        │
       Screen  Success ─┤── Failure / Cancel
                       │       │
                       ▼       ▼
                     Open    Show PIN
                     App     Screen
```

## Lock Screen UI

The lock screen should display:

- Summa logo or app name
- PIN input (6 digits)
- Biometric button (if available and enabled)
- Error message on incorrect PIN
- Lockout message after too many attempts

## Lockout Policy

| Attempts | Action |
|---|---|
| 1-4 | Allow retry immediately |
| 5 | 30 second cooldown |
| 10 | 5 minute cooldown |
| 20 | 15 minute cooldown |

After 20 failed attempts, the user must wait or use biometrics.

---

# PIN Authentication

## PIN Requirements

- Exactly 6 digits
- Numeric only (0-9)
- Cannot be all same digit (e.g., 111111)
- Cannot be sequential (e.g., 123456)

## PIN Setup Flow

```text
Settings → Security → Enable App Lock
        │
        ▼
    Enter new PIN
        │
        ▼
    Confirm PIN
        │
        ▼
    PINs match?
        │
    No ─┤── Yes
    │       │
    ▼       ▼
  Retry   Hash and store PIN
          Enable app lock
```

## PIN Storage

The PIN is never stored in plain text.

```text
PIN (6 digits)
    │
    ▼
Salt (random, stored per device)
    │
    ▼
Argon2id hash
    │
    ▼
Stored in flutter_secure_storage
```

## PIN Verification

```text
User enters PIN
    │
    ▼
Retrieve salt from secure storage
    │
    ▼
Hash entered PIN with salt (Argon2id)
    │
    ▼
Compare with stored hash
    │
    ▼
Match → Allow access
No match → Increment attempt counter
```

## PIN Change Flow

```text
Settings → Security → Change PIN
        │
        ▼
    Enter current PIN
        │
        ▼
    Verify current PIN
        │
        ▼
    Enter new PIN
        │
        ▼
    Confirm new PIN
        │
        ▼
    Hash and store new PIN
```

## PIN Disable Flow

```text
Settings → Security → Disable App Lock
        │
        ▼
    Enter current PIN
        │
        ▼
    Verify current PIN
        │
        ▼
    Remove PIN from secure storage
    Disable app lock
```

---

# Biometric Authentication

## Supported Methods

| Platform | Method | Library |
|---|---|---|
| Android | BiometricPrompt | local_auth |
| iOS | LocalAuthentication | local_auth |

## Biometric Setup

Biometric authentication can only be enabled after PIN is set.

```text
Settings → Security → Enable Biometrics
        │
        ▼
    Is device biometric capable?
        │
    No ─┤── Yes
    │       │
    ▼       ▼
  Show    Is PIN already set?
  error       │
          No ─┤── Yes
          │       │
          ▼       ▼
        Prompt   Enable biometric
        to set   Store preference
        PIN first
```

## Biometric Verification

```text
Biometric prompt shown
        │
    Success ─┤── Failure
        │           │
        ▼           ▼
    Allow       Fall back to PIN
    access      screen
```

## Biometric Fallback

If biometric authentication fails:

- The user can always fall back to PIN
- After 3 biometric failures, PIN screen is shown automatically
- Biometric is not required — it is a convenience feature

---

# Secure Key Storage

## flutter_secure_storage

All sensitive values are stored using flutter_secure_storage.

| Platform | Backend |
|---|---|
| Android | Android Keystore (EncryptedSharedPreferences) |
| iOS | iOS Keychain |

## Stored Values

| Key | Value | Purpose |
|---|---|---|
| `pin_hash` | Argon2id hash of PIN | PIN verification |
| `pin_salt` | Random salt | PIN hashing |
| `biometric_enabled` | Boolean | Biometric preference |
| `lock_timeout_seconds` | Integer | Background lock timeout |

## Values NOT Stored in Secure Storage

| Value | Storage | Reason |
|---|---|---|
| Financial data | drift (SQLite) | Too large for key-value store |
| User preferences | shared_preferences | Not sensitive |
| App settings | shared_preferences | Not sensitive |

---

# Local Database Protection

## Phase 1 Approach

In Phase 1, the SQLite database is stored in the application's private directory.

- Android: `/data/data/com.projectsumma.summa/databases/`
- iOS: App sandbox (protected by iOS app sandboxing)

The database is NOT encrypted at rest in Phase 1.

## Why No Encryption Yet

- iOS already encrypts app data when the device is locked (Data Protection)
- Android provides file-based encryption on modern devices
- Adding application-level encryption complicates backup and restore
- Encryption at rest is under review for Phase 2

## Future Consideration

If encryption is added:

- Use SQLCipher for SQLite encryption
- Store the encryption key in flutter_secure_storage
- Ensure backup/restore handles encrypted databases
- Consider performance impact on large databases

---

# Screen Protection

## Screenshot Prevention

On Android, prevent screenshots in sensitive screens:

```dart
import 'package:flutter_windowmanager/flutter_windowmanager.dart';

// Disable screenshots
await FlutterWindowManager.addFlags(
  FlutterWindowManager.FLAG_SECURE,
);

// Re-enable screenshots
await FlutterWindowManager.clearFlags(
  FlutterWindowManager.FLAG_SECURE,
);
```

## Screens to Protect

- Application lock screen
- PIN entry screens
- Backup restore confirmation
- Any screen showing full financial data

## iOS Limitation

iOS does not provide a programmatic way to prevent screenshots.

Mitigation:

- Hide sensitive content in app switcher preview
- Use `WidgetsBindingObserver` to detect app lifecycle
- Blur or hide content when app goes to background

---

# Data Deletion

## Soft Delete

All entity deletions are soft deletes.

```text
User deletes transaction
    │
    ▼
Set deleted_at = current timestamp
    │
    ▼
Record disappears from UI
    │
    ▼
Data remains in database for backup/restore
```

## Hard Delete (Data Deletion Request)

When a user requests complete data deletion:

```text
Settings → Data → Delete All Data
        │
        ▼
    Show warning dialog
    "This will permanently delete all data"
        │
        ▼
    Confirm with PIN
        │
        ▼
    Delete all database records
    Delete all attachments
    Delete all backups
    Clear secure storage
    Reset app to initial state
```

## Deletion Verification

After deletion:

- Database contains zero user records
- Attachment files are removed
- Backup files are removed
- Secure storage is cleared
- App returns to onboarding state

---

# Sensitive Data Handling

## What Is Sensitive

| Data | Sensitivity | Handling |
|---|---|---|
| PIN | Critical | Argon2id hashed, stored in secure storage |
| Financial amounts | High | Never logged, never in error messages |
| Transaction notes | High | Never logged |
| Profile names | Medium | Never logged in error context |
| Category names | Low | Acceptable in debug logs |

## Rules

- Financial data never appears in logs
- Financial data never appears in error messages sent to crash reporting
- Financial data never appears in analytics (if analytics are ever added)
- PIN is never stored in plain text
- PIN is never logged
- Backup files are stored in app-private directory

---

# Logging Restrictions

## Forbidden in Logs

- PIN values
- Financial amounts
- Transaction details
- Profile information
- Database contents
- Secure storage contents
- Backup file contents

## Allowed in Logs

- Feature navigation events
- Error categories (not details)
- Performance metrics
- Database operation success/failure (not content)
- Authentication success/failure (not credentials)

## Implementation

```dart
// Forbidden
logger.debug('Transaction amount: ${transaction.amount}');

// Allowed
logger.debug('Transaction created successfully');

// Forbidden
logger.error('PIN verification failed for PIN: $pin');

// Allowed
logger.error('PIN verification failed');
```

---

# Security Testing

## Unit Tests

- PIN validation rules
- PIN hashing and verification
- Lockout policy enforcement
- Soft delete behavior
- Data deletion completeness

## Integration Tests

- Full PIN setup and verification flow
- Biometric enable and disable
- Lock screen appearance and dismissal
- Backup and restore integrity
- Data deletion and app reset

## Security Test Cases

| Test | Expected Result |
|---|---|
| Enter correct PIN | Access granted |
| Enter incorrect PIN | Access denied, attempt counted |
| 5 failed attempts | 30 second lockout |
| Biometric success | Access granted |
| Biometric failure | Fallback to PIN |
| App backgrounded 60s | Lock screen shown |
| Screenshot on lock screen | Blocked (Android) |
| Delete all data | Database empty, app reset |
| Backup after delete | Backup contains no user data |

---

# Future Security Phases

## Phase 2

- Database encryption at rest (SQLCipher)
- Encrypted backup archives
- Enhanced biometric policies

## Phase 3

- Transport security (HTTPS)
- Authentication tokens
- Workspace authorization
- Device management
- Session management
- Remote data wipe

## Phase 4

- Cloud security infrastructure
- Penetration testing
- Security audit
- Bug bounty program

---

# Definition of Done

Security implementation is complete when:

- PIN setup, change and disable work correctly
- Biometric authentication works on supported devices
- Lock screen appears on app launch and resume
- Lockout policy prevents brute force
- Sensitive data never appears in logs
- Screenshots are blocked on Android lock screens
- Data deletion removes all user data
- All security flows have unit and integration tests
- No sensitive data is stored in plain text

---

# Guiding Principle

Security should be invisible when it works and clear when it fails.

Users should feel confident that their financial data is protected without needing to understand the implementation details.

Every security feature should degrade gracefully — biometric failure falls back to PIN, lockout has a timeout, data deletion requires confirmation.

The goal is protection, not paranoia.
