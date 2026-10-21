# ongoing

- Need to push the repo to GitHub (`gh repo create`) and run `vercel login` + `vercel` to deploy — both need interactive browser auth that wasn't available in the bootstrap session.
- Only 3 puzzle types shipped so far (Staircase, Portal, Seal). More variety would help the daily rotation feel less repetitive.
- Calibration controller is a simple stochastic-approximation walk (mu += lr*(result-0.7)) — works but untested against real usage data; may need tuning of LEARNING_RATE/JITTER once there's real play history.
- "The Seal" puzzle's browser-engine detection is feature-probe based, not bulletproof — a browser that spoofs CSS.supports results or lacks userAgentData could misclassify. Low priority since it's a flavor puzzle, not security-critical.
- No persistence across devices for a given player (by design — no backend) — stats/streaks are per-browser only. If this ever wants cross-device streaks, that's the fork point for adding Supabase.
