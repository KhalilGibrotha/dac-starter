# Documentation-as-Code — Claude Instructions

This repo authors Markdown; the dac-toolkit container image lints and renders
it. Generated DOCX in `exports/` is a build artifact — regenerate it, never
edit it.

## How This Repo Is Built

- **Engine:** `ghcr.io/khalilgibrotha/dac-toolkit`. Every tool (`docx-build`,
  `docx-build-all`, the lint scripts, Vale, markdownlint-cli2, pre-commit) is
  an image command, available in the Dev Spaces workspace and CI. Never clone
  the toolkit repo.
- **`dac/` is the machinery folder:** build config, org identity, templates,
  Vale styles. The repo root keeps only files their tools require there
  (devfile, `.github/`, `.vscode/`, dotfiles).
- **Content folders** (`docs/`, `decisions/`, `patterns/`, ...) are yours.
- `dac-init` installs missing starting points; it never overwrites existing
  files. Tune the configs freely — they are yours to own.

## Golden Rules

1. Output Markdown only. Build DOCX only when asked.
2. Every formal document: YAML front matter, then a one-sentence purpose
   statement directly after the title.
3. Never manually number headings — the builder auto-numbers.
4. One primary type per document: concept, procedure, reference, tutorial, or
   ADR. If it mixes, split and link.
5. File naming: `<doc_type>_<domain>_<descriptor>.md` — lowercase, hyphens
   inside segments, underscores between them.
6. Mermaid fences use `flowchart`, never `graph` — the lint gate rejects
   `graph`. No space between the backticks and `mermaid`.
7. Start new documents from `dac/templates/`.

## Document Types

| Type | `doc_type` values | Home |
|---|---|---|
| concept | `overview`, `gap-analysis`, `sad`, `proposal` | `docs/`, `initiatives/` |
| procedure | `pattern`, `checklist`, `runbook` | `patterns/`, `governance/` |
| reference | `reference`, `standard`, `spec` | `references/`, `governance/` |
| tutorial | `guide` | `docs/` |
| decision | `adr` | `decisions/proposed/` |

Concepts explain why; procedures give steps with no embedded explanation;
references optimize for scanning. Link between types instead of mixing them.

## Front Matter

```yaml
---
title: "Full Document Title"
doc_type: "<approved type>"
domain: "<subject area>"
department: "Architecture"
owner: "Owning Team"
status: "Draft"
version: "0.1"
date: "YYYY-MM-DD"
author: "Author Name"
---
```

`audience`, `related_docs`, `revision_history`, and `published_url` are
optional but encouraged.

## Build Commands

```bash
docx-build-all              # incremental: only documents changed since last run
docx-build-all --dry-run    # show what would build
docx-build-all --force      # rebuild everything
docx-build doc.md --org dac/org.yaml --output exports/doc.docx   # one file
```

## Lint Gate — run before pushing (mirrors CI one for one)

| Command | CI job |
|---|---|
| `lint-frontmatter.py --path .` | Front matter validation |
| `lint-mermaid.py --path .` | Mermaid preflight |
| `lint-encoding.py --path .` | Encoding artifacts |
| `markdownlint-cli2 "**/*.md" "!dac/vale" "!exports" "!archive"` | Markdown lint |
| `vale docs/ decisions/ patterns/ references/` | Vale (advisory) |

If a CI job is added or removed, update this table in the same commit — a
local gate that is a subset of CI produces green local runs and red PRs.

## Writing Quality

Vale includes the `ai-tells` style (AI-prose detection, advisory). It catches
vocabulary tells; you own structure:

- Vary sentence and paragraph length; let some be short.
- Prefer "is" over "serves as / represents"; cut filler transitions ("it's
  worth noting", "importantly") and marker words (delve, leverage, seamless).
- No "it's not X, it's Y" as a crutch; at most one rule-of-three in a row.
- Em dashes sparingly. Don't open every bullet with a bolded phrase.
- Advisory means judgment: a flagged construct that carries real meaning
  ("rollback is a promotion, not a rebuild") stays.

## In This Starter Repo Itself

Keep all examples generic — no organization names, hostnames, or identities.
Teams add their own in `dac/org.yaml`, which is never committed here.
