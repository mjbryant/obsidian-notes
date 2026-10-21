# Many Chickens, Many Eggs — daily browser puzzle app

Bootstrapped 2026-10-20: see [[many-chickens-many-eggs/index|the project notes]]. Repo: `/Users/mbryant/bed/many-chickens-many-eggs`.

Wordle-style app/site. Once a day, AI generates a new HTML/CSS/JS puzzle to solve, seeded with example puzzles already in my head. Set time limit to solve; if time runs out, it reveals the answer/how to solve it.

Core mechanic: app self-improves, targeting ~70% success rate — but uniformly distributed across puzzles, not 70% trivial + 30% impossible. Each individual puzzle should feel calibrated to roughly a 70% chance of success, sampled across instances, not just true in aggregate. Needs a generate -> track outcome -> adjust generation loop.

Puzzle style: small graphical/interactive puzzles, often solvable multiple ways, exercising real browser APIs/behaviors so solving teaches how browsers work.
- Ball bounces down a set of stairs on screen; click the right stair, it opens, ball falls through — win.
- Resize the browser viewport to a specific size/aspect ratio to trigger success (needs distinct behavior for mobile vs desktop).
- Open the app in a specific browser — e.g. does nothing in Chrome, succeeds in Safari — exploits browser/API quirks.

Anti-cheese: HTML/CSS/JS minimized/obfuscated so you can't just view-source or dump the page into an LLM for the answer. Always provide hints (progressive, not just the raw solution) so it stays solvable without being a free pass.

Open questions:
- Scoring/telemetry pipeline for tuning difficulty toward 70%.
- Hint system design — how many hints, how/when triggered.
- How puzzles actually get generated (LLM-authored templates? parameterized puzzle library?).
- Obfuscation approach that survives inspection but stays fair.
