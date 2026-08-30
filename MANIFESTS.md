# Manifests

## Do not hand-edit `bucket/*.json`

Every manifest in this repository is written by the source repository's release
pipeline, through the `xberg-io/actions/publish-scoop-manifests` action. Each source
repo owns exactly one manifest and holds the template that produces it:

| Manifest | Written by | Template |
|----------|-----------|----------|
| `bucket/xberg.json` | `xberg-io/xberg` | `scripts/publish/xberg.json.tmpl` |
| `bucket/crawlberg.json` | `xberg-io/crawlberg` | `scripts/publish/crawlberg.json.tmpl` |
| `bucket/html-to-markdown.json` | `xberg-io/html-to-markdown` | `scripts/publish/html-to-markdown.json.tmpl` |
| `bucket/liter-llm.json` | `xberg-io/liter-llm` | `scripts/publish/liter-llm.json.tmpl` |
| `bucket/ts-pack.json` | `xberg-io/tree-sitter-language-pack` | `scripts/publish/ts-pack.json.tmpl` |
| `bucket/alef.json` | `xberg-io/alef` | `scripts/publish/alef.json.tmpl` |

A hand-edit here survives only until that repository's next release, which overwrites
the file wholesale. Fix the template in the source repository instead.

There is no Excavator bot on this bucket. Nothing here polls for new versions; the
release pipeline pushes.

## Manifest anatomy

```json
{
  "version": "1.4.2",
  "architecture": {
    "64bit": {
      "url": "https://github.com/xberg-io/crawlberg/releases/download/v1.4.2/crawlberg-cli-x86_64-pc-windows-msvc.zip",
      "hash": "6a4db45...",
      "extract_dir": "crawlberg-cli-x86_64-pc-windows-msvc"
    }
  },
  "bin": "crawlberg.exe"
}
```

- **`version`** — the release version without the `v` prefix. The publish action
  refuses to commit a manifest whose `version` does not match the release it rendered.
- **`hash`** — SHA256 of the exact `.zip` at `url`, computed by the publish action from
  the downloaded asset. Scoop verifies it on every install; a wrong hash makes the app
  uninstallable for everyone.
- **`extract_dir`** — the directory *inside* the zip that Scoop should treat as the app
  root. This is not cosmetic: get it wrong and `bin` resolves to nothing.
- **`bin`** — must be `<manifest filename>.exe`. Scoop names the app after the file, not
  after anything inside it, and CI enforces the match.

### `extract_dir` per app

The six repositories do not package their Windows zips identically, so the value
differs and cannot be derived from a rule:

| App | Zip layout | `extract_dir` |
|-----|-----------|---------------|
| `crawlberg` | nested, dir named after the asset | `crawlberg-cli-x86_64-pc-windows-msvc` |
| `html-to-markdown` | nested, dir named after the asset | `cli-x86_64-pc-windows-msvc` |
| `liter-llm` | nested, **version in the dir name** | `liter-llm-<version>-x86_64-pc-windows-msvc` |
| `ts-pack` | **flat — the `.exe` is at the zip root** | omitted |
| `alef` | nested, dir named after the asset | `alef-x86_64-pc-windows-msvc` |
| `xberg` | nested, dir named after the asset | `xberg-cli-x86_64-pc-windows-msvc` |

## `$version` in `autoupdate`

Scoop's own `autoupdate` block uses a literal `$version` placeholder that Scoop expands
when it detects a new release. The templates are rendered with Python's
`string.Template`, which consumes `$version` itself, so templates must write `$$version`
inside `autoupdate` — `string.Template` collapses `$$` to a single `$`.

An unescaped `$version` there is not a syntax error. It renders to the current release's
version and silently freezes `autoupdate` on it. The rendered manifests in `bucket/` are
the place to check: `autoupdate` URLs must contain a literal `$version`, and the
top-level `architecture` URLs must contain a concrete version number.

## Native dependencies

All six binaries import `VCRUNTIME140.dll`, so every manifest carries
`"suggest": {"vcredist": "extras/vcredist2022"}`.

Only `xberg.exe` has a third-party hard import, `onnxruntime.dll`. Its publish job
vendors the ONNX Runtime DLL closure into the zip and Scoop's shim resolves DLLs from
the app directory, so no `depends` entry is needed — but the DLL must actually be in the
archive. Tesseract, used for OCR, is looked up on `PATH` at runtime and is documented in
`notes` rather than declared as a dependency.

## CI

`.github/workflows/validate-manifests.yml` runs on every change under `bucket/`:

1. **Schema** — validates each manifest against the upstream Scoop `schema.json` and
   asserts `bin` matches the filename.
2. **Install** — on `windows-latest`, adds this checkout as a bucket, runs
   `scoop install` for each changed app, and executes `<app> --version`.

The install job is the only check that proves the shim, `extract_dir`, and the DLL
closure work. A manifest can be schema-valid, correctly hashed, and still install a
binary that cannot start.
