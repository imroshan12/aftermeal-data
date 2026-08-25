# aftermeal-data

The food reference table AfterMeal downloads at launch. **This is live data** —
whatever is on `main` here reaches every installed copy of the app within a
GitHub Pages cache window (roughly 10 minutes). There is no App Review in
between.

Served at:

    https://imroshan12.github.io/aftermeal-data/starter-nutrition.json

## What the app does with it

It's a cold-start aid, nothing more. AfterMeal looks up nutrition in this
order:

1. **Your own history** — what you logged for that food before. Always wins.
2. **This table** — used only until you've logged a food once yourself.
3. Nothing.

The app never uploads anything here. This is a one-way, read-only download of
public reference data; no identifiers are sent.

## Format

```json
{
  "schemaVersion": 1,
  "foods": [
    { "name": "roti", "unit": "piece", "amount": 1, "kcal": 100, "proteinG": 3, "carbsG": 18 }
  ]
}
```

- `name` — lowercase; matched case-insensitively, and by substring
- `unit` — **must** be one of the units the shipped app knows:
  `piece, bowl, plate, cup, glass, ml, g, serving, tbsp`.
  A unit outside that list is silently dropped for that row.
- `amount` — what the numbers describe (e.g. `1` piece). Must be > 0; it's a divisor when scaling.
- `kcal`, `proteinG`, `carbsG`, `fatG` — all optional. Omit rather than guess;
  a null row simply doesn't prefill. `fatG` and `carbsG` are stored but not
  yet shown in the app.
- Bumping `schemaVersion` makes older apps **ignore the file entirely** and
  keep their bundled table. That's the intended escape hatch, not a mistake.

## Publishing

Don't edit this repo directly. Edit
`AfterMeal/Resources/starter-nutrition.json` in the app repo and run:

    scripts/publish-nutrition-table.sh

It validates first — units cross-checked against the actual `FoodUnit` enum
in the Swift source, plus duplicate names, negatives, and zero amounts — and
refuses to publish if anything fails.

## If you break it

The app is defensive, deliberately:

- One malformed row costs that row, not the table.
- A file that fails to parse entirely is ignored; the app keeps its last good
  cached copy, or the version bundled at release.
- A table that parses to zero rows is refused rather than swapped in.

So a bad publish degrades rather than breaks. Fix it and push again.
