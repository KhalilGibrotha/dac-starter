# Troubleshooting

Problems adopters actually hit, in rough order of how often. Every entry here
was found the hard way, so the symptom is written the way you will experience
it rather than the way the tool reports it.

---

## The workspace disconnects while committing

**Symptom.** Dev Spaces shows "Disconnected. Attempting to reconnect" partway
through a commit, usually the first one after starting a workspace. Memory
climbs toward the workspace limit and the session dies.

**Cause.** Not the linting. `pre-commit` caches its hook environments under
`$HOME`, and in Dev Spaces `$HOME` is ephemeral — only the source mount is on
the persistent volume. Every workspace restart therefore re-runs `npm install`
for the markdownlint environment on your next commit. Node makes it worse: it
sizes its heap from the *host node's* RAM, not the container limit, so it will
happily grow past your quota.

**Fix.** Both settings ship in `devfile.yaml`, so a workspace started from the
current file is already correct:

```yaml
env:
  - name: NODE_OPTIONS
    value: "--max-old-space-size=768"
  - name: PRE_COMMIT_HOME
    value: /projects/.pre-commit-cache
```

On a workspace that predates them, export both in your shell, then warm the
cache once, deliberately, before doing any work:

```bash
pre-commit install-hooks
```

That is the expensive step. Let it finish. Afterwards it never repeats,
including across restarts.

**If you need a commit through immediately**, skip the hooks rather than fight
them. CI still gates the branch:

```bash
git commit --no-verify -m "..."
SKIP=markdownlint-cli2 git commit -m "..."   # or skip just the heavy one
```

**Still dying?** Commit in smaller batches, and close the Source Control panel
while you do — an unrendered panel does not re-scan. Then check whether the
install alone is the problem: `pre-commit install-hooks` on an otherwise idle
workspace tells you whether you have the headroom at all.

---

## Every document fails the gates on day one

Expected, and it does not block you. `pre-commit` only inspects **staged**
files, so existing documents are not touched until you edit them. Adoption is
therefore incremental by default: the gates apply to what you are working on,
not to everything you have ever written.

Treat the backlog as a runway rather than a wall:

- Fix documents as you touch them. A file you were editing anyway costs little
  extra to bring to standard.
- To see the whole picture without committing anything:
  `pre-commit run --all-files`. Read it as a survey, not a to-do list.
- Fixer-style hooks rewrite files and fail the commit so you can review. Re-add
  and commit again — the second attempt passes.
- If a rule genuinely does not fit your content, change the rule. The configs
  are yours: `.markdownlint.json` for structure, `.vale.ini` for prose.

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
