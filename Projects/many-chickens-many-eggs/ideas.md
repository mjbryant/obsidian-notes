# ideas

- More puzzle types: double-click timing, devicePixelRatio/zoom-level puzzle, clipboard-paste puzzle, drag-and-drop, `prefers-color-scheme`/`prefers-reduced-motion` media query puzzle, geolocation/timezone puzzle, service-worker/offline puzzle, Web Audio puzzle (listen for a tone), drag-select puzzle.
- Share-card result (Wordle-style emoji grid) for today's result, shareable without spoiling the puzzle mechanic.
- Optional Supabase backend later: aggregate (anonymous) success-rate telemetry across all players to tune the calibration engine's learning rate/target globally instead of per-browser-only; cross-device streaks if a player wants to log in.
- A small in-app "how puzzles are generated" explainer page, once there are enough puzzle types to make the pattern (canvas rendering + browser APIs + seeded daily rotation) clear without spoiling individual mechanics.
- Consider genuinely AI-generated (LLM-authored) puzzle variants rather than only hand-authored templates, per the original idea — would need a review/sandboxing step before shipping AI-written HTML/CSS/JS to visitors.
