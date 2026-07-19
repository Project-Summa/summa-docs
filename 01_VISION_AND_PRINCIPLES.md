# Vision & Core Principles

> Every successful long-term project is guided by a clear philosophy rather than a collection of features.

---

# Table of Contents

- Purpose
- Vision
- Mission
- Core Values
- Core Principles
- Product Philosophy
- Long-Term Goals
- Non-Goals
- Decision Framework

---

# Purpose

Summa exists to give people complete control over their personal financial data.

Modern finance applications often require users to:

- Create an online account
- Store financial information on third-party servers
- Trust proprietary cloud providers
- Accept limited export capabilities
- Lose access to their data if the service shuts down

Summa takes the opposite approach.

Users should always remain in control of:

- Their data
- Their backups
- Their synchronization
- Their privacy

---

# Vision

Our vision is to build the most transparent, privacy-respecting and flexible personal finance platform available.

Whether someone prefers using a completely offline application, hosting their own synchronization server, or subscribing to an official cloud service, the experience should remain consistent.

The user chooses where their data lives.

Never the application.

---

# Mission

Build software that people can trust.

Trust is earned through:

- Open source development
- Transparent architecture
- Local-first design
- Optional cloud services
- User ownership of data

Every technical decision should reinforce these principles.

---

# Core Values

## Privacy

Financial information is among the most sensitive data a person owns.

Privacy is not an optional feature.

It is a core requirement.

---

## Transparency

Every major architectural decision should be documented.

Every important component should be understandable.

Users should never have to guess what the application is doing.

---

## Simplicity

The interface should remain approachable for everyone.

Advanced functionality should never make basic tasks more difficult.

---

## Reliability

Users should be able to depend on Summa every day.

The application should prioritize stability over rapidly shipping features.

---

## Ownership

Users own their financial history.

Not Summa.

Not a cloud provider.

Not a subscription.

---

# Core Principles

## 1. Local First

The application must always function without an internet connection.

Offline is the default.

Online is optional.

---

## 2. User Owns The Data

Users should always be able to:

- Export their data
- Import their data
- Back up their data
- Delete their data

Data portability is a first-class feature.

---

## 3. Cloud Is Optional

Cloud synchronization is a convenience.

Never a requirement.

Every feature should be evaluated under the assumption that the user has no internet connection.

---

## 4. Open Source

The project should remain fully open source.

The community should always be able to:

- Audit the code
- Build from source
- Contribute improvements
- Host their own infrastructure

---

## 5. Privacy by Default

Privacy should not require additional configuration.

The safest option should also be the default option.

Examples include:

- No advertising
- No financial telemetry
- No hidden tracking
- No unnecessary permissions

---

## 6. Native Experience

Every supported platform should feel native.

Android should behave like an Android application.

iOS should behave like an iOS application.

Platform conventions should be respected.

---

## 7. Performance Matters

The application should remain responsive even with years of financial history.

Performance should be considered during architecture decisions rather than treated as an optimization later.

---

## 8. Long-Term Maintainability

Readable code is preferred over clever code.

Documentation is considered part of the product.

Consistency is more valuable than novelty.

---

# Product Philosophy

Summa is not trying to become another banking application.

It is not trying to replace banks.

It is not trying to become an accounting system.

Instead, Summa aims to become the central place where users understand and organize their personal finances.

---

## One Backend, Multiple Deployment Options

Self-hosted Summa and Summa Cloud use the same open-source backend.
The hosted service provides convenience, not exclusive functionality
or a separate closed implementation.

# Long-Term Goals

Over the coming years, Summa should evolve into a complete personal finance platform with support for:

- Native Android application
- Native iOS application
- Desktop applications
- Self-hosted synchronization
- Official cloud synchronization
- Intelligent automation
- Household finance management
- Open API
- Community plugins (future consideration)

---

# Non-Goals

To remain focused, Summa intentionally avoids several directions.

Examples include:

## Banking Platform

Summa will not become a bank.

---

## Investment Platform

Investment tracking may be supported.

Investment execution will not.

---

## Cryptocurrency Exchange

Cryptocurrency may eventually be tracked.

Trading is outside the project's scope.

---

## Social Network

Financial information remains private.

The project will not evolve into a social platform.

---

## Advertising Platform

Advertisements will never become part of the user experience.

---

# Decision Framework

Whenever a new feature is proposed, it should be evaluated using the following questions.

## Does it respect privacy?

If not, reject or redesign it.

---

## Does it work offline?

If not, reconsider the design.

---

## Does it give users more control?

If not, determine whether it truly belongs in the project.

---

## Can users export their data?

If not, the implementation is incomplete.

---

## Does it increase unnecessary complexity?

If yes, search for a simpler solution.

---

# Success Criteria

A successful version of Summa should allow a user to:

- Install the application
- Never create an account
- Use it for years offline
- Back up their data
- Restore their data
- Migrate to another device
- Self-host synchronization if desired
- Subscribe to Summa Cloud if preferred

without ever losing ownership of their financial information.

---

# Closing Statement

Technology changes.

Business models change.

Companies change.

The principles of Summa should not.

Every future feature, architecture decision and line of code should reinforce the philosophy described in this document.