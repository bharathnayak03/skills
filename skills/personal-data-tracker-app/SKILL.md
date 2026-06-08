---
name: personal-data-tracker-app
description: "Build Hermes-driven personal data trackers: SQLite persistence, CLI interface, summaries, suggestions, privacy-safe git."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [personal-tracking, sqlite, cli, hermes-interface, privacy, health-data]
    related_skills: [test-driven-development, writing-plans, github-workflows]
    created_by: agent
---

# Personal Data Tracker Apps

Use this when building or extending a Hermes-operated tracker for recurring personal logs: calories, medicines, habits, symptoms, finances, workouts, sleep, or similar domains.

## Core Pattern

1. **Use Hermes as the first interface.**
   - The user should be able to say natural-language entries like “I ate 2 eggs” or “log today’s medicine refill.”
   - Hermes translates the request into deterministic CLI/database operations.
   - Keep a CLI command available so future automation, cron jobs, Telegram, or voice messages can call the same code path.

2. **Persist user data in SQLite.**
   - Use a stable local DB path such as `data/<tracker>.sqlite` inside the project.
   - Store structured rows rather than only free text.
   - Include timestamps, notes, and source/assumption fields when the user gives approximate data.

3. **Separate code from private data.**
   - Commit source code, tests, README, package files, schemas, and seed/static reference data.
   - Do **not** commit personal logs or SQLite runtime databases.
   - Add `.gitignore` patterns before the first commit:
     ```gitignore
     data/*.sqlite
     data/*.sqlite-*
     .env
     node_modules/
     dist/
     coverage/
     ```

4. **Support corrections as first-class operations.**
   - Users will correct earlier assumptions (“that was coconut water, not coconut meat”).
   - Locate the row, update it in-place, and show the before/after impact on totals.
   - Keep notes honest: record assumptions like “estimated 200ml milk + 1 tsp sugar.”

5. **Produce summaries and suggestions.**
   - Summary: totals, category breakups, and target percentages.
   - Suggestions: next action based on gaps, not generic advice.
   - Clearly label estimates and uncertainty.

## TDD Workflow

For new tracker features:

1. Write a failing test for the desired behavior.
2. Run the specific test and confirm it fails for the expected reason.
3. Implement the minimal code.
4. Run the specific test, then the full suite.
5. Build the project and exercise the CLI with real commands.

Useful behavior tests:

- Logging an entry persists a row in SQLite.
- Summary aggregates the correct fields for a date.
- Suggestions respond to gaps against targets.
- Unknown/custom inputs can include explicit nutrition/metadata.
- Corrections update the intended row and change the summary.

## Schema Guidance

Every personal tracker should include:

- `id INTEGER PRIMARY KEY AUTOINCREMENT`
- domain-specific item fields (`food`, `medicine`, `habit`, etc.)
- `servings` / `quantity` / `amount` when relevant
- `category` or `meal` when relevant
- `occurred_at` or domain-specific timestamp
- `notes TEXT`
- `source TEXT` or `confidence` for common/static/custom/estimated data
- fields needed for aggregation, stored as numbers where possible

## Privacy and Git Pitfalls

- Never include the SQLite DB in the initial commit unless the user explicitly asks to version personal data.
- Check `git check-ignore -v data/<db>.sqlite` after adding `.gitignore`.
- If the user asks to initialize git, make the first commit include code/tests/docs only.
- If cloud sync is needed, prefer explicit export/backup workflows rather than silently committing logs.

## Reference Material

- `references/calorie-tracker-mvp.md` captures a concrete calorie-tracker implementation pattern, correction workflow, and CLI examples from a successful session.
