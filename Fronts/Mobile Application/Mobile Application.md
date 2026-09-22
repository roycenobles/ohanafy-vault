---
type: front
title: Mobile Application
status: active
as_of: 2026-09-22
owner: Royce Nobles
tracks:
  - App development — Jeff (dev), Emily & Elliot (domain), Royce & Josh (architecture)
  - App store registrations — Royce
  - Release planning past pilot — Elliot, Matt, Josh, Royce
milestones:
  - 2026-09-14 Gulf build pencils down (Matt)
  - 2026-09-28 Mobile milestone — NOT a go-live-ready date
  - 2026-09-24 to 2026-09-29 Royce on-site at Gulf
  - 2026-12-14 Gulf rollout window ends (14 weeks from Sep 8)
related:
  - "[[Gulf Pilot]]"
  - "[[AI Strategy]]"
sources: vault only (journal + this note); not yet reconciled against Slack or Jira
---
> Front note for the Ohanafy field mobile app. **Status** and **Open threads** are rewritten at each reconcile (see `as_of`). **Log** is append-only. **Reference** links out; nothing authoritative is copied in.

> [!quote] Requirement (Elliot Flores)
> As a tech business we need an app that supports a full route day with no connectivity issues, including printing, and starting a new order with zero signal AND supports scale (1,500 field users) following enterprise security requirements.

# Status

**Why this exists.** Salesforce discontinued Mobile Offline for new Mobile App Plus contracts effective 2026-07-31 ([[SF OEM Email.png|SF OEM email]]). The existing mobile experience does not meet the acceptance criteria above. Ohanafy is building its own field app (working name Ohanafy Native), with Gulf as the first deployment.

**Where it stands (2026-09-22).** Derisking mobile has been the stated job-one since the Sep 8 leadership meeting. Registrations are done on the Google side (verified Sep 10) and in progress with Apple; Apple TestFlight is the fallback if Apple Business review runs long. Requirements were received Sep 10 and groomed over two sessions (Sep 10–11); the scope questions that came out of grooming are the bulk of the open threads below. Gulf field population is roughly 500 users (drivers, merchandisers, reps); the requirement is sized for 1,500. The Sep 28 date is a milestone, not a go-live, and leadership has agreed the messaging is "a non-event," owned by Ian and Hogan with Elliot refining details. Royce is on-site at Gulf Sep 24–29.

**Risks I am watching.**
- Mobile was not surfaced as a risk in the Sep 17 leadership discussion; unresolved whether that was right (journal Sep 17).
- Apple Business verification timeline versus the Sep 28 milestone.
- Warehouse/inventory is the sensitive part of the system and was agreed to be delayed; load and reconciliation when a device comes back online is unresolved.
- Companion-app lifecycle (release cycle, backward compatibility, update control, multi-org configuration) has no owner or design yet.
- Security review process for the app is not yet understood.

# Open threads

Owner in parentheses; date is when the thread was captured.

## Decisions needed
- [ ] Companion app perspective: release cycle and test/verify approach, backward compatibility, update control (push/pull/scheduled), multi-org configuration, and why native over mobile web. (Royce, 2026-09-17)
- [ ] Printing: requirement is not PDF, likely HTML. When and how does the separate printing app (Thomas) fold into the mobile app? Discuss with Jeff and Thomas. (Royce, 2026-09-11 / 09-15)
- [ ] "OCR" in the requirements = a "captured offline" flag, not optical character recognition. Confirm and simplify. (Royce, 2026-09-11)
- [ ] Scope guard: filtering required data on-device is critical; pre-processing from AWS and an API hook at `mobile.ohanafy.com` were floated. Decide how much of this is in scope for pilot. (Royce/Josh, 2026-09-10)
- [ ] Reconciliation when back online: can conflicts be resolved outside the app, or at least via a fully online process? AWS may make this easier; what does that do to auth? (Royce, 2026-09-10)
- [ ] External client app (beyond Gulf): plan with Bryson once Gulf go-live temperature drops. (Royce, 2026-09-22)
- [ ] CD for mobile: provision a dedicated org for continuous delivery after Gulf settles, and pipe its credentials into the iOS security review submission. (Royce, 2026-09-22)

## Tasks
- [ ] Carefully review the [Mobile App Requirements](https://docs.google.com/document/d/1GHJZU3g-R_bHOY-oWG99vewXhBCfeLyegmLwRzbxM7M/edit?tab=t.0). (Royce, 2026-09-11)
- [ ] Get Jeff a sandbox user to test the app. (Royce, 2026-09-21)
- [ ] Remove Jeff's security review org from deletion. (Royce, 2026-09-22)
- [ ] Understand the Salesforce security review process for the app. (Royce, 2026-09-22)
- [ ] Test SSO with the mobile app. (Royce, 2026-09-22)
- [ ] Set up Ohanafy people on TestFlight. (Royce, 2026-09-22)
- [ ] Understand the mobile check-deposit use case. (Royce, 2026-09-21)
- [ ] Check whether the org has an unlimited API limit. (Royce, 2026-09-11)
- [ ] Validate flows before polishing the app. (team, 2026-09-11)
- [ ] Watch the Warehouse Z videos in Slack. (Royce, 2026-09-11)
- [ ] Research what "enterprise ready" means for a mobile app (MDM: Gulf uses Ivanti). (Royce, 2026-09-11)
- [ ] Research the Salesforce Mobile SDK and how the SF mobile app understands org configuration. (Royce, 2026-09-08 / 09-09)
- [ ] Ask about RayRig and Sears Tech (warehouse hardware). (Royce, 2026-09-10)
- [ ] Come up with details around the benefits of the app for the Sep 28 messaging (Ian/Hogan communicate). (Royce with Elliot, 2026-09-15)

## Questions
- [ ] Do we need a partially-updated state when a device is actually online? (2026-09-10)
- [ ] How do the many business rules sync to the app? (2026-09-10)
- [ ] Is warehouse work better addressed with onsite hardware given better connectivity? (2026-09-10)
- [ ] Signature capture on the sales rep side is mostly CYA against "I didn't order this," not regulatory. Does that change the requirement? (2026-09-11)
- [ ] Bug bash scope: iPad, iPhone, and Android? (Matt, 2026-09-01)

# Decisions
- None recorded yet as DEC notes. Candidates from the log: build our own app rather than extend Salesforce mobile (implicit since early discovery); Sep 28 is a milestone not a go-live (leadership, Sep 8); warehouse/inventory delayed (Matt, Sep 10); printing is HTML not PDF (grooming, Sep 11). Each should become a `Decisions/DEC-xxxx` note when confirmed.

# People
- Jeff — developer; needs sandbox user; has a security review org
- Emily, Elliot — domain experts; Elliot owns requirement and messaging details
- Josh — architecture with Royce; created the prototype overview
- Matt — Gulf build owner; warehouse scope; bug bash
- Thomas — printing app
- Mack — mobile enablement discussion (Sep 18/22)
- Bryson — external client app conversation (later)
- Ian, Hogan — external messaging for Sep 28

# Log
## Early discovery
- Reviewed the [[Ohanafy Native.pdf|Prototype Overview]] from Josh.
- Gulf uses [Ivanti](https://www.ivanti.com) for mobile device management (MDM).
- For iOS, [Apple TestFlight](https://testflight.apple.com) is a reliable backup strategy if business registration and security review run long.
- Confirmed that mobile device users have Salesforce Platform License, so API connectivity should not be problematic.
- Working from [[Hardware Breakdown.png|Hardware Breakdown]] to determine platform priority.
## 2026-09-01
- Leadership: Gulf build pencils down 9/14; 9/28 reinforced as not a go-live-ready date. Bug bash to be defined (iPad, iPhone, Android?). Printing app exists (Thomas). Open question of building our own app versus working with Salesforce, and hybrid options.
## 2026-09-08
- Created [[Mobile Launch Brief.pdf|Mobile Launch Brief]] to guide timeline for delivery.
- Began registration process for Apple Business and Google Play.
- Leadership: derisking mobile is job one this week. Need access to Apple and Google accounts. Researching how the SF mobile app understands org configuration. Sep 28 messaging: less is more, a right move six months in the making, not a last-minute audible; Hogan and Matt to align by EOD Wed.
## 2026-09-09
- Met with Matt, Josh, Elliot and Jeff to define progress tracks and assign responsibilities (see `tracks` above).
## 2026-09-10
- Received the [Requirements Document](https://docs.google.com/document/d/1GHJZU3g-R_bHOY-oWG99vewXhBCfeLyegmLwRzbxM7M/edit?tab=t.0) for review.
- Replied to additional information requests from Apple Business and Google Play. Google Play confirmed verification.
- Requirements review: data scope filtering is critical; AWS pre-processing and a `mobile.ohanafy.com` API hook floated; consider tracking device update state. Warehouse/inventory agreed to be delayed (Matt). Gulf field users ≈ 500. Load and reconciliation flagged as an issue.
## 2026-09-11
- Grooming day 2: signature capture is CYA not regulatory; printing is HTML not PDF; research "enterprise ready"; OCR likely means a captured-offline flag; check unlimited API limit; validate flows before polish.
## 2026-09-15
- Leadership: discuss printing app with Jeff and Thomas. Messaging on the app to stay a non-event; communication from Ian/Hogan; Royce to supply benefit details with Elliot.
## 2026-09-17
- Captured the companion-app perspective questions (release cycle, compatibility, updates, multi-org). Scheduled mobile enablement with Jeff and Mack. Noted that mobile was not surfaced as a risk in leadership; unresolved whether that was right.
## 2026-09-21
- Need a sandbox user for Jeff. Check-deposit use case needs understanding.
## 2026-09-22
- Jeff's security review org must be kept from deletion. Takeaway on CD: provision a dedicated org for mobile continuous delivery after Gulf settles and feed its credentials into the iOS security review submission. Test SSO. Set up TestFlight for Ohanafy people. External client app: talk to Bryson later.

# Reference
- [Mobile App Requirements](https://docs.google.com/document/d/1GHJZU3g-R_bHOY-oWG99vewXhBCfeLyegmLwRzbxM7M/edit?tab=t.0) (Google Doc, received 2026-09-10)
- [[Mobile Launch Brief.pdf|Mobile Launch Brief]] (2026-09-08)
- [[Ohanafy Native.pdf|Prototype Overview]] (Josh)
- [[Hardware Breakdown.png|Hardware Breakdown]]
- [[SF OEM Email.png|Salesforce OEM email]] — Mobile Offline discontinued for new Mobile App Plus contracts, 2026-07-31
- [Apple TestFlight](https://testflight.apple.com)
- hub.ohanafy.com (noted 2026-09-09; purpose to confirm)
- Slack channel and Jira epic: **not yet identified** — add at first reconcile
