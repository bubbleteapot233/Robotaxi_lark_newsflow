# Robotaxi_lark_newsflow: Robotaxi Industry Monitor

An open-source AI agent skill suite for automated tracking of the global Robotaxi industry. Built for [QoderWork](https://docs.qoder.com/qoderwork/introduction) — a desktop agentic assistant platform.

AlphaLab combines two complementary skills into a complete data pipeline:

- **robotaxi-monitor** — Daily web search across 30+ data sources (company blogs, IR pages, industry media, regulatory filings), with a tiered fetch strategy, deduplication, relevance filtering, and structured daily briefing output.
- **robotaxi-bitable-automation** — Structured archiving to Feishu/Lark Bitable, with event fingerprinting, verification workflows, human-in-the-loop confirmation, and master table synchronization.

## What It Does

Every day, AlphaLab:

1. **Searches** 30+ sources across 4 tiers — from core players (Waymo, Tesla, Zoox, Baidu Apollo Go, Pony.ai, WeRide) to industry media and policy filings.
2. **Filters** through deduplication, relevance checks, and quality validation.
3. **Generates** a structured daily briefing in Chinese, with overseas/China split sections, key highlights, and editor's observations.
4. **Archives** verified events to a Feishu Bitable "Projects & Milestones" table, with full provenance tracking.
5. **Syncs** confirmed entries to master tables (Company Info, Region Dictionary, Operational Status, Partner Details) after human review.

## Project Structure

```
AlphaLab/
├── robotaxi-monitor/              # Daily report generation skill
│   ├── SKILL.md                   # Skill definition & workflow
│   ├── watchlist.json             # Monitored companies & data sources
│   ├── fetch-strategy.md          # Tiered fetch strategy (P0-P4)
│   ├── filter-layer.md            # Dedup + relevance + quality filters
│   └── digest-template.md         # Daily report format & rules
│
├── robotaxi-bitable-automation/   # Bitable archiving skill
│   ├── SKILL.md                   # Full data pipeline definition
│   ├── references/
│   │   ├── table-schema.md        # Bitable table & field schema
│   │   ├── confirmation-sync-template.md  # Post-confirmation sync flow
│   │   └── daily-cron-template.md # Cron task prompt template
│   └── evals/
│       └── evals.json             # Skill evaluation test cases
│
├── .gitignore
└── README.md
```

## Monitored Companies

### Tier 1 — Core Players (6)
Waymo, Tesla Robotaxi, Zoox (Amazon), Baidu Apollo Go, Pony.ai, WeRide

### Tier 2 — Industry Players (14)
Momenta, Wayve, 42 Dot (Hyundai), Hyundai RoboRide, Mobileye, Motional, MOIA (VW), Avride, Cruise/GM, Aurora, XPeng AD, DeepRoute, Woven by Toyota, Nuro

### Tier 3 — Industry Media (9)
TechCrunch, Reuters, Electrek, The Verge, QbitAI, LeiPhNet, 36Kr, AutoCar AI, D1EV

### Tier 4 — Policy & Research (2)
NHTSA (US), China Autonomous Driving Policy

## How to Use

### Prerequisites

- [QoderWork](https://docs.qoder.com/qoderwork/introduction) desktop app installed
- (Optional) [lark-cli](https://github.com/nicepkg/lark-cli) configured with Feishu bot credentials for Bitable integration

### Installation

1. Clone this repository
2. Copy the skill directories into your QoderWork skills folder:

```bash
cp -r robotaxi-monitor ~/.qoderwork/skills/
cp -r robotaxi-bitable-automation ~/.qoderwork/skills/
```

### Manual Trigger

In QoderWork, say any of:

- "Robotaxi daily report"
- "Run monitor"
- "Today's dynamics"
- "robotaxi monitor"

### Scheduled Trigger

Set up a QoderWork cron job for daily 8:00 AM (Beijing time) execution. See `robotaxi-bitable-automation/references/daily-cron-template.md` for the task prompt template.

### Customization

- **Add/remove companies**: Edit `robotaxi-monitor/watchlist.json`
- **Change report format**: Edit `robotaxi-monitor/digest-template.md`
- **Adjust filtering rules**: Edit `robotaxi-monitor/filter-layer.md`
- **Configure your Bitable**: Update table IDs and app token in `robotaxi-bitable-automation/SKILL.md` and `references/table-schema.md`

## Hard Rules (Non-negotiable)

1. **Every item must have a URL** — information without links is never published
2. **No fabrication** — content outside the watchlist is never written; uncertain items are marked "to be verified"
3. **No guessing** — person titles, fleet sizes, operational status, and city names must come from sources

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Trigger (manual/cron)                  │
└───────────────────────┬─────────────────────────────────┘
                        │
           ┌────────────▼────────────────┐
           │  robotaxi-monitor (Skill 1)  │
           │  watchlist → fetch → filter  │
           │  → generate daily digest     │
           └────────────┬────────────────┘
                        │
                        ▼
           ┌────────────────────────────┐
           │  Daily Digest (Markdown)    │
           │  → pushed to Feishu/Lark    │
           └────────────┬───────────────┘
                        │
           ┌────────────▼───────────────────────────┐
           │  robotaxi-bitable-automation (Skill 2)  │
           │  extract events → dedup → write to      │
           │  Projects table → human confirm →       │
           │  sync master tables                     │
           └────────────────────────────────────────┘
```

## Bitable Schema

The Bitable automation skill works with 5 interconnected tables:

| Table | Purpose |
|-------|---------|
| Company Info | Company watchlist & master data |
| Projects & Milestones | Daily event intake & verification queue |
| Company × Region Ops Status | Operational facts per company per city |
| Partner Details | Partnership relationships |
| Region Dictionary | City/country/region reference data |

See `robotaxi-bitable-automation/references/table-schema.md` for complete field definitions.

## Workflow: Data Pipeline

```
Information Intake → Verification → Archiving → Human Confirmation → Master Table Sync
```

Phase A (automated): Search → Deduplicate → Write to Projects table → Output daily report
Phase B (human-in-the-loop): User confirms → Sync to master tables → Write back sync status

## Contributing

Contributions are welcome! Areas where help is needed:

- Expanding the watchlist with new companies or data sources
- Improving filter rules for better signal-to-noise ratio
- Translating documentation to other languages
- Adding support for other Bitable/messaging platforms

## License

MIT
