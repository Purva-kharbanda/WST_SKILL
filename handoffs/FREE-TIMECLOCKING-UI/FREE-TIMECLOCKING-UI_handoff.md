# Triage: FREE-TIMECLOCKING-UI

- Mode: Draft triage
- Input form: Free text + screenshot
- Verdict: NEEDS_MORE_INFO
- Confidence: MEDIUM

## Key findings

1. The roster Time Clocking page shows `No Data Found` when `selectedAssociateTimeClockingDetailsList` is empty in `WebContent/ngtemplates/roster/timeClocking/timeClocking.html`.
2. The fetch endpoint chain is:
   - UI call `fetchRosterAssociateTimeClockingInfo` in `WebContent/scripts/roster.common.actions.js`
   - Controller `getAssociateTimeClocking` in `WebContent/src/com/reflexis/rws/v5/controller/WebRosterController.java`
   - Service `getAssociateTimeClockingInformation` in `DefaultAssociateCoreService`
   - DAO `getAssociateTimeClockingInformation` selecting from `TA_EMP_TL_STATUS` in `AssociateDAO.xml`.
3. The DAO query is person-scoped and returns all rows for that person, so "No Data Found" is expected if no rows exist in `TA_EMP_TL_STATUS` for that associate.
4. A layout artifact is plausible: fixed `pageContentHeight` is reused by nested containers and adjusted with manual constants in `roster.timeClocking.renderers.js`, which can create visible bottom whitespace on some viewport sizes.

## Evidence needed to confirm root cause

- Browser network response for `fetchRosterAssociateTimeClockingInfo` (to confirm empty vs non-empty payload).
- One DB check on `TA_EMP_TL_STATUS` for the displayed `PERSON_ID`.
- Repro sequence clarifying whether the issue is:
  - data absence (`No Data Found` unexpected), or
  - UI layout gap (red highlighted bottom blank space), or both.
