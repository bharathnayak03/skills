# Hermes Public Skills Tap

This repository hosts multiple Hermes skills and is intended for **public sharing**.

The repo currently contains:

- `personal-data-tracker-app`

Add this repository as a Hermes skill tap:

```bash
hermes skills tap add bharathnayak03/hermes-skills-public
```

Install a skill from this tap:

```bash
hermes skills install bharathnayak03/hermes-skills-public/<skill-name>
# Example:
hermes skills install bharathnayak03/hermes-skills-public/personal-data-tracker-app
```

You can also install directly from GitHub:

```bash
hermes skills install bharathnayak03/hermes-skills-public/personal-data-tracker-app --yes
```

## Publishing updates

When you add or update skills in this repo, publish by committing then pushing to this
repository. Because this repo is the source of truth for community taps, consumers can
find the latest version from the public repo.

```bash
git add .
git commit -m "Update skills"
git push
```
