---
id: WFM-142519
summary: "Roster associate tabs show blank space at page bottom when Add panel opens (Pay Rule, Time Clocking, AWL, Time Rounding, Time Collection)"
verdict: GENUINE_BUG
confidence: HIGH
step_reached: KB_STEP2
input_form: jira_json
module: HR
customer: COPPEL
flavour: null
affects_version: "45.1.22.0"
fix_version: "45.1.22.3"
priority: Medium
blast_radius: global
next_owner: L3-Engineering
sla: P3-next-sprint
status: Draft
version: R1
generated: "2026-08-05T07:20:00+05:30"
last_updated: "2026-08-05T07:20:00+05:30"
duration_minutes: 8
model: composer-2.5-fast
skill_version: "2.2.0"
attachments_read: null
fix_primary: web
channels_mobile: false
channels_web: true
channels_data: false
implicated_areas_count: 5
---

# Triage: WFM-142519 â€” Roster blank space at bottom when Add panel opens

## TL;DR

**Bug:** On Employee > Roster associate detail tabs (Pay Rule, Time Clocking, Alternate Work Location, Time Rounding, Time Collection), clicking **Add** opens the bottom edit panel but leaves a visible blank gap at the bottom of the page because the add panel is shown before `pageContentHeight` is reduced.
**Fix:** In each affected roster renderer, call `updatePageHeight(false)` **before** setting `showAddTemplateTab = true` (and set `showAddTemplateTab = false` only after `updatePageHeight(true)` when closing). Pattern already present as uncommitted local diff on five renderer files.
**Action:** L3-Engineering â€” P3-next-sprint â€” Apply the reorder fix across all implicated roster renderers, regression-test Add/Close on the five tabs listed in WFM-130904, and scan other roster tabs using the same `showAddTemplateTab = show` anti-pattern.

## Ticket snapshot

- **Module:**             HR / Employee > Roster
- **Customer / flavour:** COPPEL (COPPEL_SB â€” n/a)
- **Affects version:**    45.1.22.0 (build WFM.45.1.22.0.20260416.U003851)
- **Priority:**           Medium
- **Stage:**              To Do (dev story for WFM-130904)
- **Original input:**     [NOT APPLICABLE]

## Fix routing (from ticket + code)

**Primary fix location:** web â€” AngularJS roster client layout; no server/API fault.

**Channels (fix may be required):**

| Channel | Required? | Evidence |
|---------|-----------|----------|
| Mobile (Shift App) | no | Repro is RWS web Roster only |
| Web / server | yes | Employee > Roster Angular templates + `WebContent/scripts/roster/*.renderers.js` |
| Data / master data | no | Pure UI height calculation |
| Shared mobile shell (kernel_auth / ZDS) | n/a | â€” |

**Implicated areas:**

| Area | Layer | Fix owner | Confidence | Notes |
|------|-------|-----------|------------|-------|
| Roster > Pay Rule | client_emission (web_js) | web | HIGH | WFM-130904 repro step 7 |
| Roster > Time Clocking | client_emission (web_js) | web | HIGH | same |
| Roster > Alternate Work Location | client_emission (web_js) | web | HIGH | same |
| Roster > Time Rounding | client_emission (web_js) | web | HIGH | same |
| Roster > Time Collection | client_emission (web_js) | web | HIGH | same |

**Routing reconciliation:** confirmed â€” Step 2 code walk confirms client-side layout ordering bug; no E1 network evidence required.

## Prior analysis critique

No prior analysis present in input.

---

## Probable cause

- **Symptom:**     After clicking **Add** on several Roster associate sub-tabs, a blank white gap appears at the bottom of the page below the add/edit bottom panel.
- **Fault:**       `WebContent/scripts/roster/roster.payRule.renderers.js:showHideContactInfoPanel` (and parallel methods in timeClocking, alternateWorkLocation, timeRounding, timeCollection renderers) â€” `showAddTemplateTab` was toggled before `updatePageHeight()`, so the bottom panel rendered while the outer container still used the full viewport height.
- **Trigger:**     User opens Add on any affected tab; Angular digest renders `payRuleBottomPanel.html` (via `ng-if="showAddTemplateTab"`) before `pageContentHeight` is decremented to reserve space for `entityDetailsHeaderHeight + entityDetailsContentHeight`.
- **Consequence:** Outer `tab-content` height (`ng-style="{height: pageContentHeight}"`) and inner list card height disagree with the newly visible bottom panel â€” unused vertical space appears as blank area at page bottom.

---

## Probable fix

> **[SKETCH]** Triage-time analysis from static read. Validate in browser before merge.

- **File(s):**       `WebContent/scripts/roster/roster.{payRule,timeClocking,alternateWorkLocation,timeRounding,timeCollection}.renderers.js` â€” reorder panel show/hide vs height update
- **Change sketch:** Move `vm.showAddTemplateTab = true` to **after** `vm.updatePageHeight(false)` when opening; move `vm.showAddTemplateTab = false` to **after** `vm.updatePageHeight(true)` when closing. Do not set the flag before height recalculation.
- **Blast radius:**  All tenants using Roster associate detail tabs with bottom add panels; pattern may exist in other roster renderers still using `vm.showAddTemplateTab = show` at function entry (contactInfo, timeOff, availability, etc.) â€” audit recommended.
- **Migration:**     None
- **PR scope:**      Five renderer JS files (minimum); optional follow-up PR for remaining roster tabs with identical anti-pattern; manual UI regression on COPPEL_SB repro path.

```javascript
// [SKETCH] â€” triage-time only
// Current (buggy):
this.showHideContactInfoPanel = function (show) {
    vm.showAddTemplateTab = show;   // panel renders at full pageContentHeight
    if (!show) { vm.updatePageHeight(true); ... }
    else { vm.updatePageHeight(false); }
};

// Suggested:
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
| TC-NEW-WFM142519-01 | SYSADMIN | Roster > Associate > Pay Rule > Add | COPPEL-style filter (Tienda + Location applied) | Add bottom panel fills cleanly; no blank gap below panel | HIGH | all |
| TC-NEW-WFM142519-02 | SYSADMIN | Repeat Add/Close on Time Clocking, AWL, Time Rounding, Time Collection tabs | Same associate selected | No blank bottom space; list area height adjusts on open/close | HIGH | all |
| TC-NEW-WFM142519-03 | SYSADMIN | Pay Rule > Add then Close (X) | Panel open | `pageContentHeight` restores to pre-add value; no residual gap | MED | all |

- **Closest existing TC:** none identified for Roster bottom-panel height regression â€” why it missed: `scope` â€” layout defect on add-panel open not covered in SIT roster functional suites.
- **Extend (optional):** any existing Roster UI BAT â€” add viewport screenshot assertion after Add click.

---

## Handoff envelope

**Next owner:**  L3-Engineering
**Next action:** Commit the five-file reorder fix (already in local working tree), audit remaining roster renderers for `showAddTemplateTab = show` at function entry, and verify on COPPEL_SB repro.
**SLA:**         P3-next-sprint

## Open questions

| ID | State | Owner | blocks_fix | Text |
|---|---|---|---|---|
| OQ-1 | [OPEN] | QA | no | Confirm whether Contact Info and other roster tabs outside the five listed in WFM-130904 exhibit the same gap (same anti-pattern exists in `roster.contactInfo.renderers.js`). |
| OQ-2 | [OPEN] | Reporter | no | Attach screenshot or docx from original WFM-130904 if visual confirmation of gap size is needed for sign-off. |

---

## Evidence trail

**KB citations:**
- Module map: Employee > Roster (HR) â€” layout-only symptom; no matching RNI for blank-space add-panel defect in `release_notes_registry.json`.

**Code citations:**
- `rfx-rwsdev300:WebContent/scripts/roster/roster.payRule.renderers.js:showHideContactInfoPanel:200-212`
- `rfx-rwsdev300:WebContent/scripts/roster/roster.timeClocking.renderers.js:showHideContactInfoPanel:478-490`
- `rfx-rwsdev300:WebContent/scripts/roster/roster.alternateWorkLocation.renderers.js:showHideContactInfoPanel:386-398`
- `rfx-rwsdev300:WebContent/scripts/roster/roster.timeRounding.renderers.js:showHideTimeRoundBottomPanel:454-466`
- `rfx-rwsdev300:WebContent/scripts/roster/roster.timeCollection.renderers.js:showHideTimeCollectionPanel:505-517`
- `rfx-rwsdev300:WebContent/ngtemplates/roster/payRule/payRule.html:23,51,76` â€” dual `pageContentHeight` bindings + conditional bottom panel include

**Candidate-cause ledger (Step 2 Â§S2.2):**

| Layer | Status | Confidence | Evidence |
|---|---|---|---|
| client_emission (web_js) | implicated | HIGH | showAddTemplateTab / updatePageHeight ordering |
| angular_template | checked-clear | HIGH | Templates bind correctly to pageContentHeight; fault is JS timing |
| server_api | checked-clear | HIGH | No save/submit failure; pure display layout |
| jsp_scriptlet | not-checked | â€” | Roster is AngularJS module, not JSP submit path |
| data_config | checked-clear | HIGH | No config key involved |
| i18n | checked-clear | HIGH | Not locale-specific |

**Attachments read:** none

**Skipped layers:**
- [INFO: sws_zta (ZTA sources) not in workspace â€” optional repo; jar fallback if ZTA path implicated]
- [INFO: rflx-wfm-shiftbuilding not in workspace â€” optional]
- [INFO: rflx-wfm-frameworkscheduling not in workspace â€” optional]
- [INFO: rflx-wfm-scheduling not in workspace â€” optional]

## Provenance

### Confidence flags

- Step 1 INCONCLUSIVE (no KB/RNI match for layout symptom); Step 2 HIGH on code path.
- Uncommitted local diff on five renderer files matches proposed fix â€” strengthens confidence.
- Jira attachment `Blank space at bottom of the page.docx` referenced in description but not downloaded to intake bundle.

### Refinement log

| Version | Date | Trigger | Summary |
|---|---|---|---|
| R1 | 2026-08-05 | /triage WFM-142519 | Initial triage â€” GENUINE_BUG, client layout ordering in roster renderers |

---

*Triage duration: ~8m Â· Model: composer-2.5-fast Â· Skill: wst-defect-triage v2.2.0*
