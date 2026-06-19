# Ideas — dotfiles

## Feature ideas
### more structured work items
The flow is something like:
- Look at open work items somehow
- Join some persistent session, have it spawn another session for a particular work item
- That session figures out what the deliverables should be and starts work
- Progress is tracked in a persistent way, potentially checked in somewhere
- I can join the session to nudge it along if it gets stuck
- When it’s done, it summarizes the deliverables and gives me the option to accept/reject. 
- reject just keeps the session alive and it keeps going
- Accept executes the deliverables, closes the work item, and shuts down the session

### idea inbox
Figure out how to have a rambly input, probably voice, that ends up as an idea in a particular project

### Machine bootstrapping
Cross-platform (Mac + Linux) way to declare what tools should be available on a machine (e.g. `fzf`, `ripgrep`). Running a bootstrap script on a new machine installs everything. Should handle the case where package names differ between Homebrew and apt/dnf.

## Improvements
<!-- Things that could be better. -->

- Improve session log goal capture: using the first user message doesn't reflect real usage — find a better heuristic (e.g. infer goal from the arc of the session, ask at session end, or use a summary model).
- Use structured prefixes for session log entries, e.g. `[repo; type]` — the brackets improve scannability compared to plain text prefixes.

## Technical debt
<!-- Code quality issues to address. -->

## Experiments
- set up a persistent coordinator agent always running, waiting for input and coordinating other tasks. Example things I want it to be able to do: receive a text from me and take an action, like start another agent to perform a coding task. I really want to be able to see what’s going on and initiate tasks from my phone. Is this just OpenClaw?
