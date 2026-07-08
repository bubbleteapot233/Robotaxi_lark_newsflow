# Filter Layer

> The core quality control layer between fetching and report generation. Ensures the daily report only contains valuable Robotaxi operations-related information.

## Filtering Pipeline

```
Raw search results (raw data)
    │
    ├─ Step 1: Deduplication
    │    └─ Same event from multiple sources → keep most authoritative source
    │
    ├─ Step 2: Relevance Filter
    │    └─ Remove non-Robotaxi-operations-related content
    │
    ├─ Step 3: Quality Check
    │    └─ Ensure every item has URL, timestamp, and summary
    │
    └─ Step 4: Empty Day Handler
         └─ No valid content after filtering → output "No new content"
```

---

## Step 1: Deduplication

### Dedup Rules

**Criteria for "same event":**
- Title or summary describes the same core event (e.g., "Waymo launches in Miami" reported by both TechCrunch and Reuters)
- Same company publishes highly similar content on the same day (e.g., official blog and media reprint)

**Dedup strategy:**

| Situation | Retention Policy |
|-----------|-----------------|
| Same event reported by T1 official + T3 media | Keep T1 official (more authoritative) + T3 as supplementary citation |
| Same event reported by multiple T3 media | Keep the most detailed article |
| Same event reported in both English + Chinese | Keep Chinese (more comprehensive), English as "international perspective" supplement |
| Multiple highly similar items from same company | Merge into one item, annotate multiple sources |

**Post-dedup record format:**

```json
{
  "event_id": "evt-2026-06-22-001",
  "title": "Waymo officially launches commercial Robotaxi service in Miami",
  "date": "2026-06-21",
  "region": "overseas",
  "primary_source": {
    "name": "Waymo",
    "url": "https://waymo.com/blog/miami-launch",
    "tier": "T1"
  },
  "supplementary_sources": [
    {
      "name": "TechCrunch",
      "url": "https://techcrunch.com/waymo-miami",
      "tier": "T3"
    }
  ],
  "summary": "Waymo announces official commercial Robotaxi service launch in Miami, initially deploying 200 vehicles...",
  "companies_involved": ["Waymo"]
}
```

---

## Step 2: Relevance Filter

### Content to KEEP (Robotaxi operations-related)

The following content types are **retained**:

- New city launches / operational area expansions
- Fleet size changes / new vehicle deployments
- Order volume / revenue / profitability data
- Technology milestones (new sensors, new software versions, safety data)
- Regulatory approvals / license acquisitions / policy changes
- Strategic partnerships (OEM collaborations, platform integration, supply chain)
- Funding / IPO / financial report related
- Safety incidents / recalls / safety reports
- Competitive landscape changes (new entrants, exits)

### Content to FILTER OUT (non-Robotaxi-operations-related)

The following content types are **removed**:

| Filter Type | Examples | Keywords/Signals |
|-------------|----------|------------------|
| **Event promotions** | "XX Summit registration", "Limited-time ride discounts", "Brand collaboration events" | "register", "discount", "promo", "summit", "conference", "forum" |
| **Personnel changes** | "XX appoints new CTO", "Executive departure", "Team hiring" | "appointment", "joins", "leaves", "hiring" |
| **Job postings** | "Hiring autonomous driving engineers", "We're hiring" | "hiring", "careers", "jobs", "open position" |
| **Product reviews/ads** | "Test drive XX model experience", "Brand advertorial" | "test drive", "review", "sponsored", "advertorial" |
| **Pure academic/papers** | "Paper proposes new algorithm" (no commercialization progress) | "paper", "arXiv" (unless involving major breakthroughs) |
| **Stock market movements** | "XX stock up 3% today" (no substantive news) | "stock price" (unless accompanied by major announcements) |
| **Recycled old news** | Old stories re-reported days later with no new information increment | Date older than 48 hours with no new information increment |

### Edge Case Handling

| Situation | Handling |
|-----------|----------|
| Personnel change also contains business progress | Extract business progress, remove personnel content |
| Event promotion reveals operational data | Extract operational data, remove promotion content |
| Uncertain if relevant | Keep, but annotate in report as "relevance to be confirmed" |

---

## Step 3: Quality Check

Perform quality check on each item that passes filtering:

| Check Item | Fail Handling |
|------------|---------------|
| Missing URL | Remove item (hard rule: no link = no publish) |
| Missing date | Mark as "date to be confirmed" |
| Summary too vague | Attempt to extract more detail from original page |
| Unreliable source | Mark as "to be verified" |

---

## Step 4: Empty Day Handler

### Decision Logic

```
Valid content count after filtering:

≥ 1 item → Generate report normally
0 items  → Output empty report template
```

### Empty Report Template

```
🚕 Robotaxi Daily | {YYYY-MM-DD}

No new content today.

Monitoring scope: {N} core companies + {M} media sources
Next update: {tomorrow's date} 08:00
```

### Post-Empty-Report Recommendations

If empty reports occur for 2+ consecutive days, output watchlist adjustment recommendations:

```
💡 Watchlist Adjustment Recommendations:
- No content updates for {N} consecutive days, recommend checking query strategies for the following sources:
  - {source_name}: No hits for the last {N} days
  - Recommendation: Adjust query_template or add alternative data sources
```

---

## Filter Log

Each run generates a filter log saved to `data/episodes/{date}-filter-log.json`:

```json
{
  "date": "2026-06-22",
  "raw_count": 25,
  "after_dedup": 18,
  "after_relevance_filter": 12,
  "after_quality_check": 10,
  "removed_items": [
    {
      "title": "XX Summit registration notice",
      "reason": "activity_promotion",
      "source": "36kr"
    },
    {
      "title": "Waymo hires new VP",
      "reason": "personnel_change",
      "source": "techcrunch"
    }
  ],
  "duplicate_groups": [
    {
      "event": "Waymo Miami launch",
      "sources_merged": ["waymo-official", "techcrunch-av", "reuters-av"],
      "kept": "waymo-official"
    }
  ]
}
```
