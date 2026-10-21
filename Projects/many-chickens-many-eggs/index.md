# many-chickens-many-eggs

## What this is
A Wordle-style daily browser puzzle app. Every day the site serves one small, graphical, interactive puzzle (ball bouncing down stairs, resize-the-viewport, browser-only-responds puzzles, etc). Solve it within a time limit or get shown the answer. No backend — pure Next.js/TypeScript static app, state lives in the visitor's `localStorage`. Deployed to Vercel.

Originated from an [[many-chickens-many-eggs app idea|Inbox idea note]] (see `Inbox/Many chickens many eggs app idea.md` in this vault).

## Active context
Bootstrapped 2026-10-20: repo scaffolded at `/Users/mbryant/bed/many-chickens-many-eggs`, three puzzle types shipped (Staircase, Portal, Seal), client-side adaptive difficulty calibration targeting 70% success rate, daily-attempt gating + practice mode, hints, build verified, smoke-tested in browser. Next: push to GitHub and deploy on Vercel (needs `vercel login` — device-flow auth, interactive).

## Key notes
- [[architecture]] — codebase structure, key files, patterns
- [[ongoing]] — current work in progress, open questions
- [[ideas]] — planned features, future work
- [[session-log]] — history of Claude Code / Hermes sessions

## Quick reference
- Repo: `/Users/mbryant/bed/many-chickens-many-eggs`
- Local dev: `npm run dev`
- Build: `npm run build`
- Deploy: `vercel` (not yet authenticated on this machine)
