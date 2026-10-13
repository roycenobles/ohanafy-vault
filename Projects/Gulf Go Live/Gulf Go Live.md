# Overview
This is the single most important step in our progression as a company. There are dozens of large distributors watching how this process goes, and success here will secure our future. It cannot fail. It will not fail.

https://gulfdistributing.com
## Key Dates
- September 28, 2026 - Pilot began
- October 19, 2026 - One year of data loaded in sandbox
- October 26, 2026 - [[Mobile Offline]] testing rollout
- January 4, 2026 - Go-Live (updated from 12/14)
# Areas of Concern
## Offline Mobile Experience
- Salesforce Mobile Offline SKU was dropped and also deemed insufficient for Gulf.
- Started the [[Mobile Offline]] project to deliver durable offline mode.
	- Consists of Sales and Driver experience for pilot
	- Need clarification on warehouse for go-live
	- Eventually needs full experience
## Integrations
Gather the following for each of the [[Integrations]] and estimate impact at scale.
Validate capacity in [[Stress Testing]] plan.
- Business criticality
- Type and timing (is it scheduled or run on demand?)
- Involved SObjects and triggers
- Aproximate processing volume
## Historical Data
Validate volume in [[Stress Testing]] plan.
- Gulf environment must support 3 years of historical data without performance issues or instability.
- Integrations must perform reliably at the scale of data volume in Gulf environment.
- Prove this on 9 years (3x) of data in a Large Data Volume environment.
### Open Concerns
#### Data Loading
What if any problems might arise with this process?
* Difficulty staying in synch
* Mapping issues going undetected
#### Trigger scaling
Understand what triggers exist and how selective they are with data.
* Bulkification issues
* DML limits
* Sensitivity to bad data
#### Data-heavy workloads
Are any current workloads data-dependent is complicated ways?
- Unbounded SELECT queries
- Sensitivity to data skew (Account with 10k contacts, etc)
##### Examples
- Forecasting (needs to forecast based on SKUs)
- Accounting (journal entries - high volume)
- Inventory Adjustments (movement of inventory - wide footprint)
- Pricing (likely to become more problematic over time - initial volume + historical)
### Testing Plan
#### 1. Build a volume model
- **Records per object:** For each high-volume object (orders, invoice lines, inventory transactions, deliveries, depletions, pricing), estimate the daily count, the busiest-hour count, and the month-end count.
- **Account for monthly variance:** Business may be highly variable by month. Estimate data volume across each month and base targets data volume targets on at least 2 x (12 x max month).
#### 2. Run a “day in the life” test in the full copy
- Run the integrations at production cadence and volume while scripted users work through the peak windows: morning order cutoff, driver load-out, end-of-day settlement.
- Test recovery too. If an integration is down for 4 hours, what happens when it catches up all at once? That’s often worse than steady-state traffic.
- Watch Salesforce limits: daily API calls, concurrent long-running requests, the async Apex daily limit, the 100-job flex queue, and Bulk API batches.
- Full copy sandboxes don’t run on production-equivalent hardware, so treat results as directional. Check whether Gulf’s edition includes Salesforce Scale Center or Scale Test.
#### 3. Things not on your list that often cause go-live problems
- **Record locking (`UNABLE_TO_LOCK_ROW`):** This is the classic failure for distributors. Inventory and item records get hit at the same time by orders, receipts, adjustments and integrations, and roll-ups make it worse. Test concurrent writes to the same item and warehouse specifically.
- **Data skew:** Watch for any parent with more than about 10,000 children, such as a generic “cash customer” account, a catch-all item, or one integration user who owns every record. Skew causes locking and slow sharing recalculation.
- **Query selectivity at full volume:** Queries that work in a small org can fail or time out once objects hold millions of rows. Review trigger and report filters against indexed fields, and request custom indexes from Salesforce early, because that takes time.
- **Overlapping automation:** Ohanafy’s package triggers, Gulf-specific Flows, validation rules and roll-ups all fire on the same save and share one set of limits. Map out what runs on your heaviest objects.
- **Batch job collisions:** List every scheduled job (pricing, invoicing, EDI, supplier depletion reporting, route planning) on a 24-hour timeline. Look for overlaps with each other and with peak user hours.
- **Mobile and handheld sync:** Many drivers and reps syncing at 6am is a concurrency spike of its own.
- **Reports and list views:** Make sure the reports Gulf depends on don’t time out at full data volume.
- **Storage:** Check data and file storage against the production allocation, including field history tracking and logs.
- **How much history to load:** Currently targeting "3 years across the board". Push back in areas with outsized effect and unclear need. (e.g. daily routes, etc)
#### 4. Data load and cutover
- **Rehearse the load at least twice,** timing each step. One run should be a full rehearsal of the actual cutover, including the delta load.
- **Avoiding locks during the load:** Sort child records by parent ID, use serial mode where locking appears, defer sharing calculations, and turn off email deliverability.
- **Reconciliation:** Prepare record-count and dollar-total checks against the legacy system, with sign-off owners named.
- **Backout plan:** Decide what triggers a rollback and what the legacy fallback looks like.
#### 5. Hypercare readiness
- **Monitoring:** Set up dashboards for API usage, failed async jobs, integration errors and a custom error log, with alerts set below the limits rather than at them.
- **Integration resilience:** Integrations should be idempotent (upsert by external ID) and retry with backoff on lock errors.
- **Change control:** Freeze code and configuration around go-live, and check Salesforce’s maintenance calendar for December.