# Overview
Sends Gulf's pallet pick data to Rehrig Pacific's VOR (Vision Object Recognition) system. VOR runs on the pallet wrappers and RFID dock towers and checks that what's on each pallet matches the order before it leaves the warehouse and goes onto the right truck. Rehrig also wants status pushed back to Ohanafy (pallet verified, loaded, etc.). IRIS, Rehrig's pallet-jack IoT platform, is part of the same vendor relationship.
- Repo: `OHFY-Gulf-Rehrig` (`gulf-rehrig`). The 08-12 plan was a new declared 2GP package, `OHFY-Rehrig`, alongside the existing packaged directories in the repo. Not in OHFY-Split.
- Gulf: **blocked**. gulf-prod's credential points at Rehrig dev and has no secret. Waiting on Rehrig for the production endpoint and API key. The roster shows cutover 10/21.
- Criticality: warehouse dock-out verification at the Rehrig-equipped warehouses. Other warehouses verify by scanning license plates in Ohanafy instead.
- Owners: Bryson Carroll (dev), Drew Heath (PM), Dave Gillette (Rehrig relationship). Gulf: Joshua Smith (owner), Billy Clark, Ashton Pond, Chandler Marston, James Nevin (users). Rehrig contact: Lance.
## Data Flows
- **Out:** PickList (pallet → stops → SKUs) to the VOR API. Sandbox is `vordevapi.rehrig.com/PickList/GULFTSTU/v1`. Our key works on v1 only. The spec we have is v2.3.9, and posting a v2 payload to v1 returns 502 (as of 05-14).
- **In:** status callbacks from Rehrig. The contract was still a spike as of 08-12 (CS-570 / CS-348).
- Required fields with no clean OHFY-Core source (05-14): `distribution_center`, `brand` (no `Brand__c` on Item), `qty_of_packages_per_case`, `pallet_pick_completed_time`. Fields we'd like but can defer: operator, dock door, load number, item dimensions.
## Open Items
- Production credentials from Rehrig (BMS-7115, blocked).
- Kickoff: Gulf contact not confirmed and no recurring sign-off meeting yet (BMS-8374).
- Was v1 vs v2 resolved? Was pallet type taxonomy (CS-569) decided? Confirm with Bryson Carroll.
# Architecture
Planned as a scheduled bulk outbound that is idempotent and credentialed, plus an inbound endpoint that shares the same credential, config, and alerting. Ohanafy is the source of truth for pallets: `autoPalletize` and height-based splitting (BMS-7105) build the pallets, and the outbound fires from `Pallet__c`. Unknown whether this is Batchable or Queueable. Confirm in the repo.
```
Route released for pick ─▶ autoPalletize ─▶ Pallet__c + Pallet_Item__c
                                                  │
                          Scheduled bulk PickList (callout, API key)
                                                  │
                                    Rehrig VOR API ─▶ wrapper / RFID dock tower
                                                  │
                          Status callback (verified, loaded) ─▶ inbound endpoint ─▶ Pallet__c
```
## Cadence
- Unknown. "Scheduled bulk" is all the plan says. Confirm the frequency and how close to pick or load time it runs.
# Load Testing
## SObjects
- `Pallet__c`: the payload root. Status changes to COMPLETED trigger the outbound. There's no pick-completed timestamp field.
- `Pallet_Item__c`: SKU lines
- Indirect: `Item__c` (case dimensions, packs per case), `Delivery__c` / `Route__c` (stops, `Pallet_Type__c` → `Pallet_Type__mdt`), Invoice (Gulf pallets are 1:1 with invoices)
## Triggers
- Unknown. No SOQL or DML measurements yet. Not on the capacity board.
## Known Limits
- Callouts per transaction depend on whether the bulk batches pallets per request. Not designed or measured yet.
- Migrated VIP data has 2,393 itemless pallets (1,999 of them are generated staging placeholders). The outbound should skip pallets with no items.
- See [[Stress Testing]] for record locking on Delivery and Pallet during pick and load.
## Baseline
- No Gulf volumes. No other customer runs Rehrig, so there's no TBM baseline either.
- Rough size from migrated VIP data: 744 real pallets in the history load. Build daily pallet and SKU counts from Gulf's pick history.
