# Troubleshooting

Problems adopters actually hit, in rough order of how often. Every entry here
was found the hard way, so the symptom is written the way you will experience
it rather than the way the tool reports it.

---

## The workspace disconnects while committing

**Symptom.** Dev Spaces shows "Disconnected. Attempting to reconnect" partway
through a commit, usually the first one after starting a workspace. Memory
climbs toward the workspace limit and the session dies.

**Cause.** Usually not the linting itself. `pre-commit` caches hook
environments under `$HOME`, and in Dev Spaces `$HOME` is ephemeral — only the
source mount is on the persistent volume. If your `.pre-commit-config.yaml`
gains a hook backed by a downloaded runtime (an npm-based `markdownlint-cli2`
hook is the usual one), every workspace restart re-installs it on your next
commit. Node compounds it: it sizes its heap from the *host node's* RAM rather
than the container limit, so it grows past the workspace quota.

The hooks shipped in this starter are all `language: system` and run tools
already baked into the image, so they install nothing. If you hit this, the
first thing to check is whether a hook with its own runtime has been added.

**Fix.** Both settings ship in `devfile.yaml`, so a workspace started from the
current file is already correct:

```yaml
env:
  - name: NODE_OPTIONS
    value: "--max-old-space-size=768"
  - name: PRE_COMMIT_HOME
    value: /projects/.pre-commit-cache
```

`PRE_COMMIT_HOME` puts the cache on the persistent volume, so an install
happens once rather than after every restart. On a workspace that predates
these, export both and warm the cache deliberately before working:

```bash
pre-commit install-hooks
```

**If you need a commit through immediately**, skip the hooks rather than fight
them. CI still gates the branch:

```bash
git commit --no-verify -m "..."
SKIP=<hook-id> git commit -m "..."   # or skip just the expensive one
```

**Still dying?** Commit in smaller batches, and close the Source Control panel
while you do — an unrendered panel does not re-scan. Then test the install
alone: `pre-commit install-hooks` on an otherwise idle workspace tells you
whether you have the headroom at all.

---

## Every document fails the gates on day one

Expected on an existing corpus, and worth understanding before you fight it.

**These hooks scan the whole repository, not just what you staged.** Every hook
in `.pre-commit-config.yaml` sets `pass_filenames: false` and ends in
`--path .`, so a one-line edit to one document runs the gates across
everything. That is deliberate for a clean repo — it catches a file broken by
something other than your edit — but it means adoption on an existing corpus is
not incremental by default.

Pick the approach that matches where you are:

- **Migrating an existing corpus.** Commit with `--no-verify` while you work
  through the backlog. CI still gates the branch, so nothing unreviewed reaches
  the default branch. Fix by category rather than by file — one rule at a time
  across all documents is far faster than one document at a time.
- **Want the gates to be genuinely incremental.** Change the hooks to
  `pass_filenames: true` and drop `--path .`, so pre-commit hands each script
  the staged files. The configs are yours; this is a supported change, not a
  workaround.
- **See the whole picture first:** `pre-commit run --all-files`. Read it as a
  survey to plan from, not a to-do list to clear in one sitting.
- **If a rule genuinely does not fit your content, change the rule.**
  `.markdownlint.json` for structure, `.vale.ini` for prose. A gate nobody can
  pass gets bypassed, and then it gates nothing.

---

## Documents render but nothing appears in `exports/`

Builds are incremental, and the trigger is the document's `version:` field, not
its text. Edit a document and rebuild and you will see
`Skipped: 1 (up to date)`.

Bump `version:` in the front matter to publish the change, or force a full
rebuild:

```bash
docx-build-all --force
```

---

## `Scanned: 0` and nothing renders

The scanner only picks up files with a YAML front matter block. If it reports
finding Markdown but scanning none, the documents need front matter — copy the
shape from `dac/templates/`.

One non-obvious cause: a **UTF-8 BOM** before the opening `---` used to hide a
document completely. Current versions strip it, but if you are on an older
image and a file looks invisible for no reason, check for a BOM. Windows
editors and PowerShell's `Set-Content -Encoding utf8` add one by default.

---

## The cover page has no logo

The file must be named exactly `logo.png`. Auto-detection looks for
`dac/logo.png`, then legacy `assets/logo/logo.png`, and anything else is passed
over **silently** — a missing logo is not an error, the cover just falls back
to styled text.

Set the key explicitly in `dac/docx-build.yml` so a wrong path becomes a
visible error instead of a quiet omission:

```yaml
logo: dac/logo.png
```

To confirm a logo actually embedded rather than trusting the build log:

```bash
python3 -c "import zipfile,sys; z=zipfile.ZipFile(sys.argv[1]); print([n for n in z.namelist() if 'media' in n])" exports/path/to/doc.docx
```

---

## Vale fails with "config/vocabularies/... does not exist"

Vale 3 reads vocabularies from `dac/vale/styles/config/vocabularies/<Name>/`.
The old Vale 2 location, `dac/vale/styles/Vocab/<Name>/`, is dead — entries
there are ignored with **no error**, so the file looks maintained while doing
nothing.

If both directories exist, delete the `Vocab/` one. A repo carrying both drifts
apart silently, which is exactly how a vocabulary ends up dozens of words
behind.

---

## Adding a word Vale keeps flagging

Add it on its own line in
`dac/vale/styles/config/vocabularies/<Name>/accept.txt` and commit it. Prefer
that over rewording when the flagged word is the precise technical term.

Note that `reject.txt` is **not enforced** by default: the stock `.vale.ini`
leaves the built-in `Vale` style out of `BasedOnStyles`, so `Vale.Terms` never
runs. Words added there do nothing. Enforce terminology with a `DocOps` rule
instead.

---

## Running the toolchain locally on Windows

Use the container. Two shell-specific forms, both verified:

```powershell
function dac { podman run --rm -v "${PWD}:/work:Z" -w /work `
  ghcr.io/khalilgibrotha/dac-toolkit:latest @args }
```

```bash
dac() { MSYS_NO_PATHCONV=1 podman run --rm \
  -v "$(pwd -W)://work:z" -w //work \
  ghcr.io/khalilgibrotha/dac-toolkit:latest "$@"; }
```

PowerShell needs the braces — a bare `$PWD:` parses as a drive reference. Git
Bash needs `MSYS_NO_PATHCONV=1`, or the mount path is rewritten and the run
fails with a confusing "workdir does not exist".

If podman appears to hang or refuses to connect, its VM is probably stopped:
`podman machine start`.

---

## Dev container: Vale exits with "style does not exist on StylesPath"

**Symptom.** Vale reports `E100 [loadStyles] Runtime error` and
`style 'ai-tells' does not exist on StylesPath`, then lints nothing.

**Cause.** Style packages are gitignored, so a fresh clone has none, and
`vale sync` cannot install them onto a Windows-backed bind mount: sync chmods
what it downloads, chmod is not permitted there, and it dies partway. Vale
stops on a declared-but-absent style rather than skipping it.

**Fix.** `postCreateCommand` handles this by syncing against a redirected
`StylesPath` inside the container filesystem and copying the result onto the
mount. If your container predates that, or sync was offline when the container
was created, run it by hand:

```bash
mkdir -p /tmp/vs
sed 's#^StylesPath.*#StylesPath = /tmp/vs/styles#' .vale.ini > /tmp/vs/.vale.ini
vale --config /tmp/vs/.vale.ini sync
cp -rn /tmp/vs/styles/. dac/vale/styles/
```

Do not reach for `vale-bootstrap.sh` in the image. It resolves `StylesPath`
from `.vale.ini`, but has CRLF line endings and silently falls back to the
legacy `.vale/styles` path — leaving styles exactly where Vale will not look.

---

## Dev container: "git failed. Is it installed?"

`pre-commit` reports git missing when git is plainly present. The real message
is underneath: git refuses the workspace for **dubious ownership**, because the
files belong to your host user while the container runs as uid 1001.

`postCreateCommand` adds the workspace to `safe.directory` before anything else
runs. To repair it by hand:

```bash
git config --global --add safe.directory "$PWD"
```

---

## Dev container: git shows files modified that you never touched

Windows checkouts made with `core.autocrlf=true` sit on disk with CRLF endings.
Your host git normalizes those away and reports a clean tree; the container's
git does not, so the same files look modified inside it.

This stays cosmetic until you commit, at which point the line endings become
real churn in the history. Check which side you are on first:

```bash
git config --get core.autocrlf
file <a-tracked-file>
```

The durable fix is a `.gitattributes` pinning text files to LF so both sides
agree. That renormalizes the whole repository, so treat it as its own change
rather than a drive-by.
