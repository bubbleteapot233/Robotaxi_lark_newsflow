# Fetch Strategy

## Core Principles

- **Overseas-first**: Fetch overseas signals first (T1 English + T2 English), then China (T1 Chinese + T2 Chinese + T3 Chinese)
- **Execute by Tier**: P0 → P1 → P2 → P3, complete each layer before proceeding to the next
- **One search per source**: Execute each query_template once via WebSearch, do not repeat searches for the same source
- **Preserve raw data**: All search results written to `data/episodes/{date}-raw.json`

## Execution Flow

### Phase 1: Overseas Track (P0-P1 English)

**P0 - T1 Core Players Official (6 companies, sequential):**

Search in the following order:
1. Waymo (`site:waymo.com blog robotaxi 2026`)
2. Tesla Robotaxi (`site:tesla.com OR site:ir.tesla.com robotaxi 2026`)
3. Zoox (`site:zoox.com robotaxi 2026`)

For each company, extract:
- Publication date
- Title
- One-sentence summary
- Original URL

**P1 - T2 Industry Players (14 companies, can run 3 in parallel):**

Search in the following order:
1. Momenta / Wayve / 42 Dot / Hyundai RoboRide
2. Mobileye / Motional / MOIA / Avride
3. Cruise/GM / Aurora / XPeng AD / DeepRoute
4. Woven by Toyota / Nuro

Search each company's `query_template`, extracting the same information as above.

### Phase 2: China Track (P0-P2 Chinese)

**P0 - T1 Chinese Players (3 companies, sequential):**
1. Baidu Apollo Go
2. Pony.ai
3. WeRide

**P2 - T3 Industry Media (5 Chinese media, can run 3 in parallel):**
1. QbitAI / LeiPhNet / 36Kr
2. AutoCar AI / D1EV

### Phase 3: Policy & Research (P3)

Search the two T4 sources: NHTSA and China autonomous driving policy.

### Phase 4: Fallback Layer (P4)

**Trigger condition**: Phase 1-3 total valid hits < 5

**Execution**: Sequentially search the 8 topic keywords in `fallback_queries`, taking the top 3 results from each.

**Value of the fallback layer**: Captures industry signals missed by T1-T3, especially from the Chinese media landscape.

## Data Record Format

Each search result is recorded as:

```json
{
  "source_id": "waymo-official",
  "source_name": "Waymo",
  "tier": "T1",
  "title": "Waymo Expands to Miami",
  "url": "https://waymo.com/blog/miami-launch",
  "date": "2026-06-21",
  "snippet": "Waymo announces commercial robotaxi service in Miami...",
  "lang": "en",
  "fetched_at": "2026-06-22T08:00:00+08:00"
}
```

## Failure Handling

| Scenario | Handling |
|----------|----------|
| No results for a company | Record `{name}: No new content today`, do not fabricate |
| WebSearch timeout/failure | Retry once, if still failing then skip and mark |
| Total hits < 5 | Trigger fallback layer |
| Still < 5 after fallback | Output normally, annotate "Limited information today" |
