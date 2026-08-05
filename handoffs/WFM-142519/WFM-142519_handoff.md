---
id: WFM-142519
summary: "Dev story - WFM-130904 - Roster blank space at bottom when adding new record"
verdict: GENUINE_BUG
confidence: MEDIUM
step_reached: KB_TER_STEP2
input_form: jira_json
module: RWS / Employee > Roster
customer: COPPEL_SB CORP
flavour: null
affects_version: WFM.45.1.22.0.20260416.U003851
fix_version: 45.1.22.2
priority: Medium
blast_radius: global
next_owner: L3-Engineering
sla: P3-next-sprint
status: Applied-Fix-Captured
version: R1
generated: 2026-08-03T10:12:00+05:30
last_updated: 2026-08-03T15:47:00+05:30
duration_minutes: 8
model: null
skill_version: "2.2.0"
attachments_read: null
fix_primary: web
channels_mobile: false
channels_web: true
channels_data: false
implicated_areas_count: 5
---

# Triage: WFM-142519 â€” Roster blank space at bottom when adding new record

## TL;DR

**Bug:** On Employee > Roster > Associate, clicking **Add** on Pay Rule, Time Clocking, Alternate Work Location, Time Rounding, or Time Collection leaves a large blank gap at the bottom of the page.

**Fix:** Correct the Roster 2.0 client-side page-height budget in affected `roster.*.renderers.js` controllers and `ngtemplates/roster/` HTML templates â€” inner list panels reuse full `pageContentHeight` without subtracting static headers, and the Add bottom-panel toggle recalculates height in the wrong order.

**Action:** L3-Engineering â€” P3-next-sprint â€” Review the fix sketch in *Probable fix*; implement height corrections for all five repro tabs; verify on COPPEL_SB build.

## Ticket snapshot

- **Module:**             RWS / Employee > Roster
- **Customer / flavour:** COPPEL_SB CORP (n/a)
- **Affects version:**    WFM.45.1.22.0.20260416.U003851
- **Priority:**           Medium
- **Stage:**              Open (Story / To Do)
- **Original input:**     [NOT APPLICABLE]

## Fix routing (from ticket + code)

**Primary fix location:** web â€” Pure AngularJS Roster 2.0 layout defect; no server API or mobile channel implicated.

**Channels (fix may be required):**

| Channel | Required? | Evidence |
|---------|-----------|----------|
| Mobile (Shift App) | no | Symptom on RWS web Roster only |
| Web / server | yes | Repro path Employee > Roster; client height model in `WebContent/scripts/roster/` |
| Data / master data | no | No data/config symptom |
| Shared mobile shell (kernel_auth / ZDS) | n/a | Not applicable |

**Implicated areas:**

| Area | Layer | Fix owner | Confidence | Notes |
|------|-------|-----------|------------|-------|
| Pay Rule | client_app (web JS/HTML) | web | HIGH | Repro step 7 |
| Time Clocking | client_app | web | HIGH | Repro step 7 |
| Alternate Work Location | client_app | web | HIGH | Repro step 7 |
| Time Rounding | client_app | web | HIGH | Repro step 7 |
| Time Collection | client_app | web | HIGH | Repro step 7 |

**Routing reconciliation:** confirmed â€” Step 2 code walk confirms client-side height miscalculation; no server path.

## Prior analysis critique

No prior analysis present in input.

| Prior claim | Source | Verdict | Reason |
|---|---|---|---|
| â€” | â€” | â€” | â€” |

---

## Probable cause

- **Symptom:**     Empty white space below the Add form after clicking Add on multiple Roster associate detail tabs.
- **Fault:**       `WebContent/scripts/roster/roster.payRule.renderers.js:updatePageHeight` (and parallel controllers for Time Clocking, Time Collection, Time Rounding, Alternate Work Location) â€” page height budget does not account for static header rows above the list panel.
- **Trigger:**     Add click â†’ bottom panel shown via `showHideContactInfoPanel(true)` â†’ `updatePageHeight(false)` shrinks `pageContentHeight` while inner list `ws-card` still binds to the same height and static headers sit above it uncounted.
- **Consequence:** Cosmetic layout defect; visible blank gap at page bottom; no data loss or save failure implied.
- **What raises to HIGH:** Screenshot from `Blank space at bottom of the page.docx` attached to parent WFM-130904, or DevTools confirmation that outer container height exceeds sum of visible child elements.

---

## Probable fix

> **[SKETCH]** This code is triage-time analysis based on a static read of the fault site. Before applying, re-read the surrounding method and any downstream caller of the changed variable or return value. The fix stage (`/capture-fix`) will validate the diff against this sketch and flag any divergence in `Fix stage â†’ Technical notes` with `[WARN: diff does not match Pattern B]`.

- **File(s):**       `roster.payRule.renderers.js`, `roster.timeClocking.renderers.js`, `roster.timeCollection.renderers.js`, `roster.timeRounding.renderers.js`, `roster.alternateWorkLocation.renderers.js` and matching `ngtemplates/roster/*/` HTML + bottom-panel templates â€” adjust height model and panel toggle order.
- **Change sketch:** Introduce a list-area height (e.g. `listPanelHeight = pageContentHeight - headerStackOffset`) for the inner list `ws-card` instead of binding it to raw `pageContentHeight`. In `showHide*Panel()`, call `updatePageHeight()` **before** setting `showAddTemplateTab`. Remove one-time `::` bindings on dynamic heights in bottom-panel templates (e.g. `payRuleBottomPanel.html` line 48) so `entityDetailsContentHeight` adjustments apply after the +50 minimum-height hack.
- **Blast radius:**  All tenants using Roster 2.0 associate detail tabs; ~20+ roster renderers share the same pattern â€” fix the five repro tabs first, then spot-check others (Accrual Balance, Devices).
- **Migration:**     No
- **PR scope:**      `WebContent/scripts/roster/roster.{payRule,timeClocking,timeCollection,timeRounding,alternateWorkLocation}.renderers.js` + corresponding `ngtemplates/roster/` HTML; manual UI verify on all five tabs.

```javascript
// [SKETCH] â€” triage-time only; verify per-screen header offset before applying.

// Current (buggy) â€” showHideContactInfoPanel in payRule:
this.showHideContactInfoPanel = function (show) {
    vm.showAddTemplateTab = show;          // panel renders before height recalc
    if (!show) { vm.updatePageHeight(true); }
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

// Template â€” payRule.html: bind inner list card to listPanelHeight, not pageContentHeight
// ng-style="{height: payRuleCtrl.listPanelHeight + 'px'}"
// where listPanelHeight = pageContentHeight - agreementHeaderOffset - columnHeaderOffset
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

- **Closest existing TC:** none identified (Roster WCAG BAT exists; no viewport-height Add-panel coverage) â€” why it missed: scope â€” existing tests focus on accessibility labels, not dynamic height budgeting on Add panel open.
- **Extend (optional):** Roster BAT suite â€” add viewport assertion after Add click on each associate sub-tab.

---

## Handoff envelope

**Next owner:**  L3 Engineering
**Next action:** Review the fix sketch in *Probable fix*; confirm scope; implement height corrections for Pay Rule, Time Clocking, Time Collection, Time Rounding, and Alternate Work Location; link PR to WFM-130904 / WFM-142519.
**SLA:**         P3 next sprint

**Open questions:**
- **Q1** [OPEN, owner: QA, blocks_fix: no] Attach/read `Blank space at bottom of the page.docx` from WFM-130904 â€” confirm gap is below Add form, not a scrollbar artifact.
- **Q2** [OPEN, owner: QA, blocks_fix: no] Repro confirmed only on COPPEL_SB or also on generic SIT (B0)?
- **Q3** [OPEN, owner: Dev, blocks_fix: no] Parent bug WFM-130904 â€” any prior fix attempt or partial PR to reconcile?

**Distribution:**
- Primary: Eng DL + Scrum team
- CC: CSE (COPPEL_SB account)

## Evidence trail

- **KB citations:**      none (Step 1 INCONCLUSIVE â€” no matching BR/RNI/config for Roster blank-space layout)
- **Code citations:**    `WebContent/scripts/roster/roster.payRule.renderers.js:updatePageHeight:163`, `WebContent/scripts/roster/roster.payRule.renderers.js:showHideContactInfoPanel:200`, `WebContent/ngtemplates/roster/payRule/payRule.html:23,51,76`, `WebContent/scripts/roster/roster.timeClocking.renderers.js:updatePageHeight:492`, `WebContent/scripts/roster/roster.timeCollection.renderers.js:showHideTimeCollectionPanel:505`, `WebContent/scripts/roster/roster.timeRounding.renderers.js:updatePageHeight:505`, `WebContent/scripts/roster/roster.alternateWorkLocation.renderers.js:updatePageHeight:417`, `WebContent/rws/staff/roster.jsp:1590` (global `pageContentHeight` bootstrap)
- **Mobile citations:**  not checked (mobile repos missing) â€” not applicable
- **Cross-channel:**     web_only â€” RWS Roster AngularJS client layout only
- **TER evidence:**      not triggered â€” ticket already supplies build, URL, and full repro; screenshot docx not downloaded
- **Attachments read:**  none
- **Skipped layers:**    [INFO: sws_zta (ZTA sources) not in workspace â€” optional repo; jar fallback if ZTA path implicated] Â· [INFO: rflx-wfm-shiftbuilding not in workspace â€” optional] Â· [INFO: rflx-wfm-frameworkscheduling not in workspace â€” optional] Â· [INFO: rflx-wfm-scheduling not in workspace â€” optional] Â· [INFO: Step 2 included WebContent client-emission layer (web_js) â€” payload checked before back-end]

---

## Fix stage

**Fix status:**     Applied Fix Captured
**Fix revision:**   F1
**Captured at:**    2026-08-03T15:47:00+05:30
**Last updated:**   2026-08-03T15:47:00+05:30
**Diff scope:**     Working tree (staged + unstaged + untracked) vs HEAD
**Repos scanned:**  rfx-rwsdev300 (5 files), rfx_wfm_dbscripts (0 files), rfx_wfm_i18n (0 files), sws_zta (0 â€” ZTA_ROOT not in workspace), rflx-wfm-shiftbuilding (0 â€” SHIFTBUILDING_ROOT not in workspace), rflx-wfm-frameworkscheduling (0 â€” FRAMEWORKSCHEDULING_ROOT not in workspace), rflx-wfm-scheduling (0 â€” SCHEDULING_ROOT not in workspace)
**Generated by:**   wst-defect-fix skill
**Skill version:**  2.0.0

### Fix log

| Rev | Timestamp | Triggered by | Files touched | Summary of change |
|-----|-----------|--------------|---------------|-------------------|
| F1  | 2026-08-03T15:47:00+05:30 | /capture-fix | 5 | Minimal fix â€” panel toggle order: recalculate page height before showing/hiding Add bottom panel on five Roster associate tabs |

### Root cause

When a manager opened the Add panel on Roster associate detail tabs, the page layout resized the list area and showed the Add form in the wrong order, leaving unused blank space at the bottom of the screen.

### Analysis

On Employee > Roster, after selecting an associate and clicking Add on Pay Rule, Time Clocking, Alternate Work Location, Time Rounding, or Time Collection, a large empty gap appeared below the Add form. The issue was a layout timing problem: the Add panel became visible before the main list area had finished shrinking to make room for it, so the combined page height did not fit the viewport correctly.

### Fix

- The Add bottom panel now waits until the main list area has been resized before it is shown.
- When the Add panel is closed, the list area is restored to full height before the panel is hidden.
- The same behaviour was applied consistently across all five associate detail tabs reported in the defect.

### Area impacted

- [DIRECT] Roster associate detail â€” Add panel (Pay Rule, Time Clocking, Alternate Work Location, Time Rounding, Time Collection) (all flavours): triggered when a manager clicks Add on any of these tabs after opening an associate from the Roster filter
- [REGRESSION] Roster associate detail â€” Edit panel and Close on the same five tabs (all flavours): triggered when editing an existing row or closing the Add/Edit panel without saving
- [REGRESSION] Other Roster associate sub-tabs using the same layout pattern (all flavours): tabs not in this fix that share the show-panel / resize-page-height behaviour

### Regression scope

| # | Persona | Action | Condition | Expected outcome | Risk | Flavour |
|---|---------|--------|-----------|------------------|------|---------|
| RS-1 | STRADM | Roster > Associate > Pay Rule > Add | Associate selected; CORP profile; filter applied | No blank gap below Add panel; form and list fill viewport without empty footer space | HIGH | all |
| RS-2 | STRADM | Roster > Associate > Pay Rule > Add then Close | Same as RS-1 | Add panel closes; list area returns to full height with no residual gap | HIGH | all |
| RS-3 | STRADM | Repeat Add on Time Clocking, Time Collection, Time Rounding, Alternate Work Location | Same associate context as RS-1 | Same layout as RS-1 on each tab â€” no bottom blank space | HIGH | all |
| RS-4 | STRADM | Edit existing row on any of the five tabs | At least one existing policy/record on the tab | Edit bottom panel opens with correct layout; no new blank gap | MEDIUM | all |

### Technical notes

- **Code-level root cause:** `showHideContactInfoPanel` / `showHideTimeCollectionPanel` / `showHideTimeRoundBottomPanel` set `showAddTemplateTab` before calling `updatePageHeight()`, so Angular rendered the bottom panel before `pageContentHeight` was reduced for the list area.
- **Why this fix approach:** Developer chose minimal scope (suggestion [2] from triage only) â€” reorder toggle calls in five renderer files without template or `listPanelHeight` refactor.
- **Diff vs triage:** Partial match to Pattern B sketch â€” panel toggle order implemented as triage suggested; `listPanelHeight` template changes and `::` binding removal from Probable fix were **not** applied (`[SKIPPED BY DEVELOPER]` minimal-change request).
- **[WARN: diff does not match Probable fix Pattern B file list â€” diff touched 5 JS renderer files only; brief also specified ngtemplates/roster HTML and bottom-panel template binding changes]**
- **Caveats:** If blank space persists after deploy, apply follow-up: introduce `listPanelHeight` on inner list cards and remove one-time `::` height bindings in bottom-panel HTML.

**Changed files (by repo):**

- **rfx-rwsdev300**
  - `WebContent/scripts/roster/roster.payRule.renderers.js:showHideContactInfoPanel` (+2/-1 lines) â€” updatePageHeight before showAddTemplateTab
  - `WebContent/scripts/roster/roster.timeClocking.renderers.js:showHideContactInfoPanel` (+2/-1 lines) â€” same
  - `WebContent/scripts/roster/roster.timeCollection.renderers.js:showHideTimeCollectionPanel` (+2/-1 lines) â€” same
  - `WebContent/scripts/roster/roster.timeRounding.renderers.js:showHideTimeRoundBottomPanel` (+2/-1 lines) â€” same
  - `WebContent/scripts/roster/roster.alternateWorkLocation.renderers.js:showHideContactInfoPanel` (+2/-1 lines) â€” same

### Test cases executed

[NOT PROVIDED] â€” fix captured from working tree; manual UI verification on COPPEL_SB pending. Recommended checks: TC-NEG-WFM142519-01 through -05 from Test gap section.

---

## Provenance

**Triage duration:** 8m 0s
**Model:**           [NOT PROVIDED]
**Skill version:**   2.2.0
**Generated by:**    wst-defect-triage skill

### Refinement log

| Rev | Timestamp | Triggered by | Summary of change |
|-----|-----------|--------------|-------------------|
| R1  | 2026-08-03T10:12:00+05:30 | Initial triage | First analysis â€” GENUINE_BUG, MEDIUM |
| R2  | 2026-08-03T15:47:00+05:30 | /capture-fix (wst-defect-fix) | Fix stage F1 added â€” 5 files across rfx-rwsdev300 |

### Confidence flags

- [INFO: TER skipped â€” ticket supplies build + repro; visual screenshot not in intake bundle]
- [INFO: Linked parent bug WFM-130904 â€” dev story tracks same defect]
- [INFO: Step 2 candidate-cause ledger â€” client_emission implicated; server_api checked-clear]
- [INFO: Fix capture â€” minimal apply scope; template/listPanelHeight changes deferred]

---

## Clarifications

<!-- REFINE: paste new context, Q&A answers, or follow-up findings below this line.
     Then attach this file in a chat with the wst-defect-triage skill to trigger refinement. -->

<!-- FIX: paste fix context, test outcomes, or follow-up diff notes below this line.
     Then attach this file in a chat with the wst-defect-fix skill to trigger /update-fix. -->

---
