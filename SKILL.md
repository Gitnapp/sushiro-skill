---
name: sushiro
description: Query Sushiro China (寿司郎/壽司郎) store list, single-store detail, and live wait/queue status via the wechat-mini-program backend. Use when the user asks about 寿司郎/sushiro 排队/等位/wait/queue, wants to compare wait times across cities or areas, look up a specific Sushiro store by id/name/area, or fetch the official areas list. Only covers mainland China stores.
---

# Sushiro Queue Skill

A thin curl + jq wrapper over the three public endpoints of Sushiro China's wechat-mini-program backend (`crm-cn-prd.sushiro.com.cn`). No API key required — the Bearer token is a shared mini-program credential and is hardcoded in the script.

## Quick start

The CLI is at `scripts/sushiro`. Invoke it directly (it's executable). Common usage:

```bash
scripts/sushiro summary                        # nationwide aggregate
scripts/sushiro stores --waiting --limit=10    # 10 busiest stores now
scripts/sushiro stores --city=深圳              # all Shenzhen stores
scripts/sushiro store 1012                     # single-store detail
scripts/sushiro wait 1012                      # one-line wait count
scripts/sushiro search 来福士                   # name/area/address search
scripts/sushiro areas                          # all 区 names
scripts/sushiro raw 'stores?latitude=22.5&longitude=113.9&numresults=3'
```

Add `--json` to `stores`, `store`, or `areas` for raw JSON when piping or post-processing. `summary`, `search`, and `raw` always emit JSON or table; pipe through `jq` as needed.

## Output policy

- For human-readable queries (`how many people waiting at X?`, `which Sushiro is least busy in 深圳?`): use the default table/pretty output and just relay the answer.
- For programmatic follow-up (graphing, filtering, storing): pass `--json` and pipe to `jq`.

## Field cheat sheet

| Field | Meaning |
|---|---|
| `wait` | Groups currently in queue (not minutes). 0 = walk-in. |
| `waitTimeCap` | Upper bound of wait time in minutes (usually 180). |
| `storeStatus` | `OPEN` / `CLOSED` / `PRE_OPEN`. |
| `netTicketStatus` | `ONLINE` (queue accepts remote tickets) / `OFFLINE`. |
| `nameKana` | City (城市). Field name is a legacy carry-over from JP schema. |
| `area` | Neighborhood (e.g. `深圳南山区`). |
| `id` | Store id (4-digit, e.g. `1012`). Use with `store` / `wait`. |

## When data is stale

The backend itself updates in real-time. There is no local cache — every call hits Sushiro. If a result looks wrong, try `sushiro raw stores?latitude=1&longitude=1&numresults=1` to confirm the upstream is responsive; a 4xx/5xx means Sushiro changed or rotated the token.

## Endpoint reference

For full URL schema, parameter semantics, and a sample raw response, see [references/api.md](references/api.md).

## Constraints

- **Mainland China only.** This backend (`crm-cn-prd`) does not cover Japan / HK / TW. Tell the user explicitly if they ask about other regions.
- **Token may rotate.** If every call returns 401, the wechat mini-program token has been rotated. Override with `SUSHIRO_TOKEN=...` env var, or update the default in the script.
- **Python/requests is blocked** by upstream CDN fingerprinting. Stick to curl (the script already uses it).
- **Don't hammer.** This is an unofficial use of a wechat-internal API. For repeated polling, add `sleep` or use the upstream's own cron (1 req/min is fine).
