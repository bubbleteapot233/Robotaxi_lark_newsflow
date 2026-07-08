# Robotaxi Bitable Schema Reference

## App

- URL: `<YOUR_BITABLE_URL>`
- app_token: `<YOUR_APP_TOKEN>`

## Table List

| Table | table_id | Purpose |
| --- | --- | --- |
| Company Info | `<your_table_id>` | Company watchlist and master data |
| Projects & Milestones | `<your_table_id>` | Auto-archive target table, verification queue, sync task queue |
| Company × Region Ops Status | `<your_table_id>` | Current operational status facts per company per city/region |
| Partner Details | `<your_table_id>` | Partnerships and partnership relationships |
| Region Dictionary | `<your_table_id>` | City, country, region, policy status |

## Projects & Milestones Table Fields

### Existing Basic Fields

- Event Name: Text, primary field
- Associated Company: Link to Company Info
- Target City: Link to Region Dictionary
- Entity Classification: Single select, Internal Project / Competitor Dynamics / Industry Dynamics
- Date: Date
- Status Label: Single select, Not Started / In Progress / Delayed / Completed / Cancelled / Ended
- ODD Change: Text
- Event Description: Text
- Event Type: Single select, Autonomous Driving Test / Strategic Partnership / Commercial Operations / Policy Development
- Link: URL

### New Source / Verification Fields

- Source Name: Text
- Source Type: Single select, Company Official/IR / Government/Regulatory / Partner Official / Authoritative Media / Chinese Industry Media / Secondary Reprint/Social Media / Other
- Original Title: Text
- Confirmation Status: Single select, Confirmed / To Verify / Partially Confirmed / Debunked / Deferred
- Confidence Level: Single select, High / Medium / Low
- Pending Confirmation Points: Text
- Verification Notes: Text
- Event Fingerprint: Text
- Latest Verification Date: Date

### New Sync Task Fields

- Affects Master Tables: Single select, Yes / No / To Be Determined
- Tables to Update: Multi select, Company Info / Company × Region Ops Status / Partner Details / Region Dictionary / No update, record event only
- Sync Status: Single select, Unprocessed / Pending Human Confirmation / Synced / No Sync Needed / Sync Failed
- Sync Notes: Text

### New Candidate Structured Fields

- Target City (Original): Text
- Target City (Chinese Name): Text
- Operating Model: Multi select, Self-operated Platform / Third-party Mobility Platform / Technology Supplier / Fleet/Capacity Supplier / Local Operator Partnership / Government/Public Transit / OEM Joint Operations / Undisclosed
- Platform Partner: Text
- Fleet Supplier: Text
- Local Operator: Text
- Current Fleet Size: Text
- Planned Fleet Size: Text

## Region Dictionary Table Key Fields

- City: Text, Chinese primary key
- City English Name: Text, for matching overseas news
- Country: Single select
- Region: Single select, Asia / Middle East / Europe / Americas
- Region Label: Single select, Domestic / Overseas / HK-MO-TW
- Policy Access Status: Single select
- Local Policy Friendliness: Single select

## Company × Region Ops Status Table Fields (complete, as of 2026-06-23)

### Core Identity Fields

- Company - Region: Formula, `Associated Company & " - " & Associated City`
- Associated Company: Link to Company Info table
- Associated City: Link to Region Dictionary table
- Country: Lookup from Region Dictionary "Country"
- Region: Lookup from Region Dictionary "Region"

### Operational Status & Form

- Current Operational Status: Single select, Announced Entry / Road Testing / Demonstration Ops / Paid Ops (with driver) / Fully Driverless Ops / Suspended / Exited
- Project Form: Single select, Robotaxi Commercial Ops / Robotaxi Testing/Pilot / Robobus/Robovan Public Transit / OEM Demo/Product Dev / Technology Licensing / Undisclosed/TBD
- Business Form: Multi select, Robotaxi / Robobus / Robosweeper
- Partnership Model: Single select, RT Tech + Fleet Supplier + Local Operator + Platform Partner / RT Tech + Local Operator / RT Tech + Local Operator + Platform Partner / RT Tech + Fleet Supplier / RT Tech + Local Govt Transit Operator / Vertically Integrated / Undisclosed/TBD

### Partner Association Fields (all link to Partner Details)

- Associated Partners: General association
- Partners: Formula, summary of associated partner display names
- Local Operator/Fleet Manager: Link to Partner Details
- Fleet Supplier: Link to Partner Details
- Platform Partner: Link to Partner Details
- Government/Regulatory: Link to Partner Details

### Fleet & Vehicles

- Current Fleet Size: Text, with status description, e.g., "~50 vehicles (operational)," "200 vehicles (testing)"
- Planned Fleet Size: Text
- Fleet Size Confirmation Status: Single select, Confirmed / Partially Confirmed / To Verify / Undisclosed
- Vehicle/Model: Text

### Intelligence & Sources

- Industry Intelligence: Text, relatively reliable intelligence gathered through private or special channels
- PR Intelligence: Text, publicly available information, may be PR, authenticity to be verified
- Model Source Notes: Text
- Related Links: Text

### Time

- Status Update Date: Date

## Partner Details Table Fields (complete, as of 2026-06-23)

- Partner: Text, partner name
- Partner Category: Single select, Mobility Platform / Fleet Operations / Local Government Agency / Auto OEM / Operations Service Provider / Local Operator/Fleet Manager / Technology Licensee / Other
- Partnership Status: Single select, In Negotiation / Active / Suspended / Ended
- Partnership Summary: Text
- Partnership Effective Date: Date
- Associated Companies: Lookup from associated Company × Region records
- Associated Cities: Lookup from associated Company × Region records
- Reverse Association Fields: 4 links to Company × Region Ops Status (corresponding to Associated Partners, Local Operator, Fleet Supplier, Platform Partner, Government/Regulatory)

## Company Info Table Key Fields (as of 2026-06-23)

- Company: Text
- Company Attribute: Single select, Internal / External Competitor
- HQ (Country): Single select, China / US / UK / South Korea / Germany
- Product Lines: Multi select, Robotaxi / Robobus / Robosweeper / RoboVan / RoboTruck
- OEM Partners: Multi select, Geely / Toyota / Nissan / Hyundai / Renault / Stellantis / VW / BAIC / JMC / Mercedes / Jaguar
- Revenue Structure Tiers: Multi select, Vehicle Sales / Technology Licensing/Software / Operations Revenue Share / Self-operated Ticket Revenue / Government/Public Transit Procurement / Undisclosed/TBD
- Overseas Business Model Overview: Text
- Financial & Strategic Updates: Text
- Fleet Scale: Text
- Operations Details: Link to Company × Region Ops Status
- Operations Overview: Lookup
- Partners: Lookup
- Associated Partners: Lookup (deduplicated count)

## Feishu Field Value Format Reminders

- Single select: pass string, e.g., `"Confirmed"`
- Multi select: pass string array, e.g., `["Robotaxi", "Robobus"]`
- Date: pass millisecond timestamp
- URL: pass object, e.g., `{ "link": "https://example.com", "text": "Source Title" }`
- Link fields: need target table record ID, e.g., `[{"id": "recXXXX"}]`
- Batch update (+record-batch-update): does not support `--yes` flag; JSON format `{"record_id_list":[...],"patch":{...}}`
- Batch write interval: when writing to the same table consecutively, add 0.8-1 second delay between batches to avoid rate limiting (error code 800004135)
- Windows environment: when passing JSON via `@file`, must use Python to write file (`open` + `json.dump` + `encoding=utf-8`), not shell echo redirection, otherwise Chinese will be garbled
