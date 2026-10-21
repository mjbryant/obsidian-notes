# architecture

Next.js 16 (App Router) + React 19 + TypeScript + Tailwind v4. No backend, no database — Vercel static/edge deploy, state in `localStorage`.

## Key files
- `src/lib/seed.ts` — mulberry32 seeded RNG + daily date-key hashing. Everyone gets the same puzzle instance on a given calendar day.
- `src/lib/calibration.ts` — the self-improving difficulty engine. Per puzzle type keeps a `mu` (difficulty estimate 0..1) in localStorage; after each attempt, `mu += LEARNING_RATE * (result - 0.7)` (stochastic approximation toward 70% success). Each instance samples `mu ± small jitter`, which is what keeps per-puzzle success uniform around the target instead of bimodal (trivial vs impossible).
- `src/lib/daily.ts` — one official attempt per calendar day, gates replays; streak tracking.
- `src/lib/client-id.ts` — anonymous per-browser id, used only to vary practice-mode seeds per visitor.
- `src/lib/puzzles/types.ts` — `PuzzleRuntimeProps` (seedKey, difficulty, hintLevel, onResult) and `PuzzleDef` (name/blurb/instructions/hints/timeLimit/Component) contracts.
- `src/lib/puzzles/registry.ts` — the list of puzzle types. Add a new type here.
- `src/components/GameShell.tsx` — orchestrates instructions → countdown timer → mounted puzzle → win/lose/reveal screen. Handles daily-vs-practice mode switching.
- `src/components/puzzles/StairDrop.tsx` — canvas-rendered; ball bounces down N stairs, click the correct one before the ball arrives.
- `src/components/puzzles/ResizePortal.tsx` — reshape the page: window resize (desktop) or device rotation (mobile, via `matchMedia('(orientation:...)')`).
- `src/components/puzzles/BrowserTell.tsx` — only "solvable" in one browser engine per day, detected via feature probes (`CSS.supports`, `navigator.userAgentData`) rather than raw UA string parsing.

## Obfuscation approach
Puzzle visuals are drawn at runtime (canvas, computed inline styles) rather than spelled out as readable DOM/CSS, and production builds are minified by Next.js. This is a speed bump against casual view-source/LLM-dump, not real security — documented as an honest tradeoff in the repo README. Hints are always available via the in-game Hint button regardless.

## Adding a puzzle type
1. Add a `PuzzleTypeId` string union member in `calibration.ts`.
2. Build a component matching `PuzzleRuntimeProps`.
3. Register it in `registry.ts` with copy, hints, and `timeLimit(difficulty)`.
