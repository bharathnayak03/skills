# Calorie Tracker MVP Reference

This reference captures a reusable implementation shape for a Hermes-operated calorie tracker. It is not a transcript; use it as a template for similar personal trackers.

## Project Shape

A successful MVP used:

- TypeScript
- `better-sqlite3` for local persistence
- `commander` for a Hermes-friendly CLI
- `vitest` for behavior tests
- SQLite database at `data/calorie-tracker.sqlite`

Recommended commands:

```bash
npm install
npm test
npm run build
npm run cli -- log --food egg --servings 2 --meal breakfast
npm run cli -- summary
npm run cli -- suggest
```

## Minimum CLI Surface

- `log` — add an entry.
- `summary` — show totals for a date.
- `suggest` — suggest next action/meal based on gaps.
- `foods` or equivalent — list static/common reference items.

For unknown foods, accept explicit per-serving nutrition JSON rather than blocking:

```bash
npm run cli -- log \
  --food "homemade dal" \
  --servings 1.5 \
  --meal lunch \
  --nutrition-json '{"caloriesKcal":180,"proteinG":9,"carbsG":25,"fatG":4,"fiberG":6,"sugarG":3,"sodiumMg":420,"potassiumMg":500,"calciumMg":45,"ironMg":2.5,"vitaminCMg":2}'
```

## Nutrition Fields Used

The MVP tracked:

- Calories: `caloriesKcal`
- Macros: `proteinG`, `carbsG`, `fatG`, `fiberG`, `sugarG`
- Micros: `sodiumMg`, `potassiumMg`, `calciumMg`, `ironMg`, `vitaminCMg`

## Correction Workflow

When the user corrects logged food details:

1. Identify the exact row by date/food/id.
2. Show or inspect the prior row.
3. Update the row in-place with corrected values and notes.
4. Re-run `summary` and `suggest` for that date.
5. Report the delta in human terms.

Examples of corrections that occurred:

- “Coconut quarter” was corrected from coconut meat to coconut water. This changed calories/fat substantially, so the summary needed to be recomputed.
- Coffee was corrected from “little milk” to about 200ml milk with little sugar. This increased calories, protein, calcium, and sugar.

## Estimation Practice

When the user gives approximate food descriptions:

- Choose a reasonable default serving size.
- Put the assumption in `notes`.
- Tell the user what assumption was used.
- Encourage corrections without requiring a full questionnaire.

Good note examples:

- `Corrected from coconut meat: coconut water, estimated 1 cup / 240ml`
- `Corrected: coffee with about 200ml milk and little sugar; assumed 1 tsp sugar`

## Git/Privacy Practice

Before initial commit, add:

```gitignore
node_modules/
dist/
coverage/
.env
*.log
.DS_Store

data/*.sqlite
data/*.sqlite-*
```

Then verify:

```bash
git check-ignore -v data/calorie-tracker.sqlite
```

Commit code and docs, not the personal database.
