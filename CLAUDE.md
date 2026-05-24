# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Skill references (read before starting)

Before beginning any task, read the relevant skill guides under `skill/`:

- **[skill/jira.md](skill/jira.md)** — How to fetch tickets, extract requirements, update status, and comment via the Jira MCP server.
- **[skill/figma.md](skill/figma.md)** — How to pull design specs, download images/icons, set up resources, and run visual comparisons via the Figma MCP server.

## Project

Android app (`MyPinApplication`) — Jetpack Compose + Material 3, built with Gradle (Kotlin DSL). Package `com.example.mypinapplication`, `minSdk 24`, `targetSdk 36`. Designs live in Figma and are pulled via the Figma MCP server.

Tickets are tracked in Jira; pull them with the Jira MCP server (`mcp__jira__jira_get`) when given a ticket key (e.g. `SCRUM-XX`).

## Architecture — MVVM + Clean Architecture

The app follows **Clean Architecture** split into three layers, with **MVVM** in the presentation layer.

- **Presentation** (`ui/`) — Jetpack Compose screens and `ViewModel`s. The ViewModel exposes immutable UI state via `StateFlow` and handles events; Composables are stateless and observe state. No business logic in Composables.
- **Domain** (`domain/`) — Pure Kotlin, no Android/framework dependencies. Holds entities (models), repository **interfaces**, and use cases. Use cases encapsulate a single business action and are the only thing ViewModels call.
- **Data** (`data/`) — Repository **implementations**, data sources (remote/local), DTOs, and mappers between DTOs and domain models.

Dependency rule: dependencies point inward — `Presentation → Domain ← Data`. Domain depends on nothing; Data and Presentation depend on Domain. ViewModels never touch data sources directly; they go through use cases.

Suggested package layout under `com.example.mypinapplication`:

```
ui/            # Compose screens, ViewModels, UI state, theme
  theme/
  <feature>/   # <Feature>Screen.kt, <Feature>ViewModel.kt, <Feature>UiState.kt
domain/
  model/       # domain entities
  repository/  # repository interfaces
  usecase/     # use cases
data/
  repository/  # repository implementations
  remote/      # API services, DTOs
  local/       # DataStore / Room
  mapper/      # DTO <-> domain mappers
di/            # dependency injection modules
```

## Design-to-code workflow (required)

When implementing a screen from a Figma design referenced by a ticket:

1. **Pull specs via MCP, never hand-copy values.**
   - `mcp__figma__get_figma_data` for layout, typography, colors, spacing, radii, effects.
   - `mcp__figma__download_figma_images` for image fills and icons.
   - Prefer SVG icons exported as vector drawables in `res/drawable/`; tint them with `tint` / `LocalContentColor`.

2. **Capture a reference screenshot of the Figma frame.**
   - Export the target frame as PNG at 2x via `mcp__figma__download_figma_images` (pass the frame's nodeId, name it `<screen>_figma.png`, save under `design-refs/`).

3. **Compare Figma reference vs. the rendered screen — implementation must match ≥ 80%.**
   - Use the `Read` tool to visually inspect the Figma reference and the rendered screen side by side.
   - Walk through each element: position, size, color, typography weight/size, corner radius, stroke, shadow, gradient stops, spacing between elements, alignment.
   - Score the match honestly. If < 80%, iterate on the Compose code (re-pull Figma specs if values disagree) and re-check until ≥ 80%.
   - Record the final score and any intentional deviations in the PR/task summary.

4. **Do not claim a UI task done without the visual comparison.** A green build verifies code correctness, not visual fidelity.

## Conventions

- Figma is the source of truth for visual specs — padding, font sizes, colors, radii, shadows. Do not hardcode values that diverge from the Figma.
- Use Material 3 typography/color tokens; define design values in `ui/theme/` rather than scattering literals across Composables.
- Keep new code in the correct Clean Architecture layer; respect the inward dependency rule.
- Composables are stateless — hoist state into the `ViewModel`; expose UI state as a single immutable `StateFlow`.
- Keep image/vector assets and strings under `app/src/main/res/`. Save Figma reference screenshots under `design-refs/` (gitignored if not needed in the repo).

## Useful commands

```bash
# Build a debug APK
./gradlew assembleDebug

# Run unit tests
./gradlew test

# Run instrumented (device) tests
./gradlew connectedAndroidTest
```