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
3. Choose where the tools run. Both give you the same toolkit image:

   **Locally with podman or docker** — nothing to install beyond the
   container runtime:

   ```bash
   podman run --rm -v "$PWD:/work:Z" -w /work \
     ghcr.io/khalilgibrotha/dac-toolkit:latest docx-build-all
   ```

   On Windows Git Bash, prefix the command with `MSYS_NO_PATHCONV=1` and
   write the mount as `-v "C:\path\to\repo://work:z" -w //work`.
   Without that, the path is rewritten and the run fails. If podman appears
   to hang or refuses to connect, start its VM first: `podman machine start`.

   **In OpenShift Dev Spaces** — if your organization runs it, paste the
   repo's Git URL into **Import from Git** on your Dev Spaces dashboard (ask
   your platform team for the URL). `devfile.yaml` starts the workspace on
   the same image with every tool ready.
4. Copy a template from `dac/templates/` into `docs/`, fill in the front
   matter, and write.
5. Build. In Dev Spaces: **Terminal → Run Task → "Build changed documents"**.
   Anywhere the toolkit is on your PATH:

   ```bash
   docx-build-all
   ```

   Your styled Word document lands in `exports/`.

6. Record which engine revision you started from, once:

   ```bash
   dac-init
   ```

   Your files are already in place, so this installs nothing. What it does
   is write `dac/.dac-manifest.json`, the provenance record that `dac-update`
   reads when you later pull in a newer engine revision. Skip it and
   `dac-update` has no baseline to compare against and will refuse to run.

## Quick Start: Existing Repository

Run `dac-init` from your repository root and it installs the managed set for
you:

```bash
podman run --rm -v "$PWD:/work:Z" -w /work \
  ghcr.io/khalilgibrotha/dac-toolkit:latest dac-init
```

It never overwrites a file you already have, so your existing `.vale.ini` or
`.markdownlint.json` survives untouched; add `--dry-run` first to see exactly
what it would place. It also writes the `dac/.dac-manifest.json` provenance
record that `dac-update` needs later. Then follow steps 2, 4, and 5 above.

To place the files by hand instead, copy: `devfile.yaml`, `.vale.ini`,
`.markdownlint.json`, `.pre-commit-config.yaml`, `.github/workflows/lint.yml`,
the whole `dac/` folder, and the content folders you plan to use (or point
`dac/docx-build.yml` `scan:` at folders you already have).

Point `scan:` in `dac/docx-build.yml` at the folders that actually exist in
your repo. The shipped list assumes the full starter layout, and any missing
folder produces a `scan path not found` warning on every build.

Nothing else is required — the tools come from the image.

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

## Checks

The same gates run in three places — editor (as you type), opt-in pre-commit
hook, and CI on every pull request — all from the same image:

| Gate | What it catches |
|---|---|
| Front matter validation | Missing or invalid YAML metadata |
| Markdown lint | Structural Markdown problems |
| Mermaid preflight | Diagram syntax the renderer would reject (`flowchart`, not `graph`) |
| Encoding artifacts | Smart quotes, no-break spaces, mojibake |
| Vale (advisory) | House prose style, plus AI-prose tells (`ai-tells`) |

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
