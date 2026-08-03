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

This repository is trunk-based:

```text
main        stable default branch; PRs merge here
feature/*   short-lived feature branches, deleted on merge
fix/*       short-lived fix branches, deleted on merge
docs/*      documentation-only changes
```

Adopters do not have to copy that. A documentation repo with a review or
publication step often wants a long-lived integration branch as well:

```text
master      published; what stakeholders receive
develop     integration branch; drafts accumulate here
```

Pick by whether "merged" and "published" are the same event for your team.
They are here — a merge to `main` is the release — so a second long-lived
branch would only be a place for work to go stale. Where publication is a
separate, slower step, `develop` earns its keep.

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
vale sync && vale docs/ decisions/ governance/ initiatives/ patterns/ references/
docx-build-all --dry-run
```

These are the same commands CI runs. The first four are the blocking gates —
clean here means clean on the pull request. Vale is advisory in CI and needs
`vale sync` once per fresh clone or workspace, because the downloaded style
packages are gitignored; without it Vale fails outright rather than
reporting findings. Read its findings and act on the ones that improve the
document — a locally "failing" Vale run does not block the merge.

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
