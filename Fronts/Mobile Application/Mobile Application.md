---
type: front
title: Mobile Application
status: active
as_of: 2026-09-23
owner: Royce Nobles
tracks:
  - App development — Jeff (dev), Emily & Elliot (domain), Royce (architecture)
  - App store registrations — Royce (complete)
  - Enablement — Mack, with Royce (pre-planning)
  - Release planning past pilot — Elliot, Matt, Josh, Royce
milestones:
  - 2026-10-12 Handoff to Gulf (moved from 2026-09-28; not a go-live-ready date)
  - 2026-12-14 Gulf rollout window ends (14 weeks from Sep 8)
related:
  - "[[Gulf Pilot]]"
  - "[[AI Strategy]]"
sources: journal through 2026-09-22, corrected with Royce 2026-09-23; not yet reconciled against Slack or Jira
---
> Front note for Ohanafy Mobile, the field app for Gulf and future customers. **Status** and **Open threads** are updated in place at each reconcile (see `as_of`). **Log** is append-only. **Reference** links out; nothing authoritative is copied in.

> [!quote] Requirement (Elliot Flores)
> As a tech business we need an app that supports a full route day with no connectivity issues, including printing, and starting a new order with zero signal AND supports scale (1,500 field users) following enterprise security requirements.

# Status

**Why this exists.** The existing mobile experience does not meet the acceptance criteria above. Ohanafy is building Ohanafy Mobile, its own field app, with Gulf as the first deployment. Context: Salesforce discontinued Mobile Offline for new Mobile App Plus contracts effective 2026-07-31 ([[SF OEM Email.png|SF OEM email]]).

**Where it stands (2026-09-23).** Handoff to Gulf is Oct 12 (moved from Sep 28); it is a handoff, not a go-live. Both store registrations are complete and TestFlight is in active use; the Apple App Store review of the iOS build is expected in the next few days, using Jeff's review org for demo credentials. Requirements review is finished; scope has grown several times and is now stabilizing. The separate printing app is being folded into the mobile app: a handoff meeting has happened and the remaining work sits with Jeff and Royce. Enablement pre-planning is underway with Mack. Gulf field population is roughly 500 users (drivers, merchandisers, reps), with a first rollout wave of 50; the requirement is sized for 1,500. Ian and Hogan own the external messaging for the handoff, with Elliot refining details; the current status of that communication is unknown.

**Risks being watched.**
- Scope growth: stabilizing, but it has moved several times since Sep 10.
- Apple App Store review outcome and timing against the Oct 12 handoff.
- Warehouse/inventory is the sensitive part of the system; deferred with no target date. Load and reconciliation when a device comes back online is unresolved.
- Companion-app lifecycle (release cycle, backward compatibility, update control, multi-org configuration) has no design yet.
- Jeff's security review org is not yet protected from deletion, and Jeff has no sandbox user.

# Open threads

Owner in parentheses; date is when the thread was captured.

## Decisions needed
- [ ] Companion app perspective: release cycle and test/verify approach, backward compatibility, update control (push/pull/scheduled), multi-org configuration, and why a companion app over mobile web. (Royce, 2026-09-17)
- [ ] Print output format: HTML was the grooming outcome on Sep 11; uncertain now that the printing app is folding in. Confirm as part of the printing work with Jeff. (Royce, 2026-09-11 / 09-23)
- [ ] "OCR" in the requirements = a "captured offline" flag, not optical character recognition. Confirm and simplify. (Royce, 2026-09-11)
- [ ] Scope guard: filtering required data on-device is critical; pre-processing from AWS and an API hook at `mobile.ohanafy.com` were floated. Decide how much of this is in scope for pilot. (Royce, 2026-09-10)
- [ ] Reconciliation when back online: can conflicts be resolved outside the app, or at least via a fully online process? AWS may make this easier; what does that do to auth? (Royce, 2026-09-10)
- [ ] External client app: packaging the app's Salesforce-side components for installation into a customer's own org. Plan with Bryson once Gulf go-live temperature drops. (Royce, 2026-09-22)
- [ ] CD for mobile: provision a dedicated org for continuous delivery after Gulf settles, and pipe its credentials into the iOS review submission. (Royce, 2026-09-22)

## Tasks
- [ ] Get Jeff a sandbox user to test the app. (Royce, 2026-09-21)
- [ ] Remove Jeff's security review org from deletion. (Royce, 2026-09-22)
- [ ] Add Mack to TestFlight. (Royce, 2026-09-23)
- [ ] Test SSO with the mobile app. (Royce, 2026-09-22)
- [ ] Understand the mobile check-deposit use case. (Royce, 2026-09-21)
- [ ] Check whether the org has an unlimited API limit. (Royce, 2026-09-11)
- [ ] Validate flows before polishing the app. (team, 2026-09-11)
- [ ] Research what "enterprise ready" means for a mobile app (MDM: Gulf uses Ivanti). (Royce, 2026-09-11)
- [ ] Research the Salesforce Mobile SDK and how the SF mobile app understands org configuration. (Royce, 2026-09-08 / 09-09)
- [ ] Ask about RayRig and Sears Tech (warehouse hardware). (Royce, 2026-09-10)
- [ ] Printing: remaining work to fold the printing app into Ohanafy Mobile. (Jeff & Royce, 2026-09-23)

## Questions
- [ ] Status of the external handoff communication (Ian/Hogan own it)? (2026-09-23)
- [ ] Do we need a partially-updated state when a device is actually online? (2026-09-10)
- [ ] How do the many business rules sync to the app? (2026-09-10)
- [ ] Is warehouse work better addressed with onsite hardware given better connectivity? (2026-09-10)
- [ ] Signature capture on the sales rep side is mostly CYA against "I didn't order this," not regulatory. Does that change the requirement? (2026-09-11)
- [ ] Bug bash scope: iPad, iPhone, and Android? (Matt, 2026-09-01)
- [ ] Does a Slack channel or Jira epic exist for mobile? Royce doubts it; to be checked at first reconcile. (2026-09-23)

# Decisions
- None recorded yet as DEC notes. Candidates from the log: build our own app rather than extend Salesforce mobile (implicit since early discovery); handoff is not a go-live (leadership, Sep 8, reaffirmed with the Oct 12 date); warehouse/inventory deferred (Matt, Sep 10); printing app folds into the mobile app (handoff meeting, Sep 2026). Each should become a `Decisions/DEC-xxxx` note when confirmed.

# People
- Jeff — developer; needs sandbox user; owns the review org used for Apple demo credentials; taking on printing with Royce
- Emily, Elliot — domain experts; Elliot owns the requirement and messaging details
- Mack — enablement lead for the rollout; pre-planning with Royce; to be added to TestFlight
- Matt — Gulf build owner; warehouse scope; bug bash; release planning
- Josh — release planning past pilot; earlier architecture partner and prototype author, now participating less
- Bryson — external client app packaging (later)
- Ian, Hogan — external messaging for the handoff

# Log
## Early discovery
- Reviewed the [[Ohanafy Native.pdf|Prototype Overview]] from Josh.
- Gulf uses [Ivanti](https://www.ivanti.com) for mobile device management (MDM).
- For iOS, [Apple TestFlight](https://testflight.apple.com) is a reliable backup strategy if business registration and security review run long.
- Confirmed that mobile device users have Salesforce Platform License, so API connectivity should not be problematic.
- Working from [[Hardware Breakdown.png|Hardware Breakdown]] to determine platform priority.
## [[2026-09-01]]
- Leadership: 9/28 reinforced as not a go-live-ready date. Bug bash to be defined (iPad, iPhone, Android?). Printing app exists (Thomas). Open question of building our own app versus working with Salesforce, and hybrid options.
## [[2026-09-08]]
- Created [[Mobile Launch Brief.pdf|Mobile Launch Brief]] to guide timeline for delivery.
- Began registration process for Apple Business and Google Play.
- Leadership: derisking mobile is job one this week. Need access to Apple and Google accounts. Researching how the SF mobile app understands org configuration. Messaging: less is more, a right move six months in the making, not a last-minute audible; Hogan and Matt to align by EOD Wed.
## [[2026-09-09]]
- Met with Matt, Josh, Elliot and Jeff to define progress tracks and assign responsibilities.
## [[2026-09-10]]
- Received the [Requirements Document](https://docs.google.com/document/d/1GHJZU3g-R_bHOY-oWG99vewXhBCfeLyegmLwRzbxM7M/edit?tab=t.0) for review.
- Replied to additional information requests from Apple Business and Google Play. Google Play confirmed verification.
- Requirements review: data scope filtering is critical; AWS pre-processing and a `mobile.ohanafy.com` API hook floated; consider tracking device update state. Warehouse/inventory agreed to be delayed (Matt). Gulf field users ≈ 500. Load and reconciliation flagged as an issue.
## [[2026-09-11]]
- Grooming day 2: signature capture is CYA not regulatory; printing is HTML not PDF; research "enterprise ready"; OCR likely means a captured-offline flag; check unlimited API limit; validate flows before polish.
## [[2026-09-15]]
- Leadership: discuss printing app with Jeff and Thomas. Messaging on the app to stay a non-event; communication from Ian/Hogan; Royce to supply benefit details with Elliot.
## [[2026-09-17]]
- Captured the companion-app perspective questions (release cycle, compatibility, updates, multi-org). Scheduled mobile enablement with Jeff and Mack. Noted that mobile was not surfaced as a risk in leadership; later settled.
## [[2026-09-21]]
- Need a sandbox user for Jeff. Check-deposit use case needs understanding.
## [[2026-09-22]]
- Jeff's review org must be kept from deletion. Takeaway on CD: provision a dedicated org for mobile continuous delivery after Gulf settles and feed its credentials into the iOS review submission. Test SSO. External client app: talk to Bryson later.
## [[2026-09-23]]
- Correction pass with Royce (23 claims reviewed). Handoff moved to Oct 12. Apple verified; TestFlight in use; App Store review expected within days. Josh off the architecture track. Printing app folding into the mobile app after a handoff meeting; work continues with Jeff and Royce. Enablement meetings with Mack held; pre-planning underway. Requirements review finished; scope grew several times, now stabilizing. First rollout wave is 50 users. App is named Ohanafy Mobile. Salesforce security review does not apply to the app. "Mobile not surfaced as a risk" question settled.
- Done since last update: reviewed the requirements document; watched the Warehouse Z videos; supplied benefit details for the handoff messaging; understood the review process that applies.
- Moved out of this front (Gulf Pilot material): Gulf Salesforce build pencils down Sep 14; Gulf game visit Sat Sep 26.

# Reference
- [Mobile App Requirements](https://docs.google.com/document/d/1GHJZU3g-R_bHOY-oWG99vewXhBCfeLyegmLwRzbxM7M/edit?tab=t.0) (Google Doc, received 2026-09-10; review complete)
- [[Mobile Launch Brief.pdf|Mobile Launch Brief]] (2026-09-08)
- [[Ohanafy Native.pdf|Prototype Overview]] (Josh; "Native" was the prototype's name, the app is Ohanafy Mobile)
- [[Hardware Breakdown.png|Hardware Breakdown]]
- [[SF OEM Email.png|Salesforce OEM email]] — Mobile Offline discontinued for new Mobile App Plus contracts, 2026-07-31
- [Apple TestFlight](https://testflight.apple.com) (in use)
- hub.ohanafy.com (noted 2026-09-09; purpose to confirm)
- Slack channel and Jira epic: unknown, probably none — check at first reconcile
