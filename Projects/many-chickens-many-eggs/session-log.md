# session-log

## 2026-10-20 — Hermes (bootstrap session)
- Scaffolded Next.js 16 / React 19 / TypeScript / Tailwind v4 app at `/Users/mbryant/bed/many-chickens-many-eggs` via `create-next-app`.
- Built: `src/lib/seed.ts` (daily seeded RNG), `src/lib/calibration.ts` (70%-target difficulty controller), `src/lib/daily.ts` (daily gating/streaks), `src/lib/client-id.ts`.
- Built 3 puzzle types: StairDrop (canvas), ResizePortal (window resize / device rotation), BrowserTell (feature-probe browser detection).
- Built `GameShell.tsx` orchestrating instructions → timer → puzzle → result, with daily/practice modes and a progressive hint system.
- `npx tsc --noEmit` clean; `npm run build` produces a fully static export; smoke-tested in a real browser (begin → hint reveal → timeout loss → practice mode cycling through Portal/Seal puzzles) — all working as designed.
- Pending at end of session: push to GitHub, `vercel login` (interactive device-flow auth needed), `vercel` deploy.
