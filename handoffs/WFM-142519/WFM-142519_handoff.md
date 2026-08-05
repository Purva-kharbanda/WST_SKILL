# WFM-142519 — Handoff Document

*Multi-skill handoff document. Each section below represents a different skill output.*

<!-- SKILL_SECTION: wst-defect-triage R1 -->
## TRIAGE (R1)
*Updated: 2026-08-05 22:58*

﻿---
id: WFM-142519
summary: "Dev story - WFM-130904 - Roster Add panel leaves blank space at bottom of page"
verdict: GENUINE_BUG
confidence: HIGH
step_reached: KB_STEP2
input_form: jira_json
module: "WFM-OM/HR/Integrations/Notifications"
customer: COPPEL_SB
flavour: null
affects_version: "45.1.22.0"
fix_version: "45.1.22.3"
priority: Medium
blast_radius: global
next_owner: L3-Engineering
sla: P3-next-sprint
status: Draft
version: R1
generated: "2026-08-05T17:30:00+05:30"
last_updated: "2026-08-05T17:30:00+05:30"
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

# Triage: WFM-142519 â€” Roster Add panel leaves blank space at bottom of page

## TL;DR

**Bug:** On Employee > Roster, clicking Add on Pay Rule, Time Clocking, Alternate Work Location, Time Rounding, or Time Collection opens the bottom add panel but leaves a large blank gap below it because `showAddTemplateTab` is toggled before `updatePageHeight()` recalculates the main grid height.
**Fix:** In each affected roster renderer, call `updatePageHeight()` before setting `showAddTemplateTab` inside the show/hide panel helper. A local working-tree fix already exists for all five ticket tabs.
**Action:** L3-Engineering â€” P3-next-sprint â€” Commit and regression-test the five renderer changes; scan remaining roster tabs for the same `vm.showAddTemplateTab = show` ordering pattern.

## Ticket snapshot

- **Module:**             WFM-OM/HR/Integrations/Notifications (HR)
- **Customer / flavour:** COPPEL_SB CORP (n/a)
- **Affects version:**    45.1.22.0 (build WFM.45.1.22.0.20260416.U003851)
- **Priority:**           Medium
- **Stage:**              To Do (dev story from bug WFM-130904)
- **Original input:**     [NOT APPLICABLE]

## Fix routing (from ticket + code)

**Primary fix location:** web â€” client-side AngularJS layout in roster bottom-panel controllers; confirmed by Step 2 code walk.

**Channels (fix may be required):**

| Channel | Required? | Evidence |
|---------|-----------|----------|
| Mobile (Shift App) | no | Web-only repro on kernel Roster URL |
| Web / server | yes | Employee > Roster JSP/AngularJS; no server API involved |
| Data / master data | no | Pure UI height calculation defect |
| Shared mobile shell (kernel_auth / ZDS) | n/a | â€” |

**Implicated areas:**

| Area | Layer | Fix owner | Confidence | Notes |
|------|-------|-----------|------------|-------|
| Roster > Pay Rule Add panel | web_ui | web | HIGH | Listed in repro step 7 |
| Roster > Time Clocking Add panel | web_ui | web | HIGH | Listed in repro step 7 |
| Roster > Alternate Work Location Add panel | web_ui | web | HIGH | Listed in repro step 7 |
| Roster > Time Rounding Add panel | web_ui | web | HIGH | Listed in repro step 7 |
| Roster > Time Collection Add panel | web_ui | web | HIGH | Listed in repro step 7 |

**Routing reconciliation:** confirmed â€” Step 2 found client-side ordering bug in `WebContent/scripts/roster/*.renderers.js`; no server path implicated.

## Prior analysis critique

No prior analysis present in input.

---

## Probable cause

- **Symptom:** After clicking Add on multiple Roster associate-detail tabs, a blank white area appears at the bottom of the page below the add form panel.
- **Fault:** `WebContent/scripts/roster/roster.payRule.renderers.js:showHideContactInfoPanel` (and parallel helpers in timeClocking, alternateWorkLocation, timeRounding, timeCollection) â€” visibility flag set before height recalculation.
- **Trigger:** User clicks Add; `showHideContactInfoPanel(true)` runs with `vm.showAddTemplateTab = show` **before** `updatePageHeight(false)`, so Angular renders the bottom panel while the main list grid still occupies full `pageContentHeight`.
- **Consequence:** Combined heights exceed the viewport allocation, producing visible blank space at the page bottom (layout double-count).

## Probable fix

> **[SKETCH]** This code is triage-time analysis based on a static read of the fault site. Before applying, re-read the surrounding method and any downstream caller of the changed variable or return value.

- **File(s):** `WebContent/scripts/roster/roster.{payRule,timeClocking,alternateWorkLocation,timeRounding,timeCollection}.renderers.js` â€” reorder statements in each `showHide*Panel` helper.
- **Change sketch:** Move `vm.showAddTemplateTab = true/false` to **after** the corresponding `updatePageHeight(false/true)` call so the main grid height is shrunk/restored before the add panel DOM is shown/hidden.
- **Blast radius:** All tenants using the AngularJS Roster module; same anti-pattern exists in ~20 other roster renderer files not listed in WFM-130904 repro (optional follow-up sweep).
- **Migration:** None.
- **PR scope:** Five renderer JS files (+ BAT/UI test on one tab minimum). **Note:** Local working tree already contains this 3-line-per-file fix (uncommitted). Exclude unrelated `roster.jsp` line-ending churn and `roster.contactInfo.renderers.js` mobile-carrier edits from the PR unless intentionally scoped.

```javascript
// [SKETCH] â€” Current (buggy) pattern in HEAD:
this.showHideContactInfoPanel = function (show) {
    vm.showAddTemplateTab = show;   // â† renders panel at full page height
    if (!show) {
        vm.updatePageHeight(true);
        vm.selectedTab = "";
    } else {
        vm.updatePageHeight(false); // â† too late; blank gap already allocated
    }
};

// [SKETCH] â€” Suggested:
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
| TC-NEW-WFM142519-1 | SYSADMIN | Roster > Associate > Pay Rule > Add | COPPEL-style filter (Tienda + Location applied) | Add panel opens flush to bottom; no blank gap below | HIGH | all |
| TC-NEW-WFM142519-2 | SYSADMIN | Repeat Add on Time Clocking, AWL, Time Rounding, Time Collection | Same associate/filter | Same â€” no bottom blank space on any tab | HIGH | all |
| TC-NEW-WFM142519-3 | SYSADMIN | Add then Close on Pay Rule | Panel open | Main grid restores full height; no residual gap | MEDIUM | all |

- **Closest existing TC:** none identified in KB for roster page-height layout â€” why it missed: `visual/layout` defect not covered by functional roster CRUD tests.
- **Extend (optional):** Any existing Roster SIT BAT â€” add viewport screenshot assertion after Add click.

---

## Handoff envelope

**Next owner:**  L3-Engineering
**Next action:** Commit the five renderer reorder fixes (already present locally), open PR targeting fix version 45.1.22.3, and verify all five repro tabs on COPPEL_SB or equivalent SIT.
**SLA:**         P3-next-sprint

**Open questions:**
- **Q1** [OPEN, owner: QA, blocks_fix: no] Should the same `showAddTemplateTab` ordering fix be applied to the ~20 other roster renderer files with the identical pattern (contactInfo, dayOff, availability, etc.) even though not in WFM-130904 repro?

**Distribution:**
- Primary: WFM OM/HR Engineering
- CC: QA (Roster module)

## Evidence trail

- **KB citations:**      none (Step 1 INCONCLUSIVE â€” no matching RNI for roster blank-space layout)
- **Code citations:**    `WebContent/scripts/roster/roster.payRule.renderers.js:showHideContactInfoPanel:200-213`; `WebContent/scripts/roster/roster.timeClocking.renderers.js:showHideContactInfoPanel:478-491`; `WebContent/scripts/roster/roster.alternateWorkLocation.renderers.js:showHideContactInfoPanel:387-399`; `WebContent/scripts/roster/roster.timeRounding.renderers.js:showHideTimeRoundBottomPanel:453-467`; `WebContent/scripts/roster/roster.timeCollection.renderers.js:showHideTimeCollectionPanel:507-518`; `WebContent/ngtemplates/roster/timeClocking/timeClocking.html:23` (pageContentHeight binding); `WebContent/ngtemplates/roster/timeClocking/timeClockingBottomPanel.html:45` (showAddTemplateTab ng-if)
- **Mobile citations:**  not checked (web-only defect)
- **Cross-channel:**     web_only â€” kernel Roster AngularJS UI only
- **TER evidence:**      not triggered
- **Attachments read:**  none
- **Skipped layers:**    [INFO: sws_zta (ZTA sources) not in workspace â€” optional repo; jar fallback if ZTA path implicated] | [INFO: rflx-wfm-shiftbuilding not in workspace â€” optional] | [INFO: rflx-wfm-frameworkscheduling not in workspace â€” optional] | [INFO: rflx-wfm-scheduling not in workspace â€” optional] | [INFO: Step 2 included WebContent client-emission layer (web_js) â€” payload checked before back-end]

### Candidate-cause ledger (UI-triggered)

| Layer | Status | Confidence | Evidence |
|-------|--------|------------|----------|
| client_emission (web_js) | implicated | HIGH | showAddTemplateTab/updatePageHeight ordering in 5 roster renderers |
| angular_template | checked-clear | HIGH | Templates bind pageContentHeight correctly; issue is controller timing |
| server_api | checked-clear | HIGH | No REST call on Add click; pure client layout |
| jsp_scriptlet | checked-clear | MEDIUM | roster.jsp provides pageContentHeight bootstrap only |
| data_config | checked-clear | HIGH | No data dependency |
| i18n | checked-clear | HIGH | Not a string/display issue |

---

## Provenance

**Confidence:** HIGH â€” Step 2 code comparison of HEAD vs working-tree fix; repro steps map 1:1 to five patched files.

**Triage duration:** ~8m
**Model:** composer-2.5-fast

### Refinement log

| Rev | Date | Trigger | Verdict | Confidence | Summary |
|-----|------|---------|---------|------------|---------|
| R1 | 2026-08-05 | /triage WFM-142519 | GENUINE_BUG | HIGH | Client-side roster panel height ordering bug; local fix exists for 5 tabs |

### Confidence flags

- Working-tree uncommitted fix detected â€” verify before PR (do not assume committed).
- roster.jsp shows large line-ending-only diff â€” exclude from fix PR unless intentional.

<!-- /SKILL_SECTION -->
