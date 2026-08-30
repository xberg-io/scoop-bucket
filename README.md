# xberg-io/scoop-bucket

[Scoop](https://scoop.sh) bucket for the xberg.io command-line tools on Windows.

## Installation

```powershell
scoop bucket add xberg https://github.com/xberg-io/scoop-bucket
scoop install crawlberg
```

Add `xberg/` in front of an app name (`scoop install xberg/crawlberg`) if another
bucket you have installed provides an app by the same name.

## Apps

| App | Architectures | Source repository |
|-----|---------------|-------------------|
| `crawlberg` | 64bit | [xberg-io/crawlberg](https://github.com/xberg-io/crawlberg) |
| `html-to-markdown` | 64bit | [xberg-io/html-to-markdown](https://github.com/xberg-io/html-to-markdown) |
| `liter-llm` | 64bit | [xberg-io/liter-llm](https://github.com/xberg-io/liter-llm) |
| `ts-pack` | 64bit, arm64 | [xberg-io/tree-sitter-language-pack](https://github.com/xberg-io/tree-sitter-language-pack) |
| `alef` | 64bit | [xberg-io/alef](https://github.com/xberg-io/alef) |

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
scoop update
scoop update crawlberg
```

Manifests are written by each source repository's release pipeline, so a new version
appears here within minutes of that repository publishing a release.

## Uninstall

```powershell
scoop uninstall crawlberg
scoop bucket rm xberg
```

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). Manifests are generated — read
[MANIFESTS.md](MANIFESTS.md) before editing anything under `bucket/`.
