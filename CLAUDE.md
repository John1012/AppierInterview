# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

`AppierInterview` is a single-module Android app skeleton (interview starter) built with Jetpack Compose, Kotlin 2.3, and a unidirectional MVVM data flow. The current UI just renders a list of greeting strings from a repository — it's a scaffold meant to be extended.

## Commands

Use the Gradle wrapper (`./gradlew`) for everything.

- Build debug APK: `./gradlew :app:assembleDebug`
- Install on a connected device/emulator: `./gradlew :app:installDebug`
- Local unit tests (JVM): `./gradlew :app:testDebugUnitTest`
- Run a single unit test: `./gradlew :app:testDebugUnitTest --tests "com.example.appierinterview.ui.main.MainScreenViewModelTest"` (append `.methodName` to target one method)
- Instrumented / Compose UI tests (needs a device or emulator running): `./gradlew :app:connectedDebugAndroidTest`
- Clean: `./gradlew clean`

There is no separate lint step wired up beyond Android's default (`./gradlew :app:lintDebug`).

## Architecture

Package root: `com.example.appierinterview`. Layers flow one direction: **UI (Compose) → ViewModel → Repository (Flow)**.

- **Entry point** — `MainActivity` (`ComponentActivity`) calls `setContent { AppierInterviewTheme { ... MainNavigation() } }` with edge-to-edge enabled.
- **Navigation** — Uses **Navigation 3** (`androidx.navigation3`), not the older Nav-Compose. `Navigation.kt` holds a `NavDisplay` driven by a `rememberNavBackStack`; screens are added/removed by pushing/popping `NavKey`s (`backStack.add(navKey)` / `removeLastOrNull()`). Destinations are declared in `NavigationKeys.kt` as `@Serializable data object`s implementing `NavKey` (e.g. `Main`) — hence the `kotlin-serialization` plugin. Add a new screen by defining a new `NavKey` there and an `entry<...> { }` block in the `entryProvider`.
- **Screens** — Under `ui/<feature>/`. Each feature has a stateful Composable that reads a `viewModel` and `collectAsStateWithLifecycle()`, plus an `internal` stateless Composable overload that takes plain data for previews/tests. See `ui/main/MainScreen.kt` for the pattern (the stateless `MainScreen(data: List<String>)` is what UI tests drive).
- **ViewModel** — Exposes a single `uiState: StateFlow<...UiState>` built by mapping the repository `Flow` into a sealed `UiState` (`Loading` / `Success` / `Error`) via `.map{}.catch{}.stateIn(viewModelScope, WhileSubscribed(5000), Loading)`. Replicate this Loading/Success/Error sealed-interface shape for new screens.
- **Data** — `data/DataRepository.kt` defines an interface exposing `Flow`s; `DefaultDataRepository` is the concrete impl. There is **no DI framework** — the ViewModel is constructed inline in the Composable (`viewModel { MainScreenViewModel(DefaultDataRepository()) }`) and tests pass a hand-written fake repository.

## Testing patterns

- Unit tests (`src/test/`) use JUnit4 + `kotlinx-coroutines-test` (`runTest`), asserting on `uiState.first()`. Inject a fake `DataRepository` implemented inline in the test file.
- UI tests (`src/androidTest/`) use `createAndroidComposeRule<ComponentActivity>()` and set content to the **stateless** screen Composable with fixed fake data, then assert with `onNodeWithText`.

## Conventions

- **2-space indentation** throughout Kotlin sources (not the Kotlin-official 4). Match the surrounding files; single-expression bodies and one-line lambdas are kept compact.
- Version catalog: all dependencies and plugin versions live in `gradle/libs.versions.toml` and are referenced as `libs.*`. Add/upgrade deps there, not inline in `build.gradle.kts`.
- Gradle **configuration cache and build cache are enabled** (`gradle.properties`); keep build scripts configuration-cache-compatible.
- `buildConfig`, `aidl`, and `shaders` build features are disabled in `app/build.gradle.kts` — don't rely on `BuildConfig`.
- SDK: `minSdk 24`, `compileSdk`/`targetSdk 36`, Java/JVM toolchain 17.
