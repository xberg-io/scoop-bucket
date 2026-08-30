# xberg-io/scoop-bucket

[![Validate manifests](https://github.com/xberg-io/scoop-bucket/actions/workflows/validate-manifests.yml/badge.svg)](https://github.com/xberg-io/scoop-bucket/actions/workflows/validate-manifests.yml)

[Scoop](https://scoop.sh) bucket for the [xberg.io](https://xberg.io) command-line tools
on Windows. The macOS and Linux counterpart is
[xberg-io/homebrew-tap](https://github.com/xberg-io/homebrew-tap).

## Installation

Scoop itself, if you do not have it:

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
Invoke-RestMethod -Uri https://get.scoop.sh | Invoke-Expression
```

Then add this bucket and install what you need:

```powershell
scoop bucket add xberg https://github.com/xberg-io/scoop-bucket
scoop install crawlberg
```

Add `xberg/` in front of an app name (`scoop install xberg/crawlberg`) if another
bucket you have installed provides an app by the same name.

## Apps

| App | What it does | Architectures | Source |
|-----|--------------|---------------|--------|
| `crawlberg` | Web crawling and scraping, with headless-Chrome fallback | 64bit | [crawlberg](https://github.com/xberg-io/crawlberg) |
| `html-to-markdown` | Fast, lossless HTML→Markdown conversion | 64bit | [html-to-markdown](https://github.com/xberg-io/html-to-markdown) |
| `liter-llm` | LLM API client, OpenAI-compatible proxy, and MCP server | 64bit | [liter-llm](https://github.com/xberg-io/liter-llm) |
| `ts-pack` | Tree-sitter grammars and code-intelligence primitives | 64bit, arm64 | [tree-sitter-language-pack](https://github.com/xberg-io/tree-sitter-language-pack) |
| `alef` | Polyglot binding generator for Rust libraries | 64bit | [alef](https://github.com/xberg-io/alef) |

`ts-pack` is the only app with a native arm64 Windows build. The rest install their
64bit build, which Windows on ARM runs under emulation.

`xberg` itself is not yet published here. Its Windows binary imports
`onnxruntime.dll`, and no released archive bundles that DLL yet — the fix is on the
source repo's main branch, so the manifest lands with the next `xberg` release.

## Runtime requirements

Every app is a native MSVC build and needs the Microsoft Visual C++ 2015-2022
redistributable. Scoop suggests it during install; install it explicitly with:

```powershell
scoop install extras/vcredist2022
```

`xberg` additionally needs [Tesseract](https://github.com/tesseract-ocr/tesseract) on
`PATH` for OCR, which is optional for every other feature.

## Updating

```powershell
scoop update            # refresh bucket metadata
scoop update crawlberg  # upgrade one app
scoop update '*'        # upgrade everything installed
```

## How a version gets here

Nothing in this repository polls for new releases, and there is no Excavator bot. Each
source repository owns its manifest and pushes it: on release, its pipeline downloads the
Windows archive, hashes it, renders the manifest from a template it keeps under
`scripts/publish/`, and commits the result here. A new version usually appears within
minutes of the source release.

That makes `bucket/*.json` generated output. See [MANIFESTS.md](MANIFESTS.md) before
touching it — a hand-edit survives only until the owning repo's next release.

## Troubleshooting

**`couldn't find manifest for <app>`** — the bucket is not added, or its metadata is
stale. Run `scoop bucket list` to confirm, then `scoop update`.

**Hash check failed** — the release asset was replaced after the manifest was written.
Open an issue rather than passing `--skip`; a hash mismatch is the one check standing
between you and an unintended binary.

**The app installs but will not start** — usually a missing Visual C++ runtime. Install
`extras/vcredist2022` as above. If it persists, include the output of
`scoop info <app>` and `scoop which <app>` in an issue.

**`scoop status` says an app is outdated but `scoop update` does nothing** — check
whether the source repository actually published a release; this bucket only ever
reflects what upstream shipped.

## Uninstall

```powershell
scoop uninstall crawlberg
scoop bucket rm xberg
```

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). Manifests are generated — read
[MANIFESTS.md](MANIFESTS.md) before editing anything under `bucket/`.
