# Triage: WFM-142519

**Triage status:** GENUINE_BUG  
**Confidence:** HIGH  
**Step reached:** Step 2 (code analysis)  
**Input form:** Jira API (hook)

## TL;DR
- Bug: Roster add/edit flows retain legacy contact-preference UI wiring (notably Mobile Carrier/SMS paths), causing layout artifacts such as blank space at the bottom while adding records.
- Fix: Remove deprecated Mobile Carrier render paths from roster/contact UI and enforce SMS preference as unsupported in server processing paths.
- Action: Validate in COPPEL SB flow that Add dialogs across Contact/PayRule/TimeClocking/TimeRounding/TimeCollection no longer reserve empty UI rows.

## Probable cause
- **Symptom:** In `Employee > Roster`, adding records in specific tabs shows blank space at page bottom.
- **Fault:** Legacy UI/renderer code still allocates/handles Mobile Carrier and SMS communication-preference fields after support changed, leaving empty render slots and stale bindings.
- **Trigger:** Contact/roster templates and renderers executed with obsolete field wiring in 45.1.22.0 build lineage.
- **Consequence:** Visual dead space appears in add flow and creates inconsistent form UX.

## Probable fix
- Remove Mobile Carrier column/cell rendering from roster summary templates and contact renderers.
- Replace interactive Mobile Carrier control with hidden neutral value where backend contract still expects param shape.
- Add guard in processing JSPs to reject deprecated SMS delivery preference values (`S`/`B`) and persist empty mobile carrier.

## Test gap
- Missing regression UI checks for layout integrity after field deprecation (blank-row/blank-space detection in add dialogs).
- Missing negative tests for deprecated communication preference values in contact/profile save endpoints.

## Evidence trail
- Jira payload: dev story explicitly references original bug `WFM-130904`, affected build `WFM.45.1.22.0.20260416.U003851`, and fix target `45.1.22.3`.
- Code indicators in `rfx-rwsdev300` show direct remediation in roster/contact stack:
  - `WebContent/ngtemplates/roster/summary/summaryContactInfo.html` (Mobile Carrier cells removed)
  - `WebContent/scripts/roster/roster.contactInfo.renderers.js` (mobile carrier map/select handlers removed)
  - `WebContent/rws/staff/process_stf_contacts.jsp` (SMS pref blocked; mobile carrier normalized)
  - Additional related updates in `stf_contacts.jsp`, `roster.jsp`, and profile/security handlers.

## Open questions
- [OPEN] Confirm whether all impacted tabs from original repro share the same renderer baseline in the customer branch (`Pay Rule`, `Time Clocking`, `Alternate Work Location`, `Time Rounding`, `Time Collection`).
- [OPEN] Confirm if any customer-specific customization still references `eMobCarr` or SMS preference values in downstream integrations.
