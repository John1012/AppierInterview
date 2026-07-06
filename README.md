# AppierInterview

A single-module Android app skeleton (interview starter) built with Jetpack Compose, Kotlin, and a unidirectional MVVM data flow. The current UI renders a list of greeting strings from a repository — it's a scaffold meant to be extended.

## Tech stack

- **Kotlin** 2.3.20 (JVM toolchain 17)
- **Jetpack Compose** (BOM `2026.03.01`) + Material 3
- **Navigation 3** (`androidx.navigation3`) — not the older Nav-Compose
- **AndroidX Lifecycle** ViewModel + `StateFlow`
- **Coroutines** `Flow`
- Android Gradle Plugin 9.2.1 · `minSdk 24` · `compileSdk`/`targetSdk 36`

## Architecture

Data flows in one direction:

```
UI (Compose) → ViewModel → Repository (Flow)
```

- **`MainActivity`** — `ComponentActivity` entry point. Enables edge-to-edge and calls `setContent { AppierInterviewTheme { MainNavigation() } }`.
- **Navigation** (`Navigation.kt` / `NavigationKeys.kt`) — A `NavDisplay` driven by a `rememberNavBackStack`. Destinations are `@Serializable data object`s implementing `NavKey` (e.g. `Main`). Push with `backStack.add(navKey)`, pop with `removeLastOrNull()`.
- **Screens** (`ui/<feature>/`) — Each feature has a stateful Composable that reads a `viewModel` and `collectAsStateWithLifecycle()`, plus an `internal` stateless overload that takes plain data for previews and UI tests.
- **ViewModel** — Exposes a single `uiState: StateFlow<...UiState>`, mapping the repository `Flow` into a sealed `UiState` (`Loading` / `Success` / `Error`) via `.map{}.catch{}.stateIn(...)`.
- **Data** (`data/DataRepository.kt`) — An interface exposing `Flow`s, with `DefaultDataRepository` as the concrete impl. No DI framework — the ViewModel is constructed inline in the Composable, and tests pass a hand-written fake repository.

## Project layout

```
app/src/main/java/com/example/appierinterview/
├── MainActivity.kt          # Entry point
├── Navigation.kt            # NavDisplay + back stack
├── NavigationKeys.kt        # NavKey destinations
├── data/
│   └── DataRepository.kt    # Repository interface + default impl
├── theme/                   # Compose theme (Color, Type, Theme)
└── ui/main/
    ├── MainScreen.kt        # Stateful + stateless Composables
    └── MainScreenViewModel.kt
```

## Build & run

Use the Gradle wrapper (`./gradlew`) for everything.

| Task | Command |
| --- | --- |
| Build debug APK | `./gradlew :app:assembleDebug` |
| Install on device/emulator | `./gradlew :app:installDebug` |
| Unit tests (JVM) | `./gradlew :app:testDebugUnitTest` |
| Instrumented / Compose UI tests | `./gradlew :app:connectedDebugAndroidTest` |
| Lint | `./gradlew :app:lintDebug` |
| Clean | `./gradlew clean` |

Run a single unit test:

```bash
./gradlew :app:testDebugUnitTest --tests "com.example.appierinterview.ui.main.MainScreenViewModelTest"
```

Append `.methodName` to target one method.

## Testing

- **Unit tests** (`src/test/`) — JUnit4 + `kotlinx-coroutines-test` (`runTest`), asserting on `uiState.first()`. Inject a fake `DataRepository` implemented inline in the test file.
- **UI tests** (`src/androidTest/`) — `createAndroidComposeRule<ComponentActivity>()`, set content to the **stateless** screen Composable with fixed fake data, then assert with `onNodeWithText`.

## Adding a new screen

1. Define a new `NavKey` in `NavigationKeys.kt`.
2. Add an `entry<...> { }` block to the `entryProvider` in `Navigation.kt`.
3. Create `ui/<feature>/` with a stateful Composable, an `internal` stateless overload, and a ViewModel exposing a `Loading` / `Success` / `Error` `UiState`.

## Conventions

- **2-space indentation** throughout Kotlin sources (not the Kotlin-official 4).
- All dependency and plugin versions live in `gradle/libs.versions.toml`, referenced as `libs.*`. Add/upgrade deps there, not inline in `build.gradle.kts`.
- Gradle **configuration cache and build cache are enabled** — keep build scripts config-cache-compatible.
- `buildConfig`, `aidl`, and `shaders` build features are disabled — don't rely on `BuildConfig`.
