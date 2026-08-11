# WFM-140174 — Handoff Document

*Multi-skill handoff document. Each section below represents a different skill output.*

<!-- SKILL_SECTION: wst-defect-triage R1 -->
## TRIAGE (R1)
*Updated: 2026-08-11 12:47*

---
ticket_id: WFM-140174
summary: "Forecast Review department filter shows empty/wrong grid when selected departments lack PRE_METRIC rows"
verdict: GENUINE_BUG
confidence: MEDIUM
step_reached: KB_STEP2
input_form: jira_json
fix_primary: web
fault_locus: server_api
channel_confidence: MEDIUM
channels:
  mobile: false
  web: true
  data: false
implicated_areas:
  - id: forecast-review-dept-filter
    surface: "Predictive Forecast → Forecast Review → Filter Settings → Departments"
    layer: web_ui
    fix_owner: web
    confidence: HIGH
  - id: forecast-review-skeleton-fallback
    surface: "DefaultDriverForecastReviewService.getReviewForecastData / getReviewForecastDataSkeleton"
    layer: server_api
    fix_owner: web
    confidence: HIGH
attachments_read:
  - "jira:Department filter - forecast review screen.png (read)"
  - "jira:department filter incorrect data being fetched.png (read)"
status: Draft
version: R1
last_updated: "2026-08-11T07:15:00Z"
duration_minutes: 12
model: composer-2.5
---

# Triage: WFM-140174

**Summary:** WFM(PRE) - The "Department" option in the filter on the Forecast Review screen is not functioning as expected.

**Status:** In Progress · **Priority:** Medium · **Fix version:** 45.1.23.0  
**Affects:** 45.1.20.0 · **Module:** Driver Forecast / WFM-Driver/Budget/Labor Forecast  
**Environment:** QA06 (`wfmproductqa06`) · Build `WFM.45.1.20.0.20251130.U233012`  
**Linked:** WFM-109065 (same summary — prior/parallel defect)  
**Assignee:** Vishalkumar Shukla

---

## TL;DR

| | |
|---|---|
| **Bug** | Applying a Department filter on Forecast Review that targets departments with no matching `PRE_METRIC` rows triggers an empty-data skeleton response that **ignores the selected departments**, rendering all driver-mapped department rows with blank values — looks like a broken filter. |
| **Fix** | In `DefaultDriverForecastReviewService`, when `departmentIds` is non-empty and filtered `metricData` is empty, do **not** call unfiltered `getReviewForecastDataSkeleton`; either return a department-scoped empty result or intersect skeleton rows with `filterdeptSkeys`. Also apply `filterdeptSkeys` inside skeleton row building. |
| **Action** | Dev to patch server path; QA to retest Dept1/Dept3 vs Dept2/Dept4 on QA06 week 20251130–20251206 and confirm HAR `departmentIds` payload + `PRE_METRIC` row counts per dept. |

---

## Probable cause

| Field | Detail |
|---|---|
| **Symptom** | User selects Department1 (+ Department3) in Forecast Review filter → grid shows driver/department row labels but **empty values**; filter appears ineffective. Selecting Department2 / Department4 shows populated values (per reporter note). |
| **Fault** | Server-side skeleton fallback path in `DefaultDriverForecastReviewService.getReviewForecastData` (lines 887–889): when SQL returns zero rows after department-scoped fetch, code calls `getReviewForecastDataSkeleton`, which rebuilds rows from **all** driver-mapped departments via `getDepartmentsList` **without** intersecting `forecastReviewLookupFilter.getDepartmentIds()` / `filterdeptSkeys`. |
| **Trigger** | Department filter narrows `PRE_METRIC` SQL (`DEPT_SKEY IN (...)`) to dept skeys that have no forecast rows for the selected week/drivers at CORP level; empty result hits skeleton branch. |
| **Consequence** | UI displays misleading empty structure (all mapped depts, null values) instead of filtered data or a clear empty-state — reported as "incorrect data". |
| **What raises to HIGH** | HAR capture showing `departmentIds` well-formed in POST body **and** DB query confirming zero `PRE_METRIC` rows for Dept1/3 but non-zero for Dept2/4 for same week — proves whether data gap is co-factor or sole cause. |

---

## Probable fix

1. **`DefaultDriverForecastReviewService.getReviewForecastData`** (~887–889): When `!isEmpty(filterdeptSkeys)` and `metricData` is empty, skip unfiltered skeleton; return department-scoped empty grid or explicit no-data response.
2. **`getReviewForecastDataSkeleton`** (~425–600): Resolve `filterdeptSkeys` from `forecastReviewLookupFilter.getDepartmentIds()` and intersect with `metricDeptSkeys` before emitting rows (mirror main path lines 757–796).
3. **Optional UX**: Show i18n message when filter matches zero rows (instead of silent null cells).
4. **Parallel data check**: Verify QA06 `PRE_METRIC` + driver-dept mappings for Department1/3 — if rows genuinely missing, initialize forecast; code fix still required for filter UX.

---

## Test gap

| Gap | Recommendation |
|---|---|
| No BAT/API test for department-filter + empty-data skeleton path | Add integration test: filter to dept with zero `PRE_METRIC` rows → response must contain **only** filtered dept rows (or explicit empty), never all mapped depts |
| No UI automation for Forecast Review department multiselect apply | Add Robot/BAT: apply Dept filter → assert row count matches selection; compare Network tab `departmentIds` |
| Linked WFM-109065 untested regression | Close duplicate; add TC-NEG-FR-DEPT-01 to flavour B0 predictive suite |

---

## Fix routing (from ticket + code)

| Field | Value |
|---|---|
| Symptom surface | web (Forecast Review kernel / AngularJS) |
| Fault locus | **server_api** (confirmed Step 2) |
| Evidence tier | E5 surface-only (ticket) → Step 2 code HIGH |
| Channels | mobile=F web=T data=partial (missing PRE_METRIC rows may co-exist) |
| Fix primary | **web** |
| Step 2 reconciliation | **confirmed** — skeleton fallback ignores department filter |

---

## Open questions

| ID | State | Owner | Blocks fix? | Question |
|---|---|---|---|---|
| Q1 | OPEN | QA | no | Paste HAR/network body for `POST .../forecastreview/basic.json` after Apply with Dept1+Dept3 — confirm `departmentIds` array values |
| Q2 | OPEN | DBA/QA | no | Run PRE_METRIC count by DEPT_SKEY for Driver21 week 20251130–20251206 at CORP — are Dept1/3 genuinely empty vs Dept2/4? |
| Q3 | OPEN | Dev | no | Is WFM-109065 the same root cause? If yes, consolidate fix under one PR |

---

## Prior analysis

| Source | Assessment |
|---|---|
| WFM-109065 (linked Mention) | Same summary — likely duplicate; no dev comments on either ticket |
| Reporter note (Dept2/4 only work) | Consistent with skeleton bypass when filtered SQL returns data for those depts only |

---

## Evidence trail

**KB citations:** PF-S10 Forecast Review stage; DITL-SM-03 Forecast Review flow  
**Code citations:**
- `DefaultDriverForecastReviewService.java:getReviewForecastData` — lines 757–758, 795–796, 887–889 (filter → empty → skeleton)
- `DefaultDriverForecastReviewService.java:getReviewForecastDataSkeleton` — lines 577–585 (all depts, no filter intersection)
- `DefaultDriverForecastDataService.java:getMetricDataQueryParameters` — lines 270–298 (departmentIds → deptSkeys SQL filter)
- `forecast.review.actions.js:getForecastData` — line 137 (posts full selectedFilter incl. departmentIds)
- `forecastReviewFilter.html` — lines 116–121 (department multiselect binding)

**Attachments read:** jira:Department filter - forecast review screen.png (read); jira:department filter incorrect data being fetched.png (read)

**Skipped layers:** ZTA, ShiftBuilding, FrameworkScheduling, Scheduling — not implicated

---

## Provenance

### Confidence flags

- Step 2 included WebContent client-emission layer — `departmentIds` bound and posted via `selectedFilter`; no client payload truncation found
- Skeleton fallback path identified as primary code defect; DATA co-factor (missing PRE_METRIC for Dept1/3) unconfirmed without DB query (Q2)
- MEDIUM confidence pending HAR + DB evidence (Q1, Q2)

### Refinement log

| Version | Date | Trigger | Change |
|---|---|---|---|
| R1 | 2026-08-11 | /triage WFM-140174 | Initial KB + Step 2 triage |

---

**Triage duration:** ~12m · **Model:** composer-2.5

**Original input:** [NOT APPLICABLE — Jira API intake]

<!-- /SKILL_SECTION -->
