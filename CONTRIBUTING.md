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

Run the local checks. This repository has no `scripts/` directory — every
tool comes from the toolkit image, so commands run bare in Dev Spaces or a
Dev Container, or through the `dac` shell function defined in the README
under "Where the Toolchain Runs":

```bash
lint-frontmatter.py --path .
lint-mermaid.py --path .
lint-encoding.py --path .
markdownlint-cli2 "**/*.md" "!dac/vale" "!exports" "!archive"
vale docs/ decisions/ governance/ initiatives/ patterns/ references/
docx-build-all --dry-run
```

These are the same commands CI runs, so passing locally means passing on the
pull request. Vale is advisory — read its findings, act on the ones that
improve the document.

## Pull Requests

1. Describe what changed and why.
2. Call out any starter-template taxonomy changes explicitly.
3. Note whether the change should also be mirrored into downstream content
   repositories.
4. Reference the work item in the PR description, if your team tracks work
   in a ticketing system.

## Suggested Commit Style

If your team uses a tracker, put its key at the front of the commit subject:

```text
ARCH-1234 Add manifest-driven content repo entrypoints
ARCH-1288 Update guide template wording
ARCH-1402 Fix Mermaid lint examples
```

**Why key-first matters if your team uses Jira:** the GitHub-Jira
integration associates work automatically by matching the issue key. A
branch whose name **starts with the key** (`ARCH-1234-short-description`)
is linked to the issue, and its pull request and commits follow — no manual
linking. The key must lead; a key buried mid-name is not reliably matched.
Teams without Jira lose nothing by keeping the convention, and gain the
linkage for free if they adopt it later.

## Code of Conduct

This project follows the [Code of Conduct](CODE_OF_CONDUCT.md).
