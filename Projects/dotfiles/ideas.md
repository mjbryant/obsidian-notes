# Ideas — dotfiles

## Feature ideas
<!-- New features to add. -->

### Machine bootstrapping
Cross-platform (Mac + Linux) way to declare what tools should be available on a machine (e.g. `fzf`, `ripgrep`). Running a bootstrap script on a new machine installs everything. Should handle the case where package names differ between Homebrew and apt/dnf.

## Improvements
<!-- Things that could be better. -->

- Improve session log goal capture: using the first user message doesn't reflect real usage — find a better heuristic (e.g. infer goal from the arc of the session, ask at session end, or use a summary model).
- Use structured prefixes for session log entries, e.g. `[repo; type]` — the brackets improve scannability compared to plain text prefixes.

## Technical debt
<!-- Code quality issues to address. -->

## Experiments
<!-- Things to try, spikes to run. -->
