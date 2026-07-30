# dac/ — the machinery folder

Everything the docs-as-code toolchain needs that is not content lives here, so
the repo root stays clean for authors. Nothing in this folder is rendered.

| Item | Purpose |
|---|---|
| `docx-build.yml` | Batch render config for `docx-build-all`. Paths inside are relative to the repo root |
| `org.yaml` | Organization identity for cover pages. Copy `org.yaml.example` and fill in your own |
| `logo.png` *(you add)* | Cover-page logo; picked up automatically at this path |
| `templates/` | Document templates — copy one to start a new document |
| `vale/styles/` | Prose lint styles and your vocabulary. See [Prose lint layout](#prose-lint-layout) below — the vocabulary path is easy to get wrong |
| `render-manifest.yaml` | Optional explicit render manifest for `docx_manifest.py`. The `docx-build.yml` autodetect flow is the default; use the manifest when you need per-document control |

## Prose lint layout

Vale reads `.vale.ini` at the repo root, which points `StylesPath` here.

```text
dac/vale/styles/
|-- DocOps/                        committed - the house style
|-- config/
|   `-- vocabularies/
|       `-- ArchitectureDocs/      committed - your terms live here
|           |-- accept.txt         words Vale should stop flagging
|           `-- reject.txt         only enforced if you enable Vale.Terms
|-- RedHat/                        gitignored - vale sync regenerates
|-- write-good/                    gitignored - vale sync regenerates
`-- ai-tells/                      gitignored - vale sync regenerates
```

**To teach Vale a word your team uses**, add it on its own line in
`dac/vale/styles/config/vocabularies/ArchitectureDocs/accept.txt` and commit
it. The folder name must match `Vocab =` in `.vale.ini`; rename both together
if you want a different one.

> **Do not create `dac/vale/styles/Vocab/`.** Vale 2 reads vocabularies from
> there and Vale 3 does not. Words placed in that folder are ignored with no
> error, so the file looks maintained while doing nothing, and a repo holding
> both copies drifts apart quietly.

Everything under `RedHat/`, `write-good/`, and `ai-tells/` is downloaded by
`vale sync` from the `Packages =` line in `.vale.ini`. Those are gitignored on
purpose — never edit them, because the next sync overwrites your changes.

A few files cannot move here because their tools require the repo root:
`devfile.yaml` (Dev Spaces), `.github/` (GitHub), `.vscode/` (VS Code), and the
dotfiles `.vale.ini`, `.markdownlint.json`, `.pre-commit-config.yaml` (editor
and hook auto-discovery). Everything else machinery-shaped belongs in here.
