# Security Model

> Defining the security, privacy and trust boundaries of the Summa ecosystem.

---

## Table of Contents

- [Purpose](#purpose)
- [Security Objectives](#security-objectives)
- [Security Principles](#security-principles)
- [Deployment Modes](#deployment-modes)
- [Threat Model](#threat-model)
- [Data Classification](#data-classification)
- [Local Application Security](#local-application-security)
- [Local Database Protection](#local-database-protection)
- [Application Lock](#application-lock)
- [Secure Key Storage](#secure-key-storage)
- [Authentication](#authentication)
- [Password Security](#password-security)
- [Session and Token Security](#session-and-token-security)
- [Device Security](#device-security)
- [Authorization](#authorization)
- [Workspace Isolation](#workspace-isolation)
- [Transport Security](#transport-security)
- [API Security](#api-security)
- [Attachment Security](#attachment-security)
- [Backup Security](#backup-security)
- [Self-Hosted Security](#self-hosted-security)
- [Summa Cloud Security](#summa-cloud-security)
- [Logging and Telemetry](#logging-and-telemetry)
- [Secrets Management](#secrets-management)
- [Dependency and Supply Chain Security](#dependency-and-supply-chain-security)
- [Data Export and Deletion](#data-export-and-deletion)
- [Security Testing](#security-testing)
- [Vulnerability Reporting](#vulnerability-reporting)
- [Incident Response](#incident-response)
- [Security Limitations](#security-limitations)
- [Definition of Done](#definition-of-done)
- [Guiding Principle](#guiding-principle)

---

## Purpose

This document defines the security model for Summa.

Summa processes sensitive personal and household financial information. Security must therefore be treated as a core architectural requirement rather than an optional feature.

The security model covers:

- Local-only mobile usage
- Self-hosted synchronization
- Summa Cloud
- Authentication
- Authorization
- Encryption
- Backup
- Attachments
- Logging
- Incident response
- Secure development practices

---

## Security Objectives

The security architecture should protect:

- Confidentiality of financial data
- Integrity of financial records
- Availability of local and synchronized data
- User ownership of data
- Authentication credentials
- Device sessions
- Backup archives
- Attachments
- Workspace boundaries

The system should minimize the consequences of:

- Lost or stolen devices
- Compromised credentials
- Misconfigured self-hosted servers
- Malicious API requests
- Vulnerable dependencies
- Unauthorized workspace access
- Database leakage
- Backup exposure
- Server compromise

---

## Security Principles

### Privacy by Default

The safest reasonable configuration should be the default.

Users should not need advanced technical knowledge to avoid unnecessary data exposure.

---

### Least Privilege

Every user, device, service and process should receive only the permissions required for its role.

---

### Defense in Depth

Security must not depend on a single mechanism.

Examples:

- Device lock
- Application lock
- Database encryption
- Secure token storage
- Server authorization
- HTTPS
- Backup encryption

---

### Explicit Trust Boundaries

The system must clearly distinguish between:

- Trusted local application code
- Local device storage
- Operating system secure storage
- Untrusted networks
- Self-hosted infrastructure
- Summa Cloud infrastructure
- Third-party providers

---

### Minimize Sensitive Data

The system should collect, transmit and store only the information required for its functionality.

---

### Secure Failure

When a security-sensitive operation fails, the system should deny access rather than silently continue in an unsafe state.

---

## Deployment Modes

### Local-Only Mode

```text
Mobile Application
        ↓
Local Database
```

Characteristics:

- No Summa account
- No remote server
- No network transmission
- Security depends primarily on the device and application protection

---

### Self-Hosted Mode

```text
Mobile Application
        ↓
User-Managed Summa Server
        ↓
PostgreSQL and Attachment Storage
```

Characteristics:

- User controls infrastructure
- User is responsible for server configuration
- Summa provides secure defaults and deployment guidance
- Mobile and server security both apply

---

### Summa Cloud Mode

```text
Mobile Application
        ↓
Summa Cloud
        ↓
Managed Database, Storage and Backups
```

Characteristics:

- Summa manages infrastructure
- Authentication and subscription services are enabled
- Operational security becomes the responsibility of the Summa project

---

## Threat Model

The security model considers the following threat actors.

### Unauthorized Local User

A person gains physical access to an unlocked or insufficiently protected device.

Possible goals:

- Read financial history
- Export data
- Modify transactions
- Access attachments

---

### Stolen Device Attacker

A device is lost or stolen.

Possible goals:

- Extract the local database
- Reuse stored sessions
- Access backups
- Read notification content

---

### Remote Attacker

An attacker interacts with a public Summa server.

Possible goals:

- Brute-force login
- Exploit API validation
- Access another workspace
- Upload malicious attachments
- Abuse synchronization
- Cause denial of service

---

### Malicious Workspace Member

A valid workspace member attempts to access or modify data outside their permissions.

---

### Compromised Self-Hosted Administrator

A person with server-level access may access:

- PostgreSQL
- Attachment storage
- Environment variables
- Backups
- Logs

Summa cannot fully protect plaintext server-side data from a fully privileged server administrator unless end-to-end encryption is introduced in a future phase.

---

### Compromised Dependency

A third-party library, build plugin or container image introduces malicious or vulnerable code.

---

### Misconfiguration

A self-hosted deployment may accidentally expose:

- Database ports
- Internal services
- Plain HTTP endpoints
- Public attachment storage
- Weak credentials
- Unprotected backups

---

## Data Classification

### Highly Sensitive

- Financial amounts
- Transaction notes
- Merchant names
- Bank statements
- Receipt images
- Invoice images
- Account backups
- Workspace financial history

---

### Sensitive

- Email addresses
- User names
- Workspace membership
- Device information
- Session metadata
- Audit events
- Subscription status

---

### Secret

- Passwords
- Password hashes
- Access tokens
- Refresh tokens
- Encryption keys
- API credentials
- Signing keys
- Database credentials
- Storage credentials

---

### Public or Low Sensitivity

- Application version
- API version
- Public documentation
- Open-source code
- Generic health status

---

## Local Application Security

The mobile application must:

- Store financial data only in application-private storage
- Avoid writing sensitive data to shared storage
- Prevent sensitive data from appearing in logs
- Avoid including financial data in crash reports
- Use platform-secure storage for secrets
- Support application locking
- Respect operating system backup configuration
- Avoid unnecessary permissions
- Validate imported files before processing

---

## Local Database Protection

The local database contains highly sensitive information.

Recommended protection strategy:

- Store the database inside application-private storage
- Encrypt the database where technically practical
- Store encryption keys outside the database
- Protect encryption keys using operating system secure storage
- Never hardcode encryption keys
- Never derive database keys from predictable values

Encryption should use an authenticated encryption mechanism.

The exact database encryption implementation must be validated separately for Android and iOS.

---

## Database Encryption States

### Phase 1 Initial Release

At minimum:

- Application-private database storage
- Device-level protection
- No shared external database files
- Secure local backup handling

### Phase 1 Security Upgrade

Target:

- Encrypted local database
- Securely generated database key
- Key protection through Android Keystore or iOS Keychain
- Recovery strategy documented

Database encryption must not be introduced without a reliable backup and recovery plan.

---

## Application Lock

Summa should support an optional application lock.

Supported methods:

- Device biometrics
- Device credentials
- Application PIN where appropriate

The application lock should protect:

- Opening the application
- Viewing financial details
- Exporting data
- Changing security settings
- Connecting a remote server
- Restoring backups

---

## Biometric Authentication

Biometric authentication should use platform APIs.

Android:

```text
BiometricPrompt
```

iOS:

```text
LocalAuthentication
```

Summa must never receive or store raw biometric data.

Biometrics only authorize access to a key or protected operation managed by the operating system.

---

## Application PIN

An application PIN may be supported as an alternative.

Requirements:

- Never store plaintext PIN
- Apply attempt limits
- Introduce delays after repeated failures
- Avoid weak default PINs
- Provide a secure reset strategy
- Clearly explain whether reset deletes local encryption keys

A forgotten PIN must not be recoverable through insecure questions or plaintext storage.

---

## Screen Protection

Where supported, the application may prevent or obscure:

- Screenshots
- Recent-app previews
- Screen recording

This behavior should be configurable where platform conventions allow it.

The application should at minimum obscure sensitive views in the recent-app switcher.

---

## Secure Key Storage

### Android

Secrets should be protected using:

```text
Android Keystore
```

Possible stored secrets:

- Database encryption key wrapper
- Access token
- Refresh token
- Backup encryption key reference
- Device private key in future versions

---

### iOS

Secrets should be protected using:

```text
iOS Keychain
```

Appropriate accessibility classes should be selected based on whether synchronization must work while the device is locked.

---

## Authentication

Authentication is required only for remote synchronization.

Local-only users do not need an account.

Supported initial method:

- Email and password

Future methods:

- Passkeys
- OAuth providers
- Enterprise single sign-on where justified

---

## Password Security

Password requirements should prioritize length over arbitrary complexity.

The server should:

- Reject known-invalid or empty passwords
- Enforce a reasonable minimum length
- Allow password managers
- Allow long passwords
- Avoid silently truncating passwords
- Avoid requiring frequent forced rotation
- Rate-limit authentication attempts

Passwords must be hashed using:

```text
Argon2id
```

The exact parameters must be configurable and reviewed before production deployment.

Passwords must never be:

- Logged
- Emailed
- Stored in plaintext
- Returned by the API
- Included in analytics

---

## Session and Token Security

The session model should use:

- Short-lived access tokens
- Rotating refresh tokens
- Device-specific sessions
- Session revocation
- Refresh token reuse detection

---

## Access Tokens

Access tokens should:

- Have a short expiration time
- Contain minimal claims
- Be cryptographically signed
- Include user and session identifiers
- Avoid containing financial information
- Be validated on every protected request

---

## Refresh Tokens

Refresh tokens should:

- Be random and high entropy
- Be stored securely on the client
- Be stored only as hashes on the server
- Rotate after successful refresh
- Be invalidated after logout
- Be tied to a device session
- Support reuse detection

If an already-used refresh token is presented again, the associated token family should be revoked.

---

## Session Revocation

Users should be able to:

- Log out the current session
- Revoke a specific device
- Revoke all other devices
- Revoke all sessions
- Review recent session activity

Sensitive account changes may invalidate existing sessions.

---

## Device Security

Every synchronized device should have a unique identifier.

Device identifiers must not be derived from hardware identifiers that violate platform privacy expectations.

The server should track:

- Device ID
- User ID
- Platform
- Application version
- Created time
- Last seen time
- Revocation time

A revoked device must not be able to refresh its session.

---

## Authorization

Authentication confirms identity.

Authorization determines allowed actions.

Authorization must be enforced on the backend for every protected operation.

The UI may hide unavailable actions, but UI visibility is not a security boundary.

---

## Workspace Roles

Initial roles:

```text
owner
admin
member
viewer
```

Permissions should be defined explicitly.

Example:

| Action | Owner | Admin | Member | Viewer |
|---|---:|---:|---:|---:|
| View financial data | Yes | Yes | Yes | Yes |
| Create transaction | Yes | Yes | Yes | No |
| Edit transaction | Yes | Yes | Yes | No |
| Manage categories | Yes | Yes | Yes | No |
| Invite members | Yes | Yes | No | No |
| Change roles | Yes | Limited | No | No |
| Delete workspace | Yes | No | No | No |

The final permission matrix must be validated before backend implementation.

---

## Workspace Isolation

Every workspace-scoped query must include workspace ownership validation.

The backend must prevent:

- Reading another workspace's data
- Updating another workspace's data
- Referencing categories from another workspace
- Downloading attachments from another workspace
- Using another workspace's sync cursor
- Applying changes to another workspace

Workspace isolation must be verified through automated integration tests.

---

## Transport Security

Production communication must use HTTPS.

Requirements:

- TLS termination
- Valid certificates
- Secure redirect from HTTP where applicable
- No mixed-content web access
- No plaintext credentials
- No tokens in query strings

Self-hosted examples should assume a reverse proxy such as Caddy, Nginx or Traefik.

---

## Certificate Validation

Mobile clients must validate server certificates.

Disabling certificate validation must not be available in production builds.

Self-signed certificates may be supported only through an explicit development or advanced self-hosted configuration.

---

## API Security

The API must implement:

- Input validation
- Request size limits
- Rate limiting
- Authentication
- Authorization
- Safe error messages
- Content type validation
- Idempotency protection
- Pagination limits
- Attachment validation
- Request correlation IDs

---

## Brute-Force Protection

Sensitive endpoints require stricter controls.

Examples:

- Login
- Registration
- Password reset
- Refresh token
- Invitation acceptance

Controls may include:

- Per-IP rate limits
- Per-account rate limits
- Increasing delays
- Temporary lockouts
- Security alerts
- Abuse monitoring

The system must avoid allowing attackers to easily determine whether an email address exists.

---

## Attachment Security

Attachments must be considered untrusted input.

Requirements:

- File size limits
- MIME validation
- File signature validation where practical
- Randomized storage names
- No direct execution
- No public storage buckets
- Authorization before download
- Safe download headers
- Workspace ownership checks
- Optional malware scanning in hosted environments

Original filenames should be treated as metadata only.

They must not determine the physical storage path.

---

## Image Processing

Image processing libraries may contain vulnerabilities.

Receipt images should be:

- Decoded using maintained libraries
- Subject to dimension limits
- Subject to memory limits
- Processed outside sensitive request paths where practical
- Re-encoded before long-term storage where justified

---

## Backup Security

Backups may contain the complete financial history of a user or workspace.

Backups must be treated as highly sensitive.

Requirements:

- Encryption
- Integrity verification
- Documented restoration
- Restricted access
- Separate storage from the primary system
- Retention policy
- Secure deletion policy

---

## Local Backup Encryption

Encrypted local backups should use:

- A versioned archive format
- Authenticated encryption
- Strong key derivation for password-protected backups
- Integrity metadata
- Explicit format version
- Clear recovery instructions

The backup format must remain portable and documented.

---

## Backup Passwords

When users protect backups with a password:

- The password must not be stored in the backup
- The password must not be transmitted to Summa
- The user must be warned that forgotten passwords cannot be recovered
- Weak password warnings may be displayed
- Password managers should be supported

---

## Self-Hosted Security

Self-hosted deployments should include secure defaults.

Official documentation must warn against:

- Exposing PostgreSQL publicly
- Using default passwords
- Running without HTTPS
- Storing backups on the same disk only
- Using publicly readable storage
- Running outdated versions
- Exposing internal health information
- Disabling authentication

---

## Container Security

Official container images should:

- Use minimal base images
- Run as a non-root user where practical
- Pin important dependency versions
- Avoid unnecessary packages
- Include health checks
- Avoid embedded secrets
- Publish image checksums or signatures where practical

---

## Summa Cloud Security

Summa Cloud should use:

- Managed secrets
- Restricted service identities
- Private database networking
- Encrypted storage
- Encrypted backups
- Centralized audit logs
- Operational monitoring
- Controlled production access
- Multi-factor authentication for administrators

Production access should be limited, logged and reviewed.

---

## Administrative Access

Administrators must not access user financial information without a justified operational or legal reason.

Access controls should include:

- Strong authentication
- Multi-factor authentication
- Least privilege
- Approval for elevated access
- Audit logging
- Time-limited access where practical

---

## Logging and Telemetry

Financial content must not be logged.

Forbidden log fields include:

- Financial amounts
- Transaction notes
- Merchant names
- Bank statement contents
- Receipt contents
- Passwords
- Access tokens
- Refresh tokens
- Backup encryption keys

---

## Allowed Operational Logging

Logs may include:

- Request ID
- Route template
- HTTP method
- Status code
- Duration
- Error category
- User ID where justified
- Workspace ID where justified
- Application version
- Server version

Identifiers should be minimized when not required.

---

## Analytics

Summa should not include behavioral analytics that inspect financial activity.

Optional privacy-respecting operational metrics may include:

- Application version distribution
- Crash category
- Feature availability
- Sync success rate
- API latency

Telemetry should be opt-in where appropriate and must be documented transparently.

---

## Secrets Management

Secrets include:

- Database passwords
- Token signing keys
- SMTP credentials
- Storage credentials
- Push credentials
- Billing provider secrets

Requirements:

- Never commit secrets to Git
- Never include real secrets in example files
- Use environment variables or secret stores
- Rotate secrets when exposure is suspected
- Restrict access
- Separate development and production secrets

---

## Local Development Secrets

Local development should use:

```text
.env
```

The file must be ignored by Git.

The repository should provide:

```text
.env.example
```

Example values must be fictional and non-sensitive.

---

## Dependency and Supply Chain Security

The project should:

- Use dependency lock files
- Review new dependencies
- Prefer maintained libraries
- Remove unused dependencies
- Scan for known vulnerabilities
- Protect release workflows
- Restrict CI secrets
- Review automated dependency updates
- Pin GitHub Actions to trusted versions or commit hashes

---

## Build Security

Release builds should be reproducible where practical.

Production signing keys must:

- Never be stored in the repository
- Be access-controlled
- Be backed up securely
- Be rotated according to platform limitations
- Be available only to authorized release workflows

---

## Data Export and Deletion

Users must be able to export their data before deletion.

Deletion workflows must clearly distinguish between:

- Local application data deletion
- Remote account deletion
- Workspace deletion
- Member removal
- Device revocation
- Backup retention

---

## Local Data Deletion

Deleting local data should remove:

- Local database
- Local attachments
- Local cached exports
- Stored tokens
- Encryption keys where appropriate

The user must receive a clear confirmation before irreversible deletion.

---

## Cloud Account Deletion

Cloud account deletion should:

- Require re-authentication
- Provide a final export opportunity
- Revoke sessions
- Remove active data
- Document backup retention
- Document legal retention where applicable
- Notify the user when deletion is complete

---

## Security Testing

Security testing should include:

- Unit tests
- Authorization tests
- Workspace isolation tests
- API validation tests
- Dependency scanning
- Static analysis
- Secret scanning
- Container scanning
- Backup restoration tests
- Authentication abuse tests
- Attachment upload tests

---

## Required Security Test Cases

The test suite must verify:

- One workspace cannot access another workspace
- A viewer cannot modify transactions
- A revoked session cannot refresh
- A removed member loses access
- Expired tokens are rejected
- Invalid signatures are rejected
- Duplicate sync requests remain safe
- Attachment paths cannot be manipulated
- Oversized payloads are rejected
- Sensitive data is not written to logs

---

## Vulnerability Reporting

The root repository should contain:

```text
SECURITY.md
```

The document should explain:

- How to report a vulnerability privately
- Which versions are supported
- Expected response process
- What information to include
- Why public disclosure should wait for coordination

Security issues should not initially be reported through public GitHub Issues.

---

## Incident Response

A security incident process should include:

1. Detection
2. Triage
3. Containment
4. Investigation
5. Remediation
6. Recovery
7. User communication
8. Post-incident review

Potential actions include:

- Revoking sessions
- Rotating secrets
- Disabling affected endpoints
- Releasing a security update
- Restoring from backup
- Notifying affected users

---

## Security Limitations

Summa cannot guarantee protection against:

- A fully compromised operating system
- A rooted or jailbroken device with active attacker control
- A malicious self-hosted system administrator
- Weak user passwords
- Unencrypted user-created exports
- Unsafe third-party backup destinations
- Physical access to an already unlocked device

These limitations should be documented honestly.

---

## Definition of Done

The security model is considered complete when:

- Threat actors are documented
- Sensitive data is classified
- Local storage protection is defined
- Application lock behavior is defined
- Authentication flow is documented
- Token lifecycle is documented
- Workspace authorization is defined
- Transport requirements are defined
- Attachment security is defined
- Backup protection is defined
- Logging restrictions are defined
- Self-hosted security guidance is planned
- Security testing requirements are identified
- Vulnerability reporting process is planned

---

## Guiding Principle

Summa should collect as little sensitive data as possible, protect what it stores and make security boundaries understandable to both users and contributors.

Privacy and security must remain part of every feature from design through implementation.