# WFM-140174 — Handoff Document

*Multi-skill handoff document. Each section below represents a different skill output.*

<!-- SKILL_SECTION: wst-defect-triage R1 -->
## TRIAGE (R1)
*Updated: 2026-08-06 14:35*

﻿---
id: WFM-140174
summary: "WFM(PRE) - Forecast Review Department filter shows wrong data for some departments"
verdict: GENUINE_BUG
confidence: MEDIUM
step_reached: KB_STEP2
input_form: jira_json
module: "WFM-Driver/Budget/Labor Forecast"
customer: null
flavour: null
affects_version: "WFM.45.1.20.0.20251130.U233012"
fix_version: null
priority: Medium
blast_radius: unknown
next_owner: L3-Engineering
sla: P2-2d
status: Draft
version: R1
generated: "2026-08-06T14:35:00+05:30"
last_updated: "2026-08-06T14:35:00+05:30"
duration_minutes: 12
model: composer-2.5
skill_version: "2.2.0"
attachments_read:
  - { source: jira, filename: "Department filter - forecast review screen.png", type: skipped }
  - { source: jira, filename: "department filter incorrect data being fetched.png", type: skipped }
fix_primary: web
channels_mobile: false
channels_web: true
channels_data: true
implicated_areas_count: 2
fault_locus: server_api
channel_confidence: MEDIUM
---

# Triage: WFM-140174 â€” Forecast Review Department filter not functioning as expected

## TL;DR

**Bug:** On Predictive Forecast â†’ Forecast Review, applying the Department filter for Dept1 + Dept3 shows incorrect forecast values; Dept2 + Dept4 selections display correctly. Same symptom as linked duplicate WFM-109065 (still In Progress).
**Fix:** Trace `departmentIds` from filter Apply through `ForecastReviewController` â†’ `DefaultDriverForecastReviewService` and `DefaultDriverForecastDataService.getMetricDataQueryParameters`; verify dept IDâ†’`DEPT_SKEY` resolution and SQL `DEPT_SKEY` filtering; rule out driverâ€“department mapping / metric data keyed to wrong `DEPT_SKEY` on QA06.
**Action:** L3-Engineering â€” P2-2d â€” Reproduce on wfmproductqa06 with HAR capture for `/controller/predictive/forecastreview/basic.json`; compare payload and response for Dept1+3 vs Dept2+4; fix server filter/rollup path or escalate data correction if metric rows are on wrong `DEPT_SKEY`.

## Ticket snapshot

- **Module:**             WFM-Driver/Budget/Labor Forecast (Driver Forecast)
- **Customer / flavour:** [NOT PROVIDED] (System Test / PI-4.25)
- **Affects version:**    WFM.45.1.20.0.20251130.U233012
- **Fix version:**        [NOT PROVIDED]
- **Priority:**           Medium
- **Status:**             In Progress
- **Reporter:**           Akshay Sopan Pate
- **Assignee:**           Vishalkumar Shukla
- **Environment:**        wfmproductqa06 (TESTING7.view)
- **Linked issues:**      WFM-109065 (Mention â€” identical summary, In Progress)
- **Parent epic:**        WFM-142304
- **Original input:**     [NOT APPLICABLE]

## Probable cause

| Field | Detail |
|---|---|
| **Symptom** | Department multiselect filter on Forecast Review does not constrain displayed forecast data correctly for Dept1 and Dept3; Dept2 and Dept4 work. |
| **Fault line** | Server forecast-review pipeline: department filter applied in `DefaultDriverForecastReviewService` (driver list + row build) and `DefaultDriverForecastDataService.getMetricDataQueryParameters` (metric SQL `DEPT_SKEY IN â€¦`). Client posts `selectedFilter.departmentIds` (dept IDs from multiselect `id-prop="deptId"`). |
| **Trigger** | Corp user opens Forecast Review, opens filter, selects specific department combinations and clicks Apply. |
| **Consequence** | Managers see wrong driver forecast values when filtering by certain departments â€” planning and publish decisions may use incorrect demand. |
| **Raises to HIGH** | HAR showing `departmentIds` in POST body + SQL/response proving wrong `DEPT_SKEY` rows returned, or confirming Dept1/3 metric data stored under incorrect `DEPT_SKEY` on QA06. |

**Structured micro-block**

- **Layer:** `server_api` (primary) with possible `data_config` (driverâ€“dept mapping / `PRE_METRIC` `DEPT_SKEY` rows on QA06).
- **Confidence:** MEDIUM â€” code path identified; selective per-department failure pattern and duplicate WFM-109065 suggest environment data or mapping issue not fully confirmed without network/DB evidence.
- **Code citations:**
  - `WebContent/ngtemplates/predictive/forecastReview/forecastReviewFilter.html` â€” department multiselect binds `fRC.selectedFilter.departmentIds` via `deptId`.
  - `WebContent/scripts/predictive/forecast.review.actions.js:getForecastData` â€” POST `selectedFilter` to `/controller/predictive/forecastreview/basic.json`.
  - `DefaultDriverForecastReviewService.java` (~757â€“796, ~913â€“929) â€” `getDepartmentsSkeyListByIds`, driver list filtered by mapped dept skeys; row build iterates `driverService.getDepartmentsList` per unit.
  - `DefaultDriverForecastDataService.java:getMetricDataQueryParameters` (~297) â€” resolves `departmentSkeys` via `omService.getDepartments(..., corporateUnitId, ..., departmentIds)`.
  - `DriverForecastReviewDAO.xml` â€” `DEPT_SKEY IN` clause on metric queries.
- **Suspect code smell:** `DefaultDriverForecastReviewService` line ~803 â€” `getMetricIds().stream().filter(...)` is a no-op (result not collected); may leave stale driver IDs when both drivers and departments are filtered.

## Probable fix

1. **Reproduce with evidence:** On QA06, capture Network tab for Apply with Dept1+Dept3 vs Dept2+Dept4 â€” confirm `departmentIds` array in POST and which drivers/values return.
2. **If payload correct but response wrong:** Debug `getDepartmentsSkeyListByIds` vs `getDepartments` corporate-unit resolution for Dept1/3 IDs; ensure `filterdeptSkeys` and SQL `departmentSkeys` align; fix line ~803 to intersect `metricIds` with `baseMetricIds` when both filters active.
3. **If DB shows metric rows on wrong DEPT_SKEY for Dept1/3:** Treat as DATA_ISSUE â€” correct driverâ€“department mapping / re-initialize forecast for affected drivers; add regression TC.
4. **Dedupe:** Coordinate with WFM-109065 â€” same repro; close one as duplicate after fix verified.

## Test gap

| Gap | Recommended test |
|---|---|
| No BAT for Forecast Review department filter combinations | **TC-NEG-PF-FR-DEPT-01:** Multi-select Dept A+B â†’ grid shows only mapped drivers and values for those dept skeys |
| No cross-dept parity test | **TC-NEG-PF-FR-DEPT-02:** Repeat for each department in pilot store set; assert values match unfiltered drill-down for same dept |
| Duplicate long-standing defect | Link regression to WFM-109065 acceptance criteria |

## Fix routing (from ticket + code)

| Channel | Flag | Notes |
|---|---|---|
| Web | true | Kernel AngularJS Forecast Review screen |
| Mobile | false | Not reported |
| Data | true | Possible wrong `DEPT_SKEY` on metric rows or driverâ€“dept mapping |

**Implicated areas**

| ID | Surface | Layer | Fix owner |
|---|---|---|---|
| pf-fr-dept-filter | Forecast Review filter Apply | web_js + server_api | web (server primary) |
| pf-fr-dept-data | Driver metric data by department | data_config | data (if mapping/rows wrong) |

**Ticket hypothesis:** `partially_confirmed` â€” web surface + server pipeline implicated; per-dept selective failure may be data/mapping on QA06.

## Open questions

| ID | State | Owner | Blocks fix? | Text |
|---|---|---|---|---|
| OQ-1 | OPEN | Reporter/QA | yes | Exact `DEPT_ID` values for Dept1â€“Dept4 on QA06 and their `DEPT_SKEY` mappings |
| OQ-2 | OPEN | QA | yes | HAR or paste of POST `/controller/predictive/forecastreview/basic.json` body + response for Dept1+3 vs Dept2+4 |
| OQ-3 | OPEN | PM | no | Relationship to WFM-109065 â€” should WFM-140174 be closed as duplicate? |

## Prior analysis critique

No L1/L2/L3 comments on ticket. Linked WFM-109065 has identical description and same build â€” indicates recurring/unfixed defect, not a new regression class.

## Evidence trail

**KB citations:** `predictive_forecast.json` PF-S10 Forecast Review; `permissions_matrix.json` Forecast Review module; `test-scenarios.md` DITL-SM-03 Forecast Review flow.

**Code citations:** See Probable cause block.

**Attachments read:** jira:Department filter - forecast review screen.png (skipped â€” not downloaded locally); jira:department filter incorrect data being fetched.png (skipped).

**Skipped layers:** ZTA_ROOT, SHIFTBUILDING_ROOT, FRAMEWORKSCHEDULING_ROOT, SCHEDULING_ROOT not in workspace (optional; not implicated).

## Provenance

### Confidence flags

- MEDIUM: selective department pattern not validated with network/DB on QA06.
- Duplicate WFM-109065 increases confidence this is a real unfixed product/data issue, not reporter mistake.
- Attachments listed in Jira but not present in local intake folder.

### Refinement log

| Version | Date | Trigger | Confidence | Summary |
|---|---|---|---|---|
| R1 | 2026-08-06 | `/triage WFM-140174` | MEDIUM | Initial triage â€” GENUINE_BUG hybrid server/data; linked duplicate WFM-109065 |

## Clarifications

_None._

<!-- /SKILL_SECTION -->
