# amber-extract
**Description:** Build the recipe extractor: URL mode (JSON-LD + Claude fallback), image mode (Claude vision), text mode (Claude)
**Started:** 2026-06-20

## Context

The intelligence layer. Takes a raw input (URL, image bytes, or text) and returns a complete, formatted recipe string ready to be written to disk. The output must conform exactly to [[recipe-format]].

**Three modes:**

### URL mode
1. Fetch the page HTML (`httpx` or `requests` with a real User-Agent)
2. Look for `<script type="application/ld+json">` blocks containing `@type: Recipe` — this is schema.org structured data that most major recipe sites embed (Serious Eats, NYT Cooking, Bon Appétit, AllRecipes, etc.). Parse it directly if found — faster and more accurate than asking Claude to read HTML.
3. If no JSON-LD found, pass the HTML (or a cleaned text version) to Claude and ask it to extract the recipe.
4. Either way, pass the extracted data through a Claude call to normalize it into the canonical format — even JSON-LD data needs to be reshaped to our schema.

### Image mode
1. The image bytes are already downloaded by [[crisp-server]] (using the bot token to call `files.info` then fetch the private URL).
2. Send to Claude via the vision API (`claude-sonnet-4-6` or better, with `image/jpeg` or `image/png` media type).
3. Prompt: extract the recipe from this image and format it per the spec.
4. Handle multi-page recipes (user sends multiple images) — concatenate into one Claude call if possible, or process sequentially and merge.

### Text/description mode
1. Pass the raw text to Claude.
2. If it's a full recipe (ingredients + steps), extract and format it.
3. If it's a description ("a weeknight pasta with brown butter and sage"), generate a plausible recipe from it.
4. The Slack reply should indicate whether Claude extracted vs. generated, so the user knows to verify generated ones.

**Prompt strategy:**
- System prompt includes the full recipe-format spec (the YAML schema + unit/tag vocabularies)
- Instruct Claude to output *only* the raw file content (YAML frontmatter + Markdown body), no commentary
- Set `today's date` in the prompt so `history.added` is correct
- Validate the output parses as valid YAML before returning it

**Output:** a string — the full contents of the `.md` file to be written

**Error cases to handle:**
- URL is paywalled or returns 403 → tell the user
- Image is not a recipe (random photo) → Claude will say so; surface that to the user
- Claude returns malformed YAML → retry once with a stricter prompt; if still bad, surface the raw output and ask user to fix

**Depends on:** Anthropic API key, [[recipe-format]] spec
**Called by:** [[crisp-server]]

## Session Log

### 2026-06-20 — Session 1
- Session started
- Initial context written
