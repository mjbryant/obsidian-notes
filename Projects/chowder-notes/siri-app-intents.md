# Siri & App Intents — chowder

How Chowder's voice-first invocation actually reaches the app. This is the front door for the [[architecture]] Input Layer.

## Bottom line
A locally-built iPhone/iPad app can expose actions to Siri via the **App Intents** framework. Siri routes matching spoken phrases to the app, runs the code (foreground or background), and speaks/shows a result. No server, no App Store submission, no special entitlement required.

## Two mechanisms — use the new one
- **SiriKit Intents (legacy)** — domain-based (messaging, payments, workouts…). Your action had to fit a predefined Apple domain. Rigid, mostly legacy.
- **App Intents (iOS 16+)** — modern, pure-Swift, arbitrary actions. No `.intentdefinition` files. **This is what Chowder uses.**

## Three pieces that connect a phrase to code
1. **`AppIntent`** — a struct defining one action; its `perform()` runs the logic and returns a result.
2. **`AppShortcut`** — wraps an intent and declares the spoken **trigger phrases**. This is what makes Siri route phrases to the app.
3. **`AppShortcutsProvider`** — lists the shortcuts so the system registers them at install time.

```swift
import AppIntents

struct StartDinnerIntent: AppIntent {
    static var title: LocalizedStringResource = "Take care of dinner"

    func perform() async throws -> some IntentResult & ProvidesDialog {
        // orchestrator logic
        return .result(dialog: "On it — planning dinner now.")
    }
}

struct ChowderShortcuts: AppShortcutsProvider {
    static var appShortcuts: [AppShortcut] {
        AppShortcut(
            intent: StartDinnerIntent(),
            phrases: [
                "Take care of dinner with \(.applicationName)",
                "\(.applicationName), take care of dinner"
            ],
            shortTitle: "Dinner",
            systemImageName: "fork.knife"
        )
    }
}
```

## Constraints that will bite
- **App name must appear in the trigger phrase.** Use `\(.applicationName)`. Siri reserves bare natural-language phrases for itself/first-party apps — a third-party app can't own a phrase with no app reference. An app-name *synonym* can be registered (e.g. "Chowder"), but app identity must be present. Our north-star phrase *"Chowder, take care of dinner"* satisfies this by construction.
- **Parameters** are spoken and Siri prompts for missing ones ("Which recipe?") via `@Parameter` + `AppEntity` + `EntityQuery` (so Siri knows valid values).
- **Personal-team signing works for testing** on your own devices, but free-account shortcuts expire with the provisioning profile (~7 days) and registration can be flaky until the app is launched once. Paid Apple Developer account ($99/yr) → stable, non-expiring installs.
- **Siri TTS vs. in-app dialogue.** App Intents get you Siri invocation + Siri's TTS reply. Richer multi-turn conversation is better handled inside the app after Siri hands off.

## Design implications for Chowder
- **App Shortcuts = cold-invocable actions** (no setup needed). Candidate first set:
  - "Chowder, take care of dinner" → [[ideas]] no-decisions mode
  - "Chowder, what's for dinner"
  - "Chowder, add X to the list"
- **Invocation vs. conversation split (decide early):** Siri = wake + route + short reply; the deep [[ideas]] voice cooking companion loop ("what's next?", "repeat that step") likely lives in-app after handoff rather than fully through Siri.
- Ties to the open [[architecture]] decision on platform stack (Swift/SwiftUI) — App Intents presumes a native Swift target.
