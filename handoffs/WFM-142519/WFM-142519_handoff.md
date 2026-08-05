---
id: WFM-142519
summary: "Dev story - WFM-130904 - Roster blank space at bottom when adding new record"
verdict: GENUINE_BUG
confidence: HIGH
step_reached: KB_STEP2
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
version: R3
generated: 2026-08-03T10:12:00+05:30
last_updated: 2026-08-05T11:15:00+05:30
duration_minutes: 12
model: composer-2.5-fast
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

**Fix:** Reorder `updatePageHeight()` and `showAddTemplateTab` in the five affected `roster.*.renderers.js` controllers so the list area resizes before the Add bottom panel renders. Fix is already present in the local working tree (5 files, uncommitted).

**Action:** L3-Engineering â€” P3-next-sprint â€” QA-verify on COPPEL_SB; open PR for the five renderer files; escalate to template `listPanelHeight` refactor only if gap persists after deploy.

## Ticket snapshot

- **Module:**             RWS / Employee > Roster (WFM-OM/HR)
- **Customer / flavour:** COPPEL_SB CORP (n/a)
- **Affects version:**    WFM.45.1.22.0.20260416.U003851
- **Priority:**           Medium
- **Stage:**              To Do (Story; parent bug WFM-130904 linked)
- **Original input:**     [NOT APPLICABLE]

## Fix routing (from ticket + code)

**Primary fix location:** web â€” Pure AngularJS Roster 2.0 client layout defect; no server API or mobile channel implicated.

**Channels (fix may be required):**

| Channel | Required? | Evidence |
|---------|-----------|----------|
| Mobile (Shift App) | no | Symptom on RWS web Roster only |
| Web / server | yes | Repro path Employee > Roster; `WebContent/scripts/roster/*.renderers.js` |
| Data / master data | no | No data/config symptom |
| Shared mobile shell (kernel_auth / ZDS) | n/a | Not applicable |

**Implicated areas:**

| Area | Layer | Fix owner | Confidence | Notes |
|------|-------|-----------|------------|-------|
| Pay Rule | client_app (web JS) | web | HIGH | Repro step 7 |
| Time Clocking | client_app | web | HIGH | Repro step 7 |
| Alternate Work Location | client_app | web | HIGH | Repro step 7 |
| Time Rounding | client_app | web | HIGH | Repro step 7 |
| Time Collection | client_app | web | HIGH | Repro step 7 |

**Routing reconciliation:** confirmed â€” Step 2 re-validated 2026-08-05; working-tree diff matches client-side toggle-order root cause.

## Prior analysis critique

No prior analysis present in input.

| Prior claim | Source | Verdict | Reason |
|---|---|---|---|
| â€” | â€” | â€” | â€” |

---

## Probable cause

- **Symptom:**     Empty white space below the Add form after clicking Add on multiple Roster associate detail tabs.
- **Fault:**       `WebContent/scripts/roster/roster.payRule.renderers.js:showHideContactInfoPanel:200` (and parallel show/hide functions in timeClocking, timeCollection, timeRounding, alternateWorkLocation renderers).
- **Trigger:**     Add click sets `showAddTemplateTab = true` **before** `updatePageHeight(false)`, so Angular renders the bottom panel while `pageContentHeight` still reflects the full viewport â€” the list area and panel together exceed the visible page, leaving a blank gap at the bottom.
- **Consequence:** Cosmetic layout defect on all five repro tabs; no data loss or save failure implied.

---

## Probable fix

> **[SKETCH]** This code is triage-time analysis based on a static read of the fault site. Before applying, re-read the surrounding method and any downstream caller of the changed variable or return value. The fix stage (`/capture-fix`) will validate the diff against this sketch and flag any divergence in `Fix stage â†’ Technical notes` with `[WARN: diff does not match Pattern B]`.

- **File(s):**       `roster.payRule.renderers.js`, `roster.timeClocking.renderers.js`, `roster.timeCollection.renderers.js`, `roster.timeRounding.renderers.js`, `roster.alternateWorkLocation.renderers.js` â€” reorder panel toggle vs height recalc.
- **Change sketch:** In each `showHide*Panel(show)` function, call `updatePageHeight(false)` **before** setting `showAddTemplateTab = true` on open; on close, call `updatePageHeight(true)` before `showAddTemplateTab = false`. **Already applied in local working tree.**
- **Blast radius:**  All tenants using Roster 2.0 associate detail tabs; ~30 other roster renderers still use the old toggle order â€” fix five repro tabs first; spot-check others if customer reports additional tabs.
- **Migration:**     No
- **PR scope:**      Five `WebContent/scripts/roster/roster.*.renderers.js` files only (minimal fix). Defer `listPanelHeight` template refactor unless QA still sees gap post-deploy.

```javascript
// [SKETCH] â€” implemented in working tree as of 2026-08-03

// Suggested (now in working tree):
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

- **Closest existing TC:** none identified (Roster BAT focuses on accessibility, not dynamic height on Add panel open) â€” why it missed: scope â€” no viewport-height assertion after Add click.
- **Extend (optional):** Roster BAT suite â€” add viewport assertion after Add on each associate sub-tab.

---

## Handoff envelope

**Next owner:**  L3 Engineering
**Next action:** QA-verify fix on COPPEL_SB (all five tabs); commit and PR the five renderer files; link PR to WFM-130904 / WFM-142519.
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
- **Code citations:**    `WebContent/scripts/roster/roster.payRule.renderers.js:showHideContactInfoPanel:200`, `WebContent/scripts/roster/roster.payRule.renderers.js:updatePageHeight:163`, `WebContent/scripts/roster/roster.timeClocking.renderers.js:showHideContactInfoPanel:478`, `WebContent/scripts/roster/roster.timeCollection.renderers.js:showHideTimeCollectionPanel:504`, `WebContent/scripts/roster/roster.timeRounding.renderers.js:showHideTimeRoundBottomPanel:453`, `WebContent/scripts/roster/roster.alternateWorkLocation.renderers.js:showHideContactInfoPanel:385`, `WebContent/rws/staff/roster.jsp:1590` (global `pageContentHeight` bootstrap)
- **Mobile citations:**  not checked â€” not applicable
- **Cross-channel:**     web_only â€” RWS Roster AngularJS client layout only
- **TER evidence:**      not triggered â€” ticket supplies build, URL, and full repro; screenshot docx not in intake bundle
- **Attachments read:**  none
- **Skipped layers:**    [INFO: sws_zta not in workspace â€” optional] Â· [INFO: rflx-wfm-shiftbuilding not in workspace â€” optional] Â· [INFO: rflx-wfm-frameworkscheduling not in workspace â€” optional] Â· [INFO: rflx-wfm-scheduling not in workspace â€” optional] Â· [INFO: Step 2 included WebContent client-emission layer (web_js)]

---

## Fix stage

**Fix status:**     Applied Fix Captured
**Fix revision:**   F1
**Captured at:**    2026-08-03T15:47:00+05:30
**Last updated:**   2026-08-03T15:47:00+05:30
**Diff scope:**     Working tree (staged + unstaged + untracked) vs HEAD
**Repos scanned:**  rfx-rwsdev300 (5 files), rfx_wfm_dbscripts (0), rfx_wfm_i18n (0)
**Generated by:**   wst-defect-fix skill
**Skill version:**  2.0.0

### Fix log

| Rev | Timestamp | Triggered by | Files touched | Summary of change |
|-----|-----------|--------------|---------------|-------------------|
| F1  | 2026-08-03T15:47:00+05:30 | /capture-fix | 5 | Minimal fix â€” panel toggle order on five Roster associate tabs |

### Root cause

When a manager opened the Add panel on Roster associate detail tabs, the page layout showed the Add form before the main list area had finished shrinking, leaving unused blank space at the bottom of the screen.

### Analysis

On Employee > Roster, after selecting an associate and clicking Add on Pay Rule, Time Clocking, Alternate Work Location, Time Rounding, or Time Collection, a large empty gap appeared below the Add form. The issue was a layout timing problem in the AngularJS client.

### Fix

- The Add bottom panel now waits until the main list area has been resized before it is shown.
- When the Add panel is closed, the list area is restored to full height before the panel is hidden.
- Applied consistently across all five associate detail tabs reported in the defect.

### Area impacted

- [DIRECT] Roster associate detail â€” Add panel (Pay Rule, Time Clocking, Alternate Work Location, Time Rounding, Time Collection) (all flavours): triggered when a manager clicks Add after opening an associate from the Roster filter
- [REGRESSION] Roster associate detail â€” Edit panel and Close on the same five tabs (all flavours): triggered when editing an existing row or closing the Add/Edit panel
- [REGRESSION] Other Roster associate sub-tabs using the same layout pattern (all flavours): tabs not in this fix that share show-panel / resize-page-height behaviour

### Regression scope

| # | Persona | Action | Condition | Expected outcome | Risk | Flavour |
|---|---------|--------|-----------|------------------|------|---------|
| RS-1 | STRADM | Roster > Associate > Pay Rule > Add | Associate selected; CORP profile; filter applied | No blank gap below Add panel | HIGH | all |
| RS-2 | STRADM | Pay Rule > Add then Close | Same as RS-1 | List restores full height; no residual gap | HIGH | all |
| RS-3 | STRADM | Repeat Add on Time Clocking, Time Collection, Time Rounding, Alternate Work Location | Same associate context | No bottom blank space on each tab | HIGH | all |
| RS-4 | STRADM | Edit existing row on any of the five tabs | At least one existing record | Edit panel layout correct | MEDIUM | all |

### Technical notes

- **Code-level root cause:** `showHide*Panel` set `showAddTemplateTab` before `updatePageHeight()`, rendering the bottom panel before `pageContentHeight` was reduced.
- **Why this fix approach:** Minimal scope â€” reorder toggle calls in five renderer files without template refactor.
- **Diff vs triage:** Partial match â€” toggle order implemented; `listPanelHeight` template changes deferred.
- **Re-triage 2026-08-05:** Working tree still contains the five-file fix; unrelated local edits also present (`roster.contactInfo.renderers.js`, `summaryContactInfo.html`) â€” exclude from WFM-142519 PR.

**Changed files (by repo):**

- **rfx-rwsdev300**
  - `WebContent/scripts/roster/roster.payRule.renderers.js`
  - `WebContent/scripts/roster/roster.timeClocking.renderers.js`
  - `WebContent/scripts/roster/roster.timeCollection.renderers.js`
  - `WebContent/scripts/roster/roster.timeRounding.renderers.js`
  - `WebContent/scripts/roster/roster.alternateWorkLocation.renderers.js`

### Test cases executed

[NOT PROVIDED] â€” manual UI verification on COPPEL_SB pending.

---

## Provenance

**Triage duration:** 12m 0s
**Model:**           composer-2.5-fast
**Skill version:**   2.2.0
**Generated by:**    wst-defect-triage skill

### Refinement log

| Rev | Timestamp | Triggered by | Summary of change |
|-----|-----------|--------------|-------------------|
| R1  | 2026-08-03T10:12:00+05:30 | Initial triage | First analysis â€” GENUINE_BUG, MEDIUM |
| R2  | 2026-08-03T15:47:00+05:30 | /capture-fix (wst-defect-fix) | Fix stage F1 added â€” 5 files |
| R3  | 2026-08-05T11:15:00+05:30 | /triage re-run (wst-defect-triage) | Re-validated root cause; confidence raised HIGH; confirmed fix still in working tree |

### Confidence flags

- [INFO: TER skipped â€” ticket supplies build + repro; screenshot docx not in intake bundle]
- [INFO: Linked parent bug WFM-130904 â€” dev story tracks same defect]
- [INFO: Step 2 client_emission implicated; server_api checked-clear]
- [INFO: Fix capture F1 â€” minimal apply scope; template/listPanelHeight changes deferred]
- [INFO: Re-triage R3 â€” working-tree diff confirms toggle-order fix matches Probable cause]

---

## Clarifications

**2026-08-05** â€” Re-triage confirmed the five-file renderer fix remains in the local working tree (uncommitted). Unrelated edits in `roster.contactInfo.renderers.js` and `summaryContactInfo.html` should not be bundled into the WFM-142519 PR.

<!-- REFINE: paste new context, Q&A answers, or follow-up findings below this line.
     Then attach this file in a chat with the wst-defect-triage skill to trigger refinement. -->

<!-- FIX: paste fix context, test outcomes, or follow-up diff notes below this line.
     Then attach this file in a chat with the wst-defect-fix skill to trigger /update-fix. -->

---
