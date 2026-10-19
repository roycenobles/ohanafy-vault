# Overview
Sends Gulf's alcohol invoices (NA beer included) to Fintech, the EFT network distributors and retailers share. Fintech drafts the retailer's EFT from the CSV we upload, pays Gulf, and returns a remittance file that we reconcile into Payments. For Walmart, alcohol goes this way (Dept 96) and Red Bull/non-alcohol goes through [[OpenText (EDI)]] (Dept 95). The products on the invoice pick the route.
- Repo: `OHFY-Fintech` (2GP, unmanaged). Moved off Tray onto Apex. Live at TBM/TBL since 07-30 and Fort George since 08-04. Lancaster runs a separate 1GP twin.
- Gulf: **sandbox only**. Upload and remittance were proven 10/7 against a Fintech test server in gulf-sandbox and gulf-full. Every leg is in Shadow, and gulf-prod holds no invoices.
- Go-live is moving from Dec 7 to Jan 4 or 11 (Hogan, 10/8).
- Criticality: high. It's Gulf's cash collection for EFT accounts, including Walmart alcohol.
- Owners: Brian Beach (Gulf owner, kickoff), Bryson Faggart (build). Gulf contacts: Whitney Rouse (Fintech owner per Cam) and Lisa Thompson (AR EDI). Fintech contact: Kristin Raub.
## Data Flows
- **Out:** a 13-column invoice CSV goes to Fintech `/IN` by FTP through the Ohanafy bridge. That write is what drafts the EFT.
- **In:** remittance files from `/OUT` (`00<division>.<n>`) → reconcile → `GulfPaymentIntake` (one parent Payment per lump sum, a child per invoice).
- **Gulf difference:** Gulf is on the split data model, so it has no `ohfy__Order__c` (266 references in the TBM code). The port targets `ohfy__Invoice__c`. Gulf bills under 7 legal entities, and Fintech sees 4 Gulf accounts (Goldring Gulf is one). The mapping of accounts to entities and divisions is unresolved.
- VIP's `FINDTLGT` (the Fintech EFT download) is being migrated to `Invoice_Item__c` from the 2010–17 archive copy. That may be the wrong copy and the wrong object (Tim, 9/30).
## Open Items
- 0 of 246 Walmart stores carry a Fintech customer ID, and only 2 test stores have `EFT_Provider = FinTech`. The data migration sets neither field.
- Fintech IDs and prod FTP credentials haven't been issued. They wait on Gulf confirming which of the 4 accounts move (email to Whitney, 9/2). Current status unknown, so confirm with Brian.
- Keg deposits: Fintech bills them but Gulf's `Total_Due` leaves them out, so a Fintech-paid invoice can show a negative balance (Matt Keeter).
- Route for `WALMART NA EFT`, plain `RED BULL`, and `FIJI WATER` invoices is unconfirmed. NA beer vendor number is unconfirmed (962 vs 960). Both are with Lisa.
- `Payment:Intake` is in Shadow on gulf-full. PR #153 to gulf-prod is waiting on a product approver.
- Tickets: BMS-8326 (Gulf kickoff), BMS-6841 (Gulf Fintech), BMS-7998 (payment intake), BMS-7401 (capacity board), BMS-7132 (data-model port)
# Architecture
Each leg is a Schedulable (`FintechLiveScheduler`) that runs once a day and hands off to a chained Queueable doing one page or one file per hop. AWS (a Lambda bridge plus the S3 archive `ohanafy-fintech-archive`) handles transport only. It turns HTTP into FTP on port 21, because Apex can't speak FTP.
```
Completed EFT invoice (gate: EFT_Provider = FinTech, not Processed)
        │
FintechInvoiceUploadQueueable: 200 orders/hop, build CSV rows
        │
Flush at 5,000 lines or end of run: 3 callouts
  bridge PUT ─FTP─▶ Fintech /IN ──▶ EFT drafted
  S3 archive · stamp Processed + Fintech_Invoiced_Amount (1 DML)
        │
Alert hop (separate transaction) → run stats
        ⋮
Fintech /OUT ─FTP─▶ FintechRemittanceQueueable (1 file per hop)
        │
Parse + guards (poison quarantine, HOLD on short file)
        │
Reconcile: 1 SOQL + 1 DML per file → Amount_Paid / status
        │
Gulf: GulfPaymentIntake → Payment records   (TBM: QBO AR4 posts it)
```
## Cadence
- TBM: outbound once a day per leg (15:00/15:30/15:35Z, FG 18:00Z). Remittance poll 18:30Z. Heartbeat 22:00Z.
- Gulf cadence and legs: unknown, likely one leg per Fintech division. Confirm with Bryson.
- The flush is the money write. The stamp that follows it is the only guard against a resend. Un-stamping an invoice drafts a second EFT that can't be undone (BMS-6934).
# Load Testing
## SObjects
- `ohfy__Order__c` at TBM / `ohfy__Invoice__c` at Gulf: `Fintech_Invoice_Processed__c` (outbound cursor, a boolean with no send date), `Fintech_Invoiced_Amount__c`, `Amount_Paid`, `fintech_payment_date`, Payment_Status
- `Fintech_Unmatched_Remittance__c`: worklist (0 rows in both orgs)
- `Integration_Run_Stats__c`, `Integration_Watermark__c` (the remittance cursor is the file number), config row `Mode__c` (fails closed to Shadow)
- Gulf: Payment (parent + child) through `GulfPaymentIntake`. Account `EFT_Provider`, Fintech customer ID.
## Triggers
- **Reconcile DML:** synchronous managed before/after-update on order payment fields. 0 SOQL / 1 DML per 200 orders (TBM, Core #810). Six writers share `Amount_Paid` with no precedence rule (BMS-6941).
- **Outbound stamp:** one bulk update (≤ 4 DML per page). No heavy cascade measured.
- Gulf triggers on Invoice/Payment: not measured.
## Known Limits
- **Page:** 200 orders per hop. Up to 5,000 lines (~565 KB) held in heap as a single string, plus serialized queueable state. Medium memory risk.
- **Flush:** 3 callouts per flush. Nothing caps the file size. Largest seen: 397 KB, 3,521 rows.
- **Remittance:** one file per transaction, up to 500 files per run. The parser holds the whole file, so heap scales with file size.
- **Inbound queueable has no try/catch.** A crash leaves no stats row and sends no email. A large re-run can throw STRING_TOO_LONG on the watermark detail.
- `Run_Detail__c` is empty on 100% of rows, so post-mortems are thin.
- `FintechCapacityTest` and `FintechEndToEndTest` are sandbox-proven (9/3) but not in TBM prod. Gulf coverage unknown.
- Overlaps with [[Stress Testing]]: catch-up bursts after an outage (the duplicate-EFT risk) and lock contention on invoices during reconcile.
## Baseline (TBM / FG prod, 14 days to 09-03, not Gulf)
- TBM outbound: 1,123 orders / 21,531 lines (max 340/day). TBL: 425 / 8,265. FG: 293 / 2,884.
- Remittances: TBM 1,544 orders / $1.47M. FG 299 / $137K. Files are 8–16 KB (20–418 rows).
- Queueable time: upload median 10.5 s (max 26 s), remittance median 4 s (max 12 s). 0 failed jobs.
- Gulf: 365-day Walmart history alone is 1,803 BEER + 1,009 NA BEER invoices on the Fintech route. Gulf's full EFT volume across all accounts is unmeasured.
- Board: https://claude.ai/code/artifact/973d730f-eb6a-4d57-a7f7-f2b36ffd5b03
- Walmart route split: https://claude.ai/artifact/7DMkMehnuabG5bqNsNw3aC
