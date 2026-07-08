---
name: robotaxi-monitor
description: Daily automated tracking of global Robotaxi industry dynamics, generating structured daily briefings. Covers Waymo, Tesla, Zoox, Baidu Apollo Go, Pony.ai, WeRide and 15+ core players. Overseas-first, China supplementary. Supports scheduled and manual triggering.
---

# Robotaxi Monitor

> Global Robotaxi industry daily monitoring system. Overseas-first, China supplementary, facts only — no fabrication, no guessing.

## Trigger Methods

### Manual Trigger

Any of the following phrases can trigger:

- "Robotaxi daily report"
- "Today's dynamics"
- "Run monitor"
- "robotaxi monitor"
- "Industry dynamics"

### Scheduled Trigger

Runs daily at 8:00 AM (Beijing time) via QoderWork cron, with results pushed to Feishu/Lark.

## Workflow

```
Trigger
  │
  ├─ Step 1: Load watchlist.json, confirm data source list
  │
  ├─ Step 2: Fetch by Tier (fetch-strategy.md)
  │    ├─ P0: T1 core players official (6 companies)
  │    ├─ P1: T2 industry players (14 companies) official + key people
  │    ├─ P2: T3 industry media
  │    ├─ P3: T4 policy & research reports
  │    └─ P4: Fallback layer (triggered when total hits < 5)
  │
  ├─ Step 3: Filter layer (filter-layer.md) ★
  │    ├─ Dedup: same event from multiple sources → keep most authoritative + most detailed
  │    ├─ Filter: remove event promotions, personnel changes, recruitment, etc.
  │    └─ Empty check: if still < 1 item after filtering → output "No new content"
  │
  ├─ Step 4: Generate daily report per digest-template.md
  │
  └─ Step 5: Push to messaging platform (e.g., Feishu/Lark bot)
```

## Messaging Platform Push Rules

### Recipient

Configure your recipient's open_id or user identifier. Example:

- Use your bot's messaging API to send to the target user
- **Do not search contacts** — use the pre-configured ID directly

### Sending Method

The daily report file must be saved as **UTF-8 encoded**, then sent via CLI or API:

```bash
# Example using lark-cli
lark-cli im +messages-send \
  --user-id <YOUR_OPEN_ID> \
  --as bot \
  --markdown "$(cat /path/to/YYYY-MM-DD-robotaxi-digest.md)"
```

### Encoding Notes (Important)

- **Report file must be UTF-8 encoded**
- **Must pass via bash `$(cat file.md)` to `--markdown`** to ensure UTF-8 parameter passing
- **Do not read files via PowerShell and pass to `--markdown`**: PowerShell defaults to GBK/GB2312 encoding, which causes Chinese and emoji garbling

### Prohibited Actions

- **Do not use `--as user`**: triggers user identity authorization flow requiring extra scopes
- **Do not search contacts**: no need for `contact:user:search` permission
- **Do not initiate `lark-cli auth login`**: bot identity does not require user authorization

### Empty Report Handling

If no valid news after filtering for the day, send a short notification:

```bash
lark-cli im +messages-send \
  --user-id <YOUR_OPEN_ID> \
  --as bot \
  --text "🚕 Robotaxi Daily | {date} — No new content today"
```

## File Description

| File | Purpose |
|------|---------|
| `watchlist.json` | Core config: all monitored entities + data sources + query templates |
| `fetch-strategy.md` | Fetch strategy: dual-track parallel + priority + fallback mechanism |
| `filter-layer.md` | Filter layer: deduplication + content filtering + empty report handling |
| `digest-template.md` | Report template: output format + 3 hard rules |
| `data/episodes/` | Runtime data: raw data archive from each run |

## 3 Hard Rules (Non-negotiable)

1. **Every item must have a URL** — information without links is never published
2. **No fabrication** — content outside the watchlist is never written; if uncertain, mark as "unverified"
3. **No guessing titles** — person descriptions must use official descriptions from the watchlist

## Output Requirements

- Language: Chinese
- Structure: Today's Highlights (3-5 items) → Overseas Dynamics Table → China Dynamics Table → Editor's Observation (optional)
- Each dynamic must include: time, event summary (one sentence), source link
- Overseas displayed first
- If no valid news after filtering for the day, output: "🚕 Robotaxi Daily | {date} — No new content today"
