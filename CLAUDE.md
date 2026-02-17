# amplify-android

## What This Is

bFAN's fork of the AWS Amplify Library for Android. This provides the Android app with high-level interfaces for AWS services (Auth, API, Storage, DataStore, Push Notifications, Geo, Analytics, Predictions).

## Fork Status

**Status:** ACTIVE -- used in production, but only for 2 of 189 app flavors.  
**Upstream:** [aws-amplify/amplify-android](https://github.com/aws-amplify/amplify-android)  
**Fork point:** Upstream ~v2.29.2 (August 2025, last synced 2025-08-25)  
**Current upstream:** v2.33.0+ (the version used by the other 187 flavors)  
**Distribution:** JitPack as `com.github.bfansports.amplify-android` (`main-SNAPSHOT`)  
**Maintainer:** Chase Lawrence (@Kirvlawc)  

### Which flavors use the fork?

Only **`fcnantes`** and **`olympiquedemarseille`** use the fork (via JitPack SNAPSHOT).  
All other 187 flavors use the upstream Amplify SDK from Maven Central (v2.33.0).  
This is controlled in `SA-User-WhiteLabelApps-Android/app/build.gradle.kts` (line 642).

### What was customized and why?

All modifications are in the `aws-auth-cognito` module. The core change replaces Chrome Custom Tabs with an in-app WebView for Cognito Hosted UI authentication, enabling SSO cookie injection for Olympique de Marseille.

| PR | Title | What it does |
|----|-------|--------------|
| #1 | Use web view instead of custom tab for hosted UI | Adds `WebViewActivity.kt`, replaces Custom Tabs with WebView for sign-in/sign-out |
| #2 | Remove auth web view title | Hides toolbar title in auth WebView |
| #4 | Enable edge-to-edge | Adds edge-to-edge display support with proper system bar insets |
| #5 | Add cookies for OM SSO | Injects `X-Requested-From=WebView` cookies for `connect.om.fr` domains |
| #6 | Trigger jitpack build | Build trigger (no code change) |
| #7 | Fix cookie typo | Corrects cookie header from `X-Requested-From` to `X-Requested-With` |
| #8 | Attempt to fix session continuity | Major rewrite of `WebViewActivity` with proper lifecycle, cookie flushing, configurable redirects |

**Key files modified:**
- `aws-auth-cognito/src/main/java/.../activities/WebViewActivity.kt` (new file, main customization)
- `aws-auth-cognito/src/main/java/.../HostedUIClient.kt` (added WebView launch methods)
- `aws-auth-cognito/src/main/java/.../actions/HostedUICognitoActions.kt` (switched to WebView)
- `aws-auth-cognito/src/main/java/.../actions/SignOutCognitoActions.kt` (switched to WebView sign-out)
- `aws-auth-cognito/src/main/res/layout/activity_auth_webview.xml` (WebView layout)
- `gradle/libs.versions.toml` (added coordinatorlayout, material dependencies)

## Tech Stack

- **Language**: Kotlin 2.2.0, Java
- **Build System**: Gradle (Kotlin DSL)
- **Minimum Android SDK**: Check `aws-auth-cognito/build.gradle.kts` for `minSdk`
- **Package Manager**: JitPack (fork), Maven Central (upstream)
- **Key Dependencies**:
  - AWS SDK for Kotlin -- underlying AWS service clients
  - Apollo GraphQL -- for AppSync integration
  - SQLite (via Room or similar) -- DataStore persistence
  - Google Play Services -- for push notifications
  - AndroidX CoordinatorLayout, Material -- for WebView auth UI

## Quick Start

```bash
# Build the project
./gradlew build

# Run tests
./gradlew test

# Run connected tests (requires emulator/device)
./gradlew connectedAndroidTest

# Build specific module (the one with bFAN customizations)
./gradlew :aws-auth-cognito:build
```

## Project Structure

```
core/                                    # Core Amplify interfaces and protocols
core-kotlin/                             # Kotlin-specific core APIs
common-core/                             # Shared utilities
aws-core/                                # AWS-specific core functionality
aws-auth-cognito/                        # Cognito authentication plugin [MODIFIED BY BFAN]
aws-api/                                 # REST API plugin
aws-api-appsync/                         # GraphQL/AppSync plugin
aws-datastore/                           # DataStore plugin (offline-first sync)
aws-storage-s3/                          # S3 storage plugin
aws-push-notifications-pinpoint/         # Push notifications plugin
aws-push-notifications-pinpoint-common/  # Push notification utilities
aws-analytics-pinpoint/                  # Analytics plugin
aws-geo-location/                        # Geo/mapping plugin
aws-predictions/                         # ML predictions plugin
aws-predictions-tensorflow/              # TensorFlow-based predictions
rxbindings/                              # RxJava bindings (legacy)
testutils/                               # Test utilities
testmodels/                              # Test data models
canaries/                                # Integration test apps
build-logic/                             # Custom Gradle build logic
configuration/                           # Build configuration
scripts/                                 # Build and release scripts
```

## Dependencies

**Upstream dependency**: This is a fork of AWS's official `amplify-android` SDK.

**Consumed by**:
- `SA-User-WhiteLabelApps-Android` -- but ONLY the `fcnantes` and `olympiquedemarseille` flavors (via JitPack SNAPSHOT). All other 187 flavors use upstream from Maven Central.

**External services**:
- AWS Cognito (Authentication) -- the only service area where bFAN has customizations
- AWS AppSync (GraphQL API)
- AWS S3 (Storage)
- AWS DynamoDB (DataStore)
- AWS Pinpoint (Analytics, Push Notifications)
- AWS Location Service (Geo)
- AWS ML Services (Predictions)

## API / Interface

Consumed via JitPack for the 2 affected flavors:

```kotlin
// In SA-User-WhiteLabelApps-Android/gradle/libs.versions.toml
amplifySnapVersion = "main-SNAPSHOT"
amplify-api-snap = { group = "com.github.bfansports.amplify-android", name = "aws-api", version.ref = "amplifySnapVersion" }
amplify-auth-snap = { group = "com.github.bfansports.amplify-android", name = "aws-auth-cognito", version.ref = "amplifySnapVersion" }
amplify-core-snap = { group = "com.github.bfansports.amplify-android", name = "aws-core", version.ref = "amplifySnapVersion" }
amplify-rxbindings-snap = { group = "com.github.bfansports.amplify-android", name = "rxbindings", version.ref = "amplifySnapVersion" }
amplify-storage-snap = { group = "com.github.bfansports.amplify-android", name = "aws-storage-s3", version.ref = "amplifySnapVersion" }
```

## Upstream Sync Strategy

**Current process:** Manual. No upstream remote configured. Last sync was 2025-08-25.  
**Who syncs:** Chase Lawrence (@Kirvlawc)  

**Recommended process:**
1. Add upstream remote: `git remote add upstream https://github.com/aws-amplify/amplify-android.git`
2. Fetch upstream: `git fetch upstream`
3. Merge: `git merge upstream/main` (conflicts will be limited to `aws-auth-cognito`)
4. Test: Build `fcnantes` and `olympiquedemarseille` flavors against the updated fork
5. Push and verify JitPack build

**Sync frequency:** Should sync at minimum when the Android app bumps its `amplifyVersion` in `libs.versions.toml`.

## Key Patterns

- **Plugin Architecture**: Amplify uses a plugin-based system where each AWS service is a separate plugin registered at runtime
- **Category-based APIs**: Services are grouped into categories (Auth, API, Storage, DataStore, Predictions, Analytics, Geo)
- **Kotlin Coroutines + RxJava**: Modern Kotlin coroutine APIs alongside legacy RxJava bindings and callbacks
- **Offline-first DataStore**: Syncs with cloud when online, works offline when disconnected
- **Escape Hatch**: Direct access to underlying AWS SDK clients when needed via `plugin.escapeHatch`
- **WebView Auth (bFAN)**: Cognito Hosted UI uses in-app WebView instead of Chrome Custom Tabs to support cookie injection for OM SSO

## Environment

No environment variables required for the library itself. Configuration is provided by the consuming Android app via `amplifyconfiguration.json`.

## Deployment

This is a library dependency distributed via JitPack:

1. Make changes in a feature branch
2. Test with the Android app (build `fcnantes` or `olympiquedemarseille` flavor)
3. Merge to `main` via PR
4. JitPack automatically builds `main-SNAPSHOT` from the latest `main` commit

**WARNING:** Since the app uses `main-SNAPSHOT`, any push to `main` immediately affects production builds of the 2 affected flavors. Consider switching to tagged releases.

## Testing

- **Framework**: JUnit 4/5, AndroidX Test
- **Unit tests**: `./gradlew test`
- **Instrumented tests**: `./gradlew connectedAndroidTest` (requires emulator/device)
- **Coverage**: Kover (see `kover.gradle`)
- **bFAN-specific tests**: None exist. The WebView auth customizations have no test coverage.

## Gotchas

- **Fork is narrowly scoped**: Only 2 of 189 flavors use this fork. Don't assume all builds use it.
- **SNAPSHOT builds are live**: Pushing to `main` immediately affects production. Test before merging.
- **Upstream drift accumulates fast**: AWS Amplify Android releases frequently. The fork is currently ~6 months behind.
- **Hardcoded OM cookies**: `WebViewActivity.kt` has `connect.om.fr` and `connect.athena.om.fr` cookies hardcoded. If another flavor needs the fork, these cookies will affect it too.
- **No upstream remote**: You must manually add it before syncing: `git remote add upstream https://github.com/aws-amplify/amplify-android.git`
- **Multi-module Gradle project**: Changes in `core` affect all plugins. Build from root to catch cascading issues.
- **Kotlin version compatibility**: Amplify Android tracks recent Kotlin versions. Ensure bFAN's Android apps use compatible Kotlin compiler versions.
- **AndroidX dependencies**: Amplify depends on AndroidX libraries. Conflicts with legacy support libraries will break the build.
- **Chrome Custom Tabs still exist in code**: The fork adds WebView auth alongside the existing Custom Tabs code. The Custom Tabs code paths are not removed, just bypassed.
- **`javaScriptEnabled = true`**: The WebView has JavaScript enabled. This is necessary for SSO but is flagged by Android Lint as a security concern. The `@SuppressLint("SetJavaScriptEnabled")` suppression is appropriate given the controlled SSO context.
