# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

UpasthitiX is a Flutter attendance tracking app that uses BLE (Bluetooth Low Energy) for proximity verification and Supabase as the backend. Teachers create attendance sessions (with codes and/or BLE broadcasting), and students mark attendance by entering session codes or scanning for BLE signals.

## Commands

```bash
# Install dependencies
flutter pub get

# Run app (debug)
flutter run

# Build for production
flutter build apk --release        # Android
flutter build ios --release        # iOS

# Analysis and linting
flutter analyze

# Run tests
flutter test
flutter test test/relative_path_test.dart    # Single test file
```

## Architecture

### Entry Point & Initialization
- `lib/main.dart` - App entry point. Initializes Supabase, locks device orientation to portrait, and sets up a custom error widget (`CustomErrorWidget`).
- `lib/routes/app_routes.dart` - Defines named routes for the three main screens.

### Core Services (`lib/services/`)
- **SupabaseService** (`supabase_service.dart`) - Singleton-style static class handling all backend operations: auth (sign in/up), user profiles, class/subject queries, session creation/teardown, attendance marking/revoking, and Supabase Realtime subscriptions for live attendance updates.
- **BleService** (`ble_service.dart`) - Static class managing BLE operations. Teachers "advertise" sessions (currently simulated via device name; production should use `flutter_ble_peripheral`). Students scan for nearby sessions with RSSI-based proximity validation (requires 3 samples for stability).

### Models (`lib/models/`)
- `user_model.dart` - User with role-based access (admin/teacher/student)
- `session_model.dart` - Attendance session with security levels, RSSI thresholds, and active/inactive state. Also includes `ClassModel`, `SubjectModel`, `AssignmentModel`, `AttendanceHistoryModel`.
- `attendance_model.dart` - Attendance records with status (present/revoked) and audit trail support.

### Presentation Layer (`lib/presentation/`)
Three main screens, each with decomposed widget files:
- **SignUpLoginScreen** - Authentication with role selection, particle background, and demo credentials
- **TeacherSessionScreen** - Session configuration, BLE broadcast indicator, session code display, attendance list, and session stats
- **StudentAttendanceScreen** - Code entry, BLE scan, RSSI meter, and attendance history

### Shared Widgets (`lib/widgets/`)
Reusable UI components: `CustomImageWidget`, `CustomIconWidget`, `GlassCardWidget`, `GlassFormFieldWidget`, `StatusBadgeWidget`, `LoadingSkeletonWidget`, `EmptyStateWidget`, `CustomErrorWidget`.

### Theme (`lib/theme/app_theme.dart`)
Dark-only theme (lightTheme redirects to darkTheme). Uses Material 3 with a custom color system (primary: `#6C63FF`, secondary: `#22D3EE`). Typography uses Google Fonts' Plus Jakarta Sans. Responsive sizing via the `sizer` package (`.w`, `.h` extensions).

### Barrel Export (`lib/core/app_export.dart`)
Central export file re-exports Flutter, Sizer, Google Fonts, theme, widgets, services, and models. Most files import only this barrel file.

## Critical Rules

- **Dependencies**: Never remove `flutter`, `flutter_test`, `flutter_lints`, or `sizer` from pubspec.yaml - they are marked CRITICAL.
- **Assets**: Only use `assets/` and `assets/images/` directories. Do NOT create sub-directories like `assets/svg/` or `assets/icons/`.
- **Fonts**: This project uses Google Fonts (`plus_jakarta_sans`) - never add local font files or a `fonts:` section to pubspec.yaml.
- **Orientation**: The app is locked to portrait mode (`DeviceOrientation.portraitUp`) in main.dart - do not change.
- **Theme**: The app is dark-theme only (`ThemeMode.light` is set but uses darkTheme). Modifying `AppTheme.lightTheme` or `AppTheme.darkTheme` requires updating both.
- **Error handling**: A custom `ErrorWidget.builder` is set in main.dart using `CustomErrorWidget` - do not remove.

## Supabase Backend

- **Project URL**: `https://fgmdixxhzwhgaiajcxal.supabase.co`
- **Anon Key**: Stored in `SupabaseService._anonKey`
- **Key Tables**: `users`, `classes`, `subjects`, `teacher_assignments`, `sessions`, `attendance`, `attendance_logs`
- **Realtime**: Used for live attendance list updates via `subscribeToSessionAttendance()`
