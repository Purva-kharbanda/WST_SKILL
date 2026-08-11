# WFM-142519 — Handoff Document

*Multi-skill handoff document. Each section below represents a different skill output.*

<!-- SKILL_SECTION: wst-defect-triage R1 -->
## TRIAGE (R1)
*Updated: 2026-08-11 11:29*

---
id: WFM-142519
summary: "Dev story - WFM-130904 - Roster blank space on Add in associate sub-tabs (Done; fix in 45.1.23.0)"
verdict: ALREADY_FIXED
confidence: HIGH
step_reached: KB_ONLY
input_form: jira_json
module: "WFM-OM/HR/Integrations/Notifications"
customer: COPPEL_SB
flavour: null
affects_version: "45.1.22.0"
fix_version: "45.1.23.0"
priority: Medium
blast_radius: global
next_owner: L3-Engineering
sla: informational
status: Draft
version: R1
generated: "2026-08-11T06:00:00+05:30"
last_updated: "2026-08-11T06:00:00+05:30"
duration_minutes: 8
model: composer-2.5-fast
skill_version: "2.2.0"
attachments_read: null
fix_primary: web
channels_mobile: false
channels_web: true
channels_data: false
fault_locus: client_app
channel_confidence: HIGH
implicated_areas_count: 5
---

# Triage: WFM-142519 — Dev story - WFM-130904 - Roster blank space on Add in associate sub-tabs

## TL;DR

**Bug:**    On Employee > Roster > Associate, clicking Add on Pay Rule, Time Clocking, Alternate Work Location, Time Rounding, or Time Collection leaves a large blank gap between the list grid and the bottom add/edit panel, pushing form fields below the visible viewport.
**Fix:**    PR #22224 (rfx-rwsdev300) — change `updatePageHeight()` in five roster renderer modules to subtract bottom-panel header/content heights from `pageContentHeightWOPopUp` instead of reusing full `pageContentHeight` in add/edit mode.
**Action:** L3-Engineering — informational — Verify PR #22224 is merged and included in 45.1.23.0 build; retest all five associate sub-tabs on COPPEL_SB and a second flavour.

## Ticket snapshot

- **Module:**             WFM-OM/HR/Integrations/Notifications (HR)
- **Customer / flavour:** COPPEL_SB (COPPEL sandbox; build WFM.45.1.22.0.20260416.U003851)
- **Affects version:**    45.1.22.0
- **Fix version:**        45.1.23.0
- **Priority:**           Medium
- **Stage:**              Closed (Done / Work Complete)
- **Original input:**     [NOT APPLICABLE]
- **Linked bug:**          WFM-130904 (Mention link)
- **Parent epic:**         WFM-138579

## Fix routing (from ticket + code)

**Primary fix location:** web — AngularJS roster renderer height calculation in `WebContent/scripts/roster/*.renderers.js` (client-side layout, not server API).

**Channels (fix may be required):**

| Channel | Required? | Evidence |
|---------|-----------|----------|
| Mobile (Shift App) | no | Repro is kernel web UI (Employee > Roster); no mobile surface cited |
| Web / server | yes | Jira repro URL `coppelsb.reflexisinc.com/RWS4/`; PR targets `rfx-rwsdev300` JS renderers |
| Data / master data | no | Pure UI layout defect; no data/config dependency |
| Shared mobile shell | n/a | Not implicated |

**Implicated areas:**

| Area | Layer | Fix owner | Confidence | Notes |
|------|-------|-----------|------------|-------|
| Roster > Associate > Pay Rule | web_ui (client_emission) | web | HIGH | Sub-tab #7 in repro |
| Roster > Associate > Time Clocking | web_ui (client_emission) | web | HIGH | Sub-tab in repro |
| Roster > Associate > Alternate Work Location | web_ui (client_emission) | web | HIGH | Sub-tab in repro |
| Roster > Associate > Time Rounding | web_ui (client_emission) | web | HIGH | Sub-tab in repro |
| Roster > Associate > Time Collection | web_ui (client_emission) | web | HIGH | Sub-tab in repro |

**Sub-issue routing:** not a composite ticket (single UI defect across five sub-tabs sharing the same `updatePageHeight` pattern).

**Routing reconciliation:** confirmed — E4 PR/dev comment names `rfx-rwsdev300` roster renderer files; fault is client-side layout (`client_app` / `web_js`), not server API.

## Prior analysis critique

| Prior claim | Source | Verdict | Reason |
|---|---|---|---|
| List container used full `pageContentHeight` in add/edit mode | Purva Kharbanda (PR comment / customfield_15600) | agree | `roster.payRule.renderers.js:updatePageHeight` (lines 167–175) subtracts panel heights from current `vm.pageContentHeight` rather than anchoring to `pageContentHeightWOPopUp`; same pattern in timeClocking, timeRounding, timeCollection |
| Fix: use `pageContentHeightWOPopUp - entityDetailsHeaderHeight - entityDetailsContentHeight` | Purva Kharbanda (PR #22224) | agree | Correct pattern already present in `roster.assocRepHierarchy.renderers.js:328` as reference implementation |
| Five roster sub-tab templates updated | Purva Kharbanda (PR #22224) | partial | Dev cites five files; local workspace pre-merge still shows legacy pattern in payRule/timeClocking/timeRounding/timeCollection — expect fix in PR branch / 45.1.23.0 |

---

## Probable cause

- **Symptom:**     Large blank space between the associate list grid and the bottom add/edit panel when clicking Add on roster associate sub-tabs; form content pushed below visible area.
- **Fault:**       `WebContent/scripts/roster/roster.payRule.renderers.js:updatePageHeight` (and parallel files for Time Clocking, AWL, Time Rounding, Time Collection) — list height not reduced for bottom-panel occupancy in add/edit mode.
- **Trigger:**     User navigates Employee > Roster > Associate > any affected sub-tab and clicks Add (or Edit); `updatePageHeight(false)` runs while bottom panel opens.
- **Consequence:** List grid retains excessive vertical space; bottom add/edit form renders off-screen or with visible gap — UX defect, no data corruption.

---

## Probable fix

### Pattern B — Code patch (already delivered in PR #22224)

- **File(s):**       `WebContent/scripts/roster/roster.payRule.renderers.js`, `roster.timeClocking.renderers.js`, `roster.alternateWorkLocation.renderers.js`, `roster.timeRounding.renderers.js`, `roster.timeCollection.renderers.js` — `updatePageHeight()` function
- **Change sketch:** In the `!param` branch (add/edit panel open), set `vm.pageContentHeight = vm.pageContentHeightWOPopUp - vm.entityDetailsContentHeight - vm.entityDetailsHeaderHeight` (retain existing AWL offset `-20` / `-50` / `-52` constants per module). Do not subtract from the already-inflated `vm.pageContentHeight`.
- **Blast radius:**  Global — all customers using Employee > Roster associate sub-tabs with list+bottom-panel layout; no server or DB impact.
- **Migration:**     No schema or data migration.
- **PR scope:**      PR #22224 (zebratechnologies/rfx-rwsdev300) — merge + BAT/UI retest on five sub-tabs.

```javascript
// [SKETCH] — reference pattern from roster.assocRepHierarchy.renderers.js:326-330
// Current (buggy — payRule example):
if (!param) {
    var temp = vm.pageContentHeight - vm.entityDetailsContentHeight - vm.entityDetailsHeaderHeight - 52;
    vm.pageContentHeight = temp;
}

// Suggested (add/edit-aware):
if (!param) {
    vm.pageContentHeight = vm.pageContentHeightWOPopUp - vm.entityDetailsContentHeight - vm.entityDetailsHeaderHeight - 52;
}
```

---

## Test gap

| ID | Persona | Action | Condition | Expected | Risk | Flavour |
|---|---|---|---|---|---|---|
| TC-NEW-WFM-130904-1 | CORP_ADMIN | Employee > Roster > Associate > Pay Rule > Add | Filter applied; associate selected; viewport 1920×1080 | No blank gap between list and bottom panel; add form fully visible without scroll | HIGH | B0+ |
| TC-NEW-WFM-130904-2 | CORP_ADMIN | Repeat Add on Time Clocking, AWL, Time Rounding, Time Collection | Same roster context | Same layout — list height shrinks; panel adjacent to list | HIGH | B0+ |
| TC-NEW-WFM-130904-3 | CORP_ADMIN | Edit existing record on each of five sub-tabs | Record exists | Edit panel visible; no regression on view-only mode | MED | COPPEL_SB |

- **Closest existing TC:** None identified in ticket (Synapse badge: 0 test cases linked).
- **Extend (optional):** Add roster UI layout BAT covering add/edit panel height for any new associate sub-tab using the shared `updatePageHeight` pattern.

---

## Handoff envelope

**Next owner:**  L3-Engineering (verification / release inclusion)
**Next action:** Confirm PR #22224 merged to 45.1.23.0; execute TC-NEW-WFM-130904-1/2 on COPPEL_SB and one additional flavour before customer promotion.
**SLA:**         informational (story Done; triage documents root cause for audit)

## Open questions

- [ANSWERED: PR #22224 documents fix] Dev story tracks completed work — no open engineering questions.
- [OPEN] Local workspace may pre-date PR merge — verify five renderer files contain `pageContentHeightWOPopUp`-anchored height on target build branch. **Owner:** L3-Engineering **blocks_fix:** no

## Evidence trail

**KB citations:**
- No matching RNI in `release_notes_registry.json` for WFM-130904 (fix may post-date registry refresh).
- Module map: Employee > Roster → OM/HR associate maintenance (WST HR Admin persona).

**Code citations (light confirm — Step 2 skipped at HIGH gate):**
- `WebContent/scripts/roster/roster.payRule.renderers.js:163-177` — `updatePageHeight` legacy pattern
- `WebContent/scripts/roster/roster.assocRepHierarchy.renderers.js:326-330` — correct reference pattern
- PR: https://github.com/zebratechnologies/rfx-rwsdev300/pull/22224

**Attachments read:** none

**Skipped layers:**
- [INFO: sws_zta (ZTA sources) not in workspace — optional repo; jar fallback if ZTA path implicated]
- [INFO: rflx-wfm-shiftbuilding not in workspace — optional; ShiftBuilding.jar fallback]
- [INFO: rflx-wfm-frameworkscheduling not in workspace — optional; RWS.jar fallback]
- [INFO: rflx-wfm-scheduling not in workspace — optional; RuleScheduling.jar fallback]

## Provenance

### Confidence flags

- HIGH: Dev PR comment + Done status + fix version 45.1.23.0 + code pattern confirmation.
- [INFO: Step 2 skipped per KB_ONLY gate — code citations from targeted grep only]

### Refinement log

| Version | Date | Trigger | Confidence | Summary |
|---------|------|---------|------------|---------|
| R1 | 2026-08-11 | /triage WFM-142519 | HIGH | ALREADY_FIXED — dev story Done; PR #22224 documents web UI height fix for WFM-130904 |

<!-- /SKILL_SECTION -->
