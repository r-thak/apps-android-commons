# Commons Android Copilot Instructions

Use the repository root `AGENTS.md` as the source of truth for structure, commands, coding conventions, architecture, security, and handoff requirements. Custom Copilot agents are defined in `.github/agents/*.agent.md`; use `pr-agent.agent.md` for implementation and `architecture-reviewer.agent.md` for focused review tasks. This file adds Copilot-specific review and cloud-agent guidance.

Keep these instructions concise and actionable. Prefer short imperative bullets, concrete examples, and distinct headings. Do not add vague goals such as “improve quality” or instructions about Copilot’s response formatting. Copilot code review is non-deterministic and context-limited; prioritize the highest-value rules and iterate using real pull requests.

Read [`docs/testing-strategy.md`](../docs/testing-strategy.md) before changing tests. This repository does not currently use Maestro; use the existing JUnit, Robolectric, MockWebServer, Espresso, UiAutomator, and AndroidX test infrastructure unless the issue explicitly proposes a tool change.

## Documentation authority

For Android, Kotlin, Gradle, Compose, lifecycle, permissions, accessibility, security, and testing questions, consult current official documentation first: [Android Developers](https://developer.android.com/), [Android app architecture](https://developer.android.com/topic/architecture), [Android testing](https://developer.android.com/training/testing), and [Kotlin coding conventions](https://kotlinlang.org/docs/coding-conventions.html). Use the Commons app documentation repository for project history and domain context, not as a substitute for current platform documentation. Avoid obsolete APIs and undocumented workarounds; if compatibility with `minSdk = 21` requires one, explain it in the PR.

## Before changing code

- Identify the affected feature package, data/API boundaries, persistence, background work, and UI entry points.
- Read existing tests and the relevant flavor configuration (`prod` or `beta`); do not assume `assembleDebug` or `testDebugUnitTest` exists.
- Preserve compatibility with the app's minimum SDK and both product flavors unless the issue explicitly narrows scope.
- Determine whether the change belongs in JVM tests, deterministic emulator smoke tests, or live Beta integration tests.

## Implementation expectations

- Prefer the smallest change that fits the existing architecture. Do not introduce a new framework, module, or abstraction without explaining its ownership and migration path.
- Keep network, storage, and Android framework calls behind testable boundaries where practical. Avoid real network calls in unit tests.
- For functional changes, add a regression test. For UI or navigation changes, add an Android instrumentation test when feasible and include manual verification if automation is not yet possible.
- Do not weaken tests, suppress lint, alter credentials, or delete existing behavior to make CI pass.
- Do not use live Wikimedia accounts or remote mutable state for PR tests. Use fake sessions, test repositories, fixtures, or MockWebServer.
- Replace fixed sleeps and swallowed `NoMatchingViewException` failures with synchronization and explicit diagnostics.

## Architecture expectations

- Keep Activities, Fragments, adapters, and composables focused on UI state and events.
- Keep business rules out of UI classes, Retrofit interfaces, Room DAOs, and Workers.
- Do not pass Retrofit DTOs, Room entities, Android framework types, or `Context` into domain logic.
- Prefer focused interfaces at external boundaries and realistic test implementations over mock-only designs.
- Preserve one source of truth and one-way state flow within each feature.
- Define retry, cancellation, offline, authentication-expiry, and duplicate-operation behavior for network and background work.
- Apply SOLID and DDD concepts when they reduce coupling or clarify a real domain concept; do not add ceremony without a named problem.

## Review expectations

- Flag only actionable issues, prioritizing: incorrect behavior, lifecycle/state bugs, concurrency, data loss, security/privacy, offline/error handling, API compatibility, performance, architectural boundary violations, and missing regression coverage.
- Trace changes across callers, persistence, workers, APIs, and both product flavors.
- Check whether domain logic depends directly on Android, Retrofit, Room, or API DTOs.
- Check whether retries, cancellation, rotation, process death, or duplicate submissions change behavior.
- Require a deterministic test seam for new external dependencies.
- Do not report formatting preferences already enforced by the repository.
- Explain the failure scenario, affected users or code path, and concrete fix for every finding.

## Verification

Run the narrowest relevant Gradle task, then `./gradlew lintProdDebug` and the affected build task. If an emulator is available, run `./gradlew connectedBetaDebugAndroidTest`; otherwise state that instrumentation was not run and provide exact manual steps.

Never claim a command was run if it was only reasoned about. Report the exact variant, device/API level, credentials or backend used, failed test class, and failure category.
