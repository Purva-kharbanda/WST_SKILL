---
id: WFM-142519
summary: "Dev story - WFM-130904 - Roster blank space at bottom when adding new record"
verdict: GENUINE_BUG
confidence: MEDIUM
step_reached: KB_STEP2
input_form: jira_api_hook
module: RWS / Employee > Roster
customer: COPPEL_SB CORP
flavour: null
affects_version: WFM.45.1.22.0.20260416.U003851
fix_version: 45.1.22.2
priority: Medium
blast_radius: global
next_owner: L3-Engineering
sla: P3-next-sprint
status: Draft
version: R2
generated: 2026-08-05T11:05:00+05:30
last_updated: 2026-08-05T11:05:00+05:30
duration_minutes: 6
model: composer-2.5-fast
skill_version: "2.2.0"
attachments_read: null
fix_primary: web
fault_locus: client_app
channel_confidence: HIGH
channels_mobile: false
channels_web: true
channels_data: false
implicated_areas_count: 5
---

# Triage: WFM-142519 â€” Roster blank space at bottom when adding new record

## TL;DR

**Bug:** On Employee > Roster > Associate, clicking **Add** on Pay Rule, Time Clocking, Alternate Work Location, Time Rounding, or Time Collection leaves a large blank gap at the bottom of the page.

**Fix:** Roster 2.0 client-side page-height budgeting â€” call `updatePageHeight()` before toggling `showAddTemplateTab` on the five affected renderer controllers; optional follow-up if gap persists: introduce `listPanelHeight` on inner list cards and remove one-time `::` height bindings in bottom-panel HTML.

**Action:** L3-Engineering â€” commit/deploy the F1 fix already present in the local working tree (5 renderer files); manual UI verify on COPPEL_SB for all five tabs; link PR to WFM-130904 / WFM-142519.

## Ticket snapshot

- **Module:**             RWS / Employee > Roster (WFM-OM/HR)
- **Customer / flavour:** COPPEL_SB CORP (n/a)
- **Affects version:**    WFM.45.1.22.0.20260416.U003851
- **Fix version:**        45.1.22.2 (target)
- **Priority:**           Medium
- **Stage:**              To Do (Story â€” dev work for parent bug WFM-130904)
- **Original input:**     [NOT APPLICABLE]

## Fix routing (from ticket + code)

**Primary fix location:** web â€” Pure AngularJS Roster 2.0 layout defect; no server API or mobile channel implicated.

**Symptom surface:** web (Employee > Roster, CORP/SYSADMIN profile)

**Fault locus:** client_app (web JS/HTML height model)

**Evidence tier:** E5 surface-only â†’ Step 2 code confirmation (no HAR/API payload; cosmetic layout)

| Channel | Required? | Evidence |
|---------|-----------|----------|
| Mobile (Shift App) | no | Repro path is RWS web Roster only |
| Web / client | yes | `WebContent/scripts/roster/*.renderers.js` + `ngtemplates/roster/` |
| Data / master data | no | No data/config symptom |
| Shared mobile shell | n/a | Not applicable |

**Implicated areas:**

| Area | Layer | Fix owner | Confidence | Notes |
|------|-------|-----------|------------|-------|
| Pay Rule | client_app (web_js) | web | HIGH | Repro step 7 |
| Time Clocking | client_app | web | HIGH | Repro step 7 |
| Alternate Work Location | client_app | web | HIGH | Repro step 7 |
| Time Rounding | client_app | web | HIGH | Repro step 7 |
| Time Collection | client_app | web | HIGH | Repro step 7 |

**Routing reconciliation:** confirmed â€” Step 2 code walk confirms client-side height miscalculation; server_api checked-clear.

## Prior analysis critique

| Prior claim | Source | Verdict | Reason |
|---|---|---|---|
| Blank space caused by Add-panel toggle ordering before `updatePageHeight()` | R1 triage (2026-08-03) + F1 fix capture | **Agree** | Code read confirms `showHide*Panel()` now calls `updatePageHeight(false)` before `showAddTemplateTab = true` in all five renderers |
| Inner list card binds full `pageContentHeight` without header offset | R1 Probable fix sketch | **Partially agree** | `payRule.html:51` still binds list `ws-card` to raw `pageContentHeight`; secondary contributor if gap persists after F1 |
| Minimal F1 fix sufficient without template changes | F1 capture (2026-08-03) | **Plausible** | Developer chose toggle-order only; UI verify still pending |

---

## Probable cause

- **Symptom:**     Empty white space below the Add form after clicking Add on multiple Roster associate detail tabs.
- **Fault:**       `WebContent/scripts/roster/roster.payRule.renderers.js:showHideContactInfoPanel` (and parallel `showHideTimeCollectionPanel`, `showHideTimeRoundBottomPanel`, `showHideContactInfoPanel` in Time Clocking / Alternate Work Location) â€” Add bottom panel was shown before the list area height was recalculated.
- **Trigger:**     Add click â†’ `onEditClick(1)` â†’ panel show path â†’ list area retains full viewport height while bottom Add panel also renders â†’ cumulative layout exceeds visible area, leaving blank footer space.
- **Consequence:** Cosmetic layout defect only; no data loss or save failure implied.
- **What raises to HIGH:** Screenshot from `Blank space at bottom of the page.docx` on parent WFM-130904, or DevTools confirmation that outer container height exceeds sum of visible child elements after F1 deploy.

---

## Probable fix

> **[SKETCH]** F1 fix already applied in local working tree (uncommitted). Re-read before commit.

- **File(s):**       Five renderer files (F1 applied); optional follow-up: `ngtemplates/roster/{payRule,timeClocking,timeCollection,timeRounding,alternateWorkLocation}/` HTML + bottom-panel templates.
- **Change (F1 â€” in working tree):** Reorder panel toggle â€” `updatePageHeight(false)` before `showAddTemplateTab = true`; on close, `updatePageHeight(true)` before hiding panel.
- **Change (follow-up if gap persists):** Introduce `listPanelHeight = pageContentHeight - headerStackOffset` for inner list `ws-card`; remove one-time `::` bindings on dynamic heights in bottom-panel templates (e.g. `payRuleBottomPanel.html:48`).
- **Blast radius:**  All tenants using Roster 2.0 associate detail tabs.
- **Migration:**     No
- **PR scope:**      Commit the 5 modified `WebContent/scripts/roster/roster.*.renderers.js` files; manual UI verify on all five repro tabs.

```javascript
// F1 fix (already in working tree) â€” payRule example:
this.showHideContactInfoPanel = function (show) {
    if (!show) {
        vm.updatePageHeight(true);
        vm.showAddTemplateTab = false;
        vm.selectedTab = "";
    } else {
        vm.updatePageHeight(false);
        vm.showAddTemplateTab = true;
    }
};
```

---

## Test gap

| ID | Persona | Action | Condition | Expected | Risk | Flavour |
|---|---|---|---|---|---|---|
| TC-NEG-WFM142519-01 | STRADM | Roster > Associate > Pay Rule > Add | Associate selected after filter | No blank gap below Add panel; panel flush with viewport bottom | HIGH | all |
| TC-NEG-WFM142519-02 | STRADM | Repeat Add on Time Clocking, Time Collection, Time Rounding, Alternate Work Location | Same associate context | Same â€” no bottom blank space on any tab | HIGH | all |
| TC-NEG-WFM142519-03 | STRADM | Open Add then Close | Any affected tab | List area restores original height; no residual gap | MEDIUM | all |
| TC-NEG-WFM142519-04 | STRADM | Edit existing row (not Add) | Row selected | Edit bottom panel height correct | MEDIUM | all |
| TC-NEG-WFM142519-05 | STRADM | Add panel at 1366Ã—768 and 1920Ã—1080 | Viewport resize | No layout regression | MEDIUM | all |

- **Closest existing TC:** none identified â€” Roster BAT covers accessibility labels, not dynamic height budgeting on Add panel open.
- **Extend (optional):** Roster BAT â€” viewport assertion after Add click on each associate sub-tab.

---

## Handoff envelope

**Next owner:**  L3 Engineering
**Next action:** Commit F1 fix from working tree; PR against 45.1.22.2; manual UI verify on COPPEL_SB for all five tabs; close loop on WFM-130904.
**SLA:**         P3 next sprint

**Open questions:**
- **Q1** [OPEN, owner: QA, blocks_fix: no] Attach/read `Blank space at bottom of the page.docx` from WFM-130904 â€” confirm gap is below Add form, not a scrollbar artifact.
- **Q2** [OPEN, owner: QA, blocks_fix: no] Repro confirmed only on COPPEL_SB or also on generic SIT (B0)?
- **Q3** [ANSWERED: F1 fix captured 2026-08-03 â€” 5 renderer files modified locally; toggle order corrected; not yet committed/deployed]

**Distribution:**
- Primary: Eng DL + Scrum team
- CC: CSE (COPPEL_SB account)

## Evidence trail

- **KB citations:**      none (Step 1 INCONCLUSIVE â€” no matching BR/RNI/config for Roster blank-space layout)
- **Code citations:**    `WebContent/scripts/roster/roster.payRule.renderers.js:updatePageHeight:163`, `WebContent/scripts/roster/roster.payRule.renderers.js:showHideContactInfoPanel:200`, `WebContent/ngtemplates/roster/payRule/payRule.html:23,51,76`, `WebContent/ngtemplates/roster/payRule/payRuleBottomPanel.html:48`, `WebContent/scripts/roster/roster.timeClocking.renderers.js:showHideContactInfoPanel:478`, `WebContent/scripts/roster/roster.timeCollection.renderers.js:showHideTimeCollectionPanel:505`, `WebContent/scripts/roster/roster.timeRounding.renderers.js:showHideTimeRoundBottomPanel:454`, `WebContent/scripts/roster/roster.alternateWorkLocation.renderers.js:showHideContactInfoPanel:386`, `WebContent/rws/staff/roster.jsp` (global `pageContentHeight` bootstrap)
- **Candidate-cause ledger:**

| Layer | Status | Confidence | Evidence |
|-------|--------|------------|----------|
| client_emission (web_js) | implicated | HIGH | Panel toggle order + `pageContentHeight` binding on list card |
| web_template (HTML) | secondary | MEDIUM | List card uses full `pageContentHeight`; `::` one-time binding on bottom panel |
| server_api | checked-clear | HIGH | No save/API failure; pure layout symptom |
| data_config | checked-clear | HIGH | No master-data symptom |
| mobile_client | not-checked | n/a | Web-only repro |
| shared_shell | checked-clear | HIGH | Roster AngularJS module, not kernel_auth/ZDS |

- **Mobile citations:**  not checked â€” not applicable
- **Cross-channel:**     web_only
- **TER evidence:**      not triggered â€” ticket supplies build, URL, and full repro
- **Attachments read:**  none (`Blank space at bottom of the page.docx` referenced in description but not in Jira intake bundle)
- **Skipped layers:**    [INFO: sws_zta not in workspace â€” optional] Â· [INFO: rflx-wfm-shiftbuilding not in workspace â€” optional] Â· [INFO: rflx-wfm-frameworkscheduling not in workspace â€” optional] Â· [INFO: rflx-wfm-scheduling not in workspace â€” optional] Â· [INFO: Step 2 included WebContent client-emission layer (web_js)]

---

## Provenance

**Triage duration:** 6m 0s
**Model:**           composer-2.5-fast
**Skill version:**   2.2.0
**Generated by:**    wst-defect-triage skill

### Refinement log

| Rev | Timestamp | Triggered by | Summary of change |
|-----|-----------|--------------|-------------------|
| R1  | 2026-08-03T10:12:00+05:30 | Initial triage | GENUINE_BUG, MEDIUM â€” Step 2 client layout root cause |
| R1  | 2026-08-05T10:33:00+05:30 | Auto-sync (prior session) | Re-synced to GitHub |
| R2  | 2026-08-05T11:05:00+05:30 | /triage WFM-142519 | Re-triage confirms analysis; F1 fix present in working tree (5 files, uncommitted) |

### Confidence flags

- [INFO: Linked parent bug WFM-130904 â€” dev story tracks same defect]
- [INFO: F1 fix in local working tree â€” commit/deploy pending; UI verify not recorded]
- [INFO: Step 2 candidate-cause ledger â€” client_emission implicated; server_api checked-clear]
- [INFO: Template/listPanelHeight follow-up deferred from F1 â€” apply if gap persists post-deploy]

---

## Clarifications

<!-- REFINE: paste new context below this line -->

---
