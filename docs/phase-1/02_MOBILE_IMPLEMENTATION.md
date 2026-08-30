# Mobile Implementation

> Defining the concrete Flutter implementation patterns, conventions and structure for Phase 1 of the Summa mobile application.

---

# Table of Contents

- Purpose
- Project Structure
- Application Entry Point
- Theme Configuration
- Navigation Setup
- State Management
- Dependency Injection
- Feature Structure
- Screen Pattern
- ViewModel Pattern
- Use Case Pattern
- Repository Pattern
- Error Handling
- Logging
- Performance Guidelines
- Accessibility
- Animations
- Future Expansion
- Guiding Principle

---

# Purpose

This document translates the Phase 0 architecture decisions into concrete implementation patterns for the Summa Flutter application.

Where Phase 0 defines what the architecture should achieve, this document defines how every developer implements it in code. Every pattern described here is prescriptive. Deviations require an updated document and an explicit decision record.

The document covers:

- Project directory structure
- Application bootstrap and entry point
- Theme and design token implementation
- Navigation with GoRouter
- State management with Riverpod
- Dependency injection with Riverpod
- Feature-level organization
- Screen, ViewModel, Use Case and Repository patterns
- Error handling and logging
- Performance, accessibility and animation guidelines
- Preparation for future synchronization

---

# Project Structure

The Flutter project follows a feature-first structure with clear layer separation.

```text
summa-mobile/
├── lib/
│   ├── app/
│   │   ├── app.dart                          # Root application widget
│   │   ├── app_startup.dart                  # Startup initialization
│   │   └── app_providers.dart                # Global provider overrides
│   │
│   ├── core/
│   │   ├── constants/
│   │   │   ├── app_constants.dart
│   │   │   ├── database_constants.dart
│   │   │   └── storage_constants.dart
│   │   ├── error/
│   │   │   ├── app_exception.dart            # Sealed exception hierarchy
│   │   │   ├── error_handler.dart            # Centralized error processing
│   │   │   └── error_messages.dart           # User-facing error messages
│   │   ├── extensions/
│   │   │   ├── datetime_extensions.dart
│   │   │   ├── money_extensions.dart
│   │   │   ├── string_extensions.dart
│   │   │   └── context_extensions.dart
│   │   ├── logging/
│   │   │   ├── app_logger.dart               # Centralized logger wrapper
│   │   │   ├── log_levels.dart
│   │   │   └── log_sanitizer.dart            # Strips sensitive data
│   │   ├── theme/
│   │   │   ├── app_theme.dart                # ThemeData builder
│   │   │   ├── color_tokens.dart             # Design token colors
│   │   │   ├── text_style_tokens.dart        # Design token typography
│   │   │   ├── spacing_tokens.dart           # Design token spacing
│   │   │   ├── radius_tokens.dart            # Design token radius
│   │   │   └── elevation_tokens.dart         # Design token elevation
│   │   ├── utils/
│   │   │   ├── currency_formatter.dart
│   │   │   ├── date_formatter.dart
│   │   │   ├── uuid_generator.dart
│   │   │   └── validators.dart
│   │   └── types/
│   │       ├── money.dart                    # Money value object
│   │       ├── currency_code.dart            # ISO 4217 wrapper
│   │       ├── year_month.dart               # Accounting period type
│   │       └── entity_id.dart                # UUID wrapper type
│   │
│   ├── data/
│   │   ├── database/
│   │   │   ├── app_database.dart             # drift database class
│   │   │   ├── app_database.g.dart           # Generated code
│   │   │   ├── tables/
│   │   │   │   ├── workspace_table.dart
│   │   │   │   ├── profile_table.dart
│   │   │   │   ├── category_table.dart
│   │   │   │   ├── transaction_table.dart
│   │   │   │   ├── transaction_split_table.dart
│   │   │   │   ├── budget_table.dart
│   │   │   │   └── attachment_table.dart
│   │   │   ├── daos/
│   │   │   │   ├── workspace_dao.dart
│   │   │   │   ├── profile_dao.dart
│   │   │   │   ├── category_dao.dart
│   │   │   │   ├── transaction_dao.dart
│   │   │   │   └── budget_dao.dart
│   │   │   └── migrations/
│   │   │       └── migration_strategy.dart
│   │   ├── repositories/
│   │   │   ├── workspace_repository_impl.dart
│   │   │   ├── profile_repository_impl.dart
│   │   │   ├── category_repository_impl.dart
│   │   │   ├── transaction_repository_impl.dart
│   │   │   └── settings_repository_impl.dart
│   │   ├── mappers/
│   │   │   ├── profile_mapper.dart
│   │   │   ├── category_mapper.dart
│   │   │   └── transaction_mapper.dart
│   │   └── storage/
│   │       ├── secure_storage_service.dart
│   │       └── preferences_service.dart
│   │
│   ├── domain/
│   │   ├── models/
│   │   │   ├── workspace.dart
│   │   │   ├── profile.dart
│   │   │   ├── category.dart
│   │   │   ├── transaction.dart
│   │   │   ├── transaction_split.dart
│   │   │   ├── budget.dart
│   │   │   └── attachment.dart
│   │   ├── repositories/
│   │   │   ├── workspace_repository.dart     # Interface
│   │   │   ├── profile_repository.dart       # Interface
│   │   │   ├── category_repository.dart      # Interface
│   │   │   ├── transaction_repository.dart   # Interface
│   │   │   └── settings_repository.dart      # Interface
│   │   └── usecases/
│   │       ├── workspace/
│   │       │   └── initialize_workspace_use_case.dart
│   │       ├── profile/
│   │       │   ├── create_profile_use_case.dart
│   │       │   ├── update_profile_use_case.dart
│   │       │   ├── delete_profile_use_case.dart
│   │       │   └── get_profiles_use_case.dart
│   │       ├── category/
│   │       │   ├── create_category_use_case.dart
│   │       │   ├── update_category_use_case.dart
│   │       │   ├── archive_category_use_case.dart
│   │       │   ├── seed_default_categories_use_case.dart
│   │       │   └── get_categories_use_case.dart
│   │       ├── transaction/
│   │       │   ├── create_transaction_use_case.dart
│   │       │   ├── update_transaction_use_case.dart
│   │       │   ├── delete_transaction_use_case.dart
│   │       │   ├── get_transactions_use_case.dart
│   │       │   └── get_transaction_detail_use_case.dart
│   │       ├── statistics/
│   │       │   ├── calculate_monthly_balance_use_case.dart
│   │       │   ├── get_category_breakdown_use_case.dart
│   │       │   └── get_monthly_trend_use_case.dart
│   │       └── export/
│   │           ├── export_json_use_case.dart
│   │           └── export_csv_use_case.dart
│   │
│   ├── features/
│   │   ├── dashboard/
│   │   │   ├── ui/
│   │   │   │   ├── screens/
│   │   │   │   │   └── dashboard_screen.dart
│   │   │   │   ├── content/
│   │   │   │   │   └── dashboard_content.dart
│   │   │   │   ├── sections/
│   │   │   │   │   ├── balance_section.dart
│   │   │   │   │   ├── recent_transactions_section.dart
│   │   │   │   │   └── monthly_summary_section.dart
│   │   │   │   └── components/
│   │   │   │       ├── balance_card.dart
│   │   │   │       ├── transaction_preview_row.dart
│   │   │   │       └── summary_card.dart
│   │   │   ├── viewmodel/
│   │   │   │   └── dashboard_view_model.dart
│   │   │   ├── state/
│   │   │   │   └── dashboard_state.dart
│   │   │   ├── events/
│   │   │   │   └── dashboard_event.dart
│   │   │   └── navigation/
│   │   │       └── dashboard_navigation.dart
│   │   │
│   │   ├── transactions/
│   │   │   ├── ui/
│   │   │   │   ├── screens/
│   │   │   │   │   ├── transaction_list_screen.dart
│   │   │   │   │   ├── transaction_detail_screen.dart
│   │   │   │   │   └── transaction_form_screen.dart
│   │   │   │   ├── content/
│   │   │   │   │   ├── transaction_list_content.dart
│   │   │   │   │   ├── transaction_detail_content.dart
│   │   │   │   │   └── transaction_form_content.dart
│   │   │   │   ├── sections/
│   │   │   │   │   ├── transaction_filter_section.dart
│   │   │   │   │   ├── transaction_group_section.dart
│   │   │   │   │   └── amount_input_section.dart
│   │   │   │   └── components/
│   │   │   │       ├── transaction_row.dart
│   │   │   │       ├── transaction_type_selector.dart
│   │   │   │       ├── category_picker.dart
│   │   │   │       ├── amount_input.dart
│   │   │   │       └── date_picker_field.dart
│   │   │   ├── viewmodel/
│   │   │   │   ├── transaction_list_view_model.dart
│   │   │   │   ├── transaction_detail_view_model.dart
│   │   │   │   └── transaction_form_view_model.dart
│   │   │   ├── state/
│   │   │   │   ├── transaction_list_state.dart
│   │   │   │   ├── transaction_detail_state.dart
│   │   │   │   └── transaction_form_state.dart
│   │   │   ├── events/
│   │   │   │   ├── transaction_list_event.dart
│   │   │   │   ├── transaction_detail_event.dart
│   │   │   │   └── transaction_form_event.dart
│   │   │   └── navigation/
│   │   │       └── transaction_navigation.dart
│   │   │
│   │   ├── categories/
│   │   │   ├── ui/
│   │   │   │   ├── screens/
│   │   │   │   │   ├── category_list_screen.dart
│   │   │   │   │   └── category_form_screen.dart
│   │   │   │   ├── content/
│   │   │   │   │   ├── category_list_content.dart
│   │   │   │   │   └── category_form_content.dart
│   │   │   │   ├── sections/
│   │   │   │   │   └── category_group_section.dart
│   │   │   │   └── components/
│   │   │   │       ├── category_row.dart
│   │   │   │       ├── category_icon_picker.dart
│   │   │   │       └── category_color_picker.dart
│   │   │   ├── viewmodel/
│   │   │   │   ├── category_list_view_model.dart
│   │   │   │   └── category_form_view_model.dart
│   │   │   ├── state/
│   │   │   │   ├── category_list_state.dart
│   │   │   │   └── category_form_state.dart
│   │   │   ├── events/
│   │   │   │   ├── category_list_event.dart
│   │   │   │   └── category_form_event.dart
│   │   │   └── navigation/
│   │   │       └── category_navigation.dart
│   │   │
│   │   ├── statistics/
│   │   │   ├── ui/
│   │   │   │   ├── screens/
│   │   │   │   │   └── statistics_screen.dart
│   │   │   │   ├── content/
│   │   │   │   │   └── statistics_content.dart
│   │   │   │   ├── sections/
│   │   │   │   │   ├── monthly_overview_section.dart
│   │   │   │   │   ├── category_breakdown_section.dart
│   │   │   │   │   └── trend_section.dart
│   │   │   │   └── components/
│   │   │   │       ├── donut_chart.dart
│   │   │   │       ├── bar_chart.dart
│   │   │   │       └── trend_line.dart
│   │   │   ├── viewmodel/
│   │   │   │   └── statistics_view_model.dart
│   │   │   ├── state/
│   │   │   │   └── statistics_state.dart
│   │   │   ├── events/
│   │   │   │   └── statistics_event.dart
│   │   │   └── navigation/
│   │   │       └── statistics_navigation.dart
│   │   │
│   │   ├── profiles/
│   │   │   ├── ui/
│   │   │   │   ├── screens/
│   │   │   │   │   ├── profile_list_screen.dart
│   │   │   │   │   └── profile_form_screen.dart
│   │   │   │   ├── content/
│   │   │   │   │   ├── profile_list_content.dart
│   │   │   │   │   └── profile_form_content.dart
│   │   │   │   └── components/
│   │   │   │       ├── profile_card.dart
│   │   │   │       └── profile_selector.dart
│   │   │   ├── viewmodel/
│   │   │   │   ├── profile_list_view_model.dart
│   │   │   │   └── profile_form_view_model.dart
│   │   │   ├── state/
│   │   │   │   ├── profile_list_state.dart
│   │   │   │   └── profile_form_state.dart
│   │   │   ├── events/
│   │   │   │   ├── profile_list_event.dart
│   │   │   │   └── profile_form_event.dart
│   │   │   └── navigation/
│   │   │       └── profile_navigation.dart
│   │   │
│   │   └── settings/
│   │       ├── ui/
│   │       │   ├── screens/
│   │       │   │   └── settings_screen.dart
│   │       │   ├── content/
│   │       │   │   └── settings_content.dart
│   │       │   ├── sections/
│   │       │   │   ├── appearance_section.dart
│   │       │   │   ├── security_section.dart
│   │       │   │   ├── data_section.dart
│   │       │   │   └── about_section.dart
│   │       │   └── components/
│   │       │       ├── settings_tile.dart
│   │       │       └── settings_group.dart
│   │       ├── viewmodel/
│   │       │   └── settings_view_model.dart
│   │       ├── state/
│   │       │   └── settings_state.dart
│   │       ├── events/
│   │       │   └── settings_event.dart
│   │       └── navigation/
│   │           └── settings_navigation.dart
│   │
│   ├── navigation/
│   │   ├── app_router.dart                   # GoRouter configuration
│   │   ├── route_names.dart                  # Named route constants
│   │   ├── route_paths.dart                  # Path constants
│   │   ├── shell_route.dart                  # Bottom navigation shell
│   │   └── route_guards.dart                 # Auth and lock guards
│   │
│   ├── ui/
│   │   ├── components/
│   │   │   ├── buttons/
│   │   │   │   ├── primary_button.dart
│   │   │   │   ├── secondary_button.dart
│   │   │   │   ├── icon_action_button.dart
│   │   │   │   └── destructive_button.dart
│   │   │   ├── cards/
│   │   │   │   ├── app_card.dart
│   │   │   │   └── tappable_card.dart
│   │   │   ├── inputs/
│   │   │   │   ├── text_input_field.dart
│   │   │   │   ├── search_input_field.dart
│   │   │   │   └── dropdown_field.dart
│   │   │   ├── feedback/
│   │   │   │   ├── empty_state.dart
│   │   │   │   ├── error_state.dart
│   │   │   │   ├── loading_state.dart
│   │   │   │   └── skeleton_loader.dart
│   │   │   ├── navigation/
│   │   │   │   ├── app_bottom_navigation.dart
│   │   │   │   └── app_top_bar.dart
│   │   │   └── overlays/
│   │   │       ├── confirm_dialog.dart
│   │   │       ├── app_bottom_sheet.dart
│   │   │       └── app_snackbar.dart
│   │   └── shell/
│   │       └── app_shell.dart                # Scaffold with bottom nav
│   │
│   └── main.dart                             # Application entry point
│
├── test/
│   ├── core/
│   ├── data/
│   ├── domain/
│   ├── features/
│   └── ui/
│
├── integration_test/
│   ├── dashboard_flow_test.dart
│   ├── transaction_flow_test.dart
│   └── profile_flow_test.dart
│
├── assets/
│   ├── fonts/
│   ├── images/
│   └── animations/
│
├── pubspec.yaml
├── analysis_options.yaml
└── README.md
```

Every directory has a single clear responsibility.

No file should be placed in a location that requires explanation.

---

# Application Entry Point

## main.dart

The entry point initializes all infrastructure before the application starts.

```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  final container = ProviderContainer();

  await container.read(appStartupProvider.future);

  runApp(
    UncontrolledProviderScope(
      container: container,
      child: const SummaApp(),
    ),
  );
}
```

Key rules:

- `WidgetsFlutterBinding.ensureInitialized()` is called before any async work
- The `ProviderContainer` is created once and passed to the widget tree
- `appStartupProvider` handles database initialization, workspace check and migration
- `UncontrolledProviderScope` prevents the framework from disposing the container
- `runApp` is called only after startup completes

---

## App Startup Provider

The startup provider is a `FutureProvider` that performs one-time initialization.

```dart
@appStartupProvider
Future<AppStartupState> appStartup(Ref ref) async {
  final database = await ref.watch(databaseProvider.future);
  final workspaceRepository = ref.watch(workspaceRepositoryProvider);

  final workspace = await workspaceRepository.getCurrentWorkspace();

  if (workspace == null) {
    await workspaceRepository.initializeDefaultWorkspace();
  }

  return const AppStartupState.ready();
}
```

Startup responsibilities:

- Open and verify the drift database
- Run pending migrations
- Ensure a default workspace exists
- Load user preferences
- Prepare secure storage

If startup fails, the application shows a recovery screen rather than crashing.

---

## App Widget

The root widget configures theme, routing and global providers.

```dart
class SummaApp extends ConsumerWidget {
  const SummaApp({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final router = ref.watch(appRouterProvider);
    final themeMode = ref.watch(themeModeProvider);

    return MaterialApp.router(
      title: 'Summa',
      theme: AppTheme.light(),
      darkTheme: AppTheme.dark(),
      themeMode: themeMode,
      routerConfig: router,
      debugShowCheckedModeBanner: false,
      builder: (context, child) {
        return MediaQuery(
          data: MediaQuery.of(context).copyWith(
            textScaler: TextScaler.noScaling,
          ),
          child: child ?? const SizedBox.shrink(),
        );
      },
    );
  }
}
```

The `App` widget:

- Uses `MaterialApp.router` for GoRouter integration
- Applies light and dark themes from the design system
- Reads the current theme mode from a Riverpod provider
- Does not contain business logic
- Does not directly access the database

---

# Theme Configuration

## Material 3

Summa uses Material 3 as its design foundation.

The `ThemeData` is built from design tokens defined in the `core/theme/` directory. Material 3 provides the base component styling. Summa overrides specific tokens to match the design system.

```dart
class AppTheme {
  AppTheme._();

  static ThemeData light() {
    final colorScheme = ColorScheme.light(
      primary: ColorTokens.primary700,
      onPrimary: ColorTokens.neutral0,
      primaryContainer: ColorTokens.primary50,
      secondary: ColorTokens.primary400,
      surface: ColorTokens.neutral0,
      onSurface: ColorTokens.neutral900,
      error: ColorTokens.error500,
      outline: ColorTokens.neutral200,
    );

    return _buildTheme(colorScheme, Brightness.light);
  }

  static ThemeData dark() {
    final colorScheme = ColorScheme.dark(
      primary: ColorTokens.primary300,
      onPrimary: ColorTokens.neutral950,
      primaryContainer: ColorTokens.primary900,
      secondary: ColorTokens.primary400,
      surface: ColorTokens.neutral900,
      onSurface: ColorTokens.neutral50,
      error: ColorTokens.error500,
      outline: ColorTokens.neutral700,
    );

    return _buildTheme(colorScheme, Brightness.dark);
  }

  static ThemeData _buildTheme(ColorScheme colorScheme, Brightness brightness) {
    return ThemeData(
      useMaterial3: true,
      colorScheme: colorScheme,
      textTheme: AppTextTheme.build(),
      appBarTheme: _appBarTheme(colorScheme),
      bottomNavigationBarTheme: _bottomNavTheme(colorScheme),
      cardTheme: _cardTheme(colorScheme),
      inputDecorationTheme: _inputTheme(colorScheme),
      elevatedButtonTheme: _elevatedButtonTheme(colorScheme),
      outlinedButtonTheme: _outlinedButtonTheme(colorScheme),
      dialogTheme: _dialogTheme(),
      bottomSheetTheme: _bottomSheetTheme(),
      snackBarTheme: _snackBarTheme(colorScheme),
      dividerTheme: _dividerTheme(colorScheme),
      scaffoldBackgroundColor: colorScheme.surface,
    );
  }
}
```

---

## Design Tokens

Design tokens are Dart constants that mirror the Figma design system.

### Color Tokens

```dart
abstract class ColorTokens {
  // Primary
  static const primary50 = Color(0xFFF1F4F6);
  static const primary100 = Color(0xFFE1E7EB);
  static const primary200 = Color(0xFFC2CDD5);
  static const primary300 = Color(0xFF98A9B5);
  static const primary400 = Color(0xFF6E8595);
  static const primary500 = Color(0xFF526A7B);
  static const primary600 = Color(0xFF3E5567);
  static const primary700 = Color(0xFF2C3E50);
  static const primary800 = Color(0xFF233240);
  static const primary900 = Color(0xFF19242F);
  static const primary950 = Color(0xFF10181F);

  // Neutral
  static const neutral0 = Color(0xFFFFFFFF);
  static const neutral50 = Color(0xFFF8F9FA);
  static const neutral100 = Color(0xFFF1F3F5);
  static const neutral200 = Color(0xFFE5E7EB);
  static const neutral300 = Color(0xFFD1D5DB);
  static const neutral400 = Color(0xFF9CA3AF);
  static const neutral500 = Color(0xFF6B7280);
  static const neutral600 = Color(0xFF4B5563);
  static const neutral700 = Color(0xFF374151);
  static const neutral800 = Color(0xFF1F2937);
  static const neutral900 = Color(0xFF111827);
  static const neutral950 = Color(0xFF080C12);

  // Semantic
  static const success50 = Color(0xFFECFDF3);
  static const success500 = Color(0xFF22A06B);
  static const success700 = Color(0xFF166B48);

  static const warning50 = Color(0xFFFFF8E6);
  static const warning500 = Color(0xFFD99A00);
  static const warning700 = Color(0xFF8A6200);

  static const error50 = Color(0xFFFEF2F2);
  static const error500 = Color(0xFFDC4C4C);
  static const error700 = Color(0xFF991B1B);

  static const info50 = Color(0xFFEFF6FF);
  static const info500 = Color(0xFF3B82F6);
  static const info700 = Color(0xFF1D4ED8);
}
```

### Spacing Tokens

```dart
abstract class SpacingTokens {
  static const double xs = 4;
  static const double sm = 8;
  static const double md = 12;
  static const double lg = 16;
  static const double xl = 20;
  static const double xxl = 24;
  static const double xxxl = 32;
  static const double xxxxl = 40;
  static const double xxxxxl = 48;
  static const double xxxxxxl = 64;
}
```

### Radius Tokens

```dart
abstract class RadiusTokens {
  static const double none = 0;
  static const double small = 4;
  static const double medium = 8;
  static const double large = 12;
  static const double xlarge = 16;
  static const double full = 999;
}
```

---

## Light and Dark Theme

Both themes are defined from the same token system.

The light theme uses:

| Token | Value |
|---|---|
| background.canvas | Neutral 50 |
| background.surface | Neutral 0 |
| text.primary | Neutral 900 |
| text.secondary | Neutral 600 |
| border.default | Neutral 200 |

The dark theme uses:

| Token | Value |
|---|---|
| background.canvas | Neutral 950 |
| background.surface | Neutral 900 |
| text.primary | Neutral 50 |
| text.secondary | Neutral 300 |
| border.default | Neutral 700 |

Dark theme is never created by inverting light theme colors. Each theme is independently defined from the token palette.

Hardcoded color values are forbidden in feature code. All colors come from `ColorTokens` or from the `ThemeData` `ColorScheme`.

---

## Theme Access

Widgets access theme values through standard Flutter APIs.

```dart
// Colors
Theme.of(context).colorScheme.primary

// Text styles
Theme.of(context).textTheme.bodyMedium

// Custom tokens
SpacingTokens.lg
RadiusTokens.large
```

Riverpod providers expose theme mode for the settings feature.

```dart
@riverpod
class ThemeMode extends _$ThemeMode {
  @override
  ThemeMode build() {
    final preferences = ref.watch(preferencesServiceProvider);
    return preferences.getThemeMode();
  }

  Future<void> setThemeMode(ThemeMode mode) async {
    final preferences = ref.read(preferencesServiceProvider);
    await preferences.setThemeMode(mode);
    state = mode;
  }
}
```

---

# Navigation Setup

## GoRouter Configuration

Navigation uses GoRouter with a declarative routing approach.

```dart
@riverpod
GoRouter appRouter(Ref ref) {
  final appStartup = ref.watch(appStartupProvider);

  return GoRouter(
    initialLocation: RoutePaths.dashboard,
    debugLogDiagnostics: true,
    redirect: (context, state) {
      // Redirect to loading during startup
      if (appStartup.isLoading || appStartup.hasError) {
        return RoutePaths.splash;
      }

      // Application lock guard
      final isLocked = ref.read(appLockProvider);
      if (isLocked && state.matchedLocation != RoutePaths.lock) {
        return RoutePaths.lock;
      }

      return null;
    },
    routes: [
      GoRoute(
        path: RoutePaths.splash,
        builder: (context, state) => const SplashScreen(),
      ),
      GoRoute(
        path: RoutePaths.lock,
        builder: (context, state) => const LockScreen(),
      ),
      StatefulShellRoute.indexedStack(
        builder: (context, state, navigationShell) {
          return AppShell(navigationShell: navigationShell);
        },
        branches: [
          StatefulShellBranch(routes: [
            GoRoute(
              path: RoutePaths.dashboard,
              builder: (context, state) => const DashboardScreen(),
            ),
          ]),
          StatefulShellBranch(routes: [
            GoRoute(
              path: RoutePaths.statistics,
              builder: (context, state) => const StatisticsScreen(),
            ),
          ]),
          StatefulShellBranch(routes: [
            GoRoute(
              path: RoutePaths.scanner,
              builder: (context, state) => const ScannerPlaceholderScreen(),
            ),
          ]),
          StatefulShellBranch(routes: [
            GoRoute(
              path: RoutePaths.management,
              builder: (context, state) => const ManagementScreen(),
            ),
          ]),
          StatefulShellBranch(routes: [
            GoRoute(
              path: RoutePaths.profile,
              builder: (context, state) => const ProfileScreen(),
            ),
          ]),
        ],
      ),
      // Nested routes outside the shell
      GoRoute(
        path: RoutePaths.transactionDetail,
        builder: (context, state) {
          final id = state.pathParameters['id']!;
          return TransactionDetailScreen(transactionId: id);
        },
      ),
      GoRoute(
        path: RoutePaths.transactionCreate,
        builder: (context, state) => const TransactionFormScreen(),
      ),
      GoRoute(
        path: RoutePaths.transactionEdit,
        builder: (context, state) {
          final id = state.pathParameters['id']!;
          return TransactionFormScreen(transactionId: id);
        },
      ),
      GoRoute(
        path: RoutePaths.categoryList,
        builder: (context, state) => const CategoryListScreen(),
      ),
      GoRoute(
        path: RoutePaths.categoryCreate,
        builder: (context, state) => const CategoryFormScreen(),
      ),
      GoRoute(
        path: RoutePaths.settings,
        builder: (context, state) => const SettingsScreen(),
      ),
    ],
  );
}
```

---

## Route Definitions

All route paths are centralized as constants.

```dart
abstract class RoutePaths {
  static const splash = '/splash';
  static const lock = '/lock';
  static const dashboard = '/';
  static const statistics = '/statistics';
  static const scanner = '/scanner';
  static const management = '/management';
  static const profile = '/profile';
  static const transactionDetail = '/transactions/:id';
  static const transactionCreate = '/transactions/create';
  static const transactionEdit = '/transactions/:id/edit';
  static const categoryList = '/categories';
  static const categoryCreate = '/categories/create';
  static const settings = '/settings';
}
```

Named route constants are also defined for programmatic navigation.

```dart
abstract class RouteNames {
  static const splash = 'splash';
  static const lock = 'lock';
  static const dashboard = 'dashboard';
  static const statistics = 'statistics';
  static const scanner = 'scanner';
  static const management = 'management';
  static const profile = 'profile';
  static const transactionDetail = 'transaction-detail';
  static const transactionCreate = 'transaction-create';
  static const transactionEdit = 'transaction-edit';
  static const categoryList = 'category-list';
  static const categoryCreate = 'category-create';
  static const settings = 'settings';
}
```

---

## Navigation Shell

The bottom navigation uses `StatefulShellRoute.indexedStack` to preserve state across tab switches.

```dart
class AppShell extends StatelessWidget {
  final StatefulNavigationShell navigationShell;

  const AppShell({super.key, required this.navigationShell});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: navigationShell,
      bottomNavigationBar: AppBottomNavigation(
        currentIndex: navigationShell.currentIndex,
        onTap: (index) {
          navigationShell.goBranch(
            index,
            initialLocation: index == navigationShell.currentIndex,
          );
        },
      ),
    );
  }
}
```

Each tab branch maintains its own navigation stack. Switching tabs preserves scroll position and loaded data.

---

## Deep Links

GoRouter supports deep links out of the box.

Deep link examples:

| Deep Link | Destination |
|---|---|
| `summa://transactions/abc-123` | Transaction detail |
| `summa://transactions/create` | Transaction form |
| `summa://categories` | Category list |

Deep links are configured through platform-specific manifest files:

- Android: `AndroidManifest.xml` intent filters
- iOS: Associated Domains or custom URL schemes

Deep links respect route guards. A deep link to a protected screen redirects to the lock screen first.

---

## Navigation Rules

Navigation logic lives in each feature's `navigation/` directory.

```dart
class TransactionNavigation {
  TransactionNavigation(this._router);

  final GoRouter _router;

  void goToTransactionDetail(String transactionId) {
    _router.pushNamed(
      RouteNames.transactionDetail,
      pathParameters: {'id': transactionId},
    );
  }

  void goToTransactionCreate() {
    _router.pushNamed(RouteNames.transactionCreate);
  }

  void goToTransactionEdit(String transactionId) {
    _router.pushNamed(
      RouteNames.transactionEdit,
      pathParameters: {'id': transactionId},
    );
  }

  void goBack() {
    _router.pop();
  }
}
```

Screens never call `GoRouter.of(context)` directly. They receive navigation callbacks through their ViewModel or through a navigation class injected via Riverpod.

---

# State Management

## Riverpod

Riverpod is the sole state management solution.

Every screen has:

- A `State` class (immutable, defined with freezed)
- An `Event` class (sealed, defined with freezed)
- A `ViewModel` (extends `Notifier` or `AsyncNotifier`)

---

## Provider Types

| Provider Type | Use Case |
|---|---|
| `Provider` | Read-only values (repositories, services, formatters) |
| `FutureProvider` | One-shot async operations (startup, data loading) |
| `StreamProvider` | Observable data streams (database changes) |
| `NotifierProvider` | Synchronous mutable state (theme, settings) |
| `AsyncNotifierProvider` | Asynchronous mutable state (feature ViewModels) |
| `StateNotifierProvider` | Legacy, avoid for new code |

---

## Provider Organization

Providers are organized by scope.

### Global Providers

Defined in `app/` or `core/`. Available everywhere.

```dart
// Database
@riverpod
AppDatabase database(Ref ref) {
  return AppDatabase();
}

// Repositories
@riverpod
TransactionRepository transactionRepository(Ref ref) {
  return TransactionRepositoryImpl(
    transactionDao: ref.watch(transactionDaoProvider),
  );
}
```

### Feature Providers

Defined inside each feature directory. Scoped to that feature.

```dart
// features/dashboard/viewmodel/dashboard_view_model.dart
@riverpod
class DashboardViewModel extends _$DashboardViewModel {
  @override
  FutureOr<DashboardState> build() async {
    final balance = await ref.watch(calculateMonthlyBalanceUseCaseProvider.future);
    final recent = await ref.watch(getRecentTransactionsUseCaseProvider.future);

    return DashboardState.loaded(
      balance: balance,
      recentTransactions: recent,
    );
  }

  Future<void> onEvent(DashboardEvent event) async {
    switch (event) {
      case DashboardEventRefresh():
        ref.invalidateSelf();
      case DashboardEventAddTransaction():
        ref.read(transactionNavigationProvider).goToTransactionCreate();
    }
  }
}
```

### Provider Dependencies

Providers declare their dependencies explicitly through `ref.watch` and `ref.read`.

```text
ViewModel
    ↓ ref.watch
Use Case Provider
    ↓ ref.watch
Repository Provider
    ↓ ref.watch
DAO Provider
    ↓ ref.watch
Database Provider
```

No hidden dependencies. No service locator. Every dependency is visible in the provider definition.

---

## State Classes

Every state class uses freezed for immutability.

```dart
@freezed
sealed class DashboardState with _$DashboardState {
  const factory DashboardState.initial() = DashboardInitial;
  const factory DashboardState.loading() = DashboardLoading;
  const factory DashboardState.loaded({
    required Money balance,
    required List<Transaction> recentTransactions,
    required Money monthlyIncome,
    required Money monthlyExpenses,
  }) = DashboardLoaded;
  const factory DashboardState.empty() = DashboardEmpty;
  const factory DashboardState.error({
    required String message,
    AppException? exception,
  }) = DashboardError;
}
```

Every state class defines:

- `initial` — before first load
- `loading` — during data fetch
- `loaded` — successful data display
- `empty` — no data available
- `error` — failure with recovery option

---

## Event Classes

Every event class uses freezed sealed types.

```dart
@freezed
sealed class DashboardEvent with _$DashboardEvent {
  const factory DashboardEvent.refresh() = DashboardEventRefresh;
  const factory DashboardEvent.addTransaction() = DashboardEventAddTransaction;
  const factory DashboardEvent.transactionSelected(String id) = DashboardEventTransactionSelected;
  const factory DashboardEvent.retry() = DashboardEventRetry;
}
```

Events are plain data. They carry no behavior. The ViewModel decides what happens.

---

# Dependency Injection

## Riverpod-Based DI

Riverpod serves as the dependency injection container.

Every dependency is defined as a provider. Constructor injection is the preferred pattern.

```dart
@riverpod
CategoryRepository categoryRepository(Ref ref) {
  return CategoryRepositoryImpl(
    categoryDao: ref.watch(categoryDaoProvider),
  );
}

@riverpod
CreateCategoryUseCase createCategoryUseCase(Ref ref) {
  return CreateCategoryUseCase(
    categoryRepository: ref.watch(categoryRepositoryProvider),
  );
}
```

The dependency graph is:

```text
Database
    ↓
DAO
    ↓
Repository Implementation
    ↓
Use Case
    ↓
ViewModel
    ↓
Screen (ConsumerWidget)
```

No class instantiates its own dependencies. Everything is injected through constructors.

---

## Provider Overrides for Testing

Riverpod providers can be overridden in tests.

```dart
testWidgets('Dashboard shows balance', (tester) async {
  final mockRepository = MockTransactionRepository();
  when(mockRepository.getMonthlyBalance(any)).thenAnswer(
    (_) async => Money(amountMinor: 125050, currency: CurrencyCode('USD')),
  );

  await tester.pumpWidget(
    ProviderScope(
      overrides: [
        transactionRepositoryProvider.overrideWithValue(mockRepository),
      ],
      child: const SummaApp(),
    ),
  );

  await tester.pumpAndSettle();

  expect(find.text('\$1,250.50'), findsOneWidget);
});
```

Provider overrides allow:

- Replacing repositories with mocks
- Injecting test data
- Simulating error conditions
- Controlling async timing

No special testing framework is required beyond standard `flutter_test` and `mockito`.

---

## Provider Scoping

Providers can be scoped to specific widget subtrees.

```dart
ProviderScope(
  overrides: [
    activeProfileProvider.overrideWithValue(testProfile),
  ],
  child: const TransactionListScreen(),
)
```

Scoped providers are useful for:

- Profile-specific data
- Feature-specific configuration
- Test isolation

---

# Feature Structure

Every feature follows the same directory structure.

```text
features/<feature_name>/
├── ui/
│   ├── screens/
│   ├── content/
│   ├── sections/
│   └── components/
├── viewmodel/
├── state/
├── events/
└── navigation/
```

---

## Directory Responsibilities

### ui/screens/

Contains the top-level screen widget. One screen per file.

The screen is a `ConsumerWidget` that:

- Watches the ViewModel provider
- Reads the current state
- Passes state and event callbacks to the Content widget

```dart
class DashboardScreen extends ConsumerWidget {
  const DashboardScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final state = ref.watch(dashboardViewModelProvider);

    return state.when(
      initial: () => const LoadingState(),
      loading: () => const LoadingState(),
      loaded: (balance, recent, income, expenses) => DashboardContent(
        balance: balance,
        recentTransactions: recent,
        monthlyIncome: income,
        monthlyExpenses: expenses,
        onEvent: ref.read(dashboardViewModelProvider.notifier).onEvent,
      ),
      empty: () => const EmptyState(
        title: 'No transactions yet',
        message: 'Add your first transaction to get started.',
      ),
      error: (message, exception) => ErrorState(
        message: message,
        onRetry: () => ref.read(dashboardViewModelProvider.notifier)
            .onEvent(const DashboardEvent.retry()),
      ),
    );
  }
}
```

### ui/content/

Contains the main content layout for a screen.

Content widgets are stateless. They receive data through constructor parameters.

```dart
class DashboardContent extends StatelessWidget {
  const DashboardContent({
    super.key,
    required this.balance,
    required this.recentTransactions,
    required this.monthlyIncome,
    required this.monthlyExpenses,
    required this.onEvent,
  });

  final Money balance;
  final List<Transaction> recentTransactions;
  final Money monthlyIncome;
  final Money monthlyExpenses;
  final ValueChanged<DashboardEvent> onEvent;

  @override
  Widget build(BuildContext context) {
    return CustomScrollView(
      slivers: [
        const AppTopBar(title: 'Dashboard'),
        SliverToBoxAdapter(
          child: BalanceSection(
            balance: balance,
            monthlyIncome: monthlyIncome,
            monthlyExpenses: monthlyExpenses,
          ),
        ),
        SliverToBoxAdapter(
          child: MonthlySummarySection(
            income: monthlyIncome,
            expenses: monthlyExpenses,
          ),
        ),
        RecentTransactionsSection(
          transactions: recentTransactions,
          onTransactionTap: (id) =>
              onEvent(DashboardEvent.transactionSelected(id)),
        ),
      ],
    );
  }
}
```

### ui/sections/

Contains logical groupings of related components.

```dart
class BalanceSection extends StatelessWidget {
  const BalanceSection({
    super.key,
    required this.balance,
    required this.monthlyIncome,
    required this.monthlyExpenses,
  });

  final Money balance;
  final Money monthlyIncome;
  final Money monthlyExpenses;

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.all(SpacingTokens.lg),
      child: BalanceCard(
        balance: balance,
        income: monthlyIncome,
        expenses: monthlyExpenses,
      ),
    );
  }
}
```

### ui/components/

Contains reusable visual components specific to the feature.

Components are small, focused widgets that render a single piece of UI.

```dart
class BalanceCard extends StatelessWidget {
  const BalanceCard({
    super.key,
    required this.balance,
    required this.income,
    required this.expenses,
  });

  final Money balance;
  final Money income;
  final Money expenses;

  @override
  Widget build(BuildContext context) {
    final theme = Theme.of(context);

    return AppCard(
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Text(
            'Current Balance',
            style: theme.textTheme.labelMedium?.copyWith(
              color: theme.colorScheme.onSurfaceVariant,
            ),
          ),
          const SizedBox(height: SpacingTokens.xs),
          Text(
            balance.formatted,
            style: theme.textTheme.displaySmall?.copyWith(
              fontWeight: FontWeight.w700,
            ),
          ),
          const SizedBox(height: SpacingTokens.lg),
          Row(
            children: [
              _SummaryItem(
                label: 'Income',
                amount: income,
                color: ColorTokens.success700,
              ),
              const SizedBox(width: SpacingTokens.xxl),
              _SummaryItem(
                label: 'Expenses',
                amount: expenses,
                color: theme.colorScheme.onSurface,
              ),
            ],
          ),
        ],
      ),
    );
  }
}
```

---

## Feature Independence

Features do not import from other features directly.

When a feature needs data from another feature, it accesses it through shared domain use cases or repositories.

```text
features/dashboard/
    ↓ uses
domain/usecases/transaction/get_recent_transactions_use_case.dart
    ↓ uses
domain/repositories/transaction_repository.dart
```

Forbidden:

```text
features/dashboard/
    ↓ imports
features/transactions/ui/components/transaction_row.dart
```

Shared UI components live in `ui/components/`, not inside any feature.

---

# Screen Pattern

## Screen Hierarchy

Every screen follows a four-level hierarchy.

```text
Screen
  ↓
Content
  ↓
Section
  ↓
Component
```

### Screen

- Is a `ConsumerWidget`
- Watches the ViewModel
- Handles state transitions (loading, error, empty, loaded)
- Delegates rendering to Content

### Content

- Is a `StatelessWidget`
- Receives data as constructor parameters
- Receives event callbacks as function parameters
- Arranges Sections into a layout
- Handles scrolling and safe areas

### Section

- Is a `StatelessWidget`
- Groups related Components
- Handles section-level spacing and padding
- May contain section headers

### Component

- Is a `StatelessWidget`
- Renders a single visual element
- Receives all data through constructors
- Has no knowledge of ViewModels or state
- Is reusable within the feature

---

## Screen Rules

- Screens never contain business logic
- Screens never access repositories directly
- Screens never call `setState`
- Screens never start async operations during `build`
- Screens use `const` constructors where possible
- Screens handle all possible states explicitly

---

## Shared UI Components

Reusable components that are not feature-specific live in `ui/components/`.

```text
ui/components/
├── buttons/
│   ├── primary_button.dart
│   ├── secondary_button.dart
│   ├── icon_action_button.dart
│   └── destructive_button.dart
├── cards/
│   ├── app_card.dart
│   └── tappable_card.dart
├── inputs/
│   ├── text_input_field.dart
│   ├── search_input_field.dart
│   └── dropdown_field.dart
├── feedback/
│   ├── empty_state.dart
│   ├── error_state.dart
│   ├── loading_state.dart
│   └── skeleton_loader.dart
├── navigation/
│   ├── app_bottom_navigation.dart
│   └── app_top_bar.dart
└── overlays/
    ├── confirm_dialog.dart
    ├── app_bottom_sheet.dart
    └── app_snackbar.dart
```

These components use design tokens exclusively. They contain no business logic.

---

# ViewModel Pattern

## Responsibilities

ViewModels are the bridge between UI and domain.

A ViewModel:

- Receives events from the UI
- Executes use cases
- Transforms domain results into UI state
- Handles errors and maps them to user-friendly messages
- Manages loading and error states

A ViewModel must not:

- Access DAOs directly
- Perform SQL queries
- Know about widget lifecycle
- Hold references to `BuildContext`
- Navigate directly (delegates to navigation classes)

---

## Implementation

ViewModels extend `AsyncNotifier` from Riverpod.

```dart
@riverpod
class TransactionFormViewModel extends _$TransactionFormViewModel {
  @override
  FutureOr<TransactionFormState> build({String? transactionId}) async {
    if (transactionId != null) {
      final transaction = await ref
          .read(getTransactionDetailUseCaseProvider)
          .execute(transactionId);
      return TransactionFormState.editing(transaction: transaction);
    }

    final categories = await ref
        .read(getCategoriesUseCaseProvider)
        .execute();

    return TransactionFormState.creating(categories: categories);
  }

  Future<void> onEvent(TransactionFormEvent event) async {
    switch (event) {
      case TransactionFormEventSubmit(:final command):
        await _submitTransaction(command);
      case TransactionFormEventCategorySelected(:final categoryId):
        _updateSelectedCategory(categoryId);
      case TransactionFormEventTypeChanged(:final type):
        _updateTransactionType(type);
      case TransactionFormEventValidate():
        _validateForm();
    }
  }

  Future<void> _submitTransaction(CreateTransactionCommand command) async {
    state = const AsyncValue.loading();

    state = await AsyncValue.guard(() async {
      final useCase = ref.read(createTransactionUseCaseProvider);
      await useCase.execute(command);
      ref.read(transactionNavigationProvider).goBack();
      return state.valueOrNull ?? const TransactionFormState.initial();
    });
  }

  void _updateSelectedCategory(String categoryId) {
    final current = state.valueOrNull;
    if (current != null) {
      state = AsyncData(current.copyWith(selectedCategoryId: categoryId));
    }
  }
}
```

---

## Event Handling

ViewModels receive events through a single `onEvent` method.

```dart
Future<void> onEvent(TransactionFormEvent event) async {
  switch (event) {
    case TransactionFormEventSubmit():
      await _submitTransaction(event.command);
    case TransactionFormEventCategorySelected():
      _updateSelectedCategory(event.categoryId);
    case TransactionFormEventTypeChanged():
      _updateTransactionType(event.type);
    case TransactionFormEventValidate():
      _validateForm();
  }
}
```

The exhaustive `switch` ensures every event is handled. Adding a new event forces the developer to handle it.

---

## Error Handling in ViewModels

ViewModels catch domain exceptions and map them to UI-friendly error states.

```dart
Future<void> _submitTransaction(CreateTransactionCommand command) async {
  state = const AsyncValue.loading();

  state = await AsyncValue.guard(() async {
    try {
      final useCase = ref.read(createTransactionUseCaseProvider);
      await useCase.execute(command);
      ref.read(transactionNavigationProvider).goBack();
      return const TransactionFormState.success();
    } on ValidationException catch (e) {
      return TransactionFormState.validationError(
        fieldErrors: e.fieldErrors,
      );
    } on DatabaseException catch (e) {
      ref.read(appLoggerProvider).error('Transaction save failed', e);
      return const TransactionFormState.error(
        message: 'Could not save transaction. Please try again.',
      );
    }
  });
}
```

`AsyncValue.guard` is the standard pattern for catching exceptions in Riverpod notifiers.

---

# Use Case Pattern

## Single Responsibility

Every use case performs exactly one business operation.

```dart
class CreateTransactionUseCase {
  CreateTransactionUseCase({
    required TransactionRepository transactionRepository,
    required CategoryRepository categoryRepository,
  })  : _transactionRepository = transactionRepository,
        _categoryRepository = categoryRepository;

  final TransactionRepository _transactionRepository;
  final CategoryRepository _categoryRepository;

  Future<Transaction> execute(CreateTransactionCommand command) async {
    // Validate
    if (command.amountMinor <= 0) {
      throw ValidationException('Amount must be greater than zero');
    }

    if (command.categoryId != null) {
      final category = await _categoryRepository.findById(command.categoryId!);
      if (category == null) {
        throw NotFoundException('Category not found');
      }
    }

    // Create domain model
    final transaction = Transaction.create(
      workspaceId: command.workspaceId,
      profileId: command.profileId,
      categoryId: command.categoryId,
      amountMinor: command.amountMinor,
      currency: command.currency,
      transactionType: command.transactionType,
      note: command.note,
      merchant: command.merchant,
      occurredAt: command.occurredAt,
    );

    // Persist
    return await _transactionRepository.create(transaction);
  }
}
```

---

## Use Case Rules

- One public method named `execute` (or `call` for simple cases)
- Receives a command object or primitive parameters
- Returns a domain model or void
- Throws domain exceptions on failure
- Never accesses UI, Flutter framework or database directly
- Validates business rules before persistence
- Is testable in isolation with mocked repositories

---

## Command Objects

Complex use case inputs use command objects.

```dart
class CreateTransactionCommand {
  const CreateTransactionCommand({
    required this.workspaceId,
    required this.profileId,
    required this.amountMinor,
    required this.currency,
    required this.transactionType,
    required this.occurredAt,
    this.categoryId,
    this.note,
    this.merchant,
  });

  final String workspaceId;
  final String profileId;
  final int amountMinor;
  final CurrencyCode currency;
  final TransactionType transactionType;
  final DateTime occurredAt;
  final String? categoryId;
  final String? note;
  final String? merchant;
}
```

Command objects are immutable. They carry data, not behavior.

---

## Use Case Provider

Use cases are provided through Riverpod.

```dart
@riverpod
CreateTransactionUseCase createTransactionUseCase(Ref ref) {
  return CreateTransactionUseCase(
    transactionRepository: ref.watch(transactionRepositoryProvider),
    categoryRepository: ref.watch(categoryRepositoryProvider),
  );
}
```

---

# Repository Pattern

## Interface in Domain

Repository interfaces live in the domain layer.

```dart
abstract class TransactionRepository {
  Future<Transaction> create(Transaction transaction);
  Future<Transaction> update(Transaction transaction);
  Future<void> delete(String transactionId);
  Future<Transaction?> findById(String transactionId);
  Future<List<Transaction>> findByProfile(
    String profileId, {
    DateTime? startDate,
    DateTime? endDate,
    TransactionType? type,
    int? limit,
    int? offset,
  });
  Future<Money> calculateBalance(String profileId, {DateTime? forMonth});
  Stream<List<Transaction>> watchByProfile(String profileId);
}
```

Interfaces define the contract. They contain no implementation details.

---

## Implementation in Data

Repository implementations live in the data layer.

```dart
class TransactionRepositoryImpl implements TransactionRepository {
  TransactionRepositoryImpl({
    required TransactionDao transactionDao,
    required TransactionMapper mapper,
  })  : _dao = transactionDao,
        _mapper = mapper;

  final TransactionDao _dao;
  final TransactionMapper _mapper;

  @override
  Future<Transaction> create(Transaction transaction) async {
    final companion = _mapper.toCompanion(transaction);
    final row = await _dao.insertTransaction(companion);
    return _mapper.toDomain(row);
  }

  @override
  Future<Transaction?> findById(String transactionId) async {
    final row = await _dao.findTransactionById(transactionId);
    if (row == null) return null;
    return _mapper.toDomain(row);
  }

  @override
  Future<List<Transaction>> findByProfile(
    String profileId, {
    DateTime? startDate,
    DateTime? endDate,
    TransactionType? type,
    int? limit,
    int? offset,
  }) async {
    final rows = await _dao.findTransactionsByProfile(
      profileId,
      startDate: startDate,
      endDate: endDate,
      type: type?.name,
      limit: limit,
      offset: offset,
    );
    return rows.map(_mapper.toDomain).toList();
  }

  @override
  Future<Money> calculateBalance(String profileId, {DateTime? forMonth}) async {
    final result = await _dao.calculateBalance(profileId, forMonth: forMonth);
    return Money(amountMinor: result, currency: CurrencyCode('USD'));
  }

  @override
  Stream<List<Transaction>> watchByProfile(String profileId) {
    return _dao
        .watchTransactionsByProfile(profileId)
        .map((rows) => rows.map(_mapper.toDomain).toList());
  }
}
```

---

## Mapper Classes

Mappers convert between database rows and domain models.

```dart
class TransactionMapper {
  Transaction toDomain(TransactionRow row) {
    return Transaction(
      id: row.id,
      workspaceId: row.workspaceId,
      profileId: row.profileId,
      categoryId: row.categoryId,
      amountMinor: row.amountMinor,
      currency: CurrencyCode(row.currency),
      transactionType: TransactionType.values.byName(row.transactionType),
      note: row.note,
      merchant: row.merchant,
      occurredAt: row.occurredAt,
      createdAt: row.createdAt,
      updatedAt: row.updatedAt,
    );
  }

  TransactionsCompanion toCompanion(Transaction transaction) {
    return TransactionsCompanion.insert(
      id: transaction.id,
      workspaceId: transaction.workspaceId,
      profileId: transaction.profileId,
      categoryId: Value(transaction.categoryId),
      amountMinor: transaction.amountMinor,
      currency: transaction.currency.code,
      transactionType: transaction.transactionType.name,
      note: Value(transaction.note),
      merchant: Value(transaction.merchant),
      occurredAt: transaction.occurredAt,
    );
  }
}
```

---

## Repository Provider

Repositories are provided through Riverpod.

```dart
@riverpod
TransactionRepository transactionRepository(Ref ref) {
  return TransactionRepositoryImpl(
    transactionDao: ref.watch(transactionDaoProvider),
    mapper: TransactionMapper(),
  );
}
```

---

# Error Handling

## Error Type Hierarchy

All application errors extend a sealed base class.

```dart
sealed class AppException implements Exception {
  const AppException(this.message, [this.cause]);

  final String message;
  final Object? cause;
}

class ValidationException extends AppException {
  const ValidationException(super.message, {this.fieldErrors});

  final Map<String, String>? fieldErrors;
}

class NotFoundException extends AppException {
  const NotFoundException(super.message);
}

class ConflictException extends AppException {
  const ConflictException(super.message);
}

class DatabaseException extends AppException {
  const DatabaseException(super.message, [super.cause]);
}

class StorageException extends AppException {
  const StorageException(super.message, [super.cause]);
}

class NetworkException extends AppException {
  const NetworkException(super.message, [super.cause]);
}

class AuthenticationException extends AppException {
  const AuthenticationException(super.message);
}

class UnknownException extends AppException {
  const UnknownException(super.message, [super.cause]);
}
```

---

## Error States

Every screen handles errors explicitly.

```dart
state.when(
  initial: () => const LoadingState(),
  loading: () => const LoadingState(),
  loaded: (data) => ContentWidget(data: data),
  empty: () => const EmptyState(
    title: 'No data',
    message: 'Nothing to show yet.',
  ),
  error: (message, exception) => ErrorState(
    message: message,
    onRetry: () => viewModel.onEvent(const Event.retry()),
  ),
);
```

---

## Error Recovery

Errors should be recoverable whenever possible.

| Error Type | Recovery Strategy |
|---|---|
| Validation | Show field-level error, preserve input |
| Database | Retry once, then show error with retry button |
| Storage | Show error, suggest freeing space |
| Network | Queue for later, show offline indicator |
| Authentication | Redirect to lock screen |
| Unknown | Log details, show generic message |

---

## Error Translation

Infrastructure errors are translated before reaching the UI.

```text
SQLiteException: UNIQUE constraint failed
    ↓
DatabaseException: 'Transaction already exists'
    ↓
TransactionFormState.error(message: 'This transaction could not be saved.')
```

The UI never displays raw exception messages or stack traces.

---

## Error Boundary

The root application widget catches unhandled errors.

```dart
ErrorWidget.builder = (FlutterErrorDetails details) {
  return MaterialApp(
    home: Scaffold(
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Icon(Icons.error_outline, size: 48),
            const SizedBox(height: 16),
            const Text('Something went wrong'),
            const SizedBox(height: 8),
            ElevatedButton(
              onPressed: () {
                // Restart the app
              },
              child: const Text('Restart'),
            ),
          ],
        ),
      ),
    ),
  );
};
```

---

# Logging

## Centralized Logger

All logging goes through a single logger wrapper.

```dart
class AppLogger {
  AppLogger(this._logger);

  final Logger _logger;

  void debug(String message, [Object? error, StackTrace? stackTrace]) {
    _logger.d(message, error: error, stackTrace: stackTrace);
  }

  void info(String message, [Object? error, StackTrace? stackTrace]) {
    _logger.i(message, error: error, stackTrace: stackTrace);
  }

  void warning(String message, [Object? error, StackTrace? stackTrace]) {
    _logger.w(message, error: error, stackTrace: stackTrace);
  }

  void error(String message, [Object? error, StackTrace? stackTrace]) {
    _logger.e(message, error: error, stackTrace: stackTrace);
  }
}
```

---

## Log Levels

| Level | When | Example |
|---|---|---|
| Debug | Development detail | `Database query executed in 12ms` |
| Info | Lifecycle events | `Workspace initialized` |
| Warning | Recoverable issues | `Category not found, using default` |
| Error | Failed operations | `Transaction save failed` |

Production builds disable debug logging.

---

## Sensitive Data Rules

The following must never appear in logs:

- Financial amounts
- Transaction notes
- Merchant names
- Category names (in some contexts)
- User names or emails
- PINs or passwords
- Access tokens
- Database encryption keys
- Backup contents

A log sanitizer strips sensitive fields before output.

```dart
class LogSanitizer {
  static String sanitize(String message) {
    // Remove patterns that look like amounts
    // Remove UUIDs that could be real entity IDs in production
    // Remove any key/token patterns
    return message;
  }
}
```

---

## Log Provider

```dart
@riverpod
AppLogger appLogger(Ref ref) {
  return AppLogger(
    Logger(
      printer: PrettyPrinter(methodCount: 0),
      level: kDebugMode ? Level.debug : Level.warning,
    ),
  );
}
```

---

# Performance Guidelines

## Rebuild Optimization

- Use `const` constructors wherever possible
- Use `ConsumerWidget` instead of `StatefulWidget`
- Watch only the specific provider needed, not a parent provider
- Use `select` to watch specific fields of a state object
- Avoid creating new objects inside `build` methods

```dart
// Bad: rebuilds on any state change
final state = ref.watch(viewModelProvider);

// Good: rebuilds only when balance changes
final balance = ref.watch(viewModelProvider.select((s) => s.balance));
```

---

## List Performance

- Use `ListView.builder` for all scrollable lists
- Provide `itemExtent` or `prototypeItem` when item height is known
- Use stable keys for list items
- Avoid building complex widgets inside list items without need
- Use `SliverList` inside `CustomScrollView` for mixed layouts

```dart
ListView.builder(
  itemCount: transactions.length,
  itemBuilder: (context, index) {
    return TransactionRow(
      key: ValueKey(transactions[index].id),
      transaction: transactions[index],
      onTap: () => onTransactionTap(transactions[index].id),
    );
  },
)
```

---

## Async Guidelines

- Never block the main isolate
- Use `compute()` for CPU-intensive work (JSON parsing, CSV generation)
- Use `Isolate.run` for heavy operations in Dart 3
- Stream database queries for reactive UI updates
- Debounce search and filter inputs
- Cancel unused subscriptions when widgets dispose

---

## Memory Management

- Dispose controllers, scroll controllers and animation controllers
- Cancel stream subscriptions in `ref.onDispose`
- Avoid caching large datasets in memory
- Use pagination for transaction history
- Release image resources when not visible

```dart
@riverpod
Stream<List<Transaction>> transactionStream(Ref ref, String profileId) {
  final repository = ref.watch(transactionRepositoryProvider);
  final stream = repository.watchByProfile(profileId);

  ref.onDispose(() {
    // Stream is automatically cancelled when provider is disposed
  });

  return stream;
}
```

---

## Startup Performance

- Defer non-critical initialization to after first frame
- Show the splash screen during database initialization
- Preload only essential data (workspace, active profile)
- Lazy-load feature-specific data when the feature is first accessed

---

# Accessibility

## Semantics

Every interactive element must have a semantic label.

```dart
IconButton(
  icon: const Icon(Icons.add),
  onPressed: onAdd,
  tooltip: 'Add transaction',
  // Semantics are provided by IconButton automatically
)

Semantics(
  label: 'Balance: \$1,250.50',
  child: BalanceCard(balance: balance),
)
```

---

## Contrast

All text must meet WCAG AA contrast requirements.

| Element | Minimum Contrast |
|---|---|
| Body text | 4.5:1 |
| Large text (18px+) | 3:1 |
| Interactive elements | 3:1 |
| Decorative elements | No requirement |

Both light and dark themes are validated for contrast.

---

## Touch Targets

Every interactive element must have a minimum touch target of 48×48 logical pixels.

```dart
SizedBox(
  width: 48,
  height: 48,
  child: IconButton(
    icon: const Icon(Icons.delete, size: 20),
    onPressed: onDelete,
  ),
)
```

---

## Dynamic Text

Layouts must support increased system font sizes.

Rules:

- No fixed-height text containers
- No truncated critical amounts
- No overlapping labels
- Forms must remain scrollable with large text
- Critical actions must not rely on icon-only buttons

---

## Screen Readers

Every screen must be navigable by screen readers.

Requirements:

- Logical focus order follows visual layout
- State changes are announced
- Loading states are announced
- Error messages are announced
- Empty states are announced
- Charts have text alternatives

```dart
Semantics(
  label: 'Monthly spending chart. January: \$2,450. February: \$1,890.',
  child: MonthlyTrendChart(data: trendData),
)
```

---

## Reduced Motion

Animations must respect the system `disableAnimations` setting.

```dart
final disableAnimations = MediaQuery.disableAnimations(context);

if (disableAnimations) {
  return child;
}

return AnimatedOpacity(
  opacity: isVisible ? 1.0 : 0.0,
  duration: const Duration(milliseconds: 200),
  child: child,
);
```

---

# Animations

## Principles

Animations should be subtle and purposeful.

They communicate:

- State transitions (loading to content)
- Navigation (screen transitions)
- Feedback (button press, success)
- Relationship (which element changed)

They should not:

- Distract from content
- Delay user interaction
- Play excessively
- Be purely decorative

---

## Duration Tokens

| Token | Duration | Use Case |
|---|---|---|
| Fast | 100–150 ms | Button press, toggle |
| Standard | 200–250 ms | Fade, slide, size change |
| Slow | 300–400 ms | Screen transitions, complex animations |

---

## Preferred Animations

- `AnimatedOpacity` for fade transitions
- `AnimatedSlide` for content entering or leaving
- `AnimatedSize` for expanding/collapsing sections
- `AnimatedCrossFade` for state switching
- `Hero` for shared element transitions between screens

---

## Screen Transitions

GoRouter handles screen transitions.

```dart
GoRoute(
  path: RoutePaths.transactionDetail,
  pageBuilder: (context, state) {
    return CustomTransitionPage(
      child: TransactionDetailScreen(
        transactionId: state.pathParameters['id']!,
      ),
      transitionsBuilder: (context, animation, secondaryAnimation, child) {
        return FadeTransition(
          opacity: animation,
          child: child,
        );
      },
    );
  },
)
```

---

## State Change Communication

Loading states use skeleton placeholders instead of spinners where content structure is known.

```dart
if (isLoading) {
  return const SkeletonLoader(
    child: BalanceCardSkeleton(),
  );
}
```

Success feedback uses a brief snackbar or inline confirmation.

```dart
AppSnackbar.show(
  context,
  message: 'Transaction saved',
  type: SnackbarType.success,
);
```

---

# Future Expansion

## Sync Engine Integration

The architecture is designed so that synchronization can be added without changing feature code.

The integration point is the repository layer.

### Current (Phase 1)

```text
ViewModel
    ↓
Use Case
    ↓
Repository Implementation
    ↓
Local DAO
    ↓
SQLite
```

### Future (Phase 3)

```text
ViewModel
    ↓
Use Case
    ↓
Repository Implementation
    ↓
Sync Engine
    ├── Local DAO → SQLite
    └── Remote API → Backend
```

The ViewModel and Use Case layers remain unchanged. The repository implementation gains a sync engine dependency.

---

## Preparing for Sync

Phase 1 code already includes forward-compatible patterns:

- UUID primary keys (no auto-increment conflicts)
- `created_at`, `updated_at`, `deleted_at` on every entity
- `version` field for optimistic concurrency
- `sync_status` field (unused but present)
- `device_id` field (unused but present)
- Soft delete instead of hard delete
- Repository interfaces that hide storage details

---

## Adding the Sync Engine

When synchronization is introduced:

1. A `SyncEngine` class is added to the data layer
2. Repository implementations are updated to route through the sync engine
3. The sync engine handles local writes, remote writes, conflict detection and retry
4. A `SyncStatusProvider` exposes connection and sync state to the UI
5. No feature-level code changes are required

---

## Other Future Expansions

The architecture supports the following without restructuring:

| Feature | Integration Point |
|---|---|
| OCR | New feature module in `features/scanner/` |
| QR Scanner | New feature module in `features/scanner/` |
| Bank Import | New use cases in `domain/usecases/import/` |
| Budgets | New feature module in `features/budgets/` |
| Recurring Transactions | New entity and use cases |
| Cloud Backup | New repository implementation |
| Desktop | Flutter desktop target, shared codebase |
| Multi-language | `l10n/` directory with ARB files |

Each addition follows the same feature structure defined in this document.

---

# Guiding Principle

The mobile implementation should remain boring.

Every feature should look the same. Every ViewModel should work the same. Every screen should follow the same pattern.

A developer joining the project should understand any feature by reading one example.

Consistency is more valuable than cleverness. Predictability is more valuable than novelty.

The code should be easy to read, difficult to misuse and simple to test.
