# Digest Template

## 3 Hard Rules (Non-negotiable)

1. **Every item must have a URL** — information without links is never published; untraceable = not published
2. **No fabrication** — content outside the watchlist is never written; if uncertain, mark as "to be verified"
3. **No guessing titles** — person descriptions must use official descriptions from the watchlist, no self-inference

## Output Language

Chinese (configurable)

## Report Structure

```markdown
🚕 Robotaxi Daily | {YYYY-MM-DD}

📌 Today's Highlights
{3-5 core summaries, one sentence each, sorted by importance}

---

🌏 Overseas Dynamics

| Date | Event Summary | Source |
|------|---------------|--------|
| {MM-DD} | {One-sentence event description} | {Source name} [🔗]({url}) |
| ... | ... | ... |

---

🇨🇳 China Dynamics

| Date | Event Summary | Source |
|------|---------------|--------|
| {MM-DD} | {One-sentence event description} | {Source name} [🔗]({url}) |
| ... | ... | ... |

---

💡 Editor's Observation (optional, only when sufficient information available)
{Brief analysis of today's dynamics, 2-3 sentences}

---

📊 Today's Statistics
- Data source hits: {N} items
- After dedup: {M} items
- After filtering (in report): {K} items
- Monitored companies: {X} companies
```

## Field Descriptions

| Field | Rule |
|-------|------|
| **Date** | Date the event occurred, format MM-DD. If uncertain, mark as "recent" |
| **Event Summary** | One-sentence summary, 20-40 characters. Include key data (quantities, amounts, city names) |
| **Source** | Source name + link. If multiple sources merged, list the most authoritative |
| **Today's Highlights** | Extract 3-5 most important items from all dynamics, each ≤ 50 characters |
| **Editor's Observation** | Optional. Only write when the day's dynamics are sufficiently rich; avoid hollow commentary |

## Section Rules

| Assignment | Rule |
|------------|------|
| **Overseas** | Event occurs outside China, or the subject is an overseas company (Waymo, Tesla, Zoox, etc.) |
| **China** | Event occurs in China, or the subject is a Chinese company (Apollo Go, Pony.ai, WeRide, etc.) |
| **Ambiguous** | Chinese company overseas events (e.g., WeRide in Abu Dhabi) → place in "Overseas"; may also mention in China section |

## Length Control

- Total word count: 1000-2000 characters
- Today's Highlights: 3-5 items, each ≤ 50 characters
- Event Summary: each 20-40 characters
- Editor's Observation: 2-3 sentences, ≤ 100 characters
- Dynamic entries: unlimited, include as many as available

## Sort Order

- Overseas Dynamics: reverse chronological (newest first)
- China Dynamics: reverse chronological (newest first)
- Same-day events: T1 > T2 > T3 > T4 (higher Tier first)

## Empty Report Format

When the filter layer determines no valid content:

```
🚕 Robotaxi Daily | {YYYY-MM-DD}

No new content today.

📊 Monitoring Statistics
- Scanned {N} data sources
- Valid content after filtering: 0 items
- Next update: {tomorrow's date} 08:00
```

## Example

```
🚕 Robotaxi Daily | 2026-06-22

📌 Today's Highlights
- Waymo officially launches commercial operations in Miami, initially deploying 200 vehicles
- Apollo Go Q1 weekly order peak exceeds 350K orders, achieving single-city profitability
- Mobileye and VW Group reach Robotaxi solution partnership agreement

---

🌏 Overseas Dynamics

| Date | Event Summary | Source |
|------|---------------|--------|
| 06-21 | Waymo officially launches commercial Robotaxi service in Miami, initially deploying 200 vehicles | Waymo [🔗](https://waymo.com/blog/miami) |
| 06-21 | Mobileye partners with VW to provide Robotaxi technology solutions for VW models | Reuters [🔗](https://reuters.com/mobileye-vw) |
| 06-20 | Tesla expands Austin Robotaxi operating area to full metro, still only ~20 vehicles | Electrek [🔗](https://electrek.co/tesla-austin) |
| 06-20 | Motional announces Robotaxi test operations restart in Los Angeles | TechCrunch [🔗](https://techcrunch.com/motional-la) |

---

🇨🇳 China Dynamics

| Date | Event Summary | Source |
|------|---------------|--------|
| 06-22 | Apollo Go Q1 weekly order peak exceeds 350K, CEO confirms single-city profitability achieved | QbitAI [🔗](https://qbitai.com/baidu-robotaxi) |
| 06-21 | WeRide signs agreement with Geely Farizon, 2000 pre-equipped mass-production Robotaxi GXR to be delivered in 2026 | Sina Finance [🔗](https://finance.sina.com.cn/weride) |

---

💡 Editor's Observation
Overseas signal density is notably higher than domestic this week. Waymo continues accelerating city expansion, Motional's US operations restart is worth watching. Domestically, Apollo Go's profitability signal is positive — the industry is shifting from "burning cash for testing" to "commercial validation."

---

📊 Today's Statistics
- Data source hits: 28 items
- After dedup: 19 items
- After filtering (in report): 6 items
- Monitored companies: 20 companies
```
