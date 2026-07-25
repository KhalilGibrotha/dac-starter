# dac/ — the machinery folder

Everything the docs-as-code toolchain needs that is not content lives here, so
the repo root stays clean for authors. Nothing in this folder is rendered.

| Item | Purpose |
|---|---|
| `docx-build.yml` | Batch render config for `docx-build-all`. Paths inside are relative to the repo root |
| `org.yaml` | Organization identity for cover pages. Copy `org.yaml.example` and fill in your own |
| `logo.png` *(you add)* | Cover-page logo; enable the `logo:` key in `docx-build.yml` |
| `templates/` | Document templates — copy one to start a new document |
| `vale/styles/` | Prose lint styles and the vocabulary accept list. Synced packages land here and are gitignored |
| `render-manifest.yaml` | Optional explicit render manifest for `docx_manifest.py`. The `docx-build.yml` autodetect flow is the default; use the manifest when you need per-document control |

A few files cannot move here because their tools require the repo root:
`devfile.yaml` (Dev Spaces), `.github/` (GitHub), `.vscode/` (VS Code), and the
dotfiles `.vale.ini`, `.markdownlint.json`, `.pre-commit-config.yaml` (editor
and hook auto-discovery). Everything else machinery-shaped belongs in here.
