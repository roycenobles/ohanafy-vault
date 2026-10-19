# Overview
Pulls Gulf's fleet telematics from Samsara into Salesforce: vehicles, drivers, geocoded addresses, GPS trips, ELD hours-of-service, temperature readings, and equipment stats (odometer, engine hours). The goal is to link that data to Ohanafy Equipment and Deliveries for route profitability, DOT hours flags, and truck utilization. Domain is routing.
- Repo: `gulf-samsara` (also called `OHFY-Gulf-Samsara`). It's custom Apex calling the `Samsara` Named Credential. It's Gulf-only and not yet in the managed package, and its objects/fields are parked in Data-Model PRs #37/#38.
- Gulf: live in **gulf-prod since 08-25**. The first run was all Success: Vehicles 359, Drivers 380, Addresses 16,089. It ran in the Partial sandbox from July and has 180 days of history landed there.
- Criticality: low to medium. It's read-only enrichment, and nothing downstream (orders, invoicing) depends on it.
- Owners: Mackenzie Fenn (Ohanafy PM, BMS-8240), Bryson Faggart (dev). Gulf owner is Joshua Smith (Dir. Strategic Ops). Gulf users are Billy Clark, Ashton Pond, and Chandler Marston.
## Data Flows
- **In (daily pulls):** Vehicles, Drivers, Addresses, Trips, Routes, Temperature, HOS duty logs, Equipment stats
- **Bridges:** Samsara vehicle → `ohfy` Equipment (by unit number, since 10/13, PR #55). Samsara driver ↔ UKG employee by name + site: 111 auto-linked and 150 name-only matches flagged for review (sandbox, 08-05). Trip → Delivery enrichment is not working yet.
- **Out:** none today. BMS-9643 (send DVIRs from Ohanafy to Samsara) was raised 10-13. Sending routes from Ohanafy routing to Samsara is possible but not built.
## Open Items
- BMS-7190: managed `TestService` null guard is unshipped. gulf-prod was only unblocked because Emily cleared the 4 orphaned `Trigger_Configuration__mdt` rows. Any org carrying those rows can't deploy Apex whose tests touch ohfy triggered objects.
- Trip → Delivery enrichment yields nothing. This is a Gulf data task, not code: Equipment had no VIN or unit key, and delivery drivers resolve to about 23 office Users.
- Equipment upsert on gulf-full dry run: 267 linked, 15 created, and 3 with no warehouse (Panama City 1, Alexandria 2). Unit 3586 appears on two vehicles.
- Release #57 needs a product approver. gulf-prod has no Locations until Drew Heath's data load (12/11).
- Warehouse keys were switched to upsert on the `ODP:` prefix to match the migration (10-16). Josh says confirm with Tim.
- 2 Samsara devices (units 3859, 534101DC) returned permission errors in sandbox. Fix their visibility in Samsara or exclude them.
- Some duty logs show 24h "driving" (unassigned or ELD-exempt). Needs a filter rule before any DOT use.
- No managed fields yet for geolocation in Ops/Routing (09-11).
- User stories are not gathered yet (BMS-5978, BMS-5982). Epic is BMS-5605 and build is BMS-5609.
# Architecture
Schedulable → `Database.Batchable` per entity (e.g. `SamsaraTripsBatch`), making REST callouts through the `Samsara` Named Credential and upserting into custom `Samsara_*` objects. Bridge logic then writes to `ohfy` Equipment.
```
Samsara API ──REST (Named Credential)──▶ Schedulable → Batchable per entity
                                              │
        ┌──────────┬──────────┬───────────┬───┴──────┬──────────┬──────────┐
     Vehicles   Drivers   Addresses    Trips      Routes    Temp/Stats   HOS logs
        │          │          │           │                     │
        ▼          ▼          ▼           ▼                     ▼
  Samsara_Vehicle__c  Samsara_Driver__c  Samsara_Address__c  trips   Vehicle_Temperature_Reading__c
        │          │                      │
  ohfy Equipment   UKG employee bridge    Delivery enrichment (no match yet)
  (upsert on unit #)
```
## Cadence
- All daily, early morning ET. Sandbox schedule: Vehicles 06:00, Drivers 06:15, Addresses 06:30, Trips 06:45, Routes 06:50, Temperature 07:00.
- gulf-prod has 9 Samsara crons (plus 2 UKG = 11). Trips/HOS/Stats first fire between 06:00 and 07:45.
- A single per-vehicle error marks the whole run Failed. The run status overstates failures.
# Load Testing
## SObjects
- `Samsara_Vehicle__c`, `Samsara_Driver__c`, `Samsara_Address__c`, `Samsara_Route__c`, `Vehicle_Temperature_Reading__c`
- Trips, driver duty logs (HOS), equipment stat readings, driver ↔ employee bridge. Custom objects whose API names haven't been confirmed.
- Indirect: `ohfy` Equipment (live gauges, unit-number upsert), Delivery (enrichment target), Location/warehouse (`ODP:` key)
## Triggers
- Samsara's tests touch ohfy triggered objects (Equipment), which is why it hit the BMS-7190 deploy block. SOQL/DML per batch has not been measured. Unknown, confirm with Bryson.
## Known Limits
- Not measured. Volumes are small and daily, so limits risk looks low. Callouts per batch and page size are unknown.
- The Trips pull is per vehicle (about 360 vehicles).
- Uses 9 of the org's 100 scheduled-job slots. That pool is shared with EDI and others (see [[OpenText (EDI)]]).
## Baseline (Gulf)
- gulf-prod first run (08-25): Vehicles 359, Drivers 380, Addresses 16,089, 0 failures
- Partial sandbox, 180 days: 371,069 GPS trips (90% linked to Equipment), 5,923 temp readings (08-05). Trip lat-long 35,633 rows, geocoded addresses 16,140 (09-11).
- No capacity board found. The Samsara Telematics Lifecycle artifact (https://claude.ai/artifact/ELyDhBq8jpgyaAnxQpR1Gk) wasn't readable. Ask Bryson for access.
- Roster: https://claude.ai/artifact/NWvgJ9R8K5cq19Y4ULX9tV
