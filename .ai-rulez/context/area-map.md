---
type: Concept
title: Area Map
x-ai-rulez:
  kind: context
  id: area-map
  metadata:
    priority: high
---

# Area Map

Scoop bucket for the xberg-io CLIs.

- `bucket/*.json` — generated manifests, one per app, pushed by each product's release pipeline (`manifest-maintenance` skill)
- `MANIFESTS.md` — manifest ownership notes
- `.github/workflows/` — schema validation and `windows-latest` install checks
