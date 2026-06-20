# Recipe Format — chowder

Canonical format for all recipe files. Every engine that reads or writes recipes must conform to this spec.

## File layout

Each recipe is a single `.md` file: `recipes/<recipe-slug>.md`

- Slug is kebab-case, derived from the title at creation time, and **never changed** (it's the stable ID)
- File = YAML frontmatter (structured data for engines) + Markdown body (human-readable cooking guide)

## Frontmatter schema

```yaml
---
id: <string>          # kebab-case slug, matches filename
title: <string>
source: <string>      # URL, "original", or citation (e.g. "Marcella Hazan, Essentials...")
servings: <int>
prep_time: <int>      # minutes
cook_time: <int>      # minutes

tags:
  - <string>          # see tag vocabulary below

ingredients:
  - item: <string>    # canonical lowercase ingredient name (no prep notes here)
    amount: <number | null>
    unit: <string | null>   # see unit vocabulary below
    note: <string | null>   # prep/context note ("finely diced", "room temperature")
    optional: <bool>        # default false; Shopping Engine skips these by default

variations:
  - name: <string>
    substitutions:
      - from: <string>   # must match an ingredient `item` exactly
        to: <string>     # replacement description

history:
  added: <YYYY-MM-DD>
  last_made: <YYYY-MM-DD | null>
  times_made: <int>
  rating: <int | null>   # 1–5; set after cooking
---
```

## Markdown body

```markdown
## Instructions

Numbered steps. One action per step. Short sentences.

## Notes

Optional. Tips, make-ahead info, scaling notes, wine pairings, etc.
```

`## Instructions` is required. `## Notes` is optional. No other top-level sections.

## Unit vocabulary

Constrained to the following values. The Shopping Engine normalizes within dimensions for aggregation.

| Dimension | Units |
|-----------|-------|
| Volume    | `tsp`, `tbsp`, `cup`, `fl oz`, `pt`, `qt`, `l`, `ml` |
| Weight    | `oz`, `lb`, `g`, `kg` |
| Count / qualitative | `piece`, `clove`, `head`, `bunch`, `sprig`, `can`, `package`, `slice`, `pinch`, `strip` |

If no unit applies (e.g. "3 eggs"), set `unit: null` and put the count in `amount`.

## Tag vocabulary

Freeform, but prefer consistent terms. Established tags:

- **Meal type:** `breakfast`, `lunch`, `dinner`, `snack`, `dessert`, `side`
- **Dietary:** `vegetarian`, `vegan`, `gluten-free`, `dairy-free`, `low-carb`, `pescatarian`
- **Effort:** `weeknight`, `weekend`, `quick` (≤30 min total), `project`
- **Cuisine:** `italian`, `mexican`, `japanese`, `indian`, `american`, `mediterranean`, etc.
- **Format:** `soup`, `salad`, `pasta`, `grain`, `sandwich`, `stew`, `roast`, `bake`

## Ingredient naming rules

The `item` field is the Shopping Engine's join key — it must be canonical:

- Lowercase, singular: `garlic` not `Garlic` or `garlic cloves`
- No prep descriptors in `item` — those go in `note`: `item: parsley`, `note: fresh, chopped`
- No brand names unless truly irreplaceable
- Be specific enough to shop: `chicken thigh` not `chicken`; `whole milk` not `milk`

## Variations

Variations model ingredient substitutions only. They do not modify steps. If a variation requires meaningfully different steps, it is a separate recipe that may cross-reference the original via `source`.

## Example

See [[pasta-e-fagioli]] for a complete worked example.
