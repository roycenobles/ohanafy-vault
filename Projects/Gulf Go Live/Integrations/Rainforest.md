# Overview
Charges customers' stored payment methods (ACH or card) for invoiced orders through Rainforest, then polls Rainforest until each charge settles or is returned and writes the result back to the order. It also covers pay-by-email presentment, method capture, and the merchant dashboard session.
- Repo: `OHFY-Rainforest` (2GP). It sits on OHFY-Core payment fields and feeds QBO AR4 downstream. Gulf's version ships alongside the new OHFY-Split packages (Rainforest → Split, BMS-7583, Alvaro). The new package is named `OHFY-Pay`, and there is a processor-agnostic seam so we aren't locked into Rainforest.
- TBM: moved off Tray onto Apex (BMS-5597). Live 08-06 18:46Z. The first charges went out in the 21:00Z batch (32 orders).
- Gulf: **blocked** per the roster. Not started in gulf-prod. Gulf also uses it for ecomm storefront checkout (saved-card selector working in testing as of 09-11).
- Criticality: high. This leg moves money and writes `Amount_Paid__c`.
- Owners: Gulf owner Whitney Rouse (Dir. Accounting), Gulf user Lisa Thompson (AR lead), PM Brian Beach, dev Alvaro Sanchez.
## Open Items
- **Credential scope (BMS-7119, BMS-6811):** the Rainforest API key identifies the *platform*, not a merchant, and there's no server-side entitlement check. If Gulf runs on TBM's key, each company can act on the other's payments, refunds and voids included. Options were: accept the risk, get merchant-scoped keys, get a second Rainforest account (Bryson's "probably only path"), or switch processors. Separately, drop the stored `group#all` session token on `Payment_Configuration_Pmt__c`. Waiting on Alvaro.
- Ecomm: we charge after the order commits, so insufficient funds still places the order and nothing escalates it. A zero-dollar auth at checkout was proposed.
- Method sweep is still Shadow at TBM. The Tray webhook for method changes and returns died 08-30. Decide Live or unschedule before Gulf.
- Tickets: BMS-8343 (Gulf kickoff), BMS-8344 (user sign-off), BMS-7402 (capacity board), BMS-6941 / BMS-6956 (Amount_Paid writers)
- Gulf go-live date: unknown. Confirm with Brian/Alvaro.
# Architecture
Each leg is a Schedulable that enqueues a chained Queueable with callouts, one clamped page per hop. `RainforestCalloutBudget` clamps every page to the 100-callout limit (2-callout reserve plus a 100 s slice of the 120 s callout-time limit).
```
Order invoiced w/ stored method ──▶ Payin gate (no open ledger row)
                                         │
              Payin 21:00Z ─POST /payins─▶ Rainforest ──▶ ACH / card rails
                     │                                        │
         ohfy__Payment_Pmt__c (ledger row = cursor)      returns ≤60 d
                     │                                        │
     Status poll :05 ─GET─▶ settle: Amount_Paid = booked      │
     Settled re-poll :50 ─GET again─▶ return: order Failed, Amount_Paid reduced
                     │
     [managed payment triggers] ──▶ QBO AR4 posts Payment ──▶ bank rec
Flow "Charge Order via Rainforest" → 1 Queueable per order (on demand)
```
## Cadence
- Payin: daily 21:00Z, page 24 (configured 40, clamped at 4 callouts/order)
- Status poll: hourly :05, page 49 (configured 90, clamped at 2 callouts/row)
- Settled re-poll: hourly :50, page 49 over a 60-day window
- Session key: 09:00Z. Method sweep: 11:15Z, scan 100, **Shadow**
- Invoices sit in Processing 3–5 days for ACH settlement before going Paid
# Load Testing
## SObjects
- `ohfy__Payment_Pmt__c`: the ledger row. Holds the payin id, the frozen charged amount (`ohfy__Amount_Pmt__c`) and status. Both polls use it as their cursor.
- `ohfy__Payment_Method_Pmt__c`: stored methods
- Order: `ohfy__Payment_Method__c`, Payment_Status, `Amount_Paid__c` (unnamespaced), `Accounting_Payment_ID__c` (legacy, inert in the gate)
- `Payment_Configuration_Pmt__c`, `Integration_Config__c` (5 rows), run stats / heartbeat (`Run_Detail__c`, `Queue_Depth__c`)
## Triggers
- **Order + method gate:** synchronous managed Order triggers, 200 per chunk. The gate is 1 SOQL per page. No capacity test.
- **Payment fields:** managed before/after-update on every settle and reversal. 2 SOQL / 2 DML per 200 orders (Core #810, tested).
- **Amount_Paid writers:** 4 writers across 3 repos with no precedence rule in code (Dixie EFT, Fintech reconciler, status poll, re-poll). Last writer wins. Risk is HIGH for correctness.
## Known Limits
- Status poll writes the money field and has **no capacity test** at a full page of 49.
- Re-poll: 718 settled rows against 49/hr re-checks the window about 1.5×/day. At about 4× TBM volume, detection slips past a day, and there's no alarm on the ratio.
- Flow charge: 1 Queueable per order with no batching. A bulk Flow can hit the 50-enqueue governor, and a double click charges twice (idempotency key is versioned per attempt, BMS-6943).
- Only the status poll populates `Queue_Depth__c`. Payin, re-poll and session leave it null, so their stall alarms can't fire.
- Returns aren't unwound in QBO. AR corrects them by hand.
- Overlaps with [[Stress Testing]] (catch-up bursts) and [[Fintech (FTP)]] (shared `Amount_Paid__c`).
## Baseline (TBM prod, 08-20 → 09-03, not Gulf)
- 184 payins / $123,023.46 in 14 days (183 succeeded, 1 returned on day 5). 4,596 all-time. 199 stored methods.
- 360 async jobs, 0 failed. Payin median 10 s (max 26 s). Status poll median 1 s. Re-poll median 7 s.
- 3 of 11 nodes have capacity tests. `RainforestSyncTest` (58 methods) is the only one that asserts callouts.
- Gulf volume: unknown. Size it from Gulf's AR / payment-method counts.
- Board: https://claude.ai/code/artifact/7e1b8d07-a0eb-4b83-b6d6-845d550e9fd3
