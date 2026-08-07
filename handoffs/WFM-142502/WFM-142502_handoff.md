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
  - "PRE_SCENARIO kernel page â€” Forecast Scenario list"
  - "INITIALIZE_WEEK_FORECAST_USING_ML kernel page â€” Initialize Forecast / Add flow target"
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
| **Bug** | The **Add Scenario** flow (Forecast Scenario â†’ Add, which loads `initializeForecastUsingML.jsp`) shows **Clear / Initialize / Initialize & Save** but **no Previous** button, so users cannot navigate back to the Forecast Scenario list without browser back. |
| **Fix** | Add a **Previous** tertiary button to `initializeForecastTemplate.html` and a `backToScenarioList()` handler in `initialize.forecast.renderer.js` that form-posts back to `scenario.jsp` â€” mirror peer predictive add/edit screens and the existing `addScenario()` forward navigation. |
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
| **WFM module** | WFM-Driver/Budget/Labor Forecast Â· Driver Forecast |
| **Environment** | SIT â€” `wfmproductqa11.reflexisinc.com` |
| **Reporter** | Poonam Kshirsagar |

**Steps to reproduce:** CORP Admin â†’ Forecast â†’ Initialize Forecast â†’ open Add scenario window â†’ observe navigation buttons.

**Expected:** Previous button present for navigation.  
**Actual:** Previous button missing.

**Original input:** [NOT APPLICABLE]

**Input form:** Jira API (hook) â€” user instructed to **ignore** attached `.md` triage artefact (`WFM-142502-20260727-2140.md`).

---

## Fix routing (from ticket + code)

```
Ticket fix routing (from Jira text):
  Symptom surface: web (kernel AngularJS â€” Forecast / Initialize Forecast)
  Fault locus:     client_app (missing template control)
  Evidence tier:   E5 surface-only â†’ confirmed E1-n/a / Step 2 template walk
  Channel conf:    HIGH (pure UI omission; no API payload involved)
  Channels touched: mobile=F  web=T  data=F  shared_shell=F
  Fix primary:     web (template + controller navigation handler)
  Implicated:      1 area â€” Initialize Forecast Add Scenario form footer
  Sub-issues:      not a composite ticket
  Step 2 plan:     walk SERVER WebContent client-emission (Â§1b) â€” confirmed
```

**Ticket hypothesis â†’ Step 2:** confirmed

---

## Probable cause

| Field | Detail |
|---|---|
| **Symptom** | Previous navigation button absent in Add Scenario window on Forecast scenario page. |
| **Fault** | `initializeForecastTemplate.html` footer renders only **Clear**, **Initialize**, and **Initialize & Save** â€” no **Previous** control. `initialize.forecast.renderer.js` defines `clearScenario()` but **no** `backToScenarioList()` / form submit back to `scenario.jsp`. |
| **Trigger** | User clicks **Add Scenario** on Forecast Scenario list (`scenario.renderer.js` â†’ `addScenario()` posts `scenarioForm` to `initializeForecastUsingML.jsp`). |
| **Consequence** | User is stranded on Initialize Forecast page unless using browser back; inconsistent with other predictive add/edit flows (e.g. Build ML Model) that expose Previous. |
| **Raises to HIGH** | Static template inspection + navigation code path traced; peer-screen pattern confirms omission is unintentional UX gap. |

### Prior analysis critique

No L1/L2/L3 Jira comments on ticket. Attached `.md` intentionally excluded per user request â€” triage based on Jira JSON + code only.

### Candidate-cause ledger (Step 2 Â§S2.2)

| Layer | Status | Confidence | Evidence |
|---|---|---|---|
| client_emission (web_js/html) | **implicated** | HIGH | Missing button in `initializeForecastTemplate.html:201-205` |
| server_api | checked-clear | HIGH | Display-only; no REST failure |
| data_config | checked-clear | HIGH | Not data-driven |
| permission | checked-clear | MEDIUM | CORP Admin reproduces â€” not LOS |
| i18n | checked-clear | HIGH | `Previous` string used on peer templates via `ctrl.i18nFn('Previous')` |
| jsp_shell | checked-clear | HIGH | `initializeForecastUsingML.jsp` includes template correctly |

---

## Probable fix

1. **`initializeForecastTemplate.html`** â€” In `.bottom-container`, add a **Previous** button before Clear, matching peer pattern:

   [SKETCH] `<button type="button" class="btn btn-tertiary" ng-click="ctrl.backToScenarioList()">{{::ctrl.i18nFn('Previous')}}</button>`

   Reference: `buildMachineLearningModelAddEditTemplate.html` (same predictive module family).

2. **`initialize.forecast.renderer.js`** â€” Add navigation handler mirroring forward path in `scenario.renderer.js:addScenario`:

   [SKETCH]
   ```javascript
   vm.backToScenarioList = function () {
     document.initializeForecastScenarioSetup.action =
       vm.appName + '/predictive/views/scenario.jsp';
     document.initializeForecastScenarioSetup.submit();
   };
   ```

3. **Optional (confirm scope):** Add **Cancel** if product expects parity with attachment video title â€” same handler or kernel menu exit.

4. **No** `rfx_wfm_dbscripts` / `rfx_wfm_i18n` change unless new i18n key required (unlikely â€” `Previous` already used).

**Repos:** `rfx-rwsdev300` only.  
**Extension point:** N/A â€” product UI gap, not customization.  
**Upgrade safety:** RETEST / BAT â€” UI navigation only.

---

## Test gap

| ID | Gap | Recommended test |
|---|---|---|
| TC-NEG-WFM142502-01 | No SIT case asserts Previous on Add Scenario Initialize Forecast footer | Navigate PRE_SCENARIO â†’ Add Scenario â†’ assert Previous visible â†’ click â†’ lands on `scenario.jsp` list |
| TC-NEG-WFM142502-02 | No regression for direct Initialize Forecast menu entry | Open INITIALIZE_WEEK_FORECAST_USING_ML from menu â†’ document expected Previous behaviour (see OQ-1) |
| TC-NEG-WFM142502-03 | BAT locator gap on new button | Add `data-testid` on Previous (e.g. `initialize-forecast-btn-previous`) per ui-test-id rule |

**Flavour:** V45SIT / B0 corporate forecast path.

---

## Open questions

- **[OPEN] OQ-1** (owner: BA, blocks_fix: no): Should Previous appear only when entered via Forecast Scenario â†’ Add, or also for direct **Initialize Forecast** kernel menu entry?

---

## Evidence trail

**KB_ROOT resolved:** `C:\Siddhartha\GIT\wst_skills\.cursor\skills\wst-wfm-domain`

**Step 1:** INCONCLUSIVE â€” no RNI/config/BR terminal match for missing navigation control.

**Gate:** Step 1 INCONCLUSIVE â†’ Step 2 code walk (UI template inspection; TER not required for static control omission).

**Attachments read:** `jira:WFM_CORP_Forecast_scenario_Add_window_cancel_Previous_button_missing.mp4 (skipped â€” ENABLE_VIDEO_READER=false)`; `jira:WFM-142502-20260727-2140.md (skipped per user)`

**Skipped layers:** None (all core repos resolved). Optional scheduling/ZTA repos not implicated.

### Provenance

#### Confidence flags

- HIGH confidence from template + navigation code trace; video not reviewed.
- User excluded prior `.md` attachment â€” no `[WARN: prior analysis not independently re-verified]` for that file.

#### Refinement log

| Version | Date | Trigger | Summary |
|---|---|---|---|
| R1 | 2026-08-07 | `/triage WFM-142502` | Initial triage â€” GENUINE_BUG, web UI missing Previous on Add Scenario flow |

---

*Generated by Cursor AI Â· wst-defect-triage v2.2.0*

<!-- /SKILL_SECTION -->
