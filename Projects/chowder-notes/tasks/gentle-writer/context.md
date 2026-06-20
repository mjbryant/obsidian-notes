# gentle-writer
**Description:** Build the file writer: slug generation, conflict detection, write formatted recipe to Obsidian notes path
**Started:** 2026-06-20

## Context

The last step in the pipeline. Takes the formatted recipe string from [[amber-extract]] and writes it to disk as a `.md` file in the Obsidian recipes folder. Simple, but slug conflicts need to be handled gracefully.

**Target path:**
```
/Users/mbryant/Library/Mobile Documents/iCloud~md~obsidian/Documents/Michael/Projects/chowder-notes/recipes/<slug>.md
```
iCloud Drive syncs this to other devices automatically — no additional sync step needed.

**Slug generation:**
1. Take the `title` field from the parsed YAML frontmatter
2. Lowercase, strip punctuation, replace spaces with hyphens: `"Pasta e Fagioli"` → `pasta-e-fagioli`
3. Strip leading/trailing hyphens, collapse multiple hyphens

**Conflict handling:**
- If `<slug>.md` already exists, check if it's the same recipe (compare title):
  - Same title → overwrite (user is updating the recipe)
  - Different title → append `-2`, `-3`, etc. until free
- Log which case occurred; include it in the Slack reply

**What to return to [[crisp-server]]:**
- The final filename written
- The recipe title (for the Slack confirmation message)
- Whether it was a new file or an overwrite

**Slack confirmation message format:**
```
Saved: *Pasta e Fagioli* → recipes/pasta-e-fagioli.md
```
On error:
```
Couldn't save the recipe: <reason>. Raw content attached.
```

**Notes:**
- The recipes directory already exists at the target path (created during [[recipe-format]] implementation)
- File permissions: just a normal write, no special permissions needed
- No git commit needed — iCloud handles sync; git in the vault is for note-taking history, not this pipeline

## Session Log

### 2026-06-20 — Session 1
- Session started
- Initial context written
