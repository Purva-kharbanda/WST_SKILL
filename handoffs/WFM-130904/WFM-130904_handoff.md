# WFM-130904 — Handoff Document

*Multi-skill handoff document. Each section below represents a different skill output.*

<!-- SKILL_SECTION: wst-defect-triage R2 -->
## TRIAGE (R2)
*Updated: 2026-08-10 11:05*

---
id: WFM-130904
summary: "v45 Reflexis SB — Employee > Roster — blank space at bottom when Add on associate sub-tabs"
verdict: GENUINE_BUG
confidence: HIGH
step_reached: KB_STEP2
input_form: jira_api
module: HR
customer: REFLEXIS
flavour: SB
affects_version: "45.1.22.0"
fix_version: "45.1.22.3"
priority: Medium
blast_radius: global
next_owner: L3-Engineering
sla: P2-2d
status: Draft
version: R2
generated: "2026-08-10T11:02:00+05:30"
last_updated: "2026-08-10T11:02:00+05:30"
duration_minutes: 12
model: composer-2.5-fast
skill_version: "2.2.0"
fix_primary: web
fault_locus: client_app
channel_confidence: HIGH
channels:
  mobile: false
  web: true
  data: false
  shared_mobile_shell: false
attachments_read:
  - { source: jira, filename: "WFM-130904-20260707-1147.md", type: read }
  - { source: jira, filename: "Blank space at bottom of the page.docx", type: skipped }
---

# Triage: WFM-130904 — v45 Reflexis SB — Employee > Roster — blank space at bottom when Add on associate sub-tabs

## TL;DR

**Bug:**    On **Employee > Roster**, after filtering to a store/location and opening an associate, clicking **Add** on Pay Rule / Time Clocking / Alternate Work Location / Time Rounding / Time Collection shows a large blank gap between the list grid and the bottom add/edit panel — the list container keeps full viewport height while the panel slides in below it.
**Fix:**    In roster associate-detail AngularJS templates under `WebContent/ngtemplates/roster/`, bind the **list** container height conditionally when add/edit mode is active (`pageContentHeightWOPopUp − entityDetailsHeaderHeight − entityDetailsContentHeight`), not the mutated `pageContentHeight` on both outer wrapper and list; apply consistently across all five sub-tab templates (and reconcile with existing `updatePageHeight()` in each controller).
**Action:** L3-Engineering (WFM-OM/HR) — P2-2d — Patch all five sub-tab HTML templates + verify on QA06/SB with CORP profile; target fix version 45.1.22.3.

## Ticket snapshot

- **Module:**             HR — Employee > Roster (associate detail sub-tabs)
- **Customer / flavour:** REFLEXIS (updated from COPPEL per comment 2026-May-26); Source: SIT (SB); labels `FIX_MIG_UI_LOW`, `POST_MIGRATION_GENERIC`
- **Affects version:**    45.1.22.0 (Build: WFM.45.1.22.0.20260416.U003851)
- **Fix version:**         45.1.22.3 (unreleased)
- **Priority:**           Medium
- **Stage:**              To Do (Unresolved)
- **Original input:**     [NOT APPLICABLE]

## Fix routing (from ticket + code)

| Field | Value |
|---|---|
| Symptom surface | Web — Employee > Roster (CORP / SYSADMIN) |
| Fault locus | **client_app** (AngularJS roster templates + layout height bindings) |
| Evidence tier | E4 (dev RCA naming `timeClockCtrl`, `pageContentHeight`, `ng-style`) → Step 2 code confirmation |
| Channel confidence | HIGH |
| Fix primary | `web` |
| Ticket hypothesis | **confirmed** — Step 2 verified template bindings in `rfx-rwsdev300` |

**Implicated areas:**

| id | surface | layer | fix_owner |
|---|---|---|---|
| roster-pay-rule-add | Pay Rule tab Add panel | web_ui | web |
| roster-time-clocking-add | Time Clocking tab Add panel | web_ui | web |
| roster-awl-add | Alternate Work Location Add panel | web_ui | web |
| roster-time-rounding-add | Time Rounding tab Add panel | web_ui | web |
| roster-time-collection-add | Time Collection tab Add panel | web_ui | web |

## Prior analysis critique

| Prior claim | Source | Verdict | Reason |
|---|---|---|---|
| List container height hard-coded to full `pageContentHeight`; bottom panel pushed below viewport creates blank gap | Ajinkya Diware (comment, 2026-May-05) | **agree** | **Step 2 confirmed:** all five templates bind list `ng-style` to `pageContentHeight` while outer wrapper also uses the same binding; controllers mutate `pageContentHeight` in `updatePageHeight()` but templates do not use conditional add/edit height — layout overflow persists. |
| Fix via conditional `ng-style` subtracting panel header + content heights when add/edit active | Ajinkya Diware (comment, 2026-May-05) | **agree (scope expanded)** | No template in repo yet uses the suggested `entityToBeShown` conditional pattern (`grep` across `ngtemplates/roster/` = zero hits). Fix must land on **all five** tab HTML files, not Time Clocking alone. |
| Issue replicates on QA06; client changed to Reflexis | Venkatesh Mamidwar / Ajinkya Diware (2026-May-26) | **agree** | Multi-environment repro supports product UI defect, not tenant data. |
| R1 triage (2026-07-07) GENUINE_BUG MEDIUM — code unverified | Prior WST handoff | **partially superseded** | R2 re-ran Step 2 with `rfx-rwsdev300` in workspace; confidence raised to **HIGH** with file:line citations. |

---

## Probable cause

- **Symptom:**     CORP user navigates **Employee > Roster** → applies store/location filter → selects associate → opens Pay Rule, Time Clocking, Alternate Work Location Request, Time Rounding, or Time Collection → clicks **Add** → large empty white band appears between the entity list and the add/edit detail panel.
- **Fault:**       Roster associate-detail AngularJS templates bind both the **outer tab container** and the **inner list grid** to `pageContentHeight`. Controllers (`roster.*.renderers.js`) call `updatePageHeight(false)` to shrink `pageContentHeight` when the bottom panel opens, but the list still consumes the full (pre-panel) vertical budget relative to header chrome already inside the outer wrapper — total DOM height exceeds the viewport and the bottom panel renders below the fold, leaving a visible blank band.
- **Trigger:**     Roster associate detail → any sub-tab supporting inline Add → user clicks **Add** (or enters edit mode) while list `ng-style` height is not reduced by bottom-panel dimensions.
- **Consequence:** Bottom add/edit panel may appear off-screen; user sees blank space. Pure UX/layout defect — no pay-engine or data integrity impact.
- **What raises to HIGH:** ✅ Achieved — static verification of all five HTML templates + matching controller `updatePageHeight()` / `showHideContactInfoPanel()` paths in `WebContent/scripts/roster/`.

---

## Probable fix

- **File(s):**
  - `WebContent/ngtemplates/roster/payRule/payRule.html` (list container line ~51)
  - `WebContent/ngtemplates/roster/timeClocking/timeClocking.html` (outer line 23, list line 45)
  - `WebContent/ngtemplates/roster/alternateWorkLocation/alternateWorkLocation.html` (outer line 24, list line 49)
  - `WebContent/ngtemplates/roster/timeRounding/timeRounding.html` (outer line 25, list line 57)
  - `WebContent/ngtemplates/roster/timeCollection/timeCollection.html` (outer line 23, list line 53)
  - Controllers: `roster.payRule.renderers.js`, `roster.timeClocking.renderers.js`, `roster.alternateWorkLocation.renderers.js`, `roster.timeRounding.renderers.js`, `roster.timeCollection.renderers.js` — reconcile template bindings with existing `updatePageHeight()` to avoid double-shrink.

- **Change sketch:** On each **list** container (not necessarily the outer wrapper), replace unconditional `pageContentHeight` binding with add/edit-aware height using `pageContentHeightWOPopUp` (or equivalent full height) minus panel chrome:

```html
ng-style="{
  height: ((ctrl.entityToBeShown == ctrl.constantsFactory.getUiEntity().add ||
           ctrl.entityToBeShown == ctrl.constantsFactory.getUiEntity().edit)
    ? (ctrl.pageContentHeightWOPopUp - ctrl.entityDetailsHeaderHeight - ctrl.entityDetailsContentHeight)
    : ctrl.pageContentHeightWOPopUp) + 'px'
}"
```

  (Substitute `payRuleCtrl`, `timeClockCtrl`, `awlCtrl`, `trControl`, `timeCollCtrl` per tab.) Prefer `ng-style` over inline `style=` per CSP guardrails. Optionally simplify controllers by stopping redundant `pageContentHeight` mutation once templates own the conditional height.

- **Blast radius:**  All CORP/HR roster users on associate detail add/edit flows — `POST_MIGRATION_GENERIC` label + QA06 repro indicate multi-customer post-migration UI surface.
- **Migration:**     None — client-side template/layout only.
- **PR scope:**      Five roster sub-tab templates + controller hygiene; UI BAT asserting no viewport gap on Add for each tab; manual verify on QA06 with CORP profile.

---

## Test gap

| ID | Persona | Action | Condition | Expected | Risk | Flavour |
|---|---|---|---|---|---|---|
| TC-NEW-WFM130904-1 | CORP_ADMIN | Roster → filter store → associate → **Time Clocking** → **Add** | Associate with existing rows | List + bottom add panel both visible; **no blank gap** | HIGH | SB / all |
| TC-NEW-WFM130904-2 | CORP_ADMIN | Same → **Pay Rule** → **Add** | Associate on pay rule tab | No viewport gap; add form usable without scrolling | HIGH | SB / all |
| TC-NEW-WFM130904-3 | CORP_ADMIN | Same → **Alternate Work Location** → **Add** | Eligible associate | Panel visible; list height shrinks; no white band | HIGH | SB / all |
| TC-NEW-WFM130904-4 | CORP_ADMIN | Same → **Time Rounding** → **Add** | Normal associate | No blank space on Add | HIGH | SB / all |
| TC-NEW-WFM130904-5 | CORP_ADMIN | Same → **Time Collection** → **Add** | Normal associate | No blank space on Add | HIGH | SB / all |
| TC-NEW-WFM130904-6 | CORP_ADMIN | Repeat on **Edit** existing row (any tab) | Existing record | Edit panel fits viewport; list height adjusts symmetrically | MEDIUM | SB / all |

- **Closest existing TC:** none in KB roster layout coverage — missed because post-migration associate-detail Add layout is outside functional roster import TC scope.

---

## Handoff envelope

**Next owner:**  L3 Engineering (WFM-OM/HR)
**Next action:** Apply conditional list-height `ng-style` to all five sub-tab templates; reconcile controller `updatePageHeight()`; verify on QA06; link dev story WFM-142519.
**SLA:**         P2 — 2 days

**Open questions:**
- **Q1** [ANSWERED: R2] All five templates bind list height to `pageContentHeight` unconditionally — see code citations below.
- **Q2** [OPEN, owner: L3-Engineering, blocks_fix: no] Fix version 45.1.22.3 is unreleased; no linked PR in Jira dev panel — confirm whether WFM-142519 branch exists.
- **Q3** [OPEN, owner: QA, blocks_fix: no] Attach QA06 screenshot/recording on ≥2 sub-tabs for BAT oracle (DOCX attachment present but not machine-read).

**Distribution:** WFM-OM/HR/Integrations/Notifications Scrum team DL; CC Ajinkya Diware, Ankur Arvind, Kshitij Uttarwar

## Evidence trail

- **KB citations:** `code-architecture.md` §15 (`WebContent/scripts/roster`); HR Roster module map
- **Code citations:**
  - `WebContent/ngtemplates/roster/timeClocking/timeClocking.html:23,45` — outer + list bind `timeClockCtrl.pageContentHeight` unconditionally
  - `WebContent/ngtemplates/roster/payRule/payRule.html:23,51` — same pattern for `payRuleCtrl`
  - `WebContent/ngtemplates/roster/timeRounding/timeRounding.html:25,57` — same for `trControl`
  - `WebContent/ngtemplates/roster/timeCollection/timeCollection.html:23,53` — same for `timeCollCtrl`
  - `WebContent/ngtemplates/roster/alternateWorkLocation/alternateWorkLocation.html:24,49` — outer + list height bindings for `awlCtrl`
  - `WebContent/scripts/roster/roster.timeClocking.renderers.js:478-504` — `showHideContactInfoPanel` / `updatePageHeight` mutates `pageContentHeight` without template conditional
  - `WebContent/scripts/roster/roster.payRule.renderers.js:163-178,200-210` — same controller pattern
- **Candidate-cause ledger (Step 2 §S2.2):**
  - `client_emission` (web_js templates): **implicated** — list height bindings
  - `server_api`: **checked-clear** — pure layout; no REST failure
  - `data_config`: **checked-clear** — repro on QA06 + Reflexis
  - `controller_logic`: **secondary** — `updatePageHeight()` interacts with template bindings
- **TER evidence:** not triggered (Step 2 resolved with code + E4 dev RCA)
- **Attachments read:** jira:WFM-130904-20260707-1147.md (read); jira:Blank space at bottom of the page.docx (skipped)
- **Skipped layers:** `[INFO: sws_zta not in workspace — optional; not implicated]`
- **Skipped layers:** `[INFO: Step 2 included WebContent client-emission layer (web_js) — layout bindings verified before back-end walk]`

---

## Provenance

**Triage duration:** ~12m
**Model:**           composer-2.5-fast
**Skill version:**   2.2.0
**Generated by:**    wst-defect-triage skill

### Refinement log

| Rev | Timestamp | Triggered by | Summary of change |
|-----|-----------|--------------|-------------------|
| R1  | 2026-07-07T11:47:00+05:30 | Initial triage (PDF) | GENUINE_BUG MEDIUM — KB + dev RCA; code unverified |
| R2  | 2026-08-10T11:02:00+05:30 | `/wst-defect-triage WFM-130904` | Re-triage with Jira API intake + Step 2 code walk; confidence **HIGH**; five template file:line citations; Q1 answered |

### Confidence flags

- `[INFO: Prior L2 analysis (Ajinkya Diware) independently evaluated — root cause agreed; R2 confirms in source]`
- `[INFO: DOCX screenshot attachment skipped — repro steps + prior handoff describe visual gap]`
- `[INFO: Config paths in wst-workspace-config.yaml pointed at missing Siddhartha clones — workspace discovery used c:\wfmsetup_jdk21\rfx-rwsdev300]`

---

## Clarifications

<!-- REFINE: paste new context below this line -->

<!-- /SKILL_SECTION -->

<!-- SKILL_SECTION: wst-defect-fix A1 -->
## FIX (A1)
*Updated: 2026-08-10 12:57*

# WFM-130904 Apply Stage A1
Generated by: wst-defect-fix
Command: /apply-fix WFM-130904

Applied fix for roster blank-gap issue.
Updated list-container height bindings in five roster associate sub-tab templates (Pay Rule, Time Clocking, Alternate Work Location, Time Rounding, Time Collection) to use add/edit-aware computed heights based on pageContentHeightWOPopUp, entityDetailsHeaderHeight, and entityDetailsContentHeight.

<!-- /SKILL_SECTION -->
