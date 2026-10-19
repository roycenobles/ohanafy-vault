# Overview
Pulls supplier product content (descriptions, keywords, shelf life, image URLs) into Gulf's Items. There are two legs. **API** calls Salsify directly for Molson Coors' catalog. **FTP** is for Constellation: it pushes a Salsify templated export to a Gulf SFTP drop point, and we never call Salsify for it.
- Repos: `OHFY-Salsify-API` (TBM build `OHFY-Salsify`, BMS-5651) and `OHFY-Salsify-FTP-Constellation` + `OHFY-SFTP-Suppliers` (the drop point, about $216/month). Both depend on `OHFY-Integration-Shared`.
- TBM: moved off Tray onto Apex and live in tbm-prod (314 records on 08-10). It ran off a stale April export for 145 days before the live API path was proven 08-31.
- Gulf: **not live on either leg.** API is Blocked (target 10/29). FTP is To Do (target 12/9).
- Criticality: low to medium. It feeds item master data and eComm content, not orders or money.
- Ohanafy: Brian Beach (PM), Bryson Faggart (dev). Gulf owner: Taska Clayton. Gulf user: Diana Wright (lifecycle pages). The roster also lists Kevin Bender and Mindy Walker. Gulf supply-chain owner on the FTP row: Pete Teske.
## Salsify API (Molson)
- Epic BMS-7134. The 36 classes are deployed in gulf-sandbox and gulf-full, and they've been **disabled since 9/23** because there's no Gulf key. The TBM key used in August was removed 9/23. Nothing is deployed in gulf-prod.
- An earlier test with the shared key "pulls too many records" (Bryson, 08-07). Molson never answered.
- Coverage: Molson's channel only carries OSKUs that start with 7. In gulf-full, 39 active Molson items have a Molson number (all start with 7) and 171 have none, so coverage can't be checked yet.
- Build plan: Gulf fields are primary, with fallback to TBM's fields (Matt Keeter asked for package/field differences to be factored in).
## Salsify FTP (Constellation)
- Epic BMS-7539. Scope settled with Naomi Itescu (Constellation) by email 9/21.
- Drop point is live: SFTP user `constellation-gulf`, folder `constellation/gulf/`. Login, upload, and isolation from TBM were tested 10/15, and Brian sent the login to Naomi 10/15. The same login serves Gaps & Mandates (`OHFY-RAS-Constellation`).
- Salesforce reads the drop through Named Credential `ConstellationSftpS3` (IAM `sf-sftp-constellation-gulf-read`). gulf-sandbox reads `constellation/gulf/` and is refused `constellation/tbm/`.
- The parser isn't built yet. It needs Constellation's sample file with the real column headers and org IDs (Bryson, 09-15).
## Open Items
- **API:** no Gulf credential. Need a Gulf × Molson Salsify contact from Cam Koorangi, and a decision on whether TBM's Molson key can serve Gulf (as RAS's does) (BMS-8307, BMS-7116).
- **API:** gulf-prod's packaging-type picklist has 38 values against 130 in sandbox, which fails 12 tests on deploy (BMS-6543). Decide whether prod should match sandbox.
- **API:** deploy targets are TBM-only, so release PR #84 wouldn't reach Gulf. The Salsify download host is missing as a Remote Site in gulf-prod. The `Supplier_Product_Num__c` overlap with [[VIP SRS Out]] needs resolving.
- **FTP:** Naomi needs to confirm the login and drop a test file, then send the sample export (BMS-8262, BMS-8264).
- Both: kickoff contact not confirmed (BMS-8307 / BMS-8278). User stories not signed off (BMS-8340 / BMS-8359).
# Architecture
Apex in the customer org, running a daily scheduled pull (TBM). Unknown whether that's Batchable, Queueable, or Schedulable; confirm with Bryson. Run stats go to `Integration_Config__c` and `Integration_Run_Stats__c`, and a Datadog heartbeat (`salsify_catalog`) covers it at TBM.
```
API leg                                   FTP leg
Salsify API (Molson channel)              Constellation Salsify export
        │ callout (Remote Site)                   │ SFTP push
        ▼                                         ▼
  Catalog pull ─ match OSKU ─┐         S3 drop constellation/gulf/
                             │                    │ ConstellationSftpS3
                             │              Poll + parse (not built)
                             ▼                    │
                     Upsert ohfy__Item__c ◀───────┘
                     (fields, shelf life, image URLs)
```
## Cadence
- API: daily (TBM). Gulf is unknown.
- FTP: the poll interval isn't set yet. It depends on how often Constellation pushes.
# Load Testing
## SObjects
- `ohfy__Item__c`: `ohfy__Key_Words__c` (from B2B Keywords), `Average_Shelf_Life_Days__c` (holds weeks, not days, at TBM), image/asset URLs, `Supplier_Product_Num__c` (shared with VIP SRS), and Gulf org-local item fields (PR #17)
- `Integration_Config__c`, `Integration_Run_Stats__c`, `Watermark__c`
## Triggers
- Item update triggers fire. Which managed automation runs, and its SOQL/DML cost, isn't measured. Unknown; confirm with Bryson.
## Known Limits
- API pull volume: the shared key returns more records than wanted. No paging or limit numbers are known.
- One OSKU can sit on two Items (TBM example 701756), and both get the same values.
- Low risk for [[Stress Testing]]: it's a daily item-master update, not a transactional path.
## Baseline (TBM, not Gulf)
- tbm-sandbox 08-31: 597 products, 313 items updated, 0 failures. tbm-prod 08-10: 314 records/day.
- Gulf has no volume data. Molson coverage is unclear (39 numbered items), and Constellation is waiting on the sample file.
- Board (Tray transition): https://claude.ai/code/artifact/bb444e25-7a14-45de-8b87-0a6f763b1a06
- Readiness: API https://claude.ai/artifact/RMmyhMmkZBPY7yJS55i6KR · FTP https://claude.ai/artifact/R3sD9iNwtMgXb6vgTUKaAR
