---
type: Playbook
title: Manifest Maintenance
description: "Scoop manifest upkeep: bucket/*.json is generated, per-app extract_dir values, $version escaping in autoupdate, local validation. Use when editing a manifest under bucket/ or debugging a failed scoop install."
paths:
  - "bucket/**"
x-ai-rulez:
  kind: skill
  id: manifest-maintenance
  metadata:
    name: manifest-maintenance
---

## Manifest Maintenance

### `bucket/*.json` is generated output

Each manifest is written by its source repository's release pipeline through
`xberg-io/actions/publish-scoop-manifests`. A hand-edit here survives only until that
repository's next release. Fix the template in the source repo:

| Manifest | Source repo | Template |
|----------|-------------|----------|
| `xberg.json` | `xberg-io/xberg` | `scripts/publish/xberg.json.tmpl` |
| `crawlberg.json` | `xberg-io/crawlberg` | `scripts/publish/crawlberg.json.tmpl` |
| `html-to-markdown.json` | `xberg-io/html-to-markdown` | `scripts/publish/html-to-markdown.json.tmpl` |
| `liter-llm.json` | `xberg-io/liter-llm` | `scripts/publish/liter-llm.json.tmpl` |
| `ts-pack.json` | `xberg-io/tree-sitter-language-pack` | `scripts/publish/ts-pack.json.tmpl` |
| `alef.json` | `xberg-io/alef` | `scripts/publish/alef.json.tmpl` |

There is no Excavator bot. Nothing polls; the release pipeline pushes.

### The two things that break installs silently

**`extract_dir`** — the zip layouts differ per repo and no rule derives them. Get it wrong and
`bin` resolves to nothing: the manifest hashes and validates and installs a broken shim.

| App | Zip layout | `extract_dir` |
|-----|-----------|---------------|
| `xberg` | nested, dir named after the asset | `xberg-cli-x86_64-pc-windows-msvc` |
| `crawlberg` | nested, dir named after the asset | `crawlberg-cli-x86_64-pc-windows-msvc` |
| `html-to-markdown` | nested, dir named after the asset | `cli-x86_64-pc-windows-msvc` |
| `liter-llm` | nested, **version in the dir name** | `liter-llm-${version}-x86_64-pc-windows-msvc` |
| `ts-pack` | **flat — the `.exe` is at the zip root** | omitted entirely |
| `alef` | nested, dir named after the asset | `alef-x86_64-pc-windows-msvc` |

Confirm with `unzip -l <asset>` against the actual release before trusting any of these; a repo
can change its packaging without anyone noticing the manifest went stale.

**`$version` in `autoupdate`** — templates render through Python `string.Template`, so
Scoop's literal `$version` must be written `$$version` in the template. An unescaped one
renders to the current version and freezes `autoupdate` on it without erroring.

### Validate locally

```bash
curl -fsSL -o /tmp/scoop-schema.json https://raw.githubusercontent.com/ScoopInstaller/Scoop/master/schema.json
uvx check-jsonschema==0.38.2 --schemafile /tmp/scoop-schema.json bucket/*.json
```

Schema validity proves nothing about installability. The `windows-latest` job in
`.github/workflows/validate-manifests.yml` is the only real check: it adds the checkout
as a bucket, installs each changed app, and runs `<app> --version`.

### Invariants CI enforces

- `bin` must be `<manifest filename>.exe` — Scoop names the app from the filename
- `version` must match the release the pipeline rendered
- no zero-SHA placeholder may reach the bucket (dry-run renders use one)
