# Overview
Sends Red Bull North America (RBNA) a daily file of Gulf's invoices and inventory through its MSA portal (SFTP). VIP sends this file for Gulf today, and we replace it.
- Repo: `Ohanafy/OHFY-RedBull` (listed as `OHFY-Gulf-RedBull` on the roster). Gulf-only, deliberately kept out of VIP SRS. Runs on the shared self-healing integration runtime.
- Gulf: in **shadow mode** in gulf-sandbox and gulf-prod since 09-01. It builds and archives the files but sends nothing until Mode is flipped to Live. Credentials (MSA, MFT Hub / FTP) are in NordPass as of 09-16.
- Criticality: high. Red Bull is a major Gulf supplier, and Joey wants no gap in RBNA reporting at the 12/14 cutover.
- Owners: Brian Beach (PM), Bryson Faggart (dev). Gulf: Joey Irelan (Sr. VP Red Bull, owns the RBNA relationship), Erin Fuqua (Red Bull admin, user). Allen Johnson and Aaron Linton are also listed.
## Files
- Ten record types: HIR/FIR, SIR, IIR, IHR/IDR/IFR, SHR/SDR/SFR. Sales and item files are fixed-format `.txt`. Samples are in `ohfy-tray/docs/redbull-outbound-samples`.
- Every file checks its own trailer totals before it leaves (each SFR against its SDRs).
- Parity: 577 accounts and 175 items diffed field by field against VIP's 7/29 file. 5 mapping bugs were found and fixed, now 0 diff. 106 tests pass in both orgs.
- Layout: RBNA accepts one file per warehouse or one file for all warehouses (10/8). MSA wants the test run as **one combined file**. Bryson says consolidation is done (10/14).
## Open Items
- BMS-8316 (RBNA/Gulf answers, no Gulf owner): does RBNA want a test file before cutover, how to send dollar-only credits, who at Gulf signs off. Credit question never sent per roster.
- Gulf data gaps (for the data load, not exporter work): ship-to address 45%, phone 44%, legal name and chain banner on 259/577, case GTIN on 116/175 items, RBNA per-item number 0/1,837, no store-closure flag (VIP closes 147/577), no promo-code mapping, Birmingham 00407803 vs 00410731. Ship-to comes first, because RBNA says incomplete ship-to can make a file unreadable.
- Related, separate scope: Red Bull weekly reports come from retail execution records (surveys, displays, mandates, gaps). BMS-9458 covers 11 day-one stories. Whether to keep Gulfstream as a bridge or build natively is undecided, and the 10/14 deadline passed.
- Tickets: BMS-8314 (Gulf kickoff; the roster shows BMS-8245), BMS-7123 (build), BMS-7117 (SFTP credential), BMS-8316
# Architecture
A scheduled Apex job builds one complete file per warehouse × section (never chunked) and archives it to S3. An AWS Lambda uploader does the SFTP transfer. A heartbeat watches the flow.
```
Invoices / Inventory / Accounts / Items (gulf-prod)
        │
  Scheduled Apex build (per warehouse × section)
        │  trailer reconcile
        ▼
  S3 archive ──(Live only)──▶ consolidate ──▶ Lambda uploader ──SFTP──▶ RBNA MSA
        │
  Heartbeat (shadow: stops here)
```
## Cadence
- Daily. Scheduled for 13:00 CT in prod as of 09-01. Described as nightly as of 10-01, so confirm the current CronTrigger time.
- Until 09-01 no CronTrigger existed and every run was manual. The heartbeat also couldn't see the flow. Both were fixed.
# Load Testing
## SObjects
- Read-only export. The likely sources are Invoice, Invoice lines, Inventory, Account, and Item. Exact objects and fields aren't documented, so confirm in the repo.
- Config and run ledger come from the shared runtime. Specific objects unknown.
## Triggers
- Doesn't write to the business objects, so it should fire no managed triggers. Confirm. No SOQL or DML measurements yet.
## Known Limits
- **Apex heap:** the combined file is about 46 MB, too big to hold in heap. That is why the build is split per warehouse × section.
- **Lambda payload:** the uploader takes up to 6 MB, about 4,000 lines. One warehouse's file hit it and had to be split in two (10/13). The fix stages the files in S3, then sends the full set at once. Bryson plans to reuse this for VIP SRS.
- Shares the org's scheduled-job slots and async limits with the other integrations. See [[Stress Testing]].
## Baseline (Gulf sandbox, test data)
- 577 accounts, 175 items, 17 real invoices in parity runs. Production daily volume and file sizes per warehouse are unknown. Get them from the shadow archive in S3.
- Board: no capacity board found. Roster: https://claude.ai/artifact/NWvgJ9R8K5cq19Y4ULX9tV
