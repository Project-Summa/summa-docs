# Release Plan

> Defining how Phase 1 is released to users.

---

# Table of Contents

- Purpose
- Release Philosophy
- Release Phases
- Beta Testing
- Store Preparation
- Android Release
- iOS Release
- Crash Reporting
- Release Notes
- Post-Release
- Definition of Done
- Guiding Principle

---

# Purpose

This document defines the release process for Phase 1.

The goal is to ship a stable, usable application to real users while maintaining quality and catching issues before they reach everyone.

---

# Release Philosophy

## Ship Incrementally

Don't wait for perfection. Ship when the MVP is stable and useful.

Internal testing → Closed beta → Open beta → Public release

Each phase catches different types of issues.

---

## Quality Over Speed

A delayed release is better than a broken release.

Financial data is sensitive. Users trust the application with their money. Bugs that corrupt or lose data are unacceptable.

---

## Learn From Users

Beta users provide feedback that internal testing cannot.

Real usage patterns reveal:

- Unexpected workflows
- Edge cases in financial data
- Performance issues on real devices
- Confusing UI patterns
- Missing features

---

# Release Phases

## Phase 1 — Internal Testing

Duration: 1-2 weeks

Participants: Core team

Goal: Verify all features work correctly on real devices

Activities:

- Test all P0 features on Android and iOS
- Test on multiple device sizes
- Test with real financial data
- Verify backup and restore
- Verify application lock
- Test dark theme
- Check performance
- Fix critical bugs

Exit Criteria:

- No crash bugs
- All P0 features work
- Data is never lost
- Application lock works reliably

---

## Phase 2 — Closed Beta

Duration: 2-4 weeks

Participants: 10-30 invited users

Goal: Validate usability and catch edge cases

Activities:

- Distribute via Firebase App Distribution (Android) or TestFlight (iOS)
- Collect feedback via form or email
- Monitor crash reports
- Track common issues
- Iterate on feedback

Exit Criteria:

- Crash rate below 1%
- No data loss reports
- Core workflows are understandable
- Major usability issues are fixed

---

## Phase 3 — Open Beta

Duration: 2-4 weeks

Participants: Public opt-in

Goal: Scale testing and validate stability

Activities:

- Open beta on Google Play (Android)
- Open beta via TestFlight (iOS)
- Monitor crash reports at scale
- Collect ratings and reviews
- Fix remaining issues

Exit Criteria:

- Crash rate below 0.5%
- No critical bugs
- Performance is acceptable on mid-range devices
- Store listing is ready

---

## Phase 4 — Public Release

Duration: Ongoing

Participants: All users

Goal: Ship the stable release

Activities:

- Submit to Google Play Store
- Submit to Apple App Store
- Monitor crash reports
- Respond to user reviews
- Plan next release

---

# Beta Testing

## Beta Tester Recruitment

Target beta testers:

- Personal finance enthusiasts
- Privacy-conscious users
- Flutter developers (for technical feedback)
- Users who currently use spreadsheets for budgeting
- Users of competing apps (for comparison feedback)

## Beta Feedback Collection

Provide beta testers with:

- Feedback form (Google Form or similar)
- Bug report template
- Feature request channel
- Direct email for urgent issues

## Beta Feedback Categories

| Category | Priority | Action |
|---|---|---|
| Crash / Data loss | Critical | Fix immediately |
| Core feature broken | High | Fix before next beta |
| Usability issue | Medium | Fix before public release |
| Feature request | Low | Add to backlog |
| Visual issue | Low | Fix if time permits |

---

# Store Preparation

## App Icon

Requirements:

- Android: 512x512 PNG, adaptive icon
- iOS: 1024x1024 PNG, no alpha channel
- Consistent branding across platforms

## Screenshots

Requirements:

- Android: At least 2 screenshots, recommended 4-8
- iOS: At least 3 screenshots per device size
- Show key features: dashboard, transactions, statistics, categories

Screenshot Plan:

1. Dashboard with balance and recent transactions
2. Transaction creation screen
3. Category list with icons and colors
4. Monthly statistics with chart
5. Dark theme variant
6. Application lock screen

## App Description

Short description (80 characters max):

```text
Privacy-first personal finance tracker. Your data stays on your device.
```

Full description:

```text
Summa is a personal finance application that works entirely on your device.
No cloud required. No data harvesting. No subscriptions needed to access
your own financial history.

Features:
• Track expenses, income and transfers
• Organize with custom categories
• View balance and spending trends
• Export data as JSON or CSV
• Local backup and restore
• Secure with PIN or biometrics
• Light and dark themes
• Works completely offline

Your money, your data, your rules.
```

## Store Categories

- Android: Finance
- iOS: Finance

## Content Rating

- Android: Everyone (no objectionable content)
- iOS: 4+ (no objectionable content)

## Privacy Policy

Required for both stores.

Key points:

- No data collection
- No analytics
- No tracking
- No network requests
- All data stored locally on device
- No third-party services

---

# Android Release

## Build Configuration

```text
android/
    app/
        build.gradle
            - applicationId: com.projectsumma.summa
            - minSdkVersion: 26
            - targetSdkVersion: 34
            - versionCode: 1
            - versionName: 1.0.0
```

## Signing

- Generate upload keystore
- Store keystore securely (not in repository)
- Configure signing in build.gradle
- Use Play App Signing for distribution

## Release Build

```bash
flutter build appbundle --release
```

## Google Play Submission

1. Create Google Play Developer account
2. Create application listing
3. Upload AAB to internal testing track
4. Promote to closed beta
5. Promote to open beta
6. Promote to production

---

# iOS Release

## Build Configuration

```text
ios/
    Runner/
        Info.plist
            - CFBundleIdentifier: com.projectsumma.summa
            - CFBundleShortVersionString: 1.0.0
            - CFBundleVersion: 1
```

## Signing

- Apple Developer account required
- Configure signing in Xcode
- Use automatic signing for development
- Use manual signing for CI/CD

## Release Build

```bash
flutter build ipa --release
```

## App Store Submission

1. Create App Store Connect listing
2. Upload IPA via Transporter or Xcode
3. Submit for TestFlight beta
4. Submit for App Review
5. Release to App Store

---

# Crash Reporting

## Tool

Firebase Crashlytics

## Integration

```dart
// Initialize in main.dart
await Firebase.initializeApp();
FlutterError.onError = FirebaseCrashlytics.instance.recordFlutterFatalError;
PlatformDispatcher.instance.onError = (error, stack) {
  FirebaseCrashlytics.instance.recordError(error, stack, fatal: true);
  return true;
};
```

## Crash Reporting Rules

- Financial data must NEVER appear in crash reports
- User notes and merchant names must be filtered
- Profile names must be filtered
- Transaction amounts must be filtered
- Only stack traces and device info are sent

## Crash Triage

| Severity | Response Time | Action |
|---|---|---|
| Crash on launch | < 24 hours | Hotfix |
| Crash in core flow | < 48 hours | Hotfix |
| Crash in edge case | < 1 week | Next release |
| Non-crash bug | Next release | Backlog |

---

# Release Notes

## Format

```text
## 1.0.0

First release of Summa — your privacy-first finance tracker.

### Features
- Track expenses, income and transfers
- Organize transactions with custom categories
- View balance and spending on the dashboard
- Monthly and yearly statistics
- Export data as JSON or CSV
- Local backup and restore
- Secure with PIN or biometrics
- Light and dark themes

### Notes
- Works completely offline
- No account required
- No data leaves your device
```

## Version Numbering

Follow semantic versioning:

- Major: Breaking changes or major feature additions
- Minor: New features, backward compatible
- Patch: Bug fixes and improvements

Phase 1 releases: 1.0.0 → 1.0.1 → 1.1.0 → 1.2.0 → ...

---

# Post-Release

## Week 1

- Monitor crash reports hourly
- Respond to critical issues immediately
- Monitor user reviews
- Collect feedback

## Weeks 2-4

- Monitor crash reports daily
- Fix high-priority bugs
- Plan first patch release
- Analyze usage patterns (if analytics are added)

## Month 2+

- Monitor crash reports weekly
- Plan feature releases
- Begin Phase 2 planning
- Maintain release cadence

---

# Definition of Done

Release is complete when:

- Application is available on Google Play Store
- Application is available on Apple App Store
- Crash reporting is active and monitored
- Privacy policy is published
- Store listings are complete with screenshots and description
- Beta testing has been completed
- No critical bugs are known
- Release notes are published

---

# Guiding Principle

The release is not the end — it is the beginning.

Shipping the MVP validates the architecture, the design and the value proposition. Everything after this is iteration based on real user feedback.

Release when the application is stable and useful, not when it is perfect. Perfection is a direction, not a destination.
