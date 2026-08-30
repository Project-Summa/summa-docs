# Feature Specifications

> Defining the detailed specifications for every Phase 1 feature in the Summa mobile application.

---

## Table of Contents

- [Purpose](#purpose)
- [Scope](#scope)
- [Specification Format](#specification-format)
- [Feature 1 — Workspace Management](#feature-1--workspace-management)
- [Feature 2 — Profile Management](#feature-2--profile-management)
- [Feature 3 — Category Management](#feature-3--category-management)
- [Feature 4 — Transaction Management](#feature-4--transaction-management)
- [Feature 5 — Dashboard](#feature-5--dashboard)
- [Feature 6 — Statistics](#feature-6--statistics)
- [Feature 7 — Data Export](#feature-7--data-export)
- [Feature 8 — Backup and Restore](#feature-8--backup-and-restore)
- [Feature 9 — Settings](#feature-9--settings)
- [Feature 10 — Application Lock](#feature-10--application-lock)
- [Cross-Feature Requirements](#cross-feature-requirements)
- [Definition of Done](#definition-of-done)
- [Guiding Principle](#guiding-principle)

---

## Purpose

This document defines the detailed specification for every feature included in Phase 1 of Summa.

Each specification covers the goal, scope, user stories, UI requirements, data requirements, business rules, edge cases, acceptance criteria, design references and test requirements.

This document serves as the contract between design, implementation and testing. A feature is considered complete only when every acceptance criterion in its specification has been met and verified.

---

## Scope

All features in this document are local-only. No network calls, no user accounts, no server dependencies.

The features are ordered by implementation dependency, not by user-facing priority.

---

## Specification Format

Every feature follows the same structure:

```text
Goal and User Value
Scope
User Stories and Scenarios
UI Requirements
Data Requirements
Business Rules
Edge Cases
Acceptance Criteria
Design References
Test Requirements
```

Consistency in structure ensures that nothing is missed during implementation and review.

---

## Feature 1 — Workspace Management

### Goal and User Value

The Workspace is the top-level container for all financial data. In Phase 1, the Workspace is invisible to the user. It exists to enforce data isolation and to prepare the schema for future multi-workspace synchronization in Phase 3.

The user never creates, names or selects a Workspace. The application handles this automatically.

---

### Scope

#### Included

- Automatic Workspace creation on first launch
- Single Workspace per local installation
- All financial entities scoped to the Workspace
- Workspace UUID generated once and persisted
- Workspace used as parent for profiles, categories and transactions

#### Excluded

- Multiple Workspaces
- Workspace naming or editing
- Workspace switching
- Workspace sharing
- Workspace deletion by user

---

### User Stories and Scenarios

#### US-W1 — First Launch

As a new user, I want the application to be ready to use immediately so that I do not need to configure infrastructure before adding my first transaction.

**Scenario:** User opens the app for the first time. A default Workspace is created in the background. The user proceeds directly to profile creation or the onboarding flow.

---

#### US-W2 — Data Isolation

As a user, I want my financial data to be organized in a consistent structure so that future synchronization and sharing work correctly.

**Scenario:** All profiles, categories and transactions created by the user are automatically associated with the default Workspace. No data exists outside a Workspace.

---

### UI Requirements

#### Screens

No dedicated screen. Workspace creation is a background operation.

#### Components

None visible to the user.

#### Behavior

- On first launch, if no Workspace exists, one is created before the onboarding or profile selection screen appears.
- The Workspace creation must complete before any user-facing data operation is allowed.
- If Workspace creation fails, the application must display an error screen and prevent further interaction until the issue is resolved.

---

### Data Requirements

#### Entity

Workspace (see `docs/phase-0/02_DATABASE.md`)

#### Fields

| Field | Value |
|---|---|
| id | Generated UUID v4 |
| name | `"Default"` |
| created_at | Current UTC timestamp |
| updated_at | Current UTC timestamp |
| deleted_at | null |
| version | 1 |
| sync_status | `local` |
| device_id | Current device UUID |

#### Queries

- `getOrCreateDefaultWorkspace()` — Returns the single local Workspace. Creates one if none exists.
- `getWorkspaceById(id)` — Returns a Workspace by ID. Used internally by repositories.

#### Indexes

Primary key on `id`.

---

### Business Rules

1. Exactly one Workspace exists in local-only mode.
2. The Workspace is never exposed to the user through the UI.
3. The Workspace `id` is stored in application preferences for fast lookup.
4. All repository queries must scope results by `workspace_id`.
5. No financial entity may be created without a valid `workspace_id`.
6. The Workspace must exist before any Profile, Category or Transaction is created.

---

### Edge Cases

| Case | Expected Behavior |
|---|---|
| Database corrupted, Workspace row missing | Recreate Workspace on next launch. Existing data is orphaned and should be handled by a migration or recovery flow. |
| Multiple Workspace rows exist | Use the oldest non-deleted Workspace. Log a warning. Future cleanup should merge or remove duplicates. |
| Application killed during Workspace creation | On next launch, check for existing Workspace before creating a new one. Use database transaction to prevent partial creation. |

---

### Acceptance Criteria

1. A Workspace row exists in the database after the first successful launch.
2. The Workspace `id` is persisted in application preferences.
3. No screen or UI element references the Workspace directly.
4. All financial entity creation paths include a valid `workspace_id`.
5. Re-launching the application does not create a second Workspace.
6. Unit tests verify Workspace creation and retrieval.
7. Integration tests verify that entities are scoped to the Workspace.

---

### Design References

- `docs/phase-0/02_DATABASE.md` — Entity: Workspace
- `docs/phase-0/04_BACKEND_ARCHITECTURE.md` — Workspace Model
- `docs/05_GLOSSARY_AND_DOMAIN_LANGUAGE.md` — Workspace definition

---

### Test Requirements

| Type | Coverage |
|---|---|
| Unit | Workspace creation, singleton retrieval, ID persistence |
| Integration | Entity scoping to Workspace, no orphaned entities |
| Widget | None (no UI) |

---

## Feature 2 — Profile Management

### Goal and User Value

Profiles allow users to track finances for themselves (personal mode) or for multiple members of a household (household mode). A user can switch between profiles to view and manage separate financial contexts within the same application.

This is the first user-facing feature. It establishes the onboarding flow and the primary identity within the app.

---

### Scope

#### Included

- Create a new Profile with name, type and currency
- Personal profile type
- Household profile type
- Set a default Profile
- Switch between Profiles
- Edit Profile name, currency and type
- Delete a Profile (soft delete)
- Onboarding flow for first Profile creation
- Profile selector in the top app bar

#### Excluded

- Profile avatars or photos
- Profile-specific themes
- Profile sharing across devices
- Profile import or export independently
- Profile-level PIN or lock

---

### User Stories and Scenarios

#### US-P1 — Create Personal Profile

As a new user, I want to create a personal profile so that I can start tracking my individual finances.

**Scenario:** User opens the app for the first time. The onboarding screen asks for a profile name and currency. The user enters "My Finances" and selects EUR. A personal profile is created and set as default.

---

#### US-P2 — Create Household Profile

As a user, I want to create a household profile so that I can track shared expenses with my family.

**Scenario:** User navigates to Profile Management. Taps "Add Profile." Selects "Household" type. Enters "Family Budget" and selects RSD. The profile is created. The user can now switch to this profile from the top app bar.

---

#### US-P3 — Switch Profile

As a user with multiple profiles, I want to switch between them so that I can manage different financial contexts.

**Scenario:** User taps the profile selector in the top app bar. A bottom sheet displays all active profiles. User taps "Family Budget." The dashboard and all screens refresh to show data for the selected profile.

---

#### US-P4 — Edit Profile

As a user, I want to edit my profile details so that I can correct mistakes or update my preferences.

**Scenario:** User navigates to Profile Management. Selects a profile. Taps "Edit." Changes the name from "My Finances" to "Personal." Saves. The profile is updated throughout the app.

---

#### US-P5 — Delete Profile

As a user, I want to delete a profile I no longer need so that my profile list stays clean.

**Scenario:** User navigates to Profile Management. Selects a profile. Taps "Delete." A confirmation dialog warns that all associated transactions will be hidden. User confirms. The profile is soft-deleted. If it was the default, another profile becomes the default.

---

### UI Requirements

#### Screens

| Screen | Description |
|---|---|
| Onboarding Screen | First-launch flow. Collects profile name and currency. Creates the first personal profile. |
| Profile List Screen | Lists all active profiles. Accessible from Settings or the profile selector. Shows name, type and currency. |
| Create Profile Screen | Form to create a new profile. Fields: name, type (personal/household), currency. |
| Edit Profile Screen | Same form as Create, pre-filled with existing values. |
| Profile Selector Bottom Sheet | Overlay triggered from the top app bar. Lists profiles with radio selection. |

#### Components

| Component | Description |
|---|---|
| Profile Card | Displays profile name, type badge and currency. Used in the list screen. |
| Profile Selector | Top app bar widget showing the active profile name. Tapping opens the bottom sheet. |
| Type Badge | Chip indicating "Personal" or "Household." |
| Currency Selector | Searchable list of ISO 4217 currency codes with flag or symbol. |

#### Navigation

```text
Onboarding → Dashboard (after first profile created)
Settings → Profile List → Create Profile
Settings → Profile List → Profile Detail → Edit Profile
Top App Bar → Profile Selector Bottom Sheet → Switch Profile
```

---

### Data Requirements

#### Entity

Profile (see `docs/phase-0/02_DATABASE.md`)

#### Fields

| Field | Constraint |
|---|---|
| id | UUID v4, required |
| workspace_id | FK → workspace, required |
| name | 1–50 characters, required |
| type | `personal` or `household`, required |
| currency | ISO 4217 code, required |
| is_default | 0 or 1, exactly one profile has 1 |
| created_at | UTC timestamp |
| updated_at | UTC timestamp |
| deleted_at | null or UTC timestamp |
| version | Starts at 1 |
| sync_status | `local` |
| device_id | Current device UUID |

#### Queries

- `createProfile(profile)` — Insert a new profile.
- `getProfiles(workspaceId)` — All non-deleted profiles for the workspace.
- `getProfileById(id)` — Single profile by ID.
- `getDefaultProfile(workspaceId)` — The profile where `is_default = 1`.
- `updateProfile(profile)` — Update name, type, currency.
- `softDeleteProfile(id)` — Set `deleted_at`.
- `setDefaultProfile(id)` — Unset current default, set new default. Must be atomic.
- `getProfileCount(workspaceId)` — Count of non-deleted profiles.

#### Indexes

- `workspace_id`
- `is_default`

---

### Business Rules

1. Every Workspace must have at least one active Profile.
2. Exactly one Profile is marked as `is_default = 1` at all times.
3. The first Profile created is automatically set as default.
4. A Profile cannot be deleted if it is the only active Profile.
5. Deleting the default Profile requires setting another Profile as default first.
6. Profile names must be between 1 and 50 characters.
7. Profile names are trimmed of leading and trailing whitespace.
8. Currency must be a valid ISO 4217 code.
9. Profile type is immutable after creation. (Review: may be relaxed in future.)
10. Soft-deleted Profiles are excluded from all queries and the profile selector.
11. Switching profiles refreshes all data-driven screens to the new profile context.

---

### Edge Cases

| Case | Expected Behavior |
|---|---|
| User tries to delete the only Profile | Show error: "You must have at least one profile." Prevent deletion. |
| User tries to create a Profile with a duplicate name | Allowed. Profile names are not unique. |
| User enters only whitespace as Profile name | Validation error: "Enter a profile name." |
| User pastes extremely long name (>50 chars) | Validation error: "Name must be 50 characters or fewer." |
| Currency list is empty (should not happen) | Display error state with retry option. |
| Profile switch while a form has unsaved data | Confirm navigation with a discard confirmation dialog. |
| Default Profile is deleted programmatically | System must auto-assign another Profile as default. Log a warning. |

---

### Acceptance Criteria

1. On first launch, the onboarding screen appears and creates the first Profile.
2. After onboarding, the user lands on the Dashboard with the new Profile active.
3. The profile selector in the top app bar displays the active Profile name.
4. Tapping the profile selector opens a bottom sheet with all active Profiles.
5. Selecting a different Profile switches the context and refreshes the Dashboard.
6. Creating a new Profile adds it to the list and makes it available in the selector.
7. Editing a Profile updates the name and currency everywhere it appears.
8. Deleting a Profile removes it from the selector and list.
9. Attempting to delete the only Profile shows an error and prevents the action.
10. Exactly one Profile is always marked as default.
11. Profile creation, editing and deletion are persisted across app restarts.
12. All Profile operations are covered by unit tests.
13. The Profile List, Create and Edit screens have widget tests.

---

### Design References

- `docs/phase-0/07_DESIGN_SYSTEM.md` — Core Components, Navigation, Bottom Sheet
- `docs/phase-0/02_DATABASE.md` — Entity: Profile
- `docs/phase-0/03_MOBILE_ARCHITECTURE.md` — Feature Structure, Navigation
- `docs/05_GLOSSARY_AND_DOMAIN_LANGUAGE.md` — Profile, Workspace

---

### Test Requirements

| Type | Coverage |
|---|---|
| Unit | Create, edit, delete, set default, get profiles, validation rules |
| Integration | Profile scoped to Workspace, default enforcement, soft delete |
| Widget | Onboarding screen, profile list, create/edit form, profile selector |

---

## Feature 3 — Category Management

### Goal and User Value

Categories organize transactions into meaningful groups. Built-in categories provide an immediate starting point. Custom categories allow users to adapt the system to their personal financial habits.

Well-chosen categories make the Dashboard and Statistics screens useful from the first week of usage.

---

### Scope

#### Included

- Seed default expense categories on Workspace creation
- Seed default income categories on Workspace creation
- Create custom categories (name, icon, color, type)
- Edit categories (name, icon, color)
- Archive categories (soft delete)
- Reorder categories (sort order)
- Category type: expense or income
- Category icons from a predefined set
- Category colors from a predefined palette
- Category list screen with filtering by type

#### Excluded

- Nested or hierarchical categories
- Category merging
- Category import or export independently
- User-uploaded custom icons
- Category budgets (Phase 2)
- Category-level permissions

---

### User Stories and Scenarios

#### US-C1 — Default Categories Available

As a new user, I want common categories to already exist so that I can start categorizing transactions immediately.

**Scenario:** After creating the first Profile, the application seeds default categories. The user sees categories like "Food," "Transport," "Salary" and "Entertainment" in the category list without any manual setup.

---

#### US-C2 — Create Custom Category

As a user, I want to create a custom category so that I can track spending in areas specific to my life.

**Scenario:** User navigates to Category Management. Taps "Add Category." Enters name "Pet Care," selects a paw icon, picks a teal color and selects "Expense" type. The category is saved and appears in the category list and in transaction creation forms.

---

#### US-C3 — Edit Category

As a user, I want to edit a category so that I can correct its name or change its appearance.

**Scenario:** User selects "Food" from the category list. Taps "Edit." Changes the color from orange to green. Saves. The new color appears everywhere the category is used.

---

#### US-C4 — Archive Category

As a user, I want to archive a category I no longer use so that it stops appearing in transaction forms without losing historical data.

**Scenario:** User selects "Gym" from the category list. Taps "Archive." A confirmation dialog warns that existing transactions will keep this category but it will not appear in new transaction forms. User confirms. The category is hidden from the active list and from transaction creation.

---

#### US-C5 — Reorder Categories

As a user, I want to reorder categories so that my most-used ones appear first.

**Scenario:** User enters the category list. Taps "Reorder." Drags "Food" to the top of the expense list. Saves. The new order is reflected in the transaction creation category picker.

---

### UI Requirements

#### Screens

| Screen | Description |
|---|---|
| Category List Screen | Two tabs: Expense and Income. Lists active categories with icon, name and color dot. Accessible from Settings or Management tab. |
| Create Category Screen | Form with fields: name, type (expense/income), icon picker, color picker. |
| Edit Category Screen | Same form as Create, pre-filled. |
| Icon Picker | Grid of predefined icons. Searchable by name. |
| Color Picker | Grid of predefined colors from the design palette. |

#### Components

| Component | Description |
|---|---|
| Category Row | Icon, name, color indicator. Used in list screens and pickers. |
| Category Chip | Compact representation used in transaction rows and filters. |
| Icon Picker Grid | Scrollable grid of category icons. |
| Color Picker Grid | Grid of available colors. |
| Reorder Handle | Drag handle for reordering categories. |

#### Navigation

```text
Settings → Category List → Create Category
Settings → Category List → Category Detail → Edit Category
Settings → Category List → Reorder Mode
Transaction Form → Category Picker (filtered by transaction type)
```

---

### Data Requirements

#### Entity

Category (see `docs/phase-0/02_DATABASE.md`)

#### Fields

| Field | Constraint |
|---|---|
| id | UUID v4, required |
| workspace_id | FK → workspace, required |
| profile_id | null (workspace-wide in Phase 1) |
| name | 1–30 characters, required |
| icon | Icon identifier string, required |
| color | Hex color string (e.g. `#2C3E50`), required |
| type | `expense` or `income`, required |
| is_default | 1 for seeded categories, 0 for user-created |
| sort_order | Integer, user-defined ordering |
| created_at | UTC timestamp |
| updated_at | UTC timestamp |
| deleted_at | null or UTC timestamp |
| version | Starts at 1 |
| sync_status | `local` |
| device_id | Current device UUID |

#### Default Categories

Expense defaults:

| Name | Icon | Color |
|---|---|---|
| Food & Dining | restaurant | #E67E22 |
| Transport | directions_car | #3498DB |
| Shopping | shopping_bag | #9B59B6 |
| Bills & Utilities | receipt | #E74C3C |
| Entertainment | movie | #1ABC9C |
| Health | local_hospital | #2ECC71 |
| Education | school | #34495E |
| Groceries | local_grocery_store | #F39C12 |
| Personal Care | spa | #E91E63 |
| Other | more_horiz | #95A5A6 |

Income defaults:

| Name | Icon | Color |
|---|---|---|
| Salary | work | #27AE60 |
| Freelance | laptop | #2980B9 |
| Investment | trending_up | #8E44AD |
| Gift | card_giftcard | #E67E22 |
| Other Income | attach_money | #16A085 |

The exact icons and colors are subject to design review.

#### Queries

- `seedDefaultCategories(workspaceId)` — Insert default categories. Called once during Workspace initialization.
- `getCategories(workspaceId, type)` — Active categories filtered by type.
- `getCategoryById(id)` — Single category by ID.
- `createCategory(category)` — Insert a new custom category.
- `updateCategory(category)` — Update name, icon, color.
- `softDeleteCategory(id)` — Archive by setting `deleted_at`.
- `reorderCategories(categoryIds)` — Update `sort_order` for a list of category IDs.
- `getCategoryCount(workspaceId)` — Count of active categories.

#### Indexes

- `workspace_id`
- `type`
- `sort_order`

---

### Business Rules

1. Default categories are seeded once per Workspace during initialization.
2. Default categories can be edited but not deleted. (Review: may allow archiving defaults.)
3. Custom categories can be edited and archived.
4. Category names must be between 1 and 30 characters.
5. Category names are unique within a type (expense or income) per Workspace. (Review: may allow duplicates.)
6. Archived categories remain associated with existing transactions.
7. Archived categories do not appear in the transaction creation category picker.
8. The category picker in transaction creation is filtered by the transaction type (expense categories for expenses, income categories for incomes).
9. Transfer transactions do not use categories.
10. `sort_order` determines display order. Lower values appear first.
11. When a new category is created, it receives the highest `sort_order` value (appears last).
12. Icon and color must be selected from the predefined sets.

---

### Edge Cases

| Case | Expected Behavior |
|---|---|
| User creates a category with a name that matches a default | Allowed if types differ. If same type, warn but allow (names are not strictly unique). |
| User tries to archive a default category | If defaults cannot be archived, show an explanation. If allowed, proceed with archive. |
| User tries to create a category with an empty name | Validation error: "Enter a category name." |
| All categories of one type are archived | The category picker shows an empty state prompting the user to create a new category. |
| Category seeding fails mid-way | Use a database transaction. Roll back and retry on next launch. |
| Very long category name overflows the UI | Truncate with ellipsis. Full name visible on detail screen. |
| Category used by transactions is archived | Transactions retain the category. Statistics still include archived categories for historical accuracy. |

---

### Acceptance Criteria

1. Default expense and income categories are seeded on first launch.
2. The category list screen shows two tabs: Expense and Income.
3. Creating a custom category adds it to the correct tab.
4. Editing a category updates its name, icon and color everywhere.
5. Archiving a category removes it from the active list and from the transaction creation picker.
6. Archived categories remain visible in existing transactions and statistics.
7. Reordering categories changes their display order in lists and pickers.
8. The category picker in transaction creation is filtered by transaction type.
9. All category operations are persisted across app restarts.
10. Unit tests cover creation, editing, archiving, reordering and seeding.
11. Widget tests cover the category list, create/edit form, icon picker and color picker.

---

### Design References

- `docs/phase-0/07_DESIGN_SYSTEM.md` — Iconography, Color System, Category Row
- `docs/phase-0/02_DATABASE.md` — Entity: Category
- `docs/05_GLOSSARY_AND_DOMAIN_LANGUAGE.md` — Category

---

### Test Requirements

| Type | Coverage |
|---|---|
| Unit | Seed defaults, create, edit, archive, reorder, validation, type filtering |
| Integration | Category scoped to Workspace, transactions retain archived categories |
| Widget | Category list with tabs, create/edit form, icon picker, color picker, reorder mode |

---

## Feature 4 — Transaction Management

### Goal and User Value

Transactions are the core of Summa. Every financial event — a purchase, a salary payment, a transfer between accounts — is recorded as a Transaction. This feature covers the complete lifecycle: creation, editing, deletion, listing, filtering and detail viewing.

Accurate transaction tracking is the foundation for the Dashboard, Statistics, Export and Backup features.

---

### Scope

#### Included

- Create expense transaction
- Create income transaction
- Create transfer transaction
- Edit any transaction field
- Delete transaction (soft delete)
- Transaction list with date grouping
- Transaction filtering by type, category, date range and profile
- Transaction detail view
- Transaction notes
- Transaction merchant field
- Amount input with currency formatting
- Date and time picker for transaction date

#### Excluded

- Transaction splits (Phase 2 decision pending)
- Recurring transactions (Phase 2)
- Transaction attachments (Phase 2)
- Transaction search by text (may be added)
- Bulk operations (select multiple, bulk delete)
- Transaction import from bank statements (Phase 2)

---

### User Stories and Scenarios

#### US-T1 — Create Expense

As a user, I want to record an expense so that I can track where my money goes.

**Scenario:** User taps the floating action button on the Dashboard. The transaction creation screen opens with "Expense" selected. User enters amount "25.50," selects "Food & Dining" category, adds note "Lunch with colleagues," sets today's date. Taps "Save." The transaction appears in the recent transactions list on the Dashboard.

---

#### US-T2 — Create Income

As a user, I want to record income so that I can track my earnings.

**Scenario:** User taps the FAB. Switches type to "Income." Enters amount "3000," selects "Salary" category, sets the date to the 1st of the month. Taps "Save." The Dashboard balance increases.

---

#### US-T3 — Create Transfer

As a user, I want to record a transfer so that I can track money moved between accounts or categories.

**Scenario:** User taps the FAB. Switches type to "Transfer." Enters amount "500," adds note "Moved to savings." Taps "Save." The transfer is recorded. It does not affect the income or expense totals but appears in the transaction list.

---

#### US-T4 — Edit Transaction

As a user, I want to edit a transaction so that I can correct mistakes.

**Scenario:** User opens a transaction detail. Taps "Edit." Changes the amount from "25.50" to "27.00." Saves. The transaction list and Dashboard balance update.

---

#### US-T5 — Delete Transaction

As a user, I want to delete a transaction so that I can remove entries I no longer need.

**Scenario:** User opens a transaction detail. Taps "Delete." A confirmation dialog warns that this action cannot be easily undone. User confirms. The transaction is soft-deleted and removed from all lists and calculations.

---

#### US-T6 — Filter Transactions

As a user, I want to filter my transaction list so that I can find specific entries quickly.

**Scenario:** User opens the transaction list. Taps the filter icon. Selects "Expense" type, "Food & Dining" category and "This month" date range. The list updates to show only matching transactions.

---

#### US-T7 — View Transaction Details

As a user, I want to view the full details of a transaction so that I can review all its information.

**Scenario:** User taps a transaction row in the list. The detail screen shows: type, amount, category (with icon and color), date, note, merchant and profile.

---

### UI Requirements

#### Screens

| Screen | Description |
|---|---|
| Transaction Creation Screen | Form with type selector (expense/income/transfer), amount input, category picker, date picker, note field, merchant field, save button. |
| Transaction Edit Screen | Same form as Create, pre-filled with existing values. |
| Transaction List Screen | Scrollable list grouped by date. Filter bar at top. FAB for creation. |
| Transaction Detail Screen | Displays all transaction fields. Edit and Delete actions in app bar. |
| Filter Bottom Sheet | Type filter, category multi-select, date range picker, profile filter. Apply and Clear buttons. |
| Category Picker Bottom Sheet | List of categories filtered by transaction type. Searchable. |
| Amount Input Screen or Widget | Large numeric input with decimal support. Currency symbol displayed. |

#### Components

| Component | Description |
|---|---|
| Transaction Row | Category icon, merchant/note, category name, date, amount. Right-aligned amount. Income in success color, expense in neutral/error color. |
| Transaction Type Selector | Segmented control or tabs: Expense, Income, Transfer. |
| Amount Input | Large display-style input. Numeric keyboard. Supports decimal separator based on locale. |
| Date Picker | Native date picker (Material on Android, Cupertino on iOS). Default: today. |
| Date Group Header | Sticky header showing date (e.g. "Today," "Yesterday," "March 15, 2026"). |
| Filter Chip Row | Horizontal scrollable row of active filters. Each chip is dismissible. |
| Empty Transaction State | Illustration, message "No transactions yet," CTA "Add your first transaction." |

#### Navigation

```text
Dashboard → Transaction List (via "See All" or swipe)
Dashboard → Transaction Creation (via FAB)
Transaction List → Transaction Creation (via FAB)
Transaction List → Transaction Detail (tap row)
Transaction Detail → Transaction Edit (via Edit action)
Transaction Detail → Delete Confirmation (via Delete action)
Transaction Creation → Category Picker (tap category field)
Transaction List → Filter Bottom Sheet (tap filter icon)
```

---

### Data Requirements

#### Entity

Transaction (see `docs/phase-0/02_DATABASE.md`)

#### Fields

| Field | Constraint |
|---|---|
| id | UUID v4, required |
| workspace_id | FK → workspace, required |
| profile_id | FK → profile, required |
| category_id | FK → category, nullable (required for expense and income, null for transfer) |
| amount_minor | Integer > 0, required. Stored in minor currency units. |
| currency | ISO 4217 code, required. Inherited from Profile. |
| transaction_type | `expense`, `income` or `transfer`, required |
| note | 0–500 characters, nullable |
| merchant | 0–100 characters, nullable |
| occurred_at | UTC datetime, required. Default: current time. |
| created_at | UTC timestamp |
| updated_at | UTC timestamp |
| deleted_at | null or UTC timestamp |
| version | Starts at 1 |
| sync_status | `local` |
| device_id | Current device UUID |

#### Queries

- `createTransaction(transaction)` — Insert a new transaction.
- `getTransactionById(id)` — Single transaction by ID, excluding soft-deleted.
- `updateTransaction(transaction)` — Update all mutable fields.
- `softDeleteTransaction(id)` — Set `deleted_at`.
- `getTransactions(workspaceId, profileId, filters, pagination)` — Paginated, filtered list. Supports type, category, date range. Ordered by `occurred_at` descending.
- `getTransactionsByDateRange(workspaceId, profileId, startDate, endDate)` — For statistics and dashboard.
- `getRecentTransactions(workspaceId, profileId, limit)` — Most recent N transactions.
- `calculateBalance(workspaceId, profileId, startDate, endDate)` — Sum of income minus sum of expenses.
- `calculateTotalByType(workspaceId, profileId, type, startDate, endDate)` — Sum for a specific type.
- `getTransactionsGroupedByCategory(workspaceId, profileId, startDate, endDate)` — For statistics.
- `getTransactionsGroupedByDate(workspaceId, profileId, startDate, endDate)` — For list display.

#### Indexes

- `workspace_id`
- `profile_id`
- `category_id`
- `occurred_at`
- `transaction_type`
- Composite: `(workspace_id, profile_id, occurred_at)`

---

### Business Rules

1. Amount must be greater than zero.
2. Amount is stored as integer minor units. The UI converts for display.
3. Currency is inherited from the active Profile and cannot be changed per transaction.
4. Expense and income transactions require a category. Transfer transactions do not have a category.
5. The `occurred_at` date defaults to the current time but can be set to any past date.
6. Future dates are not allowed for `occurred_at`. (Review: may allow scheduling in Phase 2.)
7. Note is optional and limited to 500 characters.
8. Merchant is optional and limited to 100 characters.
9. Deleting a transaction is a soft delete. The record remains in the database.
10. Editing a transaction updates `updated_at` and increments `version`.
11. The transaction list is ordered by `occurred_at` descending (newest first).
12. Transactions are grouped by date in the list view.
13. Transfer transactions are neutral to income/expense totals but affect the overall balance calculation based on business logic (transfers may be excluded from balance or treated as neutral).

---

### Edge Cases

| Case | Expected Behavior |
|---|---|
| User enters amount with more decimal places than the currency supports | Round to the currency's minor unit precision. Display the rounded value. |
| User enters zero amount | Validation error: "Enter an amount greater than zero." |
| User enters negative amount | The sign is determined by transaction type, not the input. Amount is always positive in storage. |
| User selects a date far in the past | Allowed. No lower bound on date. |
| User selects a future date | Validation error: "Transaction date cannot be in the future." (Review decision.) |
| Category is archived after a transaction uses it | Transaction retains the category. Category name and icon are still displayed. |
| Profile is deleted that has transactions | Transactions are soft-deleted along with the profile, or re-assigned. Decision: transactions are hidden when profile is deleted. |
| Very long note (500 chars) | Allowed. Display truncated in list, full in detail view. |
| Transaction list has thousands of entries | Paginated loading. Virtualized list. No performance degradation. |
| User creates a transfer with a category | Category is ignored or cleared for transfers. UI should hide the category field when "Transfer" is selected. |
| Amount overflows integer range | Validate against a maximum amount (e.g. 999,999,999 minor units). Show validation error. |

---

### Acceptance Criteria

1. User can create an expense, income or transfer transaction.
2. Created transactions appear in the transaction list grouped by date.
3. The transaction list supports filtering by type, category and date range.
4. Tapping a transaction opens the detail screen with all fields.
5. Editing a transaction updates all fields and reflects changes in the list and Dashboard.
6. Deleting a transaction removes it from the list and recalculates the Dashboard balance.
7. Amount input uses the correct currency formatting from the active Profile.
8. Category picker is filtered by transaction type.
9. Transfer transactions do not show a category field.
10. All transaction operations are persisted across app restarts.
11. Empty state is shown when no transactions exist.
12. Unit tests cover creation, editing, deletion, filtering and balance calculation.
13. Widget tests cover the creation form, detail screen, list screen and filter sheet.

---

### Design References

- `docs/phase-0/07_DESIGN_SYSTEM.md` — Transaction Row, Financial Components, Amount Typography, Forms
- `docs/phase-0/02_DATABASE.md` — Entity: Transaction
- `docs/phase-0/03_MOBILE_ARCHITECTURE.md` — Data Flow, ViewModels
- `docs/05_GLOSSARY_AND_DOMAIN_LANGUAGE.md` — Transaction, Minor Units, Balance

---

### Test Requirements

| Type | Coverage |
|---|---|
| Unit | Create (expense, income, transfer), edit, delete, balance calculation, filtering, pagination, validation |
| Integration | Transaction scoped to Workspace and Profile, category reference integrity, soft delete |
| Widget | Creation form, edit form, detail screen, list with grouping, filter sheet, empty state, amount input |

---

## Feature 5 — Dashboard

### Goal and User Value

The Dashboard is the first screen the user sees after launching the app. It provides an at-a-glance view of the current financial state: balance, recent activity and a monthly summary. It answers the question "How am I doing financially right now?"

The Dashboard must be fast, scannable and always accurate.

---

### Scope

#### Included

- Current balance card (total income minus total expenses for the active profile)
- Balance visibility toggle (show/hide amounts)
- Monthly summary (income total, expense total, net for current month)
- Recent transactions list (last 5–10 transactions)
- "See All" link to the full transaction list
- FAB to create a new transaction
- Empty states for no transactions
- Pull-to-refresh

#### Excluded

- Budget progress (Phase 2)
- Account balances (multiple accounts)
- Upcoming bills or reminders (Phase 2)
- Financial goals
- Customizable dashboard layout
- Widget or notification summary

---

### User Stories and Scenarios

#### US-D1 — View Balance

As a user, I want to see my current balance immediately upon opening the app so that I know my financial position.

**Scenario:** User opens the app. The Dashboard displays a balance card showing the total balance calculated from all transactions for the active profile. Income is shown in green, expenses in neutral.

---

#### US-D2 — Hide Balance

As a user, I want to hide my balance so that others looking at my screen cannot see my financial details.

**Scenario:** User taps the visibility toggle (eye icon) on the balance card. All monetary amounts on the Dashboard are replaced with dots or asterisks. Tapping again reveals the amounts.

---

#### US-D3 — View Recent Transactions

As a user, I want to see my most recent transactions so that I can quickly verify recent activity.

**Scenario:** Below the balance card, the Dashboard shows the last 5 transactions with category icon, merchant/note, date and amount. User can tap any row to view details.

---

#### US-D4 — View Monthly Summary

As a user, I want to see a summary of this month's income and expenses so that I can understand my monthly pattern.

**Scenario:** The Dashboard shows a summary section with the current month's total income, total expenses and net (income minus expenses). The month name is displayed (e.g. "March 2026").

---

#### US-D5 — Navigate to Full Transaction List

As a user, I want to see all my transactions, not just recent ones.

**Scenario:** User taps "See All" next to the recent transactions header. The full transaction list screen opens.

---

### UI Requirements

#### Screens

| Screen | Description |
|---|---|
| Dashboard Screen | Main screen. Contains balance card, monthly summary, recent transactions. FAB for new transaction. |

#### Components

| Component | Description |
|---|---|
| Balance Card | Large card at the top. Shows total balance in large typography. Income and expense subtotals below. Visibility toggle in the corner. |
| Monthly Summary Section | Two or three summary cards: income total, expense total, net. Uses the Summary Card component from the design system. |
| Recent Transactions Section | Header with "Recent" title and "See All" link. List of 5 transaction rows. |
| Empty Dashboard State | Shown when no transactions exist. Illustration, message, CTA to add first transaction. |
| Visibility Toggle | Eye icon. Toggles between showing and hiding all monetary values on the Dashboard. |

#### Layout

```text
┌──────────────────────────────┐
│  Top App Bar                 │
│  [Profile Selector]          │
├──────────────────────────────┤
│  ┌────────────────────────┐  │
│  │  Balance Card          │  │
│  │  $12,450.00            │  │
│  │  Income: $15,000       │  │
│  │  Expenses: $2,550      │  │
│  └────────────────────────┘  │
│                              │
│  March 2026 Summary          │
│  ┌──────┐ ┌──────┐ ┌──────┐ │
│  │Income│ │Expense│ │ Net  │ │
│  │$3,000│ │$550  │ │$2,450│ │
│  └──────┘ └──────┘ └──────┘ │
│                              │
│  Recent Transactions         │
│  ┌────────────────────────┐  │
│  │ 🍔 Food    -$25.00    │  │
│  │ 💼 Salary  +$3,000    │  │
│  │ 🚗 Transport -$45.00  │  │
│  └────────────────────────┘  │
│                              │
│              [FAB: +]        │
├──────────────────────────────┤
│  Bottom Navigation           │
└──────────────────────────────┘
```

#### Navigation

```text
App Launch → Dashboard (default tab)
Dashboard → Transaction Detail (tap recent transaction)
Dashboard → Transaction List (tap "See All")
Dashboard → Transaction Creation (tap FAB)
Dashboard → Profile Selector (tap profile in app bar)
```

---

### Data Requirements

#### Queries Used

- `calculateBalance(workspaceId, profileId, startDate, endDate)` — For the balance card. Date range: all time.
- `calculateTotalByType(workspaceId, profileId, 'income', startOfMonth, endOfMonth)` — Monthly income.
- `calculateTotalByType(workspaceId, profileId, 'expense', startOfMonth, endOfMonth)` — Monthly expenses.
- `getRecentTransactions(workspaceId, profileId, 5)` — Last 5 transactions.

#### Computed Values

| Value | Calculation |
|---|---|
| Total Balance | Sum(income amounts) − Sum(expense amounts) for all time |
| Monthly Income | Sum(income amounts) for current month |
| Monthly Expenses | Sum(expense amounts) for current month |
| Monthly Net | Monthly Income − Monthly Expenses |

Transfers are excluded from income and expense totals.

---

### Business Rules

1. The Dashboard always reflects the active Profile. Switching profiles refreshes all values.
2. Balance is calculated, never stored. It is always derived from transactions.
3. The monthly summary uses the calendar month (1st to last day).
4. The visibility toggle hides all monetary amounts across the Dashboard. It does not affect other screens.
5. The visibility state is persisted in application preferences. It resets to "visible" on app restart. (Review: may persist across sessions.)
6. Recent transactions are limited to the 5 most recent non-deleted transactions.
7. Pull-to-refresh recalculates all Dashboard values.
8. The Dashboard must load in under 300ms on mid-range devices with up to 10,000 transactions.

---

### Edge Cases

| Case | Expected Behavior |
|---|---|
| No transactions exist | Show empty state with illustration and "Add your first transaction" CTA. |
| Only income transactions, no expenses | Balance shows positive. Expense sections show zero. |
| Only expense transactions, no income | Balance shows negative (or zero if negative balances are not supported). Display in neutral or warning color. |
| Very large balance (millions) | Format with locale-appropriate separators. Do not overflow the card. |
| Negative balance | Display with appropriate styling. Do not use error color for normal negative balances. |
| Profile switch while Dashboard is loading | Cancel current load, start new load for the new profile. |
| Transactions from archived categories | Still included in balance and summary calculations. |
| Month boundary (e.g. viewing at midnight on the 1st) | Monthly summary shows the new month. Previous month data is accessible through Statistics. |

---

### Acceptance Criteria

1. The Dashboard displays the correct total balance for the active Profile.
2. The monthly summary shows correct income, expense and net values for the current month.
3. The recent transactions list shows the 5 most recent transactions.
4. Tapping a recent transaction opens the transaction detail screen.
5. Tapping "See All" navigates to the full transaction list.
6. The FAB navigates to the transaction creation screen.
7. The visibility toggle hides and reveals all monetary amounts.
8. Switching profiles refreshes all Dashboard values.
9. The empty state appears when no transactions exist.
10. Pull-to-refresh recalculates all values.
11. The Dashboard loads within 300ms for up to 10,000 transactions.
12. Widget tests cover all Dashboard states: loading, content, empty, error.

---

### Design References

- `docs/phase-0/07_DESIGN_SYSTEM.md` — Balance Card, Summary Card, Transaction Row, Financial Components, Charts
- `docs/phase-0/03_MOBILE_ARCHITECTURE.md` — UI Architecture, Screen States

---

### Test Requirements

| Type | Coverage |
|---|---|
| Unit | Balance calculation, monthly totals, recent transactions query |
| Integration | Data refresh on profile switch, calculation accuracy with mixed transaction types |
| Widget | Dashboard screen with data, empty state, visibility toggle, loading state, error state |

---

## Feature 6 — Statistics

### Goal and User Value

Statistics transform raw transaction data into actionable insights. Users can understand spending patterns, identify top categories, compare months and track trends over time.

Statistics answer the questions: "Where does my money go?" and "Is my spending getting better or worse?"

---

### Scope

#### Included

- Monthly totals (income, expenses, net)
- Category breakdown for a selected month
- Category breakdown visualization (donut or horizontal bar chart)
- Date navigation (previous/next month)
- Yearly overview (12-month summary)
- Spending trends (line chart comparing monthly totals)
- Date range filter (month, year)
- Top spending categories ranked by amount

#### Excluded

- Custom date range selection (may be added)
- Budget vs actual comparison (Phase 2)
- Export statistics as image
- Per-category trend lines
- Income vs expense comparison chart
- Predictive analytics

---

### User Stories and Scenarios

#### US-S1 — View Monthly Statistics

As a user, I want to see a breakdown of my spending by category for this month so that I know where my money goes.

**Scenario:** User navigates to the Statistics tab. The screen shows the current month's total income, total expenses and net. Below, a donut chart shows the expense breakdown by category. Each category is labeled with name and percentage.

---

#### US-S2 — Navigate Between Months

As a user, I want to view statistics for previous months so that I can compare my spending over time.

**Scenario:** User taps the left arrow next to the month name. The statistics update to show the previous month's data. User can navigate forward and backward through all months that have transaction data.

---

#### US-S3 — View Yearly Overview

As a user, I want to see a yearly summary so that I can understand my long-term financial patterns.

**Scenario:** User switches from "Monthly" to "Yearly" view. The screen shows a bar chart with 12 months. Each bar represents the net amount for that month. Below, the yearly totals for income and expenses are displayed.

---

#### US-S4 — View Spending Trends

As a user, I want to see how my spending changes over time so that I can identify trends.

**Scenario:** User views the trend section. A line chart shows monthly expense totals for the last 6 or 12 months. The user can see whether spending is increasing, decreasing or stable.

---

#### US-S5 — View Top Categories

As a user, I want to see my top spending categories ranked by amount so that I can identify areas to cut back.

**Scenario:** Below the chart, a ranked list shows the top 5 expense categories with their total amounts and percentage of total spending.

---

### UI Requirements

#### Screens

| Screen | Description |
|---|---|
| Statistics Screen | Main statistics view. Contains period selector, summary cards, category breakdown chart, category list and trend chart. |

#### Components

| Component | Description |
|---|---|
| Period Selector | Toggle between Monthly and Yearly. Month/year navigation arrows. |
| Statistics Summary Cards | Income total, expense total, net. Similar to Dashboard summary but for the selected period. |
| Category Breakdown Chart | Donut chart or horizontal bar chart showing expense distribution by category. Each segment is colored with the category color. |
| Category Statistics Row | Category icon, name, amount, percentage bar. Ordered by amount descending. |
| Trend Line Chart | Line chart showing monthly totals over 6 or 12 months. |
| Yearly Bar Chart | Vertical bar chart with 12 bars, one per month. |
| Empty Statistics State | Shown when no data exists for the selected period. |

#### Layout (Monthly View)

```text
┌──────────────────────────────┐
│  Top App Bar: Statistics     │
├──────────────────────────────┤
│  [Monthly] [Yearly]          │
│  ◀ March 2026 ▶              │
│                              │
│  ┌──────┐ ┌──────┐ ┌──────┐ │
│  │Income│ │Expense│ │ Net  │ │
│  │$3,000│ │$550  │ │$2,450│ │
│  └──────┘ └──────┘ └──────┘ │
│                              │
│  Spending by Category        │
│  ┌────────────────────────┐  │
│  │     🍩 Donut Chart     │  │
│  │                        │  │
│  └────────────────────────┘  │
│                              │
│  1. Food       $200  36%     │
│  2. Transport  $150  27%     │
│  3. Shopping   $100  18%     │
│  4. Bills      $60   11%    │
│  5. Other      $40    7%    │
│                              │
│  Trend (Last 6 Months)       │
│  ┌────────────────────────┐  │
│  │     📈 Line Chart      │  │
│  │                        │  │
│  └────────────────────────┘  │
│                              │
├──────────────────────────────┤
│  Bottom Navigation           │
└──────────────────────────────┘
```

#### Navigation

```text
Bottom Navigation → Statistics Screen
Statistics → Transaction List (tap a category to see its transactions)
```

---

### Data Requirements

#### Queries Used

- `calculateTotalByType(workspaceId, profileId, type, startDate, endDate)` — Monthly or yearly totals.
- `getTransactionsGroupedByCategory(workspaceId, profileId, startDate, endDate)` — Category breakdown.
- `getMonthlyTotals(workspaceId, profileId, startYear, endYear)` — For trend charts and yearly view.
- `getTransactionsByCategory(workspaceId, profileId, categoryId, startDate, endDate)` — For drill-down.

#### Computed Values

| Value | Calculation |
|---|---|
| Monthly Income | Sum of income transaction amounts for the selected month |
| Monthly Expenses | Sum of expense transaction amounts for the selected month |
| Monthly Net | Income − Expenses |
| Category Amount | Sum of transaction amounts for a category in the selected period |
| Category Percentage | (Category Amount / Total Expenses) × 100 |
| Yearly Total | Sum of monthly totals for the selected year |
| Trend Point | Monthly expense total for each of the last 6 or 12 months |

---

### Business Rules

1. Statistics are always calculated from transactions, never stored.
2. The default view is the current month.
3. Month navigation is bounded by the earliest and latest months with transaction data.
4. Year navigation is bounded by the earliest and latest years with transaction data.
5. The category breakdown shows only expense categories for the expense view and income categories for the income view.
6. Categories with zero transactions in the selected period are excluded from the breakdown.
7. The category list is sorted by amount descending.
8. The trend chart shows the last 6 months by default. May be configurable to 12.
9. Months with no data show zero in the trend chart, not a gap.
10. Transfer transactions are excluded from all statistics.
11. Archived categories are included in statistics for historical accuracy.
12. Statistics update when the active Profile changes.

---

### Edge Cases

| Case | Expected Behavior |
|---|---|
| No transactions for the selected month | Show empty state: "No data for this month." |
| Only one month of data | Trend chart shows a single point. Yearly view shows one bar. |
| Category has both income and expense type | Each type is tracked separately. Expense breakdown only includes expense-type categories. |
| Very small amounts (e.g. 1 cent) | Display with proper currency formatting. Percentage rounds to nearest integer. |
| Many categories (20+) | Show top 10 in the chart. Remaining grouped as "Other." |
| Profile switch while viewing statistics | Refresh all data for the new profile. |
| Transactions at month boundary (e.g. 23:59 on the 31st) | Included in the correct month based on `occurred_at`. |

---

### Acceptance Criteria

1. The Statistics screen shows correct monthly totals for income, expenses and net.
2. The category breakdown chart accurately represents the distribution.
3. Category percentages sum to 100%.
4. Month navigation moves forward and backward correctly.
5. Yearly overview shows 12 months of data.
6. The trend chart shows monthly expense totals for the last 6 months.
7. Tapping a category navigates to its transactions for the selected period.
8. Empty state is shown when no data exists for the selected period.
9. Statistics refresh when the active Profile changes.
10. All charts are accessible (labeled, described for screen readers).
11. Unit tests cover all calculation queries.
12. Widget tests cover the statistics screen with data, empty state and navigation.

---

### Design References

- `docs/phase-0/07_DESIGN_SYSTEM.md` — Charts and Data Visualization, Chart Guidelines, Recommended Chart Types
- `docs/phase-0/03_MOBILE_ARCHITECTURE.md` — Screen States

---

### Test Requirements

| Type | Coverage |
|---|---|
| Unit | Monthly totals, category breakdown, yearly totals, trend data, percentage calculation |
| Integration | Statistics accuracy with mixed transaction types, archived categories included |
| Widget | Statistics screen with data, empty state, period navigation, chart rendering |

---

## Feature 7 — Data Export

### Goal and User Value

Data export gives users full ownership of their financial data. Users can export their data for external analysis, tax preparation, migration or personal archiving. Summa respects the principle that users own their data.

---

### Scope

#### Included

- Export all transactions as JSON
- Export all transactions as CSV
- Export scoped to the active Profile
- Include categories, profiles and transactions in JSON export
- CSV includes transaction fields with category and profile names
- Field selection for CSV export
- Share exported file through the system share sheet
- Export includes metadata (export date, profile name, currency)

#### Excluded

- Excel export (Phase 2 or P2)
- Export budgets (Phase 2)
- Export attachments (Phase 2)
- Scheduled or automatic exports
- Export to cloud storage directly
- Encrypted export

---

### User Stories and Scenarios

#### US-E1 — Export as JSON

As a user, I want to export my data as JSON so that I can keep a complete backup of my financial records in a portable format.

**Scenario:** User navigates to Settings → Export. Selects "JSON" format. Taps "Export." A JSON file is generated containing all profiles, categories and transactions for the active Workspace. The system share sheet opens so the user can save or send the file.

---

#### US-E2 — Export as CSV

As a user, I want to export my data as CSV so that I can open it in a spreadsheet application.

**Scenario:** User navigates to Settings → Export. Selects "CSV" format. Optionally selects which fields to include. Taps "Export." A CSV file is generated. The share sheet opens.

---

#### US-E3 — Select Fields for Export

As a user, I want to choose which fields are included in my CSV export so that I get only the data I need.

**Scenario:** After selecting CSV, the user sees a list of available fields: date, type, amount, currency, category, merchant, note, profile. User unchecks "note" and "merchant." Taps "Export." The CSV contains only the selected fields.

---

### UI Requirements

#### Screens

| Screen | Description |
|---|---|
| Export Screen | Accessible from Settings. Format selector (JSON/CSV). Field selection (for CSV). Export button. |

#### Components

| Component | Description |
|---|---|
| Format Selector | Radio buttons or segmented control: JSON, CSV. |
| Field Checklist | Checkbox list of available CSV fields. All selected by default. |
| Export Button | Primary action button. Shows loading state during export. |
| Export Success Feedback | Snackbar or dialog confirming export completion with file size. |

#### Navigation

```text
Settings → Export Screen → (export) → System Share Sheet
```

---

### Data Requirements

#### JSON Export Structure

```json
{
  "export_version": "1.0",
  "exported_at": "2026-03-27T12:00:00Z",
  "app_version": "1.0.0",
  "workspace": {
    "id": "...",
    "name": "Default"
  },
  "profiles": [
    {
      "id": "...",
      "name": "My Finances",
      "type": "personal",
      "currency": "EUR",
      "is_default": true
    }
  ],
  "categories": [
    {
      "id": "...",
      "name": "Food & Dining",
      "type": "expense",
      "icon": "restaurant",
      "color": "#E67E22",
      "is_default": true
    }
  ],
  "transactions": [
    {
      "id": "...",
      "profile_id": "...",
      "category_id": "...",
      "type": "expense",
      "amount_minor": 2550,
      "currency": "EUR",
      "note": "Lunch with colleagues",
      "merchant": "Restaurant ABC",
      "occurred_at": "2026-03-27T12:00:00Z",
      "created_at": "2026-03-27T12:00:00Z"
    }
  ]
}
```

#### CSV Export Columns

| Column | Description |
|---|---|
| date | `occurred_at` formatted as `YYYY-MM-DD` |
| type | `expense`, `income` or `transfer` |
| amount | Amount in major units (e.g. `25.50`) |
| currency | ISO 4217 code |
| category | Category name (empty for transfers) |
| merchant | Merchant name (empty if not set) |
| note | Transaction note (empty if not set) |
| profile | Profile name |

All columns are selectable. Date, type, amount and currency are always included.

#### File Naming

```text
summa_export_YYYYMMDD_HHmmss.json
summa_export_YYYYMMDD_HHmmss.csv
```

---

### Business Rules

1. Export includes only non-deleted (active) records.
2. JSON export includes all profiles, categories and transactions for the Workspace.
3. CSV export is scoped to the active Profile unless "All Profiles" is selected.
4. Amounts in CSV are in major units (divided by the currency's minor unit factor).
5. Amounts in JSON are in minor units (as stored).
6. The export file is generated in a temporary directory and shared through the system share sheet.
7. Temporary export files are deleted after the share operation completes.
8. Export does not include synchronization metadata (`version`, `sync_status`, `device_id`).
9. The export operation runs on a background isolate to avoid blocking the UI.
10. If export fails, an error message is shown with a retry option.

---

### Edge Cases

| Case | Expected Behavior |
|---|---|
| No transactions to export | Allow export. The file contains empty arrays. Show a warning: "No transactions to export." |
| Very large dataset (100,000+ transactions) | Export on background isolate. Show progress indicator. File may be large; warn user. |
| User cancels the share sheet | Temporary file is cleaned up. No error. |
| Storage permission denied | Show error with instructions to grant storage access. |
| CSV field with commas or newlines | Properly escape with CSV quoting rules. |
| Category or profile name contains special characters | Properly escape in both JSON and CSV. |
| Export during profile switch | Export uses the profile context at the time the export button was tapped. |

---

### Acceptance Criteria

1. JSON export produces a valid JSON file with all profiles, categories and transactions.
2. CSV export produces a valid CSV file with the selected fields.
3. CSV field selection allows toggling individual fields.
4. Exported file names include the date and time.
5. The system share sheet opens after export completes.
6. Temporary files are cleaned up after sharing.
7. Export does not block the UI.
8. Error states are handled with clear messages.
9. Unit tests verify JSON and CSV generation logic.
10. Widget tests cover the export screen and field selection.

---

### Design References

- `docs/phase-0/07_DESIGN_SYSTEM.md` — Forms, Buttons, Feedback
- `docs/phase-0/06_SECURITY_MODEL.md` — Data Export and Deletion

---

### Test Requirements

| Type | Coverage |
|---|---|
| Unit | JSON generation, CSV generation, field filtering, amount conversion, file naming, escaping |
| Integration | Full export with database data, large dataset performance |
| Widget | Export screen, format selector, field checklist, loading state, error state |

---

## Feature 8 — Backup and Restore

### Goal and User Value

Backup protects users against data loss from device failure, accidental deletion or app reinstallation. Restore allows users to recover their data from a previously created backup.

Backup and restore are critical trust features. Users must feel confident that their financial history is safe.

---

### Scope

#### Included

- Create a local backup (JSON archive)
- Backup includes all Workspace data: profiles, categories, transactions
- Backup file stored in user-accessible location (e.g. Downloads or app documents)
- Restore from a backup file
- Backup validation before restore
- Confirmation dialog before restore (warns about data replacement)
- Backup metadata (creation date, profile count, transaction count, app version)
- Backup file naming with date

#### Excluded

- Encrypted backups (depends on Decision 9 from Implementation Plan)
- Automatic scheduled backups
- Cloud backup
- Incremental backups
- Backup to external storage services
- Backup comparison or merge
- Selective restore (restore only specific profiles or date ranges)

---

### User Stories and Scenarios

#### US-B1 — Create Backup

As a user, I want to create a backup of all my data so that I can recover it if something goes wrong.

**Scenario:** User navigates to Settings → Backup. Taps "Create Backup." The app generates a JSON archive containing all Workspace data. A success message shows the file name and size. The user can share or save the file.

---

#### US-B2 — Restore from Backup

As a user, I want to restore my data from a backup so that I can recover after data loss.

**Scenario:** User navigates to Settings → Backup → Restore. Selects a backup file. The app validates the file and shows a summary: "This backup contains 3 profiles, 25 categories and 1,247 transactions from March 27, 2026. Restoring will replace all current data." User confirms. The current data is replaced with the backup data.

---

#### US-B3 — Validate Backup File

As a user, I want the app to verify a backup file before restoring so that I do not corrupt my data with an invalid file.

**Scenario:** User selects a file that is not a valid Summa backup. The app shows an error: "This file is not a valid Summa backup." The restore does not proceed.

---

### UI Requirements

#### Screens

| Screen | Description |
|---|---|
| Backup Screen | Accessible from Settings. "Create Backup" button. "Restore from Backup" button. List of recent backups (if stored locally). |
| Backup Confirmation Screen | Shows backup metadata after creation. Share and Done buttons. |
| Restore File Picker | System file picker for selecting a `.summa` or `.json` backup file. |
| Restore Confirmation Dialog | Shows backup summary (date, counts). Warning about data replacement. Confirm and Cancel buttons. |
| Restore Progress Screen | Progress indicator during restore. Cannot be cancelled once started. |
| Restore Success Screen | Confirmation that restore is complete. Option to restart the app. |

#### Components

| Component | Description |
|---|---|
| Backup Card | Shows backup file name, date, size and record counts. |
| Warning Banner | Yellow/orange banner explaining that restore replaces all current data. |
| Progress Indicator | Linear progress indicator during backup creation or restore. |

#### Navigation

```text
Settings → Backup Screen → Create Backup → Backup Confirmation
Settings → Backup Screen → Restore → File Picker → Restore Confirmation → Restore Progress → Success
```

---

### Data Requirements

#### Backup File Format

The backup file uses the same JSON structure as the export, with additional metadata:

```json
{
  "backup_version": "1.0",
  "created_at": "2026-03-27T12:00:00Z",
  "app_version": "1.0.0",
  "device_platform": "android",
  "summary": {
    "profile_count": 3,
    "category_count": 25,
    "transaction_count": 1247
  },
  "workspace": { "..." },
  "profiles": [ "..." ],
  "categories": [ "..." ],
  "transactions": [ "..." ]
}
```

#### File Extension

```text
.summa
```

Or `.json` with a recognizable naming pattern:

```text
summa_backup_YYYYMMDD_HHmmss.summa
```

#### Restore Process

1. Read and parse the backup file.
2. Validate the `backup_version` field.
3. Validate the JSON structure.
4. Display summary to the user.
5. On confirmation, begin restore inside a database transaction:
   a. Soft-delete all existing data.
   b. Insert all profiles, categories and transactions from the backup.
   c. Update the Workspace reference.
6. Refresh all providers and screens.

---

### Business Rules

1. Backup includes all non-deleted data for the Workspace.
2. Backup does not include synchronization metadata (`sync_status`, `device_id`).
3. Backup does not include soft-deleted records.
4. Restore replaces all current data. This is a destructive operation.
5. Restore must be confirmed by the user with a clear warning.
6. The restore process must be atomic (all data is replaced, or nothing changes on failure).
7. Backup validation checks for required fields and structure integrity.
8. Backup files from newer app versions may not be restorable on older versions. Show a clear error.
9. Backup files from older versions should be restorable on newer versions (forward compatibility).
10. The backup file should be shareable through the system share sheet.
11. Backup creation and restore run on background isolates.

---

### Edge Cases

| Case | Expected Behavior |
|---|---|
| Backup file is corrupted or truncated | Validation error: "This file appears to be corrupted." |
| Backup file is from a newer app version | Error: "This backup was created with a newer version of Summa. Update the app to restore." |
| Backup file is from a different app entirely | Error: "This file is not a valid Summa backup." |
| User cancels restore after selecting a file | No changes made. Return to backup screen. |
| App crashes during restore | On next launch, the database should be in a consistent state (either old data or new data, not mixed). Database transaction ensures atomicity. |
| Very large backup (50,000+ transactions) | Show progress indicator. Restore may take several seconds. |
| Backup file is extremely small (< 100 bytes) | Likely invalid. Validation catches this. |
| User tries to restore the same backup twice | Allowed. Idempotent in effect (same data replaces same data). |
| Storage is full during backup creation | Error: "Not enough storage space to create backup." |
| User selects a non-JSON file | Validation error before any data processing. |

---

### Acceptance Criteria

1. Creating a backup produces a valid `.summa` file with all Workspace data.
2. The backup file includes metadata (version, date, counts).
3. Restoring a valid backup replaces all current data with the backup data.
4. Restoring an invalid file shows an error and does not modify any data.
5. The restore confirmation dialog clearly warns about data replacement.
6. The restore process is atomic (database transaction).
7. Backup and restore run on background isolates without blocking the UI.
8. Progress indicators are shown during backup and restore.
9. Backup files are shareable through the system share sheet.
10. Unit tests cover backup generation, validation and restore logic.
11. Widget tests cover the backup screen, confirmation dialogs and progress states.

---

### Design References

- `docs/phase-0/06_SECURITY_MODEL.md` — Backup Security, Local Backup Encryption
- `docs/phase-0/04_BACKEND_ARCHITECTURE.md` — Backup and Recovery
- `docs/phase-1/01_IMPLEMENTATION_PLAN.md` — Decision 12: Backup Format

---

### Test Requirements

| Type | Coverage |
|---|---|
| Unit | Backup generation, JSON validation, restore logic, version compatibility, atomicity |
| Integration | Full backup and restore cycle, data integrity after restore, large dataset performance |
| Widget | Backup screen, create flow, restore flow, confirmation dialogs, error states, progress |

---

## Feature 9 — Settings

### Goal and User Value

Settings allow users to personalize their Summa experience. Theme, currency and language preferences make the app feel native to the user's context. Data management controls give users authority over their stored data.

---

### Scope

#### Included

- Theme selection (light, dark, system)
- Default currency setting (applies to new Profiles)
- Language selection (infrastructure; English for MVP)
- Data deletion (delete all local data with confirmation)
- App version display
- Link to backup and restore
- Link to export
- Link to profile management
- Link to category management
- Link to application lock settings

#### Excluded

- Custom theme creation
- Font size adjustment (uses system settings)
- Notification preferences (Phase 2)
- Sync settings (Phase 3)
- Account management (Phase 3)
- About page with open-source licenses (may be added)
- Debug or developer options

---

### User Stories and Scenarios

#### US-S1 — Change Theme

As a user, I want to switch between light and dark themes so that the app is comfortable to use in different lighting conditions.

**Scenario:** User navigates to Settings → Appearance. Taps "Theme." A bottom sheet shows three options: Light, Dark, System. User selects "Dark." The entire app immediately switches to the dark theme. The selection is persisted.

---

#### US-S2 — Set Default Currency

As a user, I want to set my preferred currency so that new profiles use it by default.

**Scenario:** User navigates to Settings → Currency. A searchable list of ISO 4217 currencies appears. User selects "EUR." New profiles will default to EUR. Existing profiles retain their current currency.

---

#### US-S3 — Delete All Data

As a user, I want to delete all my data from the app so that I can start fresh or remove my information before uninstalling.

**Scenario:** User navigates to Settings → Data → Delete All Data. A multi-step confirmation warns: "This will permanently delete all profiles, categories and transactions. This cannot be undone." User must type "DELETE" or confirm through a two-step dialog. After confirmation, all data is deleted. The app returns to the onboarding screen.

---

#### US-S4 — View App Version

As a user, I want to see the app version so that I can report bugs accurately.

**Scenario:** User scrolls to the bottom of Settings. The app version and build number are displayed (e.g. "Version 1.0.0 (Build 42)").

---

### UI Requirements

#### Screens

| Screen | Description |
|---|---|
| Settings Screen | Main settings list. Grouped sections: Appearance, Data Management, Security, About. |
| Theme Selection | Bottom sheet or dialog with Light, Dark, System options. |
| Currency Selection | Searchable list of ISO 4217 currencies with codes and names. |
| Language Selection | List of available languages. MVP: English only. |
| Data Management Screen | Export, Backup, Delete All Data options. |
| Delete Confirmation Dialog | Multi-step confirmation. Requires typing "DELETE" or a two-step acknowledge + confirm flow. |

#### Components

| Component | Description |
|---|---|
| Settings Group | Section header with grouped settings rows. |
| Settings Row | Icon, title, optional subtitle, optional trailing widget (chevron, switch, value). |
| Theme Preview | Small preview showing light and dark appearance. |
| Currency Row | Currency code, name and symbol. |
| Danger Zone | Red-highlighted section for destructive actions (delete data). |

#### Settings Structure

```text
Settings
├── Appearance
│   ├── Theme (Light / Dark / System)
│   └── Language (English)
├── Profiles
│   └── Manage Profiles
├── Categories
│   └── Manage Categories
├── Data Management
│   ├── Export Data
│   ├── Backup and Restore
│   └── Delete All Data (danger zone)
├── Security
│   └── Application Lock
└── About
    └── Version 1.0.0 (Build 42)
```

#### Navigation

```text
Bottom Navigation → Settings Screen
Settings → Theme Selection
Settings → Currency Selection
Settings → Language Selection
Settings → Profile Management
Settings → Category Management
Settings → Export Screen
Settings → Backup Screen
Settings → Delete Confirmation
Settings → Application Lock Settings
```

---

### Data Requirements

#### Stored Preferences

| Preference | Storage | Default |
|---|---|---|
| theme_mode | `shared_preferences` | `system` |
| default_currency | `shared_preferences` | `USD` |
| language | `shared_preferences` | `en` |

#### Queries for Data Deletion

- `deleteAllProfiles(workspaceId)` — Soft-delete all profiles.
- `deleteAllCategories(workspaceId)` — Soft-delete all categories.
- `deleteAllTransactions(workspaceId)` — Soft-delete all transactions.
- Or: `deleteWorkspace(workspaceId)` — Soft-delete the workspace and cascade.

Data deletion may use hard delete for a true "start fresh" experience. Decision: hard delete with database recreation.

---

### Business Rules

1. Theme changes take effect immediately without app restart.
2. The "System" theme option follows the operating system's light/dark mode.
3. Default currency applies only to newly created Profiles. Existing Profiles are not changed.
4. Language selection sets the app locale. MVP supports English only. The infrastructure must support adding languages without code changes.
5. Data deletion is irreversible. The confirmation must be explicit and unambiguous.
6. Data deletion removes all profiles, categories, transactions and preferences.
7. After data deletion, the app returns to the onboarding screen as if it were a fresh install.
8. The Workspace is recreated after data deletion.
9. Application lock settings are accessible from the Security section.
10. The app version is read from the package info and cannot be edited.

---

### Edge Cases

| Case | Expected Behavior |
|---|---|
| User changes theme while on a screen with custom colors | All screens update immediately. No visual glitches. |
| User changes default currency after creating profiles | Existing profiles keep their currency. New profiles use the new default. |
| User tries to delete data with no data present | Allow the operation. Effectively a no-op but the confirmation still appears. |
| App crashes during data deletion | On next launch, the app checks database integrity and offers to reset if corrupted. |
| User selects a language not yet supported | Not available in MVP. The option is hidden or disabled. |
| System theme changes while app is open (e.g. sunset) | The app updates in real time if "System" theme is selected. |

---

### Acceptance Criteria

1. Theme selection changes the app appearance immediately.
2. Light, Dark and System themes all work correctly.
3. Default currency is saved and applied to new profiles.
4. Currency selection shows all ISO 4217 codes with searchable list.
5. Language infrastructure is in place (English works).
6. Data deletion removes all data and returns to onboarding.
7. Data deletion requires explicit multi-step confirmation.
8. App version is displayed correctly.
9. All settings are persisted across app restarts.
10. Settings screen navigates correctly to all sub-screens.
11. Unit tests cover preference storage and data deletion.
12. Widget tests cover the settings screen, theme selection and delete confirmation.

---

### Design References

- `docs/phase-0/07_DESIGN_SYSTEM.md` — Dark Theme, Color System, Core Components
- `docs/phase-0/06_SECURITY_MODEL.md` — Data Export and Deletion, Local Data Deletion
- `docs/phase-1/01_IMPLEMENTATION_PLAN.md` — Decision 10: Supported Currencies, Decision 11: Supported Languages

---

### Test Requirements

| Type | Coverage |
|---|---|
| Unit | Theme persistence, currency persistence, data deletion completeness |
| Integration | Theme change propagation, data deletion with database verification |
| Widget | Settings screen, theme picker, currency picker, delete confirmation flow |

---

## Feature 10 — Application Lock

### Goal and User Value

Application Lock protects financial data from unauthorized access. When enabled, the user must authenticate (via PIN or biometrics) before accessing the app. This is critical for privacy, especially on shared or lost devices.

---

### Scope

#### Included

- PIN setup (4–8 digit numeric PIN)
- PIN verification on app launch and resume
- Biometric authentication (fingerprint, face) as alternative to PIN
- Biometric availability detection
- Lock timeout configuration (immediate, after 1 minute, after 5 minutes)
- PIN change
- PIN removal (disable lock)
- Failed attempt handling (delay after N failures)
- Secure PIN storage (hashed, not plaintext)

#### Excluded

- Pattern lock
- Password lock (alphanumeric)
- Two-factor authentication
- Remote wipe
- Per-screen lock (lock only specific screens)
- Decoy PIN (opens a fake profile)

---

### User Stories and Scenarios

#### US-L1 — Setup PIN

As a user, I want to set up a PIN so that others cannot access my financial data.

**Scenario:** User navigates to Settings → Security → Application Lock. Taps "Enable PIN." Enters a 4-digit PIN. Confirms the PIN by entering it again. PIN is set. The app will now require the PIN on launch.

---

#### US-L2 — Unlock with PIN

As a user, I want to unlock the app with my PIN so that I can access my data securely.

**Scenario:** User opens the app. A PIN entry screen appears. User enters their 4-digit PIN. The app unlocks and shows the Dashboard.

---

#### US-L3 — Unlock with Biometrics

As a user, I want to unlock the app with my fingerprint so that I can access it quickly.

**Scenario:** User opens the app. The biometric prompt appears automatically (if biometrics are available and enabled). User places their finger on the sensor. The app unlocks.

---

#### US-L4 — Configure Lock Timeout

As a user, I want to configure how quickly the app locks after I leave it so that I can balance security and convenience.

**Scenario:** User navigates to Settings → Security → Lock Timeout. Options: Immediately, After 1 minute, After 5 minutes. User selects "After 1 minute." The app will only require authentication if it has been in the background for more than 1 minute.

---

#### US-L5 — Change PIN

As a user, I want to change my PIN so that I can update it for security.

**Scenario:** User navigates to Settings → Security → Change PIN. Enters current PIN. Enters new PIN. Confirms new PIN. PIN is updated.

---

#### US-L6 — Remove PIN

As a user, I want to disable the application lock so that I can access the app without entering a PIN.

**Scenario:** User navigates to Settings → Security → Application Lock. Taps "Disable." Enters current PIN to confirm. Application lock is disabled.

---

### UI Requirements

#### Screens

| Screen | Description |
|---|---|
| PIN Entry Screen | Full-screen overlay. Numeric keypad. PIN dots (4–8). Title: "Enter PIN." Biometric button if available. |
| PIN Setup Screen | Enter new PIN. Confirm new PIN. Title: "Create PIN." |
| PIN Change Screen | Enter current PIN. Enter new PIN. Confirm new PIN. |
| Application Lock Settings | Enable/disable toggle. Lock timeout selector. Biometric toggle. Change PIN option. |
| Biometric Prompt | System-provided biometric dialog (not a custom screen). |

#### Components

| Component | Description |
|---|---|
| PIN Keypad | Numeric grid (1–9, 0). Delete button. Optional biometric button. |
| PIN Dots | Row of 4–8 dots indicating entered digits. Filled dots for entered, empty for remaining. Error animation on wrong PIN. |
| Lock Timeout Selector | Dropdown or radio options: Immediately, 1 minute, 5 minutes. |
| Biometric Toggle | Switch to enable/disable biometric unlock. Only visible if biometrics are available on the device. |

#### Navigation

```text
App Launch → PIN Entry Screen (if lock is enabled and timeout expired)
App Resume → PIN Entry Screen (if lock is enabled and timeout expired)
Settings → Application Lock Settings → PIN Setup
Settings → Application Lock Settings → PIN Change
Settings → Application Lock Settings → Lock Timeout
```

---

### Data Requirements

#### Stored Values

| Value | Storage | Description |
|---|---|---|
| pin_hash | `flutter_secure_storage` | SHA-256 hash of the PIN with salt |
| pin_salt | `flutter_secure_storage` | Random salt for PIN hashing |
| pin_enabled | `shared_preferences` | Boolean |
| biometric_enabled | `shared_preferences` | Boolean |
| lock_timeout_seconds | `shared_preferences` | 0 (immediate), 60, 300 |
| last_background_at | `shared_preferences` | Timestamp when app went to background |

#### PIN Hashing

```text
hash = SHA-256(pin + salt)
```

The PIN is never stored in plaintext. The hash and salt are stored in `flutter_secure_storage` (Android Keystore / iOS Keychain).

---

### Business Rules

1. PIN must be 4 to 8 digits. (Review: fixed at 4 for MVP simplicity.)
2. PIN is hashed with a random salt using SHA-256 before storage.
3. PIN is never stored in plaintext.
4. After 5 consecutive failed PIN attempts, a 30-second delay is enforced.
5. After 10 consecutive failed attempts, a 5-minute delay is enforced.
6. Biometric authentication is only offered if the device supports it.
7. Biometric authentication uses the `local_auth` Flutter plugin.
8. Summa never receives or stores raw biometric data.
9. The lock timeout determines when the PIN screen appears:
   - Immediately: every time the app comes to foreground.
   - 1 minute: only if the app was in the background for more than 1 minute.
   - 5 minutes: only if the app was in the background for more than 5 minutes.
10. The PIN screen appears before any financial data is rendered.
11. Disabling the lock requires entering the current PIN.
12. Changing the PIN requires entering the current PIN first.
13. The biometric prompt is offered before the PIN entry (user can skip to PIN).
14. If biometric authentication fails, the user can fall back to PIN.

---

### Edge Cases

| Case | Expected Behavior |
|---|---|
| User forgets PIN | No recovery mechanism in Phase 1. User must clear app data (which deletes all data). This must be documented clearly. (Review: may add security questions in Phase 2.) |
| Device has no biometric hardware | Biometric option is hidden. Only PIN is available. |
| User removes all biometric data from device settings | Biometric option becomes unavailable in Summa. PIN remains. |
| App is killed while PIN screen is shown | On next launch, the PIN screen appears again. |
| User rapidly switches between apps | Lock timeout is based on the time the app was backgrounded, not the number of switches. |
| PIN entry screen appears over a loading screen | The PIN screen must be the topmost layer. No financial data leaks behind it. |
| Very slow device, PIN screen takes time to appear | The app must not render any financial content before the PIN screen is ready. Use a splash screen or blank screen as placeholder. |
| User sets lock timeout to "Immediately" and uses biometrics | Biometric prompt appears every time the app comes to foreground. |

---

### Acceptance Criteria

1. User can set up a 4-digit PIN.
2. PIN is stored as a hash, never in plaintext.
3. The PIN entry screen appears on app launch when lock is enabled.
4. Correct PIN unlocks the app.
5. Incorrect PIN shows an error and increments the failure counter.
6. After 5 failed attempts, a 30-second delay is enforced.
7. Biometric authentication works on devices with fingerprint or face recognition.
8. Biometric option is hidden on devices without biometric hardware.
9. Lock timeout configuration works correctly for all options.
10. Changing the PIN requires the current PIN.
11. Disabling the lock requires the current PIN.
12. No financial data is visible behind the PIN screen.
13. Unit tests cover PIN hashing, verification and failure counting.
14. Widget tests cover the PIN entry screen, setup flow and settings screen.

---

### Design References

- `docs/phase-0/06_SECURITY_MODEL.md` — Application Lock, Biometric Authentication, Application PIN, Screen Protection
- `docs/phase-0/07_DESIGN_SYSTEM.md` — Core Components, Motion

---

### Test Requirements

| Type | Coverage |
|---|---|
| Unit | PIN hashing, PIN verification, failure counting, timeout calculation, biometric availability check |
| Integration | Lock behavior across app lifecycle (launch, background, resume), secure storage interaction |
| Widget | PIN entry screen, PIN setup flow, PIN change flow, lock settings, biometric prompt fallback |

---

## Cross-Feature Requirements

### Performance

| Metric | Target |
|---|---|
| App cold start | < 2 seconds on mid-range device |
| Dashboard load | < 300ms with 10,000 transactions |
| Transaction list scroll | 60 fps with 10,000 rows |
| Statistics calculation | < 500ms for monthly view |
| Backup creation | < 5 seconds for 10,000 transactions |
| Restore | < 10 seconds for 10,000 transactions |
| Export (JSON) | < 3 seconds for 10,000 transactions |
| Export (CSV) | < 3 seconds for 10,000 transactions |

---

### Accessibility

Every screen must:

- Support dynamic font sizes (system accessibility settings)
- Provide semantic labels for screen readers
- Maintain WCAG AA contrast ratios
- Support minimum touch targets of 44–48 px
- Announce errors and state changes
- Support reduced motion settings
- Not rely solely on color to convey meaning

---

### Localization

All user-facing strings must be externalized for future translation.

MVP language: English.

Infrastructure must support:

- String externalization (ARB files or equivalent)
- Locale-aware number formatting
- Locale-aware date formatting
- Currency formatting per locale
- RTL layout readiness (future)

---

### Error Handling

Every feature must handle:

| Error Type | Behavior |
|---|---|
| Database unavailable | Show error screen with retry option |
| Validation failure | Show inline error message, preserve entered data |
| Unexpected exception | Log error, show generic error message, offer retry |
| Operation timeout | Show timeout message, offer retry |

Error messages must be:

- Specific (not "Something went wrong")
- Actionable (suggest what the user can do)
- Non-technical (no stack traces or error codes visible to users)

---

### State Management

Every screen follows the MVVM pattern defined in `docs/phase-0/03_MOBILE_ARCHITECTURE.md`:

```text
Event → ViewModel → State → UI
```

State must include:

- Loading state (initial load, background refresh)
- Content state (data available)
- Empty state (no data)
- Error state (operation failed)

---

## Definition of Done

A Phase 1 feature is considered complete when:

1. All acceptance criteria in its specification are met.
2. Domain models are defined and tested.
3. Use cases are implemented and tested.
4. Repository interface exists in the domain layer.
5. Repository implementation exists in the data layer.
6. drift DAOs are implemented and tested.
7. ViewModel handles all events correctly.
8. UI renders all states (loading, content, empty, error).
9. Widget tests cover the screen.
10. Code follows the coding guidelines.
11. No linting warnings exist.
12. Accessibility requirements are met.
13. Performance targets are met.
14. PR is reviewed and approved.

Phase 1 is complete when all ten features meet their individual Definition of Done and the cross-feature requirements are satisfied.

---

## Guiding Principle

Every feature in this document exists to serve the user.

The specifications define what to build and why. They do not prescribe every implementation detail — those decisions belong to the developer closest to the code.

When a specification is ambiguous, prefer the simpler solution. When two approaches seem equal, prefer the one that is easier to test.

The user should never need to read this document to understand how a feature works. The feature should be self-explanatory through its interface.
