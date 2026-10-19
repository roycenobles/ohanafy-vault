# Overview
Sends Gulf's depletions to VIP Supplier Reporting (vtinfo) so suppliers see sales and inventory by distributor. Every night it builds fixed-width VIP files from completed orders and inventory moves (SLS sales, INV inventory, plus OUT outlets, ITM items and SLM salesmen snapshots), one report per warehouse. It replaces what Gulf's VIP system sends today.
- Repo: `OHFY-VIPSRS` (deployed as source, VIP objects unmanaged). Reads from Core (`ohfy__` orders, lines, inventory adjustments and history). Transport is the Ohanafy relay (AWS Lambda + S3 archive), then SFTP to vtinfo.
- Gulf build is a port to the Split data model. Gulf has no `ohfy__Order__c`, so the Core-era version wouldn't compile there (BMS-7132). Three field mappings were caught wrong during the port, before anything transmitted.
- Gulf: in gulf-prod in **shadow (Dry_Run)** since 08-25. It builds and archives files to `dryrun/Gulf/` and never contacts vtinfo (relay creds deliberately non-functional). Molson (MCJ) enrolled at Mobile and Montgomery on 10-15. The first files (10-16) were empty because Gulf's data load is still running.
- Live at TBM (since the 07-28 cutover) and Fort George. VIPRAW is not live at Gulf.
- Owners: Brian Beach (PM), Bryson Faggart (dev). Gulf owner Cam Koorangi, who took it over from Pete Teske. Gulf users are a guess on the roster (Drew Steber, Jemison Burns). Roster due date 10/29.
- Criticality: suppliers depend on it for depletion reporting. Unknown whether Gulf can go live without it. Confirm with Brian.
- Related: [[RedBull MSA]] (Red Bull runs on its own relay since 10-14). VIP SRS IN is a separate integration (BMS-8275, blocked on Gulf's renew-or-replace decision).
## Open Items
- **Enrolment IDs:** the feed produced 0 records in the Full sandbox because distributor and supplier enrolment IDs were empty (BMS-8583). There's an open question whether VIP or Gulf supplies them. Elliot argued Gulf can pull them from its own VIP setup. VIP distributor IDs are still missing for Birmingham, Huntsville, Milton, Jackson and Gulfport. Supplier codes may not carry across distributors, so VIP needs to confirm them.
- **Data:** the item file came out empty on 10-16 (items maybe not linked to Molson). The SUPPLIERT table is missing 12 supplier codes (BMS-9492).
- **Inventory history:** the history job and watermark seeding need to be scheduled at cutover (BMS-8246). Without them, INV goes out with no on-hand totals.
- **Decisions:** how to scope OUT outlets for Gulf (it has no Account Item link, so OUT currently lists every customer). Whether to turn on B records for bottles and returns (BMS-8230, built but off).
- **Filenames:** they carry the internal bucket key (`OGMOB001.OUT`). Check the naming before Live.
- **Recovery:** there's no same-day recovery cron in gulf-prod yet. Only Fort George has the 14:00Z sweep.
- Tickets: BMS-8318 (kickoff), BMS-8319 (user sign-off), BMS-8320 (parity/UAT, To Do, waiting on VIP Help Center and Pete Teske), BMS-7400 (capacity board, epic BMS-7399)
# Architecture
Every stage is a Schedulable or a scope-1 Batchable. It has been Queueable-free since #149 (09-15) under the chained-Queueable retirement (BMS-7505 / BMS-7503). The one exception is `VipSrsAssembledSend`, a self-re-enqueueing Queueable added 10-15. It runs only when `One_File_Per_Supplier__c` is on, which is off everywhere. Convert it before anyone turns it on.
```
Complete orders + lines ──▶ SLS     Inventory adj + history ──▶ INV
              │                                   │
   VipSrsSchedulable (nightly) ─┬─ VipSrsUnitJob: OUT/ITM/SLM snapshots ──┐
                                └─ VipSrsRollingWindow (D-7..D-1)         │
                                     generate segments (9,000-line scope) │
                                     stage + ledger (changed days only)   │
                                     VipSrsStagedSendJob (1 file/scope)   │
                                              │                           │
                     Relay Lambda /deliver (120 s) ◀──────────────────────┘
                                              │
                     SFTP PUT → vtinfo   ·   S3 wire archive (proof of send)
   VipSrsWeeklySweep (Sun, D-8..D-120) · WireReconcile · StatsBridge · Recovery (FG only)
```
## Cadence
- Nightly run: 01:23 local (08:23Z gulf-prod). It rebuilds the last 7 days for every enrolled supplier.
- Sunday sweep: 09:00 local, rebuilds D-8..D-120. Only changed supplier-days are sent.
- Wire reconcile: 06:45. Stats bridge: 07:15. Failure refresh: hourly at :20.
- Same-day recovery: 14:00Z, Fort George only.
# Load Testing
## SObjects
- Core: `ohfy__Order__c` / lines (TBM model; Gulf uses the Split equivalent), `ohfy__Inventory_Adjustment__c`, `ohfy__Inventory_History__c`
- `VIP_SRS_Reporting_Setup__c`: one row per supplier × location, holding `Distributor_ID__c`. gulf-prod has 2 rows (Molson at GMOB/GMON); TBM has 83.
- `VIP_SRS_Config__mdt`: one per org. `Is_Active__c` gates the run and `Dry_Run` blocks sending.
- `VIP_SRS_Staged_Segment__c`, `VIP_SRS_Segment_Ledger__c` (fingerprints delivered supplier-days), `VIP_SRS_Execution__c`, `VIP_SRS_File_Execution__c`, `VIP_SRS_Alert__c`, `Integration_Run_Stats__c`
## Triggers
- No VIPSRS triggers. It reads Core data on a schedule.
- Managed Core triggers on Order and line DML. `TA_Invoice_BU_PaymentInfo` forces the invoice date to the delivery date on every non-Draft update, so the delivery date drives SLS dates.
- No measured SOQL/DML for these triggers. An INV scope at the 6,000-row day cap passes its SOQL/DML/CPU/heap budgets (`VipSrsRollingCapacityTest`).
## Known Limits
- **File size:** 8,400 lines / 2.9 MB per file. Building a file peaks at about 2× the file in heap (47% of 12 MB). Relay requests are kept ≤ 4.4 MB against the 6 MB limit.
- **Relay:** 120 s Function URL timeout. The old 29 s API Gateway route is retired but still invocable.
- **Sunday sweep:** runtime grows with daily volume. It took 2 h 15 m (279 scopes) at TBM on 10-11. 10-18 is the first sweep with Gulf enrolled.
- **Snapshots:** OUT is the largest (up to 3 MB) and untested at volume. Gulf's OUT is split by supplier group because one transaction couldn't hold it (#290).
- **Lateness:** an order completed more than 120 days after its invoice date is never sent and needs a restatement. vtinfo sends anything older than last month to manual processing.
- **Capacity tests:** only 4 of 11 VIPSRS nodes have a capacity test (fan-out, rolling scope, send, one-file-per-supplier). Snapshots, segment generation, the sweep and recovery have none.
## Baseline (TBM prod, not Gulf)
- 16–17 files a night (down from 71–142 before the rolling protocol). Each supplier is in one file per warehouse.
- Rolling window: 17 scopes, median 640 s. Send: about 42 s a night. The week of 10-10 had 0 failed jobs.
- SLS 278k and INV 354k records over the week of 10-10..10-16. The largest SLS file was 8,398 records / 2.8 MB.
- Gulf: no volume yet (the 10-16 dry-run files had 0 records). Build Gulf's numbers once the data load lands.
- Board: https://claude.ai/code/artifact/ea8d7cbb-b10d-45dc-80fb-beab6104c31f
