# Contributing

This repository is a public architecture documentation starter. Contributions
should improve the generic workflow, template quality, and public-safe default
configuration.

## Contribution Rules

- Keep templates generic and reusable.
- Do not add organization-specific, customer-specific, or regulated internal
  information.
- Prefer changes that help multiple documentation teams adopt the starter with
  minimal rewiring.
- Keep generated artifacts out of the repo unless the change is specifically
  about sample output handling.

## Branching

Recommended branch model:

```text
main        stable default branch
develop     integration branch for upcoming changes
feature/*   short-lived feature branches
fix/*       short-lived fix branches
```

If your hosting platform supports template repositories, mark this repo as a
template after the initial baseline is stable.

For day-to-day work, branch from the team base branch with the Jira key first:

```bash
git checkout main
git pull
git checkout -b feature/ARCH-1234-short-description
```

If your team integrates through `develop`, substitute `develop` for `main`.

Recommended examples:

- `feature/ARCH-1234-reference-zone-model`
- `feature/ARCH-1288-template-cleanup`
- `fix/ARCH-1402-mermaid-frontmatter-lint`

## Before Opening a Pull Request

Run the local checks:

```bash
python3 scripts/lint-frontmatter.py --path .
python3 scripts/lint-mermaid.py --path .
python3 scripts/lint-prose.py --no-exit
markdownlint .
make docx-validate
```

## Pull Requests

1. Describe what changed and why.
2. Call out any starter-template taxonomy changes explicitly.
3. Note whether the change should also be mirrored into downstream content
   repositories.
4. Reference the Jira ticket or work item in the PR description.

## Suggested Commit Style

Use the Jira key at the front of the commit subject:

```text
ARCH-1234 Add manifest-driven content repo entrypoints
ARCH-1288 Update guide template wording
ARCH-1402 Fix Mermaid lint examples
```

## Code of Conduct

This project follows the [Code of Conduct](CODE_OF_CONDUCT.md).
