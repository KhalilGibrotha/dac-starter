# Architecture Documentation Starter Guidance

This repository is a public-safe architecture documentation starter.
Markdown with YAML front matter is the authoring format. Generated DOCX output
is a build artifact.

## Golden Rules

1. Output Markdown by default.
2. Every formal document needs YAML front matter.
3. Never manually number headings.
4. Every formal document should begin with a one-sentence purpose statement.
5. Every document should have one primary type.
6. Use the standard file naming convention: `<doc_type>_<domain>_<descriptor>.md`
7. Keep public starter content generic and reusable.

## Document Types

| Type | `doc_type` values | Typical folders |
|---|---|---|
| concept | `overview`, `gap-analysis`, `sad`, `proposal`, `policy` | `docs/`, `initiatives/`, `governance/`, `decisions/` |
| procedure | `pattern`, `checklist`, `runbook`, `rca`, `request` | `patterns/`, `governance/`, `docs/` |
| reference | `reference`, `release-notes`, `spec`, `standard` | `references/`, `governance/`, `docs/` |
| tutorial | `guide` | `docs/` |
| decision | `adr` | `decisions/proposed/` |
| informational | meeting notes and early notes | `notes/`, `initiatives/` |

## Required Front Matter

Formal documents should normally include:

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

`audience`, `related_docs`, and `revision_history` are optional but encouraged.

## Writing Guidance

- Concepts explain what something is and why it matters.
- Procedures explain how to do something.
- References optimize for scanning and lookup.
- ADRs follow the ADR structure rather than the concept/procedure/reference split.
- Keep examples generic and organization-neutral in this starter repo.

## Diagrams

- Use inline Mermaid when the diagram belongs to one document.
- Use shared image assets when the same diagram is reused across documents.
- Do not put a space between the backticks and `mermaid`.

## DOCX Rendering

When you do want document output:

```bash
docx-build <file>.md --org dac/org.yaml --output exports/<file>.docx
```

Or use wrapper tooling from `dac-toolkit` for batch and manifest-based flows.
