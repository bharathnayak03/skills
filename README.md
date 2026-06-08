# Skills

This repository is a generic collection of useful skills I want to keep, reuse, and share.
Some skills are written specifically for Hermes Agent, while others may simply document
workflows, patterns, and reusable operating knowledge that I find valuable.

At the moment, this repo contains one Hermes-compatible skill:

- [`personal-data-tracker-app`](#personal-data-tracker-app)

## Repository structure

```text
skills/
  <skill-name>/
    SKILL.md
    references/
    templates/
```

## Current skills

### `personal-data-tracker-app`

**Description**

> Build Hermes-driven personal data trackers: SQLite persistence, CLI interface,
> summaries, suggestions, privacy-safe git.

**Use this for**

- calorie trackers
- medicine/refill trackers
- habit and symptom trackers
- workout, sleep, finance, or other personal logging apps
- Hermes-first tools that accept natural language and persist structured data

**Highlights**

- uses Hermes as the primary interface for natural-language logging
- keeps deterministic CLI/database operations behind the scenes
- stores structured tracker data in SQLite
- treats edits/corrections as first-class operations
- emphasizes summaries, suggestions, and explicit uncertainty handling
- keeps private runtime data out of git by default

**Path**

- `skills/personal-data-tracker-app/SKILL.md`

## Using this repo with Hermes Agent

Add this repository as a tap:

```bash
hermes skills tap add bharathnayak03/skills
```

Install a skill from this repo:

```bash
hermes skills install bharathnayak03/skills/<skill-name>
```

Example:

```bash
hermes skills install bharathnayak03/skills/personal-data-tracker-app --yes
```

## Publishing updates

When I add or update skills here, the workflow is simple:

```bash
git add .
git commit -m "Update skills"
git push
```

## Adding more skills

1. Create a new folder under `skills/` named after the skill.
2. Add a `SKILL.md` plus any supporting `references/`, `templates/`, or `scripts/` files.
3. Update this README so the catalog stays current.

As the repository grows, it can hold both Hermes skills and other reusable skill-like
assets, but the current install examples above apply to the Hermes-compatible skills in
this repo.
