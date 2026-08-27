# Design System

> Defining a consistent, accessible and platform-aware visual language for Summa.

---

## Table of Contents

- [Purpose](#purpose)
- [Design Goals](#design-goals)
- [Design Principles](#design-principles)
- [Platform Strategy](#platform-strategy)
- [Figma Structure](#figma-structure)
- [Design Tokens](#design-tokens)
- [Color System](#color-system)
- [Typography](#typography)
- [Spacing System](#spacing-system)
- [Grid and Layout](#grid-and-layout)
- [Shape and Radius](#shape-and-radius)
- [Elevation](#elevation)
- [Iconography](#iconography)
- [Core Components](#core-components)
- [Navigation](#navigation)
- [Financial Components](#financial-components)
- [Charts and Data Visualization](#charts-and-data-visualization)
- [Screen States](#screen-states)
- [Forms and Validation](#forms-and-validation)
- [Motion](#motion)
- [Accessibility](#accessibility)
- [Responsive Behavior](#responsive-behavior)
- [Dark Theme](#dark-theme)
- [Localization](#localization)
- [Implementation Mapping](#implementation-mapping)
- [Design Review Checklist](#design-review-checklist)
- [Definition of Done](#definition-of-done)
- [Guiding Principle](#guiding-principle)

---

## Purpose

This document defines the Summa design system.

The design system acts as the shared visual and interaction language for:

- Mobile (Flutter — Android and iOS)
- Future desktop applications
- Future web interfaces
- Documentation
- Branding assets

The system should ensure consistency without forcing identical platform behavior.

---

## Design Goals

The Summa interface should feel:

- Calm
- Precise
- Trustworthy
- Structured
- Accessible
- Modern without being trendy
- Suitable for frequent everyday use

The interface should not feel:

- Promotional
- Gamified
- Aggressive
- Crypto-oriented
- Visually noisy
- Overloaded with gradients and effects

---

## Design Principles

### Clarity Before Decoration

Financial information must remain easy to scan.

Visual decoration must never reduce readability.

---

### One Clear Priority

Every screen should have one clearly dominant task or piece of information.

---

### Consistency Builds Trust

Repeated patterns should behave and look consistently.

Examples:

- Transaction rows
- Amount formatting
- Buttons
- Inputs
- Empty states
- Error messages

---

### Progressive Disclosure

Advanced settings and detailed information should appear only when needed.

---

### Native Behavior

Android and iOS should share the same design language while respecting native interaction conventions.

---

### Accessible by Default

Accessibility is a design requirement, not a later enhancement.

---

## Platform Strategy

Summa uses:

- Shared Figma design tokens
- Shared component specifications
- Flutter implementation using Material 3 widgets

The applications should remain visually aligned while allowing platform-specific differences.

Examples:

- Native back gestures
- Native navigation behavior
- Native date pickers
- Native biometric prompts
- Platform-specific system bars

---

## Figma Structure

Recommended Figma pages:

```text
00 Cover
01 Foundations
02 Tokens
03 Components
04 Patterns
05 Mobile
06 Prototypes
08 Archive
```

---

## Figma Component Organization

```text
Components/
├── Actions/
├── Inputs/
├── Navigation/
├── Cards/
├── Lists/
├── Feedback/
├── Overlays/
├── Charts/
└── Finance/
```

Every reusable component should use:

- Auto Layout
- Constraints
- Variants
- Component properties
- Named layers
- Design tokens

---

## Design Tokens

Design tokens are the source of truth for reusable values.

Token categories:

```text
color
typography
spacing
radius
elevation
size
motion
opacity
```

Example names:

```text
color.primary.default
color.text.primary
color.background.surface
spacing.md
radius.card
typography.body.medium
```

Hardcoded visual values should be avoided in implementation.

---

## Color System

### Primary Color

The selected Summa primary color is:

```text
#2C3E50
```

It represents:

- Stability
- Precision
- Control
- Professionalism

---

## Primary Palette

```text
Primary 50   #F1F4F6
Primary 100  #E1E7EB
Primary 200  #C2CDD5
Primary 300  #98A9B5
Primary 400  #6E8595
Primary 500  #526A7B
Primary 600  #3E5567
Primary 700  #2C3E50
Primary 800  #233240
Primary 900  #19242F
Primary 950  #10181F
```

Primary 700 is the default brand color.

---

## Neutral Palette

```text
Neutral 0    #FFFFFF
Neutral 50   #F8F9FA
Neutral 100  #F1F3F5
Neutral 200  #E5E7EB
Neutral 300  #D1D5DB
Neutral 400  #9CA3AF
Neutral 500  #6B7280
Neutral 600  #4B5563
Neutral 700  #374151
Neutral 800  #1F2937
Neutral 900  #111827
Neutral 950  #080C12
```

---

## Semantic Colors

### Success

```text
Success 50   #ECFDF3
Success 500  #22A06B
Success 700  #166B48
```

### Warning

```text
Warning 50   #FFF8E6
Warning 500  #D99A00
Warning 700  #8A6200
```

### Error

```text
Error 50     #FEF2F2
Error 500    #DC4C4C
Error 700    #991B1B
```

### Information

```text
Info 50      #EFF6FF
Info 500     #3B82F6
Info 700     #1D4ED8
```

Semantic colors must not be the only indicator of meaning.

Icons, labels or patterns should also be used.

---

## Financial Color Usage

Positive and negative values must be used carefully.

Income:

```text
Success 700
```

Expense:

```text
Neutral 900
```

or a controlled error tone only where emphasis is required.

A normal expense should not automatically appear as an alarming error.

---

## Background Tokens

Light theme:

```text
background.canvas       Neutral 50
background.surface      Neutral 0
background.subtle       Neutral 100
background.selected     Primary 50
```

Text:

```text
text.primary            Neutral 900
text.secondary          Neutral 600
text.tertiary           Neutral 500
text.disabled           Neutral 400
text.inverse            Neutral 0
```

Borders:

```text
border.default          Neutral 200
border.strong           Neutral 300
border.focus            Primary 700
border.error            Error 500
```

---

## Typography

Preferred shared typeface:

```text
Inter
```

Native system fonts may be used where platform-native behavior provides a meaningful advantage.

Android alternative:

```text
Roboto
```

iOS alternative:

```text
SF Pro
```

The project should avoid mixing multiple unrelated typefaces.

---

## Type Scale

| Token | Size | Line Height | Weight |
|---|---:|---:|---:|
| Display Large | 36 | 44 | 700 |
| Display Medium | 32 | 40 | 700 |
| Heading Large | 28 | 36 | 700 |
| Heading Medium | 24 | 32 | 600 |
| Heading Small | 20 | 28 | 600 |
| Body Large | 18 | 28 | 400 |
| Body Medium | 16 | 24 | 400 |
| Body Small | 14 | 20 | 400 |
| Label Large | 14 | 20 | 600 |
| Label Medium | 12 | 16 | 600 |
| Caption | 12 | 16 | 400 |

---

## Financial Amount Typography

Primary balance:

```text
32–36 px
Bold
Tabular numbers
```

Transaction amount:

```text
16 px
SemiBold
Tabular numbers
```

Secondary summary amount:

```text
14 px
SemiBold
Tabular numbers
```

Tabular figures should be enabled where available to keep columns aligned.

---

## Spacing System

Base unit:

```text
4 px
```

Primary layout rhythm:

```text
8 px
```

Token scale:

```text
spacing.0    0
spacing.1    4
spacing.2    8
spacing.3    12
spacing.4    16
spacing.5    20
spacing.6    24
spacing.8    32
spacing.10   40
spacing.12   48
spacing.16   64
```

Most layouts should use multiples of 8.

Four-pixel values are reserved for small internal adjustments.

---

## Grid and Layout

Primary mobile reference frame:

```text
390 × 844
```

Secondary validation frames:

```text
360 × 800
428 × 926
```

---

## Mobile Column Grid

```text
Columns: 4
Type: Stretch
Margin: 16
Gutter: 16
```

The column grid controls horizontal alignment.

Vertical spacing should use Auto Layout and the spacing token system.

A fixed row grid is not required.

---

## Safe Areas

All screens must respect:

- Status bar
- Camera cutout or notch
- Navigation bar
- Home indicator
- Keyboard
- System gestures

Content must not depend on a fixed device height.

---

## Content Width

Standard screen margin:

```text
16 px
```

Large-screen margin may increase to:

```text
24 px
```

Full-bleed content should be limited to:

- Camera preview
- Images
- Charts where justified
- Bottom navigation background

---

## Shape and Radius

Radius tokens:

```text
radius.none      0
radius.small     4
radius.medium    8
radius.large     12
radius.xlarge    16
radius.full      999
```

Recommended usage:

```text
Inputs            8
Cards             12
Dialogs           16
Buttons            8 or 12
Chips             Full
```

Avoid applying large rounded corners to every element.

---

## Elevation

Elevation should be subtle.

Preferred hierarchy:

```text
elevation.none
elevation.low
elevation.medium
elevation.high
```

Usage:

- Cards usually use border or background contrast
- Bottom navigation may use low elevation
- Dialogs use high elevation
- Floating action buttons use medium elevation

Shadows must not be the only method of separation.

---

## Iconography

Icon style:

- Outline
- Consistent stroke
- Rounded where appropriate
- 24 px standard size
- 20 px compact size
- 16 px metadata size

Recommended sources:

- Material Symbols for Android
- SF Symbols for iOS
- Custom Summa icons only when necessary

Do not mix filled and outline styles randomly.

---

## Bottom Navigation Icons

Recommended destinations:

```text
Main          Home or Dashboard
Statistics    Chart
Scanner       Scan or Camera
Management    Folder or Sliders
Profile       Person
```

Active state:

```text
Primary 700
```

Inactive state:

```text
Neutral 500
```

---

## Core Components

The initial component library must include:

- Primary button
- Secondary button
- Tertiary button
- Icon button
- Text input
- Search input
- Dropdown or selector
- Checkbox
- Radio option
- Switch
- Chip
- Card
- List row
- Divider
- Badge
- Top app bar
- Bottom navigation
- Dialog
- Bottom sheet
- Snackbar
- Tooltip
- Loading indicator

---

## Button Specifications

Minimum touch target:

```text
48 × 48 px
```

Common button height:

```text
48 px
```

Compact button height:

```text
40 px
```

Button states:

- Default
- Pressed
- Focused
- Disabled
- Loading
- Error where applicable

Primary buttons should use the primary color sparingly.

---

## Input Specifications

Inputs should support:

- Label
- Placeholder
- Value
- Supporting text
- Error text
- Leading icon
- Trailing action
- Disabled state
- Read-only state
- Focus state

Financial amount inputs should support:

- Numeric keyboard
- Currency indicator
- Locale-aware formatting
- Clear decimal behavior
- Validation without losing entered data

---

## Navigation

Root navigation contains:

```text
Main
Statistics
Scanner
Management
Profile
```

The scanner may use a visually emphasized central action if testing confirms that users understand it.

Navigation labels should remain visible unless a validated platform pattern supports icon-only navigation.

---

## Top App Bar

Root screen app bar may contain:

- Screen title
- Active profile or workspace
- Optional secondary action
- Optional avatar

Nested screens may contain:

- Back action
- Title
- One or two contextual actions

Avoid placing more than three actions in the top app bar.

---

## Financial Components

### Balance Card

Recommended size on a 390 px-wide screen:

```text
Width: available content width
Height: approximately 160–176 px
Padding: 20–24 px
Radius: 12 px
```

Content may include:

- Current balance
- Selected period
- Income summary
- Expense summary
- Visibility toggle

---

### Transaction Row

Recommended height:

```text
64–72 px
```

Structure:

```text
Category icon
Merchant or title
Category and date
Amount
Optional status
```

The amount should remain right-aligned.

---

### Category Row

Content:

- Category icon
- Category name
- Percentage or progress
- Amount
- Optional budget status

---

### Summary Card

Used for:

- Income
- Expenses
- Savings
- Budget remaining

Summary cards should not compete with the primary balance card.

---

## Charts and Data Visualization

Charts should answer a specific question.

Examples:

- Where was money spent?
- How did spending change?
- What is the monthly trend?
- Which category exceeds its budget?

Avoid charts used only for decoration.

---

## Chart Guidelines

- Use readable labels
- Provide accessible summaries
- Avoid excessive categories
- Avoid 3D charts
- Avoid misleading scales
- Support dark mode
- Use patterns or labels in addition to color
- Allow values to be inspected
- Use localized number formatting

---

## Recommended Chart Types

```text
Category distribution    Donut or horizontal bars
Monthly trend             Line chart
Income vs expense         Grouped bars
Budget progress           Progress bar
Daily spending            Vertical bars
```

Horizontal bar charts are preferred when category names are long.

---

## Screen States

Every data-driven screen must define:

- Loading
- Content
- Empty
- Error
- Offline
- Partial data
- Permission denied where applicable

---

## Loading State

Use:

- Skeleton placeholders for structured content
- Progress indicator for blocking actions
- Inline loading for background refresh

Avoid replacing an entire usable screen with a spinner during background refresh.

---

## Empty State

An empty state should include:

- Clear title
- Short explanation
- Relevant primary action
- Optional illustration

Example:

```text
No transactions yet

Add your first transaction to start tracking your finances.

[Add transaction]
```

---

## Error State

Errors should:

- Explain what happened
- Avoid technical jargon
- Offer recovery
- Preserve entered data
- Show request ID only when useful for support

---

## Forms and Validation

Validation should occur:

- During input for obvious local errors
- On submission for complete validation
- On the server for remote operations

Error messages should be specific.

Good:

```text
Enter an amount greater than zero.
```

Avoid:

```text
Invalid input.
```

---

## Motion

Motion should communicate relationships and state changes.

Recommended durations:

```text
Fast       100–150 ms
Standard   200–250 ms
Slow       300–400 ms
```

Preferred transitions:

- Fade
- Small slide
- Size change
- Crossfade
- Progress animation

Avoid excessive bounce, rotation or decorative motion.

Respect reduced-motion settings.

---

## Accessibility

Requirements:

- Minimum touch target of 44–48 px
- WCAG AA contrast target
- Screen reader labels
- Logical focus order
- Dynamic text support
- No color-only meaning
- Accessible chart descriptions
- Keyboard support on future desktop or web platforms
- Reduced-motion support
- Clear error announcements

---

## Text Scaling

Layouts must support increased system font size.

The design should avoid:

- Fixed-height text containers
- Truncated critical amounts
- Overlapping labels
- Icon-only critical actions
- Unscrollable forms

---

## Responsive Behavior

Mobile layouts should adapt rather than scale proportionally.

On larger mobile screens:

- Content width may expand
- More whitespace may appear
- Cards may become wider
- Text should not become unnecessarily larger

On narrow screens:

- Secondary information may wrap
- Actions may stack
- Labels may shorten only where meaning remains clear

---

## Dark Theme

Dark theme is planned from the beginning.

Suggested dark tokens:

```text
background.canvas       Neutral 950
background.surface      Neutral 900
background.subtle       Neutral 800

text.primary            Neutral 50
text.secondary          Neutral 300
text.tertiary           Neutral 400

border.default          Neutral 700
border.strong           Neutral 600
```

Primary colors may require adjusted lighter tones for adequate contrast.

Dark theme must not be created by mechanically inverting colors.

---

## Localization

Designs must support:

- Serbian Latin
- Serbian Cyrillic in the future
- English
- Longer translated labels
- Different currency formats
- Right-to-left layouts if future languages require them

Text must not be embedded in images.

---

## Currency Formatting

Currency presentation should be locale-aware.

Examples:

```text
1.250,50 RSD
€1,250.50
1 250,50 €
```

The design must not assume a fixed currency symbol position.

---

## Implementation Mapping

### Flutter

Figma tokens map to:

```text
ThemeData
ColorScheme
TextTheme
ShapeTheme
Reusable Flutter widgets
Riverpod providers for theme access
```

Token names should remain consistent across the codebase.

---

## Design Review Checklist

Before approving a screen, verify:

- Uses design tokens
- Uses Auto Layout
- Supports narrow screen
- Supports large text
- Defines loading state
- Defines empty state
- Defines error state
- Defines dark mode
- Meets touch target requirements
- Uses correct amount formatting
- Uses consistent navigation
- Uses no hardcoded unexplained values
- Includes platform-specific notes where needed

---

## Definition of Done

The design system is ready for Phase 1 when:

- Primary and neutral palettes are finalized
- Light theme is complete
- Dark theme tokens are defined
- Typography scale is finalized
- Spacing scale is finalized
- Grid is documented
- Core components exist in Figma
- Bottom navigation is finalized
- Main dashboard pattern is approved
- Transaction row is approved
- Form states are defined
- Accessibility requirements are documented
- Android and iOS token mapping is defined

---

## Guiding Principle

Summa should look calm, understandable and dependable.

The design system exists to reduce uncertainty for users and inconsistency for developers.