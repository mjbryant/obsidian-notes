# Architecture — chowder

## Intended system layers

Chowder is organized as a pipeline with six layers:

- **Input Layer** — Siri/App Intents (priority), conversational text, URL ingestion, image ingestion, structured editing
- **Orchestrator** — central engine: interprets requests, maintains context, clarifies ambiguity, coordinates workflows, invokes AI models, manages state
- **Recipe Engine** — meal generation, recipe normalization, metadata generation (prep/cook time, servings, tags), variation management
- **Pantry Engine** — pantry assumptions, staple management (salt, oil, common spices), future inventory awareness
- **Shopping Engine** — ingredient aggregation across meals, deduplication, quantity summation, unit normalization, package translation (e.g. 3 eggs → 1 dozen)
- **Fulfillment Layer** — external grocery APIs, cart generation, delivery integration (long-term goal)
- **Storage Layer** — canonical household knowledge: recipes, preferences, pantry assumptions, meal history, household context

## Repository layout
<!-- To be filled once scaffolded. -->

## Key files
<!-- To be filled once scaffolded. -->

## Patterns and conventions

- **Human-readable, portable data** — all canonical data (recipes, preferences, meal plans) in markdown or similarly portable format. Users can read, edit, and export without Chowder.
- **Model agnostic** — AI provider is configurable. Users bring their own API keys and can swap providers. Orchestration belongs to Chowder; intelligence is interchangeable.
- **Flexible input, structured output** — natural language in; structured objects out (meal plans, recipes, shopping lists). Never walls of generated prose.
- **Gradual complexity reveal** — no-decisions mode works out of the box; power user access (editing prompts, tuning novelty, choosing models) is always available but never required.

## Open design decisions
These must be resolved before coding:
- Platform stack (Swift/SwiftUI for iOS? macOS companion app?)
- Data storage format (iCloud Drive markdown files? JSON? SQLite?)
- Sync strategy between iPhone/iPad and Mac mini deployment
- AI integration pattern (SDK choice, prompt versioning, model configuration)
- Canonical recipe schema

## Dependencies
<!-- To be determined during architecture planning. -->

## Build / run
<!-- To be filled once scaffolded. -->
