# Overview
Pulls employees, time punches, earnings (hours per pay code) and time off from UKG Workforce Dimensions into Gulf's orgs once a day. It's inbound only and writes nothing back to UKG. The data feeds labor cost for budgeting (BMS-5023) and labor availability for routing (BMS-5124).
- Repo: `OHFY-Gulf-UKG`, which depends on `OHFY-Integration-Shared` (`Integration_Config__c`, `Integration_Run_Stats__c`). Auth runs through the `UKG_Dimensions` Named Credential.
- Gulf: **live** in gulf-prod since 08-25 with 5 clean days confirmed (BMS-8310). Since 10-15 it also runs Live in gulf-full and gulf-sandbox on the same code, at staggered times.
- Criticality: low for go-live. Nothing in order-to-cash depends on it, and today no Gulf person can even see the data.
- Owners: Bryson Faggart (dev), Brian Beach (PM). Gulf owner Beth Sayler (MD, HR). Gulf user Rachael Franklin (Sr HR Ops Mgr).
- Alerts go to Ohanafy only (`#datadog-alerts` plus a FAC ticket). Gulf-facing alerts were dropped 10-06.
## Data
- **Employees:** `/commons/persons/{key}` → `Employee__c`
- **Timecards:** one `multi_read` call → `Time_Punch__c` and `Earnings_Line__c`. `/timekeeping/setup/pay_codes` classifies the pay codes.
- **Time off:** → `Time_Off_Day__c`. `Blocks_Assignment__c` keeps the person off routes that day.
- Pay is a placeholder. Real pay left scope on 08-28, so `Hourly_Rate__c` holds $1 and reports sum hours.
- Leavers are re-read from UKG and marked Terminated/Inactive (release #68, 10-15).
- [[Samsara]] drivers are linked to UKG employees by name plus site: 111 auto-linked and 150 flagged for review (sandbox, 08-05).
## Open Items
- **No Gulf users in gulf-prod.** The only 2 active users are service accounts. Gulf users are due 12-04 (Drew Heath).
- **Employee ↔ User link: 0 linked in every org.** UKG returns no work email, so the join uses the UKG person number in `User.ohfy__Employee_Code__c`. That field shipped in the package (BMS-9353) but isn't installed in gulf-prod, and no owner is named to install it.
- **Acceptance (BMS-8308, blocked):** no Gulf user has validated the data. Still open is who checks a week of punches and earnings against UKG: Rachael or payroll.
- **Rate limit (10-16):** the hourly runs logged in once per batch, about 4,000 logins a day. UKG locked the client with 429s, and all 3 orgs went back to daily. Fix #94 (one login per sync) has to be proven before going hourly again.
- `ohfy__Employee__c` isn't in Gulf's data-model package yet (PR #608 / BMS-6012, open since 07-22).
- Location codes need a warehouse map. PCB, ALX, DOT and MCS (140 employees) have no Ohanafy warehouse. Entities and warehouses won't reach gulf-prod until Gulf's data load (12-11).
- Tickets: BMS-5604 (epic), BMS-8239 (kickoff, done), BMS-5977 (user stories, to do), BMS-5176 (REQ-249 paid hours vs pick productivity, in review)
# Architecture
Scheduled Apex batches, one per sync (Employees, Timecards, Time Off), page through UKG REST callouts and upsert on UKG keys. Nothing is ever deleted: punches UKG replaces are flagged `Superseded__c`. The exact class names and the Batchable/Queueable split are unconfirmed (confirm with Bryson Faggart).
```
UKG Workforce Dimensions (REST, Named Credential)
        │
  Employees sync ──▶ Employee__c (upsert Person_Key__c)
        │                    ▲ lookup
  Timecards sync ──▶ Time_Punch__c (Punch_Key__c, Superseded__c)
        │          └▶ Earnings_Line__c (Earnings_Line_Key__c)
  Time Off sync ───▶ Time_Off_Day__c (Time_Off_Key__c)
        │
  UKG_Entity_Stats__c ──▶ UKG Sync Alert ──▶ #datadog-alerts / FAC
  Employee__c.Salesforce_User__c ◀── User.ohfy__Employee_Code__c (not in prod)
```
## Cadence
- Daily: Employees first, then Timecards 30 min later, then Time Off. Reported times vary by source (06:00/06:30 UTC, 4:00/4:30 AM, 6:00/6:30 ET). Next runs are 10-17 from 09:00 UTC.
- Hourly was tried and reverted on 10-16 (see Open Items).
- Each sync's schedule, page size and endpoint are set in `UKG_Sync_Config__c`.
- It had 2 crons in gulf-prod at 08-25, out of 11 integration crons there (9 belong to Samsara). Time Off was added later, so the current count is unconfirmed.
# Load Testing
## SObjects
- `Employee__c`, `Time_Punch__c`, `Earnings_Line__c`, `Time_Off_Day__c`: custom objects in the repo, not in the package
- `UKG_Sync_Config__c`, `UKG_Entity_Stats__c`: config, plus the last run per sync
- `Integration_Config__c` (`UKG:Employees`, `UKG:Timecards`, `UKG:TimeOff`), `Integration_Run_Stats__c`
- Indirect: `User` (employee-code link). It doesn't touch `ohfy__Picker_Shift_Presence__c` or `ohfy__Worker_Presence__c`.
## Triggers
- None. UKG's objects have no ohfy trigger service. That's why it could deploy to gulf-prod on 08-25 while the TestService CMDT bug (BMS-7190) blocked [[Samsara]].
- SOQL/DML per page hasn't been measured.
## Known Limits
- **UKG API logins:** one login per batch hit UKG's lockout at hourly cadence. This is the binding limit, not Salesforce governor limits.
- Callouts per page and the page size haven't been measured. Confirm them in `UKG_Sync_Config__c`.
- The volume is daily and off-hours, so it's unlikely to contend with the [[Stress Testing]] scenarios. It does share the org's scheduled-job slots and its daily async limit.
## Baseline (Gulf)
- **gulf-prod (10-15):** 1,450 employees (1,336 active), 64,824 punches (since 08-23), 33,501 earnings lines, 666 time-off days. The day's run synced 1,335 employees, 8,494 timecard rows and 337 time-off records, 0 failed.
- **gulf-sandbox (08-05):** 180 days of history. Daily runs: employees 1,385 rows in 35 pages, timecards 7,258 rows in 28 pages.
- **gulf-sandbox (10-15):** 265,383 punches, 141,312 earnings lines
- Board: https://claude.ai/artifact/7waXnfHMnXMMPcaBGuWgFe · Roster: https://claude.ai/artifact/NWvgJ9R8K5cq19Y4ULX9tV
