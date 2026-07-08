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

## Bitable Self-Setup Guide

The Bitable automation requires a Feishu/Lark Bitable with 5 tables. Since we can't share a template link (corporate tenant restrictions), here's how to set it up yourself.

### Step 1: Create a New Bitable

In your Feishu/Lark workspace, create a new Bitable. Note the **app_token** from the URL:

```
https://<your-tenant>.feishu.cn/base/<app_token>
```

### Step 2: Create the 5 Tables

Create these tables in order (base tables first, then tables with cross-references):

**Phase 1 — Base tables (no dependencies):**

1. **企业基本信息表** (Company Info) — Primary field: `企业` (Text)
2. **地区字典表** (Region Dictionary) — Primary field: `城市` (Text)

**Phase 2 — Dependent tables:**

3. **项目与里程碑表** (Projects & Milestones) — Primary field: `事件名称` (Text)
4. **企业 × 地区 运营状态表** (Company × Region Ops Status) — Primary field: `企业 - 地区` (Formula: `关联企业 & " - " & 关联城市`)
5. **合作伙伴明细** (Partner Details) — Primary field: `合作方` (Text)

### Step 3: Add Fields

For each table, add fields per `references/table-schema.md`. Key field types:

| Field Type | How to Create |
|------------|---------------|
| Text | `type: "text"` |
| Single Select | `type: "select", multiple: false, options: [...]` |
| Multi Select | `type: "select", multiple: true, options: [...]` |
| Date | `type: "datetime"` |
| Link (cross-table) | `type: "link", link_table: "<target_table_name>"` |
| Formula | `type: "formula", expression: "..."` |
| Lookup | `type: "lookup", from: "...", select: "...", where: {...}` |

**Important field groups by table:**

- **Company Info**: 企业属性 (select), 总部国家 (select), 产品线 (multi-select), OEM合作方 (multi-select), 盈利结构层级 (multi-select), etc.
- **Region Dictionary**: 城市英文名 (text), 国家 (select), 区域 (select), 地域标签 (select), 政策准入状态 (select), 当地政策友好度 (select)
- **Projects & Milestones**: 具体归属企业 (link→Company Info), 目标城市 (link→Region Dictionary), 归属主体分类 (select), 日期 (datetime), 状态标签 (select), 事件类型 (select), 链接 (text/url), plus all source/verification/sync task fields
- **Company × Region Ops Status**: 关联企业 (link→Company Info), 关联城市 (link→Region Dictionary), 当前运营状态 (select), 合作模式 (select), plus fleet/partner fields
- **Partner Details**: 合作方 (text), 伙伴分类 (select), 合作状态 (select), plus reverse links to Company × Region

### Step 4: Configure the Skill

After creating all tables, update these files with your Bitable info:

1. **`robotaxi-bitable-automation/SKILL.md`**: Replace `<YOUR_BITABLE_URL>` and `<YOUR_APP_TOKEN>` with your actual values. Replace all `<your_table_id>` placeholders with the actual table IDs (found in each table's URL or via lark-cli `+table-list`).

2. **`robotaxi-bitable-automation/references/table-schema.md`**: Same replacements.

3. **`robotaxi-bitable-automation/references/daily-cron-template.md`**: Replace `<YOUR_BITABLE_URL>` with your actual URL.

### Step 5: Authorize lark-cli

```bash
lark-cli auth login --scope "base:table:read base:field:read base:record:read base:field:update base:field:create base:record:update base:record:create"
```

### Quick Reference: Select Options

For the most commonly needed select fields, here are the options to copy:

**归属主体分类**: 本公司项目 / 竞品企业动态 / 行业动态

**事件类型**: 自动驾驶测试 / 战略合作 / 商业化运营 / 政策动态

**当前运营状态**: 宣布进入 / 路测中 / 示范运营 / 有人商业化运营 / 全无人商业运营 / 已暂停 / 已退出

**信息确认状态**: 已确认 / 待核实 / 部分确认 / 已证伪 / 暂缓跟进

**可信度**: 高 / 中 / 低

**同步状态**: 未处理 / 待人工确认 / 已同步 / 无需同步 / 同步失败

**合作状态**: 谈判中 / 生效中 / 已暂停 / 已结束

**伙伴分类**: 出行平台 / 车队运营 / 当地政府机构 / 汽车OEM / 运营服务商 / 本地运营商/车队管理方 / 技术授权对象 / 其他

**区域**: 亚洲 / 中东 / 欧洲 / 美洲

**地域标签**: 国内 / 海外 / 港澳台

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
