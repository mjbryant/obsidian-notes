# copper-test
**Description:** End-to-end testing of all three input modes (URL, image, text) through the full pipeline
**Started:** 2026-06-20

## Context

Integration testing only — unit tests for individual components belong in those tasks. This task is about sending real Slack messages and verifying a real file appears in Obsidian.

**Prerequisites:** All of [[bold-channel]], [[quiet-tunnel]], [[crisp-server]], [[amber-extract]], [[gentle-writer]] done.

**Test cases:**

### 1. URL — well-known recipe site
- Send a message with a URL from a site known to have JSON-LD (e.g. seriouseats.com, bonappetit.com)
- Verify: bot replies with recipe title within ~15s; file appears in Obsidian `recipes/`; YAML parses cleanly; ingredients have correct units from vocabulary

### 2. URL — site without JSON-LD
- Find a recipe URL that does NOT have schema.org JSON-LD (personal blogs, older sites)
- Verify: Claude fallback fires; recipe is still extracted correctly; same checks as above

### 3. Image — photo of a printed recipe
- Take a photo of a recipe card or cookbook page, send to the bot
- Verify: Claude vision extracts it; all fields populated; ingredients parsed correctly

### 4. Image — handwritten recipe
- Harder OCR case. Good to know where the limits are.
- Verify: either succeeds or fails gracefully with a useful error message

### 5. Text — full recipe pasted
- Paste a recipe as plain text (copy/paste from a PDF or email)
- Verify: correctly identified as text mode, not URL mode; all fields extracted

### 6. Text — description only
- Send "a quick weeknight pasta with lemon and capers"
- Verify: bot generates a plausible recipe; Slack reply indicates it was generated (not extracted)

### 7. Conflict — duplicate title
- Send the same URL twice
- Verify: second send overwrites the first file (not creates `-2`); Slack reply confirms overwrite

### 8. Error — paywalled URL
- Send an NYT Cooking URL (paywalled)
- Verify: bot replies with a clear error, does not write a partial/empty file

**What to log here during testing:**
- Which test cases pass/fail
- Any edge cases discovered
- Performance: how long does each mode typically take?

## Session Log

### 2026-06-20 — Session 1
- Session started
- Initial context written
