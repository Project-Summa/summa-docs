# Design Implementation

> Implementing the Summa design system in Flutter for Phase 1.

---

# Table of Contents

- Purpose
- Design System Reference
- Design Token Implementation
- Theme Configuration
- Color Implementation
- Typography Implementation
- Spacing Implementation
- Component Library
- Financial Components
- Screen States
- Dark Theme
- Dynamic Colors
- Accessibility Implementation
- Design Review Process
- Definition of Done
- Guiding Principle

---

# Purpose

This document defines how the design system from Phase 0 is implemented in Flutter during Phase 1.

The design system defines the visual language. This document translates that language into code — theme data, reusable widgets, design tokens and component patterns.

The goal is to ensure that every screen in the application looks and feels consistent without developers making ad-hoc design decisions.

---

# Design System Reference

All visual decisions are defined in:

```text
docs/phase-0/07_DESIGN_SYSTEM.md
```

This document focuses on the Flutter implementation of those decisions.

Do not redefine colors, typography or spacing here. Reference the design system and implement what it specifies.

---

# Design Token Implementation

## Token Structure

Design tokens are organized as static constants in Dart.

```text
lib/
    core/
        design/
            tokens/
                colors.dart
                typography.dart
                spacing.dart
                radius.dart
                elevation.dart
                duration.dart
```

## Token Classes

```dart
abstract final class SummaColors {
  // Primary
  static const Color primary = Color(0xFF2C3E50);
  static const Color primaryLight = Color(0xFF34495E);
  static const Color primaryDark = Color(0xFF1A252F);

  // Neutral
  static const Color white = Color(0xFFFFFFFF);
  static const Color black = Color(0xFF000000);
  static const Color grey50 = Color(0xFFF8F9FA);
  static const Color grey100 = Color(0xFFF1F3F5);
  static const Color grey200 = Color(0xFFE9ECEF);
  static const Color grey300 = Color(0xFFDEE2E6);
  static const Color grey400 = Color(0xFFCED4DA);
  static const Color grey500 = Color(0xFFADB5BD);
  static const Color grey600 = Color(0xFF868E96);
  static const Color grey700 = Color(0xFF495057);
  static const Color grey800 = Color(0xFF343A40);
  static const Color grey900 = Color(0xFF212529);

  // Semantic
  static const Color success = Color(0xFF27AE60);
  static const Color warning = Color(0xFFF39C12);
  static const Color error = Color(0xFFE74C3C);
  static const Color info = Color(0xFF3498DB);

  // Financial
  static const Color income = Color(0xFF27AE60);
  static const Color expense = Color(0xFFE74C3C);
  static const Color transfer = Color(0xFF3498DB);
}
```

```dart
abstract final class SummaTypography {
  static const String fontFamily = 'Inter';

  static const TextStyle h1 = TextStyle(
    fontFamily: fontFamily,
    fontSize: 32,
    fontWeight: FontWeight.w700,
    height: 1.25,
  );

  static const TextStyle h2 = TextStyle(
    fontFamily: fontFamily,
    fontSize: 24,
    fontWeight: FontWeight.w700,
    height: 1.33,
  );

  static const TextStyle h3 = TextStyle(
    fontFamily: fontFamily,
    fontSize: 20,
    fontWeight: FontWeight.w600,
    height: 1.4,
  );

  static const TextStyle bodyLarge = TextStyle(
    fontFamily: fontFamily,
    fontSize: 16,
    fontWeight: FontWeight.w400,
    height: 1.5,
  );

  static const TextStyle bodyMedium = TextStyle(
    fontFamily: fontFamily,
    fontSize: 14,
    fontWeight: FontWeight.w400,
    height: 1.43,
  );

  static const TextStyle bodySmall = TextStyle(
    fontFamily: fontFamily,
    fontSize: 12,
    fontWeight: FontWeight.w400,
    height: 1.33,
  );

  static const TextStyle labelLarge = TextStyle(
    fontFamily: fontFamily,
    fontSize: 14,
    fontWeight: FontWeight.w500,
    height: 1.43,
  );

  static const TextStyle amountLarge = TextStyle(
    fontFamily: fontFamily,
    fontSize: 28,
    fontWeight: FontWeight.w700,
    height: 1.29,
    letterSpacing: -0.5,
  );

  static const TextStyle amountMedium = TextStyle(
    fontFamily: fontFamily,
    fontSize: 20,
    fontWeight: FontWeight.w600,
    height: 1.4,
    letterSpacing: -0.3,
  );
}
```

```dart
abstract final class SummaSpacing {
  static const double xxs = 2;
  static const double xs = 4;
  static const double sm = 8;
  static const double md = 12;
  static const double base = 16;
  static const double lg = 24;
  static const double xl = 32;
  static const double xxl = 48;
  static const double xxxl = 64;
}
```

```dart
abstract final class SummaRadius {
  static const double xs = 4;
  static const double sm = 8;
  static const double md = 12;
  static const double lg = 16;
  static const double xl = 24;
  static const double full = 999;
}
```

---

# Theme Configuration

## Material 3 Theme

The application uses Material 3 theming.

```dart
ThemeData buildLightTheme() {
  final colorScheme = ColorScheme.light(
    primary: SummaColors.primary,
    secondary: SummaColors.primaryLight,
    surface: SummaColors.white,
    error: SummaColors.error,
    onPrimary: SummaColors.white,
    onSecondary: SummaColors.white,
    onSurface: SummaColors.grey900,
    onError: SummaColors.white,
  );

  return ThemeData(
    useMaterial3: true,
    colorScheme: colorScheme,
    fontFamily: SummaTypography.fontFamily,
    scaffoldBackgroundColor: SummaColors.grey50,
    appBarTheme: AppBarTheme(
      backgroundColor: SummaColors.white,
      foregroundColor: SummaColors.grey900,
      elevation: 0,
      centerTitle: true,
    ),
    bottomNavigationBarTheme: BottomNavigationBarThemeData(
      backgroundColor: SummaColors.white,
      selectedItemColor: SummaColors.primary,
      unselectedItemColor: SummaColors.grey500,
      type: BottomNavigationBarType.fixed,
    ),
    cardTheme: CardTheme(
      color: SummaColors.white,
      elevation: 0,
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(SummaRadius.md),
      ),
    ),
    inputDecorationTheme: InputDecorationTheme(
      filled: true,
      fillColor: SummaColors.grey50,
      border: OutlineInputBorder(
        borderRadius: BorderRadius.circular(SummaRadius.sm),
        borderSide: BorderSide(color: SummaColors.grey300),
      ),
      enabledBorder: OutlineInputBorder(
        borderRadius: BorderRadius.circular(SummaRadius.sm),
        borderSide: BorderSide(color: SummaColors.grey300),
      ),
      focusedBorder: OutlineInputBorder(
        borderRadius: BorderRadius.circular(SummaRadius.sm),
        borderSide: BorderSide(color: SummaColors.primary, width: 2),
      ),
    ),
  );
}
```

---

# Color Implementation

## Rules

- Never use hardcoded color values in widgets
- Always reference `SummaColors` or `Theme.of(context).colorScheme`
- Financial colors (income/expense/transfer) use semantic colors
- Category colors are user-defined and stored in the database

## Color Usage

```dart
// Correct
Container(
  color: Theme.of(context).colorScheme.surface,
)

// Correct
Text(
  '+\$1,250.50',
  style: SummaTypography.bodyLarge.copyWith(
    color: SummaColors.income,
  ),
)

// Wrong
Container(
  color: Color(0xFF2C3E50), // Hardcoded
)
```

---

# Typography Implementation

## Rules

- Use `SummaTypography` constants for all text styles
- Financial amounts use `amountLarge` or `amountMedium`
- Body text uses `bodyLarge`, `bodyMedium` or `bodySmall`
- Headings use `h1`, `h2` or `h3`
- Labels use `labelLarge`

## Amount Formatting

Financial amounts should be formatted consistently.

```dart
String formatAmount(int amountMinor, String currency) {
  final amount = amountMinor / 100;
  final formatter = NumberFormat.currency(
    symbol: getCurrencySymbol(currency),
    decimalDigits: 2,
  );
  return formatter.format(amount);
}
```

---

# Spacing Implementation

## Rules

- Use `SummaSpacing` constants for all spacing
- Follow the 4px base grid
- Common spacing values:
  - Between related items: `SummaSpacing.sm` (8px)
  - Between sections: `SummaSpacing.base` (16px)
  - Page padding: `SummaSpacing.base` (16px)
  - Between major sections: `SummaSpacing.lg` (24px)

## Layout Pattern

```dart
Padding(
  padding: EdgeInsets.all(SummaSpacing.base),
  child: Column(
    crossAxisAlignment: CrossAxisAlignment.start,
    children: [
      Text('Monthly Summary', style: SummaTypography.h3),
      SizedBox(height: SummaSpacing.sm),
      BalanceCard(),
      SizedBox(height: SummaSpacing.base),
      Text('Recent Transactions', style: SummaTypography.h3),
      SizedBox(height: SummaSpacing.sm),
      TransactionList(),
    ],
  ),
)
```

---

# Component Library

## Core Components

| Component | Description | Status |
|---|---|---|
| SummaButton | Primary, secondary, text variants | Phase 1 |
| SummaInput | Text input with label and error | Phase 1 |
| SummaCard | Content container with consistent styling | Phase 1 |
| SummaListItem | List row with leading/trailing widgets | Phase 1 |
| SummaDialog | Confirmation and info dialogs | Phase 1 |
| SummaBottomSheet | Action sheets and selection | Phase 1 |
| SummaChip | Filter and selection chips | Phase 1 |
| SummaAvatar | Profile avatar with initials | Phase 1 |
| SummaIcon | Consistent icon wrapper | Phase 1 |
| SummaDivider | Consistent divider | Phase 1 |
| SummaEmptyState | Empty state with icon and message | Phase 1 |
| SummaErrorState | Error state with retry | Phase 1 |
| SummaLoadingState | Loading indicator | Phase 1 |

## Component Structure

```text
lib/
    core/
        design/
            components/
                summa_button.dart
                summa_input.dart
                summa_card.dart
                summa_list_item.dart
                summa_dialog.dart
                summa_bottom_sheet.dart
                summa_chip.dart
                summa_avatar.dart
                summa_icon.dart
                summa_divider.dart
                summa_empty_state.dart
                summa_error_state.dart
                summa_loading_state.dart
```

---

# Financial Components

These components are specific to financial data display.

| Component | Description | Status |
|---|---|---|
| BalanceCard | Displays current balance with trend | Phase 1 |
| TransactionRow | Single transaction in a list | Phase 1 |
| CategoryRow | Category with icon and color | Phase 1 |
| SummaryCard | Monthly income/expense summary | Phase 1 |
| AmountText | Formatted financial amount | Phase 1 |
| CategoryIcon | Category icon with background color | Phase 1 |
| DateHeader | Date separator in transaction lists | Phase 1 |

## Balance Card

```dart
class BalanceCard extends StatelessWidget {
  final int balanceMinor;
  final String currency;
  final int? trendPercentage;

  // Displays:
  // - Large balance amount
  // - Currency symbol
  // - Optional trend indicator (up/down arrow with percentage)
  // - "Current Balance" label
}
```

## Transaction Row

```dart
class TransactionRow extends StatelessWidget {
  final Transaction transaction;
  final Category? category;
  final VoidCallback? onTap;

  // Displays:
  // - Category icon (colored circle with icon)
  // - Category name
  // - Transaction note or merchant
  // - Amount (positive for income, negative for expense)
  // - Date
}
```

---

# Screen States

Every screen must handle four states.

## Loading State

```dart
class SummaLoadingState extends StatelessWidget {
  final String? message;

  // Displays centered CircularProgressIndicator
  // Optional message below the indicator
}
```

## Content State

The normal state when data is available.

```dart
// Screen receives data and renders content
// No special wrapper needed — just render the content
```

## Empty State

```dart
class SummaEmptyState extends StatelessWidget {
  final IconData icon;
  final String title;
  final String? message;
  final String? actionLabel;
  final VoidCallback? onAction;

  // Displays:
  // - Large icon
  // - Title text
  // - Optional message
  // - Optional action button
}
```

## Error State

```dart
class SummaErrorState extends StatelessWidget {
  final String message;
  final VoidCallback? onRetry;

  // Displays:
  // - Error icon
  // - Error message
  // - Retry button
}
```

---

# Dark Theme

## Dark Color Scheme

```dart
ThemeData buildDarkTheme() {
  final colorScheme = ColorScheme.dark(
    primary: SummaColors.primaryLight,
    secondary: SummaColors.primary,
    surface: SummaColors.grey900,
    error: SummaColors.error,
    onPrimary: SummaColors.white,
    onSecondary: SummaColors.white,
    onSurface: SummaColors.grey100,
    onError: SummaColors.white,
  );

  return ThemeData(
    useMaterial3: true,
    colorScheme: colorScheme,
    fontFamily: SummaTypography.fontFamily,
    scaffoldBackgroundColor: SummaColors.grey900,
    // ... dark theme overrides
  );
}
```

## Dark Theme Rules

- All colors from `SummaColors` adapt to dark mode via `ColorScheme`
- Financial colors (income/expense) remain the same in both themes
- Contrast ratios must remain accessible
- Test both themes for every screen

---

# Dynamic Colors

Android 12+ supports dynamic colors (Material You).

```dart
ThemeData buildTheme(Brightness brightness) {
  final colorScheme = ColorScheme.fromSeed(
    seedColor: SummaColors.primary,
    brightness: brightness,
  );

  return ThemeData(
    useMaterial3: true,
    colorScheme: colorScheme,
    // ... theme overrides
  );
}
```

Dynamic colors are optional. The app should look correct with or without them.

---

# Accessibility Implementation

## Requirements

- All text meets WCAG 2.1 AA contrast ratios (4.5:1 for normal text)
- Touch targets are at least 48x48 dp
- All images have semantic labels
- All interactive elements have semantic labels
- Dynamic font sizes are supported
- Screen reader navigation is logical

## Implementation

```dart
// Semantic labels
Semantics(
  label: 'Add new transaction',
  button: true,
  child: IconButton(
    icon: Icon(Icons.add),
    onPressed: () => navigateToCreateTransaction(),
  ),
)

// Contrast checking
// Use the contrast ratio checker in the design review checklist
```

---

# Design Review Process

## Before Merging

Every UI change should be reviewed for:

- [ ] Uses design tokens (no hardcoded values)
- [ ] Matches design system specifications
- [ ] Works in light theme
- [ ] Works in dark theme
- [ ] Handles loading state
- [ ] Handles empty state
- [ ] Handles error state
- [ ] Accessible (contrast, semantics, touch targets)
- [ ] Responsive to different screen sizes
- [ ] No overflow on small screens

## Design Review Checklist

```text
Visual Consistency
- [ ] Colors match design system
- [ ] Typography matches design system
- [ ] Spacing matches design system
- [ ] Border radius matches design system

Component Usage
- [ ] Uses standard components where possible
- [ ] New components follow existing patterns
- [ ] Component names follow naming convention

Accessibility
- [ ] Contrast ratios pass WCAG AA
- [ ] Touch targets are 48x48 minimum
- [ ] Semantic labels are present
- [ ] Dynamic font sizes work

Theme Support
- [ ] Light theme looks correct
- [ ] Dark theme looks correct
- [ ] Dynamic colors work (Android 12+)
```

---

# Definition of Done

Design implementation is complete when:

- All design tokens are implemented as Dart constants
- Material 3 theme is configured for light and dark modes
- Core component library is implemented
- Financial components are implemented
- Screen state components are implemented
- All screens use design tokens (no hardcoded values)
- Both themes are tested on all screens
- Accessibility requirements are met
- Design review checklist passes for all screens

---

# Guiding Principle

The design system exists to eliminate decision fatigue.

When a developer needs a color, there is one place to look. When they need a spacing value, there is one system to follow. When they need a component, there is a library to use.

Consistency is not about perfection — it is about predictability. Every screen should feel like it belongs to the same application.
