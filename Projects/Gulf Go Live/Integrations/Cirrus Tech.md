# Overview
Connects Gulf's warehouse system, Cirrus Tech, to Ohanafy. Cirrus calls Ohanafy's API in Gulf's org for products, pallets, bin counts and backstock. Gulf runs these steps by hand today and expects them to be automatic after go-live.
- Repo: `OHFY-Gulf-Cirrus`. Net-new for Gulf, so there is no TBM precedent. Epic BMS-5607, build BMS-7124.
- Gulf: code is in all three orgs (gulf-sandbox, gulf-full, gulf-prod), but **nothing can authenticate yet**. gulf-sandbox has run partial syncs. Roster status is Blocked, due 10/16.
- Criticality: high for warehouse ops (picking, storage, pallet labels). The board lists it under Fleet/Delivery, but it is a warehouse integration.
- Owners: Bryson Carroll (dev). PM is Brian Beach on the readiness board and Drew Heath on the roster, so confirm which. Gulf owner Joshua Smith (roles approved 10/8). Gulf user Ashton Pond (Ops Manager). Cirrus contact Chris Fonke.
## Data Flows
- **Cirrus → Ohanafy (pull):** products, pallets, bin counts, backstock.
- **Ohanafy → Cirrus (planned):** Ohanafy will host outbound requests for full pallet movements (9/15 call). Data contracts are still Bryson's to define.
- **Inventory push:** total count vs delta is undecided. Cirrus (Chris Fonke) to state a preference, Bryson to propose.
- Labels use Code 128 barcodes (confirmed 9/15).
## Open Items
- **Blocker:** Cirrus hasn't sent its real public certificate. The connected app in gulf-prod and in the repo still holds `REPLACE_WITH_CIRRUS_PUBLIC_CERT` (as of 10/8). BMS-5611, which also blocks connection proof (BMS-8566).
- Kickoff contact not confirmed (BMS-8380). User story sign-off by Ashton Pond not scheduled (BMS-8381).
- gulf-sandbox's four Cirrus config rows are **Live**. Set them to Shadow (BMS-8382). Remove 13 leftover classes from older builds in gulf-sandbox.
- gulf-full and gulf-prod have 7 classes but no config rows. Add the rows once the certificate is registered (BMS-8582, BMS-8383).
- Gulf to name the pilot warehouse for the parallel run, hypercare coverage, and trained operators by warehouse (BMS-8384). Cutover BMS-8252, live confirmation BMS-8385.
- Cirrus to provide non-prod URLs (straight IPs) for sandbox testing.
# Architecture
Inbound: Cirrus calls Ohanafy's API through a connected app that uses certificate auth. Exact endpoints and Apex classes (7 per org) aren't documented. Unknown whether any leg is Batchable or Queueable. Confirm with Bryson Carroll.
```
Cirrus Tech (WMS) ──cert auth──▶ Connected app (gulf org)
                                      │
                      Ohanafy API: products · pallets · bin counts · backstock
                                      │
                         Integration_Config__c rows (4) gate each feature
                                      │
                      Ohanafy Item / inventory records (unconfirmed)

Ohanafy ──full pallet movements (planned)──▶ Cirrus Tech
```
## Cadence
- Unknown. Cirrus initiates the calls, so their schedule or event model sets the load. The inventory push model (total vs delta) is still open.
# Load Testing
## SObjects
- `Integration_Config__c`: four Cirrus feature rows (products, pallets, bin counts, backstock), plus run stats
- Target inventory objects (Item, pallet, bin) aren't documented. Confirm with Bryson Carroll.
## Triggers
- Unknown. No trigger, SOQL, or DML measurements exist yet. Inventory and Item writes probably hit managed triggers. Check against [[Stress Testing]].
## Known Limits
- None measured. The open risk is volume: an inbound API call per pallet or bin update, especially if Cirrus pushes full totals instead of deltas.
## Baseline (gulf-sandbox, 10/6)
- Products: 680 synced. Pallets: 0 synced, 1 failed. Bin counts and backstock: enabled, no records yet.
- No production volumes. Build them from Gulf's warehouse transaction counts once the pilot warehouse is named.
- Board: https://claude.ai/artifact/27Hw5gV4Q4tTTyLEYMutHL · Roster: https://claude.ai/artifact/NWvgJ9R8K5cq19Y4ULX9tV
