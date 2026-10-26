# Overview
This project will become the Ohanafy Mobile product offering on both iOS App Store and Google Play.

Accurate as of `release/v1.1.0` (`ohfy-mobile`, 2026-10-23).
## Experiences
- [[Sales Experience]]: the iPad sales rep's route day (check in, visit checklist, order, swaps and credits, check out).
- [[Driver Experience]]: the iPhone delivery driver's route day (Start Route, deliver, collect, print, End Route).
# The App
- **Platform:** native iOS (iOS 18+), one app for iPhone and iPad. Android/Google Play is not started.
- **Version:** 1.1.0, submitted to App Store review from `release/v1.1.0`.
- **Built on:** Salesforce Mobile SDK (SmartStore + MobileSync). It writes standard `ohfy__` objects through the Salesforce REST API, with no custom Apex endpoints.
- **Offline-first:** everything a user does is saved on the device first and sent when there is signal. Offline and online run the same code.
- **Out of scope for this track:** warehouse, manager, and merchandiser workflows; sell-from-truck (G13).
## Shared Features
- **Sign-in:** Salesforce OAuth through the system browser, so passkeys and password managers work.
- **Persona routing:** the app decides from data, not role. A user named as `Driver__c` on a delivery or route in the day window gets the driver experience; otherwise they get the rep experience. A user with both can switch from the account menu.
- **Screen lock:** after time in the background, the app asks for Face ID or the device passcode.
- **Data on the device:** 51 Salesforce objects cached and scoped to the user. Today's routes are covered from yesterday through two days out (`[-1, +2]`).
- **Sync:** pull to refresh, or Sync now, sends queued work and then refreshes data without losing progress in the day. iOS can also refresh in the background.
- **Sync sheet:** three views:
	- **Status:** last sync and health
	- **Data:** cached objects and counts
	- **Changes:** the outbox queue
- **Needs your attention:** Salesforce rejections show with the reason and a Fix and resubmit path. They never block other work.
- **Help:** Submit Feedback / report a problem files a ticket into Support Triage (BMS-8906).
- **Telemetry:** Datadog metrics tagged with the Salesforce org ID.
## Known Gaps
- Account Team accounts only reach a rep's book once reps hold Split's Account Team permission sets (Q78).
- Order cutoff only warns. The answered "hold" behaviour needs a server hook that doesn't exist yet (G3).
- Short-pay holds are out of scope (G5). Short pay is recorded but never blocks.
- Promotion discounts are computed on the device, not by Apex (Q66, a known breach pending sign-off).
## Source of Truth
In the `ohfy-mobile` repo:
- `docs/prd-requirements-status.md` for requirement status
- `docs/architecture.md` for the design
- `docs/open-questions.md` for undecided items