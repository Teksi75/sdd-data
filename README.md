# sdd-data

Curated LLM data feed for **sdd_agent_selector** and the **gentle-ai** ecosystem.
This repo is the runtime data source for the "Actualizar ahora" button in
[sdd_agent_selector](https://github.com/Teksi75/sdd_agent_selector).

The runtime fetches the 5 JSON files from
`https://raw.githubusercontent.com/Teksi75/sdd-data/main/data/*.json`.

## Files

| File | Contents |
|---|---|
| `data/models.json` | 24+ active models, 2 reference, tier, lifecycle, pricing, BenchLM scores |
| `data/phases.json` | 9 SDD phases (init, explore, propose, spec, design, tasks, apply, verify, archive) |
| `data/configs.json` | 5 strategy presets (min-cost, balanced, max-quality, etc.) |
| `data/agent-roles.json` | 18 agent role matrix (minReasoning, costRatio, description) |
| `data/agent-request-profiles.json` | Token profile per agent (input/output defaults) |

## How it stays in sync

The [sdd_agent_selector CI](https://github.com/Teksi75/sdd_agent_selector/blob/main/.github/workflows/sync-data.yml)
watches `data/*.json` on pushes to `main` and mirrors any changes here via
`gh api` (contents endpoint). The sync is automatic — no manual copy needed.

For safety, the sync also runs on a daily cron (06:00 UTC) as a self-heal
in case a push was missed or a file got out of sync.

## Schema

- Schema version: **2**
- `models.json._meta.lastSynced` is the source of truth for freshness
- All other `_meta` blocks (per-file) are kept for backward compat

## How to consume

```js
const files = [
  'data/models.json',
  'data/phases.json',
  'data/configs.json',
  'data/agent-roles.json',
  'data/agent-request-profiles.json',
];
const base = 'https://raw.githubusercontent.com/Teksi75/sdd-data/main/';
const payload = await Promise.all(
  files.map(f => fetch(base + f).then(r => r.json()))
);
```

The `sdd_agent_selector` page bakes the same files into `dist/data/` at
build time, so the runtime fetch from this repo is a **progressive
enhancement** — it lets the user pick up new pricing/benchmarks between
deploys without waiting for the next PR merge.
