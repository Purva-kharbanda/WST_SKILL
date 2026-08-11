# WFM-141456 — Handoff Document

*Multi-skill handoff document. Each section below represents a different skill output.*

<!-- SKILL_SECTION: wst-defect-triage R1 -->
## TRIAGE (R1)
*Updated: 2026-08-11 23:32*

---
id: WFM-141456
summary: "ESS Alternate Work Location page — console 422 errors on add/edit, missing CSS asset 404, ui-bootstrap TypeError"
verdict: GENUINE_BUG
confidence: MEDIUM
step_reached: KB_STEP2
input_form: jira_api_hook
module: Employee Self-Service
customer: REFLEXIS
flavour: null
affects_version: "45.1.22.1, 45.1.23.0"
fix_version: "45.1.23.0"
priority: Medium
blast_radius: global
next_owner: L3-Engineering
sla: P3-next-sprint
status: Draft
version: R1
generated: "2026-08-11T23:30:00+05:30"
last_updated: "2026-08-11T23:30:00+05:30"
duration_minutes: 14
model: composer-2.5
skill_version: "2.2.0"
attachments_read:
  - { source: jira, filename: "image-2026-07-21-16-35-14-401.png", type: skipped }
  - { source: jira, filename: "screenshot-1.png", type: skipped }
fix_primary: web
channels_mobile: false
channels_web: true
channels_data: false
implicated_areas_count: 4
---

# Triage: WFM-141456 — ESS Alternate Work Location: security/console warnings on add/edit/delete

## TL;DR

**Bug:** On ESS **Alternate Work Locations** (`shareRequests.jsp`), add/edit POST calls return **HTTP 422** in the browser console; a unit-picker CSS background 404s; INTQA02 also throws **`TypeError: m.$isEmpty is not a function`** after update.
**Fix:** Align **CROSS_UNIT_DOMAIN `@AuthParam` bindings** on `updateShareRequestDetail` / `addShareRequestDetail` with `addShareRequest` (include `EFFECTIVE_DATE_SKEY`, `END_DATE_SKEY`, and `IS_FROM_ESS` where applicable); fix `isteven-multi-select.css` image path; investigate ui-bootstrap/ngModel compatibility on roster redirect path.
**Action:** L3-Engineering — P3-next-sprint — reproduce on wfmproductqa06, capture 422 response body, then patch LOS AuthParam + CSS.

## Ticket snapshot

- **Module:**             Employee Self-Service
- **Customer / flavour:** REFLEXIS (RTM / Prod QA — label `45ProdQA_Security`)
- **Affects version:**    45.1.22.1, 45.1.23.0
- **Priority:**           Medium
- **Stage:**              To Do
- **Original input:**     [NOT APPLICABLE]

## Fix routing (from ticket + code)

**Primary fix location:** web — ESS web Alternate Work Location screen and its `/controller/ess/sharerequests/*` REST endpoints (server-side LOS + client CSS/JS).

**Channels (fix may be required):**

| Channel | Required? | Evidence |
|---------|-----------|----------|
| Mobile (Shift App) | no | Ticket repro is web ESS `shareRequests.jsp`; WFM module = Employee Self-Service |
| Web / server | yes | Console 422 on `POST .../sharerequests/add/request.json` and `.../update/requestDetails/{personId}/{requestNo}.json` |
| Data / master data | verify | CROSS_UNIT resolver validates share criteria eligibility — config could contribute but code AuthParam gap is primary |
| Shared mobile shell | n/a | — |

**Implicated areas:**

| Area | Layer | Fix owner | Confidence | Notes |
|------|-------|-----------|------------|-------|
| ESS Alternate Work Locations page | web_ui | web | HIGH | `shareRequests.jsp` + Angular templates |
| ESS share-request REST API | server_api | web | HIGH | `ESSController` sharerequests mappings |
| CROSS_UNIT LOS resolver | los_auth | web | MEDIUM | Missing date keys on detail update AuthParam |
| isteven-multi-select CSS | static_asset | web | HIGH | Wrong `innercontainer-expand.png` path → 404 |

**Routing reconciliation:** confirmed — ticket text alone suggested "security warnings"; Step 2 ties 422 to `DefaultRestController` 422 handling + `CrossUnitDomainResolver` share-request overloads; CSS/JS issues are separate genuine defects.

## Prior analysis critique

No prior L1/L2/L3 analysis in Jira comments. Linked dev story WFM-143280 mirrors summary but has no technical root-cause yet.

---

## Probable cause

- **Symptom:** Associate on ESS Alternate Work Locations sees browser console errors when adding/editing/deleting requests; on productqa06 POST calls fail with **422 (Undefined)**; INTQA02 shows success banner but page goes blank with **`m.$isEmpty is not a function`**.
- **Fault:** `(primary)` `ESSController.updateShareRequestDetail` — `@AuthParam` omits `EFFECTIVE_DATE_SKEY` / `END_DATE_SKEY` (and `IS_FROM_ESS`) while `addShareRequest` includes them; `(secondary)` `WebContent/css/isteven-multi-select.css:109` — background URL missing `DEFAULT/` segment; `(tertiary)` ui-bootstrap datepicker/tooltip stack on roster path expects `ngModel.$isEmpty`.
- **Trigger:** Save/edit on Alternate Work Location details POSTs to sharerequests endpoints; LOS dispatcher binds incomplete key-set → `CrossUnitDomainResolver` evaluates with eff/end skey 0 or wrong overload → throws → HTTP 422. CSS loads on multi-select unit picker. Post-update navigation to roster invokes deprecated ui-bootstrap path.
- **Consequence:** Add/edit appears to fail silently in UI (Angular error handler gets undefined body); console noise flagged under security QA label; secondary JS error can blank roster view after apparent success.
- **What raises to HIGH:** Paste Network-tab **422 response JSON** (message text) from wfmproductqa06 repro; confirm whether message is LOS ("Please check the units…") vs business validation (`rws.share.*`).

---

## Probable fix

### Pattern B — Code fix (GENUINE_BUG)

**1. LOS AuthParam parity (primary — security + functional)**

Compare `addShareRequest` (working pattern) vs detail endpoints:

```java
// addShareRequest — HAS full key set + IS_FROM_ESS
@AuthParam("{PERSON_ID:personId,...,EFFECTIVE_DATE_SKEY:shareRequestDetails.effDateSkey,END_DATE_SKEY:shareRequestDetails.endDateSkey,IS_FROM_ESS:true}")

// updateShareRequestDetail — MISSING date skeys and IS_FROM_ESS
@AuthParam("{REQUEST_NO:requestNo,MAPPING_NO:mappingNo,UNIT_ID:unitId}")
```

**Action:** Extend `@AuthParam` on `updateShareRequestDetail`, `updateShareRequestDetailsWithEmpStatDate`, and `addShareRequestDetail` to include `EFFECTIVE_DATE_SKEY` and `END_DATE_SKEY` extracted from list body (mirror nested binding used on add). Add `IS_FROM_ESS:true` on ESS-only mutating endpoints for session personId guard in `CrossUnitDomainResolver.resolveCrossDomainOperationBasedShareObj`.

**Where:** `RWS4/src/com/reflexis/rws/v5/controller/ESSController.java` (~3670–3747); validate overload dispatch in `CrossUnitDomainResolver.java` (~920–990).

**Who:** L3 Engineering (server + ESS web).

**Risk:** global — CROSS_UNIT_DOMAIN affects all tenants using ESS Alternate Work Location.

**Verify:** Re-run ticket repro (180000001 on wfmproductqa06): add/edit/delete AWL for cross-store unit; console shows no 422; Network returns 200 + TxnStatus success.

**2. CSS 404 (secondary — cosmetic/console noise)**

**Action:** Change `isteven-multi-select.css` line 109 from `../images/innercontainer-expand.png` to `../images/DEFAULT/innercontainer-expand.png` (consistent with `request.calendar_new.css`, `annual.vacation.plan.css`).

**Where:** `RWS4/WebContent/css/isteven-multi-select.css`

**3. ui-bootstrap `$isEmpty` (INTQA02 — separate track)**

**Action:** Trace roster redirect after AWL update on INTQA02; ensure datepicker/tooltip module loaded on `associate.common.renderers.js` path provides `ngModel.$isEmpty` or upgrade ui-bootstrap-tpls bundle to match Angular version.

**Where:** `WebContent/scripts/associate.common.renderers.js`; `WebContent/tpc/bootstrap/ui-bootstrap-tpls-min.js`

---

## Test gap

| Gap | Recommended TC | Setup |
|-----|----------------|-------|
| No BAT for ESS web AWL add/edit LOS path | **TC-NEG-AWL-ESS-001** | Associate at Store A; criteria allows Store B; add + edit AWL via `shareRequests.jsp`; assert no 422 and DB row in share request tables |
| AuthParam drift regression | **TC-NEG-AWL-ESS-002** | Code-level: static check that all ESS `sharerequests` POST `@AuthParam` strings include `EFFECTIVE_DATE_SKEY` + `END_DATE_SKEY` when CROSS_UNIT_DOMAIN share overload is used |
| CSS asset 404 | **TC-NEG-AWL-ESS-003** | Load AWL page; assert no 404 for `innercontainer-expand.png` in network log |
| KB note | test-scenarios.md | **ESS Alternate Work Location — No SIT/BAT coverage** (confirmed) |

---

## Open questions

| ID | State | Owner | Blocks fix? | Question |
|----|-------|-------|-------------|----------|
| Q1 | OPEN | Reporter/QA | no | Paste full **422 response body** from Network tab for `add/request.json` and `update/requestDetails/227/6232.json` on wfmproductqa06 |
| Q2 | OPEN | CSE/QA | no | Confirm **share criteria** configured for associate 180000001 home store → target STORE15 for eff date range 08/09–08/15/2026 |
| Q3 | OPEN | QA | no | Is INTQA02 **`m.$isEmpty`** failure in scope for this ticket or a separate defect? |

---

## Evidence trail

**KB citations:** test-scenarios.md (ESS Alternate Work Location — no SIT/BAT coverage); release_notes_registry RNI-0141 (Criteria Configuration Warning for Alternate Work Requests — adjacent WEAK, not direct fix).

**Code citations:**
- `ESSController.java:addShareRequest` — full AuthParam with `IS_FROM_ESS:true` (~3673–3675)
- `ESSController.java:updateShareRequestDetail` — incomplete AuthParam (~3733–3738)
- `CrossUnitDomainResolver.java:resolveCrossDomainOperationBasedShareObj` — ESS session guard (~921–926) and criteria unit check (~988–990)
- `DefaultRestController.java` — HTTP 422 for business exceptions (~146)
- `isteven-multi-select.css:109` — wrong image path
- `employee.share.actions.js:updateShareRequestDetail` — POST body is detail list (~167–170)

**Candidate cause ledger (UI-triggered save):**

| Layer | Status | Confidence | Evidence |
|-------|--------|------------|----------|
| client_emission | checked-clear | MEDIUM | Client posts standard detail list; 422 is server response |
| server_api | implicated | HIGH | 422 on documented REST endpoints |
| los_auth | implicated | MEDIUM | AuthParam key-set mismatch vs add endpoint (los-key-drift R2) |
| config_data | possible | LOW | Criteria could reject unit; needs Q2 |
| static_asset | implicated | HIGH | 404 `innercontainer-expand.png` per ticket |
| ui_framework | implicated | MEDIUM | `$isEmpty` TypeError on INTQA02 screenshot only |

**Attachments read:** jira:image-2026-07-21-16-35-14-401.png (skipped — not downloaded locally); jira:screenshot-1.png (skipped — not downloaded locally)

**Skipped layers:** sws_zta, rflx-wfm-shiftbuilding, rflx-wfm-frameworkscheduling, rflx-wfm-scheduling — not in workspace (optional; not implicated).

---

## Provenance

### Confidence flags

- [INFO: Step 2 included WebContent client-emission layer — 422 confirmed server-side before back-end walk]
- [INFO: DBSCRIPTS_ROOT and I18N_ROOT not in workspace — Step 2 limited to SERVER_ROOT]
- [WARN: Jira attachment PNGs not present under jira-intake/attachments — console evidence inferred from ticket description + prior export]
- [WARN: 422 exact exception message not captured — confidence capped at MEDIUM until Q1 answered]

### Refinement log

| Version | Date | Trigger | Confidence | Summary |
|---------|------|---------|------------|---------|
| R1 | 2026-08-11 | /triage WFM-141456 | MEDIUM | Initial triage — LOS AuthParam gap + CSS 404 + ui-bootstrap error |

---

*Generated by Cursor AI · Version: R1 · Date: 2026-08-11 · Base: WFM-141456*

<!-- /SKILL_SECTION -->
