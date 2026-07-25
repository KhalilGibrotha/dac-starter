# templates

Reusable document skeletons. Copy a template to its working location, rename
it to match the document you are creating, complete the front matter, and
replace all placeholder content.

## How to Use

1. Identify the `doc_type` for the document you are creating.
2. Copy the matching template file to the correct working folder.
3. Rename using the standard convention: `<doc_type>_<domain>_<descriptor>.md`.
4. Complete YAML front matter and replace all placeholder text.
5. Render through `dac-toolkit` when you are ready to generate DOCX output.

Meeting notes are the exception to the naming rule. Use the date-based naming
guidance in `template_meeting-notes.md`.

## Available Templates

| Template File | doc_type | Class | Suggested Location |
|---|---|---|---|
| `template_architecture-overview.md` | `overview` | Concept | `docs/` or `initiatives/<name>/` |
| `template_gap-analysis.md` | `gap-analysis` | Concept | `docs/` or `initiatives/<name>/` |
| `template_sad.md` | `sad` | Concept (all views) | `docs/` or `initiatives/<name>/` |
| `template_solution-concept.md` | `overview` or `gap-analysis` | Concept | `initiatives/<name>/` |
| `template_proposal.md` | `proposal` | Decision input | `decisions/proposed/` |
| `template_adr.md` | `adr` | Governance artifact | `decisions/proposed/` |
| `template_pattern.md` | `pattern` | Procedure | `patterns/<subdomain>/` |
| `template_checklist.md` | `checklist` | Procedure | `governance/` or `patterns/` |
| `template_runbook.md` | `runbook` | Procedure | `docs/` or `initiatives/<name>/` |
| `template_rca.md` | `rca` | Procedure | `governance/` or `incidents/` |
| `template_reference-sheet.md` | `reference` | Reference | `references/` or `initiatives/<name>/` |
| `template_release-notes.md` | `release-notes` | Reference | `docs/` or repo root |
| `template_standard.md` | `standard` | Directive reference | `governance/` |
| `template_policy.md` | `policy` | Directive concept | `governance/` |
| `template_guide.md` | `guide` | Tutorial | `docs/` |
| `template_meeting-notes.md` | informal | Informational capture | `notes/` or `initiatives/<name>/` |
| `template_automation-request.md` | `request` | Intake | `governance/` or repo root |
| `template_automation-spec.md` | `spec` | Structured reference | `docs/` or `initiatives/<name>/` |
| `template_automation-discovery.md` | `checklist` | Discovery procedure | `docs/` or `initiatives/<name>/` |
| `template_architecture-idea-brief.md` | informal | Early concept capture | `notes/` |

## Choosing Between Types

| If you need to... | Use |
|---|---|
| Explain what something is and why it matters | `overview` |
| Compare current versus target state across a domain | `gap-analysis` |
| Document all architectural views of a system | `sad` |
| Propose an architectural change for review | `proposal` |
| Record an architectural decision | `adr` |
| Document a repeatable design-time approach | `pattern` |
| Document operational execution steps | `runbook` |
| Create a pre/post-task verification list | `checklist` |
| Document a post-incident review | `rca` |
| Provide structured lookup data | `reference` |
| Document release history | `release-notes` |
| Set mandatory technical requirements | `standard` |
| Establish intent and accountability | `policy` |
| Onboard someone to a platform or workflow | `guide` |
| Capture a meeting's decisions and actions | `meeting-notes` |

## Front Matter Stubs

The `front-matter/` subfolder contains reusable YAML stubs.
Use them when you want to start from a smaller fragment rather than a full
template file.

