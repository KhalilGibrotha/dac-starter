# DaC Starter — Documentation as Code

A drop-in starting point for a documentation-as-code repository. You write
Markdown with YAML front matter; the toolchain lints it, renders diagrams, and
generates styled Word documents. DOCX and PDF are build artifacts, never
hand-edited deliverables.

This starter stays public-safe: no organization-sensitive defaults, no
internal taxonomy, no environment-specific examples.

---

## How the System Fits Together

Two parts, one boundary. The **engine** is the published
[`dac-toolkit`](https://github.com/KhalilGibrotha/dac-toolkit) container image:
docx_builder, Pandoc, Mermaid CLI, Vale, markdownlint, pre-commit, and the
lint scripts, all baked in and versioned together. The **content repo** — this
one, or yours after adoption — holds documents plus a thin layer of
configuration. You never clone the toolkit; the workspace, CI, and hooks all
consume the same image.

```text
your-repo/
|-- devfile.yaml         One-click Dev Spaces workspace (must stay at root)
|-- .github/workflows/   CI: the same image, the same gates (must stay at root)
|-- .vale.ini, .markdownlint.json, .pre-commit-config.yaml
|                        Tool dotfiles (auto-discovery needs them at root)
|-- dac/                 THE MACHINERY FOLDER - config, templates, org identity
|   |-- docx-build.yml   What to scan, where exports land, which org.yaml
|   |-- org.yaml         Your organization identity for cover pages
|   |-- templates/       Copy one to start a document
|   `-- vale/styles/     Prose rules and your vocabulary accept list
|-- docs/ decisions/ governance/ initiatives/ patterns/ references/
|                        YOUR CONTENT - the only part that is yours to fill
`-- exports/             Generated DOCX (never edit; always regenerate)
```

Everything machinery-shaped lives in `dac/` so the root stays clean for
authors. The engine knows this layout: `docx-build-all` finds
`dac/docx-build.yml` on its own, and paths inside it are relative to the repo
root. See [dac/README.md](dac/README.md) for what each file does.

---

## Quick Start: New Repository

1. Create your repository from this template: open
   <https://github.com/KhalilGibrotha/dac-starter> and click **Use this
   template → Create a new repository**. To work from a clone instead:

   ```bash
   git clone https://github.com/KhalilGibrotha/dac-starter.git my-docs
   cd my-docs
   rm -rf .git && git init
   ```

2. Copy `dac/org.yaml.example` to `dac/org.yaml` and fill in your
   organization's name, department, and address. Nothing renders with your
   branding until this file exists — the template ships only the example, so
   covers fall back to plain text without it. Add a logo as `dac/logo.png`
   if you have one; it is picked up automatically.
3. Choose where the tools run — [Where the Toolchain Runs](#where-the-toolchain-runs)
   walks through each option. Every option gives you the same toolkit image.
   In Dev Containers and Dev Spaces, commands run bare; in a local container,
   prefix them with the `dac` shell function defined there. This guide writes
   both forms where they differ.
4. Copy a template from `dac/templates/` into `docs/`, fill in the front
   matter, and write.
5. Build. In Dev Spaces: **Terminal → Run Task → "Build changed documents"**,
   or run the command directly:

   ```bash
   docx-build-all          # Dev Spaces, or any host with the toolkit installed
   dac docx-build-all      # local container, using the function from step 3
   ```

   Your styled Word document lands in `exports/`.

   Builds are incremental, and the trigger is the document's `version:`
   field rather than its text. Edit a document and rebuild and it reports
   `Skipped: 1 (up to date)` — bump `version:` to publish the change, or
   pass `--force` to re-render everything regardless.

6. Record which engine revision you started from, once:

   ```bash
   dac-init                # Dev Spaces
   dac dac-init            # local container
   ```

   Your files are already in place, so this installs nothing. What it does
   is write `dac/.dac-manifest.json`, the provenance record that `dac-update`
   reads when you later pull in a newer engine revision. Skip it and
   `dac-update` has no baseline to compare against and will refuse to run.

Stuck? [TROUBLESHOOTING.md](TROUBLESHOOTING.md) covers the problems adopters
actually hit — workspace disconnects during commits, gates failing on existing
documents, builds that skip everything, missing cover logos.

## Quick Start: Existing Repository

First pick where the tools run — [Where the Toolchain Runs](#where-the-toolchain-runs)
covers every option, including the `dac` shell function the local-container
path needs. Then run `dac-init` from your repository root and it installs the
managed set for you:

```bash
dac-init                # Dev Spaces
dac dac-init            # local container
```

It never overwrites a file you already have, so your existing `.vale.ini` or
`.markdownlint.json` survives untouched; add `--dry-run` first to see exactly
what it would place. It also writes the `dac/.dac-manifest.json` provenance
record that `dac-update` needs later. Then follow steps 2, 4, and 5 above,
using the same prefix your environment needs.

To place the files by hand instead, copy: `devfile.yaml`, `.vale.ini`,
`.markdownlint.json`, `.pre-commit-config.yaml`, `.github/workflows/lint.yml`,
the whole `dac/` folder, and the content folders you plan to use (or point
`dac/docx-build.yml` `scan:` at folders you already have).

Point `scan:` in `dac/docx-build.yml` at the folders that actually exist in
your repo. The shipped list assumes the full starter layout, and any missing
folder produces a `scan path not found` warning on every build.

Nothing else is required — the tools come from the image.

---

## Where the Toolchain Runs

Four options, one image. Pick whichever fits how you work; nothing else in
this guide changes.

**Locally with podman or docker** — nothing to install beyond the container
runtime. The tools live in the image, so each command runs in a throwaway
container. Define this shell function once per session and prefix each
command with `dac` — for example `dac docx-build-all`:

```bash
dac() { podman run --rm -v "$PWD:/work:Z" -w /work \
  ghcr.io/khalilgibrotha/dac-toolkit:latest "$@"; }
```

**Windows PowerShell** parses a bare `$PWD:` as a drive reference, so the
braces matter:

```powershell
function dac { podman run --rm -v "${PWD}:/work:Z" -w /work `
  ghcr.io/khalilgibrotha/dac-toolkit:latest @args }
```

**Windows Git Bash** rewrites container paths unless you disable that, so
use:

```bash
dac() { MSYS_NO_PATHCONV=1 podman run --rm \
  -v "$(pwd -W)://work:z" -w //work \
  ghcr.io/khalilgibrotha/dac-toolkit:latest "$@"; }
```

If podman appears to hang or refuses to connect, start its VM first:
`podman machine start`.

**In Visual Studio Code with Dev Containers** — the same image, with the
editor inside it: linting as you type, and a terminal where commands run
bare with no `dac` prefix. Install the Dev Containers extension, open the
repository, and choose **Reopen in Container**;
`.devcontainer/devcontainer.json` handles the rest. To use podman instead of
Docker Desktop, set this once in your **User** settings:

```jsonc
{
  "dev.containers.dockerPath": "podman"
}
```

**In OpenShift Dev Spaces** — if your organization runs it, paste the repo's
Git URL into **Import from Git** on your Dev Spaces dashboard (ask your
platform team for the URL). `devfile.yaml` starts the workspace on the same
image with every tool already on your PATH, so commands run bare, with no
`dac` prefix.

On a **new Dev Spaces workspace**, warm the commit hooks once before you
start working:

```bash
pre-commit install-hooks
```

This prepares the hook environments. `devfile.yaml` keeps the cache on the
persistent volume, so it happens once rather than after every restart.
Skipping it does not break anything; it just moves the cost onto your first
commit. If a commit ever has to go through while the workspace is
struggling, `git commit --no-verify` bypasses the hooks — CI still gates the
branch.

---

## Building Documents

`docx-build-all` is incremental. After every successful run it saves a render
index (a fingerprint of each source document); the next run compares against
that index and rebuilds only what changed. Three commands cover daily use:

```bash
docx-build-all              # build only documents changed since the last run
docx-build-all --dry-run    # show what would build, render nothing
docx-build-all --force      # rebuild everything, ignore the index
```

Single document, when you want one file fast:

```bash
docx-build docs/my-doc.md --org dac/org.yaml --output exports/my-doc.docx
```

## Commits, Pull Requests, and CI

The working loop is ordinary GitHub flow, with the gates doing the policing:

1. **Branch** from your base branch (`feature/<short-description>`, or
   whatever convention your team uses — see
   [CONTRIBUTING.md](CONTRIBUTING.md) for a recommended model).
2. **Write and build.** Edit documents, run `docx-build-all`, review the
   generated DOCX in `exports/`.
3. **Commit.** Hooks are opt-in: run `pre-commit install` once per clone and
   the blocking gates below run on each commit. Skip them any time with
   `git commit --no-verify` — nothing is lost, because CI runs the same
   gates on the pull request regardless. The hooks exist to catch problems
   at your desk instead of in review.
4. **Open a pull request.** CI runs every gate inside the same toolkit image
   your workspace uses — same tools, same versions, so a clean local run
   means a clean CI run. Results appear as checks on the PR.
5. **Merge when the checks pass.** The blocking gates are the merge bar;
   Vale is advice, not a bar.

The same gates run in all three places — editor (as you type), pre-commit
hook, and CI — all from the same image:

| Gate | What it catches | Blocks the merge? |
|---|---|---|
| Secret scan (gitleaks) | Tokens, keys, credentials — anywhere in history | Yes |
| Front matter validation | Missing or invalid YAML metadata | Yes |
| Markdown lint | Structural Markdown problems | Yes |
| Mermaid preflight | Diagram syntax the renderer would reject (`flowchart`, not `graph`) | Yes |
| Encoding artifacts | Smart quotes, no-break spaces, mojibake | Yes |
| Vale | House prose style, plus AI-prose tells (`ai-tells`) | No — advisory |

Each CI job writes its result to the run's summary page — open the workflow
run and the summary reads as a report: files checked, findings counted,
secrets scanned. The secret scan runs on the runner rather than in the
toolkit image (gitleaks is not part of the image) and scans the full history,
not just the diff.

Advisory means judgment: read Vale's findings in the job log and act on the
ones that improve the document. A flagged construct that carries real
meaning stays.

**Recommended once your team is on board:** protect your base branch and
mark the five blocking gates as required status checks
(**Settings → Branches → Branch protection**). That turns "merge when the
checks pass" from a habit into a guarantee, and it is what makes auto-merge
safe to enable if your team wants it. Keep Vale off the required list — that
is what keeps advisory honest.

## Diagrams: Mermaid First

Write diagrams as ```mermaid fences inside the document. GitHub renders
Mermaid natively, so anyone reading the repo in a browser sees the diagram,
not a code block — and the build renders the same source into the DOCX. Other
diagram languages (PlantUML, Graphviz, D2, ...) do not render on GitHub:
commit a pre-rendered SVG or PNG under `diagrams/exported/` and reference it
as an image instead.

---

## Owning Your Copy

This starter is a paved road, not a cage. The configs and CI workflow in your
repo are yours: tune the Vale rules, add vocabulary to the accept list, adjust
the scan list. The heavy machinery updates centrally — bump the image tag in
`devfile.yaml` and `.github/workflows/lint.yml` to pick up new tool versions,
and pin a digest when you want reproducibility.
