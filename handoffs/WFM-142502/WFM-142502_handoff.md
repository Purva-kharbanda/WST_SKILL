# WFM-142502 — Handoff Document

*Multi-skill handoff document. Each section below represents a different skill output.*

<!-- SKILL_SECTION: wst-defect-triage R1 -->
## TRIAGE (R1)
*Updated: 2026-08-07 11:30*

﻿---
ticket_id: WFM-142502
summary: "v45 WFM CORP (RWS): Forecast scenario Add scenario window missing Previous button"
verdict: GENUINE_BUG
confidence: HIGH
step_reached: KB_STEP2
input_form: jira_api
status: Draft
version: R1
fix_primary: web
fault_locus: client_app
channel_confidence: HIGH
channels:
  mobile: false
  web: true
  data: false
  shared_mobile_shell: false
blast_radius: global
next_owner: DEVELOPER
sla: SIT_FLAVOUR
model: composer-2.5
duration_minutes: 12
attachments_read:
  - "jira:WFM_CORP_Forecast_scenario_Add_window_cancel_Previous_button_missing.mp4 (skipped)"
  - "jira:WFM-142502-20260727-2140.md (skipped per user instruction)"
kb_citations:
  - "PRE_SCENARIO kernel page — Forecast Scenario list"
  - "INITIALIZE_WEEK_FORECAST_USING_ML kernel page — Initialize Forecast / Add flow target"
code_citations:
  - "rfx-rwsdev300:WebContent/scripts/predictive/scenario.renderer.js:addScenario:187-193"
  - "rfx-rwsdev300:WebContent/ngtemplates/predictive/initializeForecast/initializeForecastTemplate.html:bottom-container:201-205"
  - "rfx-rwsdev300:WebContent/ngtemplates/predictive/buildMachineLearningModel/buildMachineLearningModelAddEditTemplate.html:Previous button pattern"
  - "rfx-rwsdev300:WebContent/scripts/predictive/initialize.forecast.renderer.js:clearScenario (no backToScenarioList)"
open_questions:
  - id: OQ-1
    state: OPEN
    owner: BA
    blocks_fix: false
    text: "Should Previous (and Cancel per attachment filename) appear only when entered via Forecast Scenario > Add, or also when Initialize Forecast is opened directly from the kernel menu?"
---

# Triage: WFM-142502

**Status:** Draft  
**Version:** R1  
**Confidence:** HIGH  
**Triage status:** GENUINE_BUG  
**Last updated:** 2026-08-07  
**Triage duration:** ~12m  
**Model:** composer-2.5  

---

## TL;DR

| | |
|---|---|
| **Bug** | The **Add Scenario** flow (Forecast Scenario → Add, which loads `initializeForecastUsingML.jsp`) shows **Clear / Initialize / Initialize & Save** but **no Previous** button, so users cannot navigate back to the Forecast Scenario list without browser back. |
| **Fix** | Add a **Previous** tertiary button to `initializeForecastTemplate.html` and a `backToScenarioList()` handler in `initialize.forecast.renderer.js` that form-posts back to `scenario.jsp` — mirror peer predictive add/edit screens and the existing `addScenario()` forward navigation. |
| **Action** | Web UI fix in `rfx-rwsdev300`; no server/DB/config change required. Confirm with QA whether **Cancel** is also in scope (attachment name suggests it; ticket text cites Previous only). |

---

## Ticket snapshot

| Field | Value |
|---|---|
| **Summary** | v45 WFM CORP (RWS): Forecast scenario Add scenario window missing Previous button |
| **Status** | To Do |
| **Priority** | Medium |
| **Affects version** | 45.1.22.1 |
| **Fix version** | 45.1.24.0 |
| **Labels** | V45SIT |
| **WFM module** | WFM-Driver/Budget/Labor Forecast · Driver Forecast |
| **Environment** | SIT — `wfmproductqa11.reflexisinc.com` |
| **Reporter** | Poonam Kshirsagar |

**Steps to reproduce:** CORP Admin → Forecast → Initialize Forecast → open Add scenario window → observe navigation buttons.

**Expected:** Previous button present for navigation.  
**Actual:** Previous button missing.

**Original input:** [NOT APPLICABLE]

**Input form:** Jira API (hook) — user instructed to **ignore** attached `.md` triage artefact (`WFM-142502-20260727-2140.md`).

---

## Fix routing (from ticket + code)

```
Ticket fix routing (from Jira text):
  Symptom surface: web (kernel AngularJS — Forecast / Initialize Forecast)
  Fault locus:     client_app (missing template control)
  Evidence tier:   E5 surface-only → confirmed E1-n/a / Step 2 template walk
  Channel conf:    HIGH (pure UI omission; no API payload involved)
  Channels touched: mobile=F  web=T  data=F  shared_shell=F
  Fix primary:     web (template + controller navigation handler)
  Implicated:      1 area — Initialize Forecast Add Scenario form footer
  Sub-issues:      not a composite ticket
  Step 2 plan:     walk SERVER WebContent client-emission (§1b) — confirmed
```

**Ticket hypothesis → Step 2:** confirmed

---

## Probable cause

| Field | Detail |
|---|---|
| **Symptom** | Previous navigation button absent in Add Scenario window on Forecast scenario page. |
| **Fault** | `initializeForecastTemplate.html` footer renders only **Clear**, **Initialize**, and **Initialize & Save** — no **Previous** control. `initialize.forecast.renderer.js` defines `clearScenario()` but **no** `backToScenarioList()` / form submit back to `scenario.jsp`. |
| **Trigger** | User clicks **Add Scenario** on Forecast Scenario list (`scenario.renderer.js` → `addScenario()` posts `scenarioForm` to `initializeForecastUsingML.jsp`). |
| **Consequence** | User is stranded on Initialize Forecast page unless using browser back; inconsistent with other predictive add/edit flows (e.g. Build ML Model) that expose Previous. |
| **Raises to HIGH** | Static template inspection + navigation code path traced; peer-screen pattern confirms omission is unintentional UX gap. |

### Prior analysis critique

No L1/L2/L3 Jira comments on ticket. Attached `.md` intentionally excluded per user request — triage based on Jira JSON + code only.

### Candidate-cause ledger (Step 2 §S2.2)

| Layer | Status | Confidence | Evidence |
|---|---|---|---|
| client_emission (web_js/html) | **implicated** | HIGH | Missing button in `initializeForecastTemplate.html:201-205` |
| server_api | checked-clear | HIGH | Display-only; no REST failure |
| data_config | checked-clear | HIGH | Not data-driven |
| permission | checked-clear | MEDIUM | CORP Admin reproduces — not LOS |
| i18n | checked-clear | HIGH | `Previous` string used on peer templates via `ctrl.i18nFn('Previous')` |
| jsp_shell | checked-clear | HIGH | `initializeForecastUsingML.jsp` includes template correctly |

---

## Probable fix

1. **`initializeForecastTemplate.html`** — In `.bottom-container`, add a **Previous** button before Clear, matching peer pattern:

   [SKETCH] `<button type="button" class="btn btn-tertiary" ng-click="ctrl.backToScenarioList()">{{::ctrl.i18nFn('Previous')}}</button>`

   Reference: `buildMachineLearningModelAddEditTemplate.html` (same predictive module family).

2. **`initialize.forecast.renderer.js`** — Add navigation handler mirroring forward path in `scenario.renderer.js:addScenario`:

   [SKETCH]
   ```javascript
   vm.backToScenarioList = function () {
     document.initializeForecastScenarioSetup.action =
       vm.appName + '/predictive/views/scenario.jsp';
     document.initializeForecastScenarioSetup.submit();
   };
   ```

3. **Optional (confirm scope):** Add **Cancel** if product expects parity with attachment video title — same handler or kernel menu exit.

4. **No** `rfx_wfm_dbscripts` / `rfx_wfm_i18n` change unless new i18n key required (unlikely — `Previous` already used).

**Repos:** `rfx-rwsdev300` only.  
**Extension point:** N/A — product UI gap, not customization.  
**Upgrade safety:** RETEST / BAT — UI navigation only.

---

## Test gap

| ID | Gap | Recommended test |
|---|---|---|
| TC-NEG-WFM142502-01 | No SIT case asserts Previous on Add Scenario Initialize Forecast footer | Navigate PRE_SCENARIO → Add Scenario → assert Previous visible → click → lands on `scenario.jsp` list |
| TC-NEG-WFM142502-02 | No regression for direct Initialize Forecast menu entry | Open INITIALIZE_WEEK_FORECAST_USING_ML from menu → document expected Previous behaviour (see OQ-1) |
| TC-NEG-WFM142502-03 | BAT locator gap on new button | Add `data-testid` on Previous (e.g. `initialize-forecast-btn-previous`) per ui-test-id rule |

**Flavour:** V45SIT / B0 corporate forecast path.

---

## Open questions

- **[OPEN] OQ-1** (owner: BA, blocks_fix: no): Should Previous appear only when entered via Forecast Scenario → Add, or also for direct **Initialize Forecast** kernel menu entry?

---

## Evidence trail

**KB_ROOT resolved:** `C:\Siddhartha\GIT\wst_skills\.cursor\skills\wst-wfm-domain`

**Step 1:** INCONCLUSIVE — no RNI/config/BR terminal match for missing navigation control.

**Gate:** Step 1 INCONCLUSIVE → Step 2 code walk (UI template inspection; TER not required for static control omission).

**Attachments read:** `jira:WFM_CORP_Forecast_scenario_Add_window_cancel_Previous_button_missing.mp4 (skipped — ENABLE_VIDEO_READER=false)`; `jira:WFM-142502-20260727-2140.md (skipped per user)`

**Skipped layers:** None (all core repos resolved). Optional scheduling/ZTA repos not implicated.

### Provenance

#### Confidence flags

- HIGH confidence from template + navigation code trace; video not reviewed.
- User excluded prior `.md` attachment — no `[WARN: prior analysis not independently re-verified]` for that file.

#### Refinement log

| Version | Date | Trigger | Summary |
|---|---|---|---|
| R1 | 2026-08-07 | `/triage WFM-142502` | Initial triage — GENUINE_BUG, web UI missing Previous on Add Scenario flow |

---

*Generated by Cursor AI · wst-defect-triage v2.2.0*

<!-- /SKILL_SECTION -->

<!-- SKILL_SECTION: wst-defect-fix F1 -->
## FIX (F1)
*Updated: 2026-08-07 12:30*

﻿---
ticket_id: WFM-142502
summary: "v45 WFM CORP (RWS): Forecast scenario Add scenario window missing Previous button"
verdict: GENUINE_BUG
confidence: HIGH
step_reached: KB_STEP2
input_form: jira_api
status: Applied-Fix-Captured
version: R2
fix_primary: web
fault_locus: client_app
channel_confidence: HIGH
channels:
  mobile: false
  web: true
  data: false
  shared_mobile_shell: false
blast_radius: global
next_owner: DEVELOPER
sla: SIT_FLAVOUR
model: composer-2.5
duration_minutes: 12
last_updated: "2026-08-07T12:30:00+05:30"
attachments_read:
  - "jira:WFM_CORP_Forecast_scenario_Add_window_cancel_Previous_button_missing.mp4 (skipped)"
  - "jira:WFM-142502-20260727-2140.md (skipped per user instruction)"
kb_citations:
  - "PRE_SCENARIO kernel page — Forecast Scenario list"
  - "INITIALIZE_WEEK_FORECAST_USING_ML kernel page — Initialize Forecast / Add flow target"
code_citations:
  - "rfx-rwsdev300:WebContent/scripts/predictive/scenario.renderer.js:addScenario:187-193"
  - "rfx-rwsdev300:WebContent/ngtemplates/predictive/initializeForecast/initializeForecastTemplate.html:bottom-container:201-205"
  - "rfx-rwsdev300:WebContent/ngtemplates/predictive/buildMachineLearningModel/buildMachineLearningModelAddEditTemplate.html:Previous button pattern"
  - "rfx-rwsdev300:WebContent/scripts/predictive/initialize.forecast.renderer.js:clearScenario (no backToScenarioList)"
open_questions:
  - id: OQ-1
    state: OPEN
    owner: BA
    blocks_fix: false
    text: "Should Previous (and Cancel per attachment filename) appear only when entered via Forecast Scenario > Add, or also when Initialize Forecast is opened directly from the kernel menu?"
---

# Triage: WFM-142502

**Status:** Applied-Fix-Captured  
**Version:** R2  
**Confidence:** HIGH  
**Triage status:** GENUINE_BUG  
**Last updated:** 2026-08-07  
**Triage duration:** ~12m  
**Model:** composer-2.5  

---

## TL;DR

| | |
|---|---|
| **Bug** | The **Add Scenario** flow (Forecast Scenario → Add, which loads `initializeForecastUsingML.jsp`) shows **Clear / Initialize / Initialize & Save** but **no Previous** button, so users cannot navigate back to the Forecast Scenario list without browser back. |
| **Fix** | Add a **Previous** tertiary button to `initializeForecastTemplate.html` and a `backToScenarioList()` handler in `initialize.forecast.renderer.js` that form-posts back to `scenario.jsp` — mirror peer predictive add/edit screens and the existing `addScenario()` forward navigation. |
| **Action** | Web UI fix in `rfx-rwsdev300`; no server/DB/config change required. Confirm with QA whether **Cancel** is also in scope (attachment name suggests it; ticket text cites Previous only). |

---

## Ticket snapshot

| Field | Value |
|---|---|
| **Summary** | v45 WFM CORP (RWS): Forecast scenario Add scenario window missing Previous button |
| **Status** | To Do |
| **Priority** | Medium |
| **Affects version** | 45.1.22.1 |
| **Fix version** | 45.1.24.0 |
| **Labels** | V45SIT |
| **WFM module** | WFM-Driver/Budget/Labor Forecast · Driver Forecast |
| **Environment** | SIT — `wfmproductqa11.reflexisinc.com` |
| **Reporter** | Poonam Kshirsagar |

**Steps to reproduce:** CORP Admin → Forecast → Initialize Forecast → open Add scenario window → observe navigation buttons.

**Expected:** Previous button present for navigation.  
**Actual:** Previous button missing.

**Original input:** [NOT APPLICABLE]

**Input form:** Jira API (hook) — user instructed to **ignore** attached `.md` triage artefact (`WFM-142502-20260727-2140.md`).

---

## Fix routing (from ticket + code)

```
Ticket fix routing (from Jira text):
  Symptom surface: web (kernel AngularJS — Forecast / Initialize Forecast)
  Fault locus:     client_app (missing template control)
  Evidence tier:   E5 surface-only → confirmed E1-n/a / Step 2 template walk
  Channel conf:    HIGH (pure UI omission; no API payload involved)
  Channels touched: mobile=F  web=T  data=F  shared_shell=F
  Fix primary:     web (template + controller navigation handler)
  Implicated:      1 area — Initialize Forecast Add Scenario form footer
  Sub-issues:      not a composite ticket
  Step 2 plan:     walk SERVER WebContent client-emission (§1b) — confirmed
```

**Ticket hypothesis → Step 2:** confirmed

---

## Probable cause

| Field | Detail |
|---|---|
| **Symptom** | Previous navigation button absent in Add Scenario window on Forecast scenario page. |
| **Fault** | `initializeForecastTemplate.html` footer renders only **Clear**, **Initialize**, and **Initialize & Save** — no **Previous** control. `initialize.forecast.renderer.js` defines `clearScenario()` but **no** `backToScenarioList()` / form submit back to `scenario.jsp`. |
| **Trigger** | User clicks **Add Scenario** on Forecast Scenario list (`scenario.renderer.js` → `addScenario()` posts `scenarioForm` to `initializeForecastUsingML.jsp`). |
| **Consequence** | User is stranded on Initialize Forecast page unless using browser back; inconsistent with other predictive add/edit flows (e.g. Build ML Model) that expose Previous. |
| **Raises to HIGH** | Static template inspection + navigation code path traced; peer-screen pattern confirms omission is unintentional UX gap. |

### Prior analysis critique

No L1/L2/L3 Jira comments on ticket. Attached `.md` intentionally excluded per user request — triage based on Jira JSON + code only.

### Candidate-cause ledger (Step 2 §S2.2)

| Layer | Status | Confidence | Evidence |
|---|---|---|---|
| client_emission (web_js/html) | **implicated** | HIGH | Missing button in `initializeForecastTemplate.html:201-205` |
| server_api | checked-clear | HIGH | Display-only; no REST failure |
| data_config | checked-clear | HIGH | Not data-driven |
| permission | checked-clear | MEDIUM | CORP Admin reproduces — not LOS |
| i18n | checked-clear | HIGH | `Previous` string used on peer templates via `ctrl.i18nFn('Previous')` |
| jsp_shell | checked-clear | HIGH | `initializeForecastUsingML.jsp` includes template correctly |

---

## Probable fix

1. **`initializeForecastTemplate.html`** — In `.bottom-container`, add a **Previous** button before Clear, matching peer pattern:

   [SKETCH] `<button type="button" class="btn btn-tertiary" ng-click="ctrl.backToScenarioList()">{{::ctrl.i18nFn('Previous')}}</button>`

   Reference: `buildMachineLearningModelAddEditTemplate.html` (same predictive module family).

2. **`initialize.forecast.renderer.js`** — Add navigation handler mirroring forward path in `scenario.renderer.js:addScenario`:

   [SKETCH]
   ```javascript
   vm.backToScenarioList = function () {
     document.initializeForecastScenarioSetup.action =
       vm.appName + '/predictive/views/scenario.jsp';
     document.initializeForecastScenarioSetup.submit();
   };
   ```

3. **Optional (confirm scope):** Add **Cancel** if product expects parity with attachment video title — same handler or kernel menu exit.

4. **No** `rfx_wfm_dbscripts` / `rfx_wfm_i18n` change unless new i18n key required (unlikely — `Previous` already used).

**Repos:** `rfx-rwsdev300` only.  
**Extension point:** N/A — product UI gap, not customization.  
**Upgrade safety:** RETEST / BAT — UI navigation only.

---

## Test gap

| ID | Gap | Recommended test |
|---|---|---|
| TC-NEG-WFM142502-01 | No SIT case asserts Previous on Add Scenario Initialize Forecast footer | Navigate PRE_SCENARIO → Add Scenario → assert Previous visible → click → lands on `scenario.jsp` list |
| TC-NEG-WFM142502-02 | No regression for direct Initialize Forecast menu entry | Open INITIALIZE_WEEK_FORECAST_USING_ML from menu → document expected Previous behaviour (see OQ-1) |
| TC-NEG-WFM142502-03 | BAT locator gap on new button | Add `data-testid` on Previous (e.g. `initialize-forecast-btn-previous`) per ui-test-id rule |

**Flavour:** V45SIT / B0 corporate forecast path.

---

## Open questions

- **[OPEN] OQ-1** (owner: BA, blocks_fix: no): Should Previous appear only when entered via Forecast Scenario → Add, or also for direct **Initialize Forecast** kernel menu entry?

---

## Evidence trail

**KB_ROOT resolved:** `C:\Siddhartha\GIT\wst_skills\.cursor\skills\wst-wfm-domain`

**Step 1:** INCONCLUSIVE — no RNI/config/BR terminal match for missing navigation control.

**Gate:** Step 1 INCONCLUSIVE → Step 2 code walk (UI template inspection; TER not required for static control omission).

**TER evidence:** not triggered

**Attachments read:** `jira:WFM_CORP_Forecast_scenario_Add_window_cancel_Previous_button_missing.mp4 (skipped — ENABLE_VIDEO_READER=false)`; `jira:WFM-142502-20260727-2140.md (skipped per user)`

**Skipped layers:** None (all core repos resolved). Optional scheduling/ZTA repos not implicated.

### Provenance

#### Confidence flags

- HIGH confidence from template + navigation code trace; video not reviewed.
- User excluded prior `.md` attachment — no `[WARN: prior analysis not independently re-verified]` for that file.

#### Refinement log

| Version | Date | Trigger | Summary |
|---|---|---|---|
| R1 | 2026-08-07 | `/triage WFM-142502` | Initial triage — GENUINE_BUG, web UI missing Previous on Add Scenario flow |
| R2 | 2026-08-07 | `/capture-fix (wst-defect-fix)` | Fix stage F1 added — 2 files in rfx-rwsdev300 |

---

## Fix stage

**Fix status:**     Applied Fix Captured  
**Fix revision:**   F1  
**Captured at:**    2026-08-07T12:30:00+05:30  
**Last updated:**   2026-08-07T12:30:00+05:30  
**Diff scope:**     Working tree (staged + unstaged + untracked) vs HEAD  
**Repos scanned:**  rfx-rwsdev300 (2 files), rfx_wfm_dbscripts (0 files), rfx_wfm_i18n (0 files), sws_zta (0 — ZTA_ROOT not implicated), rflx-wfm-shiftbuilding (0 — not implicated), rflx-wfm-frameworkscheduling (0 — not implicated), rflx-wfm-scheduling (0 — not implicated)  
**Generated by:**   wst-defect-fix skill  
**Skill version:**  2.0  

### Fix log

| Rev | Timestamp | Triggered by | Files touched | Summary of change |
|-----|-----------|--------------|---------------|-------------------|
| F1  | 2026-08-07T12:30:00+05:30 | /capture-fix | 2 | Added Previous button and back-navigation from Initialize Forecast Add Scenario screen to Forecast Scenario list |

### Root cause

The Initialize Forecast screen used for adding a forecast scenario did not include a Previous control, so users who opened it from Forecast Scenario could not return to the scenario list except via the browser back button.

### Analysis

When a corporate administrator navigated to Forecast Scenario and chose Add Scenario, the application opened the Initialize Forecast entry form showing Clear, Initialize, and Initialize & Save, but no Previous button. Other comparable predictive forecast screens already expose Previous for returning to the prior list view, so this flow was inconsistent and left users without an in-app way back to the scenario list.

### Fix

- The Initialize Forecast footer now includes a **Previous** button aligned with other predictive forecast screens.
- Selecting **Previous** returns the user to the **Forecast Scenario** list without using the browser back control.
- Existing **Clear**, **Initialize**, and **Initialize & Save** actions are unchanged.

### Area impacted

- `[DIRECT] Forecast Scenario — Add Scenario / Initialize Forecast entry (V45SIT corporate): triggered when CORP admin clicks Add Scenario from the Forecast Scenario list`
- `[REGRESSION] Initialize Forecast — direct kernel menu entry (all flavours): triggered when user opens Initialize Forecast from the menu without entering via Forecast Scenario`
- `[REGRESSION] Initialize Forecast — Clear / Initialize / Initialize & Save actions (all flavours): triggered when user completes or clears the form after the Previous control was added`

### Regression scope

| # | Persona | Action | Condition | Expected outcome | Risk | Flavour |
|---|---------|--------|-----------|------------------|------|---------|
| RS-1 | CORP Admin | Open Forecast Scenario → Add Scenario → observe footer buttons | Standard SIT forecast scenario permissions | Previous button is absent (pre-fix defect state) | HIGH | V45SIT |
| RS-2 | CORP Admin | Open Forecast Scenario → Add Scenario → click Previous | User arrived via Add Scenario from scenario list | Forecast Scenario list screen displays; no browser back required | HIGH | V45SIT |
| RS-3 | CORP Admin | Open Initialize Forecast directly from kernel menu → click Previous | User did not enter via Forecast Scenario Add | User returns to Forecast Scenario list (confirm product intent per OQ-1) | MEDIUM | all |
| RS-4 | CORP Admin | On Initialize Forecast form → Clear then Initialize & Save | Form populated with valid driver, model, location, dates, scenario name | Clear resets fields; Initialize & Save still completes without navigation regression | MEDIUM | V45SIT |

### Technical notes

- **Code-level root cause:** `initializeForecastTemplate.html` bottom-container lacked a Previous control; `initialize.forecast.renderer.js` had no back-navigation handler despite forward navigation via `scenario.renderer.js:addScenario()` posting to `initializeForecastUsingML.jsp`.
- **Fix approach:** Mirror peer predictive screens — tertiary Previous button + form POST back to `scenario.jsp` (inverse of `addScenario()`).
- **Diff matches triage Pattern B:** confirmed — files match Probable fix items [1] and [2].
- **Dropped alternative:** Cancel button (Probable fix item [3]) not implemented — ticket cited Previous only.
- **Note:** Working tree in `eclipse_gradle_workspace` clone also shows modified `gradle.properties` (local JDK paths) — **excluded** from this fix; not part of WFM-142502.

**Changed files (by repo):**

- **rfx-rwsdev300**
  - `WebContent/ngtemplates/predictive/initializeForecast/initializeForecastTemplate.html` (+2 lines) — Previous button with `data-testid="initialize-forecast-btn-previous"` before Clear
  - `WebContent/scripts/predictive/initialize.forecast.renderer.js` (+5 lines) — `backToScenarioList()` submits `initializeForecastScenarioSetup` to `scenario.jsp`

### Test cases executed

[NOT PROVIDED] — fix captured from working tree; SIT validation pending.

---

## Clarifications

<!-- Append-only. Each entry records substantial follow-up context that changed
     one or more sections. Q&A answers that fit on one line go into Open questions
     above; lengthy context or multi-section impacts go here. Oldest first. -->

<!-- REFINE: paste new context, Q&A answers, or follow-up findings below this line.
     Then attach this file in a chat with the wst-defect-triage skill to trigger refinement. -->

<!-- FIX: paste fix context, test outcomes, or follow-up diff notes below this line.
     Then attach this file in a chat with the wst-defect-fix skill to trigger /update-fix. -->

---

*Generated by Cursor AI · wst-defect-triage v2.2.0 + wst-defect-fix F1*

<!-- /SKILL_SECTION -->
