# Skills

This repository is a generic collection of useful skills I want to keep, reuse, and share.
Some skills are written specifically for Hermes Agent, while others may simply document
workflows, patterns, and reusable operating knowledge that I find valuable.

At the moment, this repo contains:

- [`personal-data-tracker-app`](#personal-data-tracker-app)
- [`headless-ui-kit-builder`](#headless-ui-kit-builder)
- [`chat-embeddable-web-apps`](#chat-embeddable-web-apps)

## Repository structure

```text
skills/
  <skill-name>/
    SKILL.md
    agents/
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

### `headless-ui-kit-builder`

**Description**

> Build or retrofit reusable React UI kits with Headless UI, shared tokens,
> wrapper primitives, Storybook coverage, and migration guidance.

**Use this for**

- creating a new UI kit for a React app
- standardizing inconsistent buttons, fields, overlays, and navigation primitives
- building accessible Headless UI wrappers
- adding Storybook coverage for design-system primitives
- planning incremental migration from one-off UI patterns to shared components

**Highlights**

- audits the existing UI layer before proposing primitives
- maps where Headless UI should be used and where native elements are enough
- emphasizes token reuse and multi-surface styling
- treats the UI kit as a context-compression layer that reduces future prompt size
- includes rollout and verification guidance for Storybook, tests, and migration

**Path**

- `skills/headless-ui-kit-builder/SKILL.md`


### `chat-embeddable-web-apps`

**Description**

> Publish produced HTML as a temporary chat-openable UI by serving it locally, exposing it with Cloudflare Tunnel, and returning a verified URL.

**Use this for**

- generated HTML UIs that need to be opened from Discord, Telegram, or WhatsApp
- one-off browser previews created by an agent
- publishing an already-produced `index.html` through Cloudflare Tunnel
- returning a verified `trycloudflare.com` URL to the user

**Highlights**

- treats the HTML as a generic browser artifact, not as a specific app type
- avoids giving users localhost links that cannot work from chat clients
- standardizes on `cloudflared tunnel --url` for publishing
- requires local and public marker verification before sharing the URL
- clearly labels Cloudflare Tunnel links as temporary previews

**Path**

- `skills/chat-embeddable-web-apps/SKILL.md`

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

For generic reusable skills in this repository, use the folder contents directly or adapt
them to the target agent runtime as needed.

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
