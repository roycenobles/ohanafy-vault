# Overview
Establish which security and compliance frameworks Ohanafy should pursue, in what order, and what each will cost. The output is a plan and budget we can defend to leadership and point customers to.

## Why now
- Larger distributors (e.g. Gulf) bring enterprise procurement, security questionnaires, and auditor scrutiny.
- Ohanafy is the system of record for orders, inventory, pricing, and AR, so it feeds customers' financial statements.
- Supplier data feeds (SRS, VIP, EDI) put us inside supplier data-sharing agreements.
- Mobile apps add device, location, and offline data-at-rest exposure.

# Drivers
## Industry
- Beverage alcohol is regulated (TTB federal, state ABC boards, three-tier system, excise tax reporting). These are product requirements more than security frameworks, but errors carry regulatory and financial risk for customers.
- Large suppliers (AB InBev, Molson Coors, etc.) impose their own security and data handling terms on distributors and their vendors.

## Customer base
- Mid-to-large US distributors, suppliers, and producers. Mostly B2B with limited consumer data.
- Some customers are large enough that their auditors will ask about controls in the systems they rely on.
- Little international exposure today.

## Platform
- Built as a Salesforce managed package, so we inherit much of Salesforce's infrastructure compliance (SOC 1/2, ISO 27001, etc.).
- Our scope is our code, our org configurations, integrations, mobile apps, any infrastructure outside Salesforce, and our people and processes.

# Frameworks
## Tier 1: Expected now
- **SOC 2 Type II** (Security, Availability, Confidentiality). The baseline ask in B2B SaaS procurement. Start with Type I, then a 6–12 month Type II window.
- **Salesforce AppExchange Security Review.** Required to list and keep a managed package listing. Re-review is needed on major releases.
- **Standard questionnaires** (SIG Lite, CAIQ). Build one maintained answer set and a trust page so we stop answering from scratch.

## Tier 2: Likely soon
- **SOC 1 Type II.** As an ERP, our controls affect customers' financial reporting (ICFR). Expect requests from larger or PE-backed distributors and their auditors.
- **PCI DSS.** Applies only if we store, process, or transmit card data, for example through payments or driver collections. Prefer tokenized processors to keep scope at SAQ A.
- **US state privacy laws** (CCPA/CPRA and others). Light exposure as a B2B company, but this covers employee, driver, and contact data. It needs a privacy policy, a DPA template, and a subprocessor list.

## Tier 3: Watch / later
- **ISO 27001.** Only if we expand internationally or customers specifically ask for it. Overlaps heavily with SOC 2.
- **GDPR.** Only with EU customers or data subjects.
- **NIST CSF / CIS Controls.** Use internally as the control baseline. Not a certification.
- **AI governance (ISO 42001 / NIST AI RMF).** Revisit as AI features ship.

# Supporting Activities
- Annual third-party penetration test covering the package, integrations, and mobile apps
- Compliance automation platform (Vanta, Drata, Secureframe) to collect evidence
- Policies: infosec, access control, incident response, BCP/DR, vendor management, SDLC
- Security awareness training and background checks
- Vendor/subprocessor risk reviews
- Cyber insurance review, aligned with the controls we commit to

# Budget Considerations
- Audit fees (SOC 2 Type I, then Type II annually; SOC 1 if pursued)
- Compliance platform subscription
- Pen testing (annual, plus retests)
- Internal time: an owner and engineering effort to remediate gaps
- Tooling gaps found during readiness (MDM, SSO, logging/SIEM, vulnerability scanning)

# Open Questions
- Have any customers or prospects already asked for SOC 2, SOC 1, or a questionnaire? Which deals depend on it?
- Does Gulf's contract include security, audit, or data handling obligations?
- Do we touch card data anywhere, now or on the roadmap?
- What runs outside Salesforce (mobile backend, integration middleware, Datadog, FTP drop site)?
- Who owns compliance? Do we hire, use a fractional vCISO, or assign it internally?
- What do supplier agreements (SRS, VIP, EDI) require of us?

# Next Steps
- Inventory systems and data flows in scope
- Collect past customer security asks and questionnaires
- Get quotes for a compliance platform, an auditor, and a pen test
- Run a SOC 2 readiness gap assessment
- Draft a timeline and budget
