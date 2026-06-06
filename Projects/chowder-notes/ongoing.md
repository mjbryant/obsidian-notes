# Ongoing — chowder

## In progress
- Design phase: translating `PROJECT_PLAN.md` vision into concrete implementation decisions
- Planning first build step (detailed architecture → bootstrap)

## Open questions

**From the design doc:**
- **Grocery package intelligence** — how does Chowder determine purchasable units and store-specific inventory? Package sizes, unit conversions, substitution strategies.
- **Preference modeling** — how do household preferences (kids' tastes, dietary constraints, novelty tolerance, time constraints) evolve and persist over time?
- **Grocery automation** — which fulfillment providers support integration? What requires explicit user approval? What workflows remain stable?

**New questions to resolve before coding:**
- Platform stack: Swift/SwiftUI for iOS? Separate macOS app or shared codebase?
- Data format: iCloud Drive markdown? JSON? SQLite? Something else portable?
- Sync strategy: How does data flow between iPhone and Mac mini deployment target?
- AI integration: Which SDK? How are prompts versioned and managed? How does model selection work at runtime?
- Recipe schema: What does the canonical portable format look like?
- Mac mini role: Server? Local sync hub? Both?

## Next steps
1. Write detailed architecture document resolving the open questions above — decisions + rationale
2. Bootstrap Xcode project with core data models and storage layer
3. Implement basic orchestrator with a single-turn conversation flow
4. Wire up Siri/App Intents for voice-first entry point
5. Build core loop end-to-end: context → meal plan → recipes → shopping list

## Blocked on
<!-- Nothing currently. -->
