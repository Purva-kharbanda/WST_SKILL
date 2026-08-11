# WFM-141485 — Handoff Document

*Multi-skill handoff document. Each section below represents a different skill output.*

<!-- SKILL_SECTION: wst-defect-triage R1 -->
## TRIAGE (R1)
*Updated: 2026-08-12 00:02*

---
ticket_id: WFM-141485
summary: "RWS(CORP - Absence Configuration) : Error and warnings observed while adding the absence configuration"
verdict: NEEDS_MORE_INFO
confidence: HIGH
step_reached: KB_ONLY
input_form: jira_json
status: Draft
version: R1
last_updated: "2026-08-11T18:30:00+05:30"
duration_minutes: 8
model: composer-2.5
next_owner: QA_REPORTER
sla: AWAITING_EVIDENCE
blast_radius: unknown
attachments_read:
  - "jira:image-2026-07-21-19-41-17-935.png (skipped — file not present under jira-intake attachments after fetch)"
kb_citations:
  - ENT-0043
  - "kernel page ABSENCE_CONFIG → /rws/staff/rws_leave_config.jsp"
code_citations: []
handoff_github_repo: https://github.com/zebratechnologies/wst_handoffs
---

# Triage: WFM-141485

**Summary:** RWS(CORP - Absence Configuration) : Error and warnings observed while adding the absence configuration  
**Status:** Draft · **Version:** R1 · **Last updated:** 2026-08-11  
**Triage status:** NEEDS_MORE_INFO · **Confidence:** HIGH · **Step reached:** KB_ONLY  
**Assignee:** Jay Deshmukh · **Reporter:** Chaitanya Sathe  
**Fix version:** 45.1.23.0 · **Affects:** 45.1.22.1, 45.1.23.0  
**Environment:** https://wfmproductqa06.reflexisinc.co.in/kernel/views/authenticate/W/REFLEXIS.view  
**Build:** WFM.45.1.23.0.20260720.U034819  
**Labels:** 45ProdQA_Security  

---

## TL;DR

| | |
|---|---|
| **Bug** | QA reports unspecified errors and warnings on CORP Absence Configuration when adding or editing a record (SYSADMIN, build 45.1.23.0 on wfmproductqa06). No exact messages, stack trace, or HAR attached; screenshot not readable locally. |
| **Fix** | Cannot recommend a code/config fix until the exact error/warning text, save outcome (blocked vs succeeds with warnings), and Network-tab payload/response are captured. |
| **Action** | Reporter/QA: paste exact UI messages + browser console output + one save attempt HAR (or REST response body). Re-run `/triage WFM-141485` with that evidence or `/update WFM-141485`. |

---

## Probable cause

| Field | Detail |
|---|---|
| **Symptom** | On Absence Configuration add/edit, user sees errors and/or warnings (content unknown). |
| **Fault (hypothesis — LOW)** | Insufficient evidence to pin fault locus. Two plausible families pending paste-back: (A) **client-side validation** on Request Type → Absence Configuration tab (`WebContent/scripts/request.type.renderers.js` — `validateNewAbsenceConfigObjectForSave`, `addNewAbsenceConfig` / `updateLeaveConfig`) or legacy standalone page `rws_leave_config.jsp`; (B) **security-regression noise** (label `45ProdQA_Security`) — browser console CSP/deprecated-JS warnings on legacy jQuery/Angular stack without functional save failure. |
| **Trigger** | SYSADMIN → navigate to Absence Configuration → Add or Edit record. |
| **Consequence** | Unknown — may block save or may be non-blocking warnings only. |
| **What raises to HIGH** | Exact error/warning strings from UI + console; whether POST to `/saveLeaveConfig` or `/updateLeaveConfig` (or `/updateRwsLeaveConfig`) returns HTTP 200 with `TxnStatus` error; screenshot text OCR. |

---

## Probable fix

**Blocked pending evidence.** Once messages are supplied, likely fix surfaces:

| If evidence shows… | Likely fix area |
|---|---|
| Client validation toast ("Please Select : …") | `request.type.renderers.js` — mandatory-field binding on ui-select arrays (`distListId[0]`, etc.) |
| REST 4xx/5xx or `TxnStatus` error message | `AssociateController.saveLeaveConfig` / `updateLeaveConfig` → `DefaultAssociateCoreService.addRwsLeaveConfig` |
| Custom absence mode JSON validation | `rws_leave_config.jsp` client validators (`customAbsenceMode.*` i18n keys) |
| Console-only CSP / third-party warnings, save succeeds | Security hardening / library upgrade follow-up (non-blocking); classify as KNOWN_BEHAVIOUR or separate security story |
| Permission denied | Security group `ABSENCE_CONFIG` D2 permissions for SYSADMIN profile on QA tenant |

---

## Test gap

| Gap | Recommended test |
|---|---|
| No BAT/SIT case for Absence Configuration add+edit with full mandatory fields on 45.1.23 security QA flavour | **TC-NEG-ABSENCE-CFG-001** — SYSADMIN adds absence config row with all mandatory fields; assert zero error toasts and successful save |
| No regression for security-labelled QA pass on legacy Absence Configuration surfaces | **TC-NEG-ABSENCE-CFG-002** — Open add/edit dialog; capture browser console; fail only on functional errors, document allowed CSP warnings separately |
| Missing negative path for partial mandatory fields | **TC-NEG-ABSENCE-CFG-003** — Omit Store Group; assert deterministic validation message matches i18n key |

---

## Open questions

| ID | State | Owner | Blocks fix | Question |
|---|---|---|---|---|
| Q1 | OPEN | QA_REPORTER | yes | What are the **exact** error and warning messages shown (copy/paste or annotate screenshot)? |
| Q2 | OPEN | QA_REPORTER | yes | Does **Save** fail (record not persisted) or do warnings appear while save succeeds? |
| Q3 | OPEN | QA_REPORTER | yes | Which navigation path: standalone **Employee → Absence Configuration** (`rws_leave_config.jsp`) or **Request Type → Absence Configuration** tab? |
| Q4 | OPEN | QA_REPORTER | yes | Browser **console** log + **Network** tab response for the save XHR (request body + JSON response). |
| Q5 | OPEN | QA_REPORTER | no | Are findings limited to **security scan** (CSP/console warnings) per label `45ProdQA_Security`, or functional incorrect behaviour? |

---

## Ticket snapshot

**Description:** Login SYSADMIN → Absence Configuration → Add or edit record → observe errors and warnings.  
**Expected / Actual:** Not specified (custom field: "Not working as expected").  
**Comments:** None.  
**Linked:** WFM-143275 (QA story), WFM-143278 (Dev story).  
**Original input:** [NOT APPLICABLE]

---

## Fix routing (ticket)

**Primary:** unknown (symptom surface = web CORP admin; fault locus unresolved)  
**Channels:** mobile=false · web=true · data=unknown  
**Implicated areas:** Absence Configuration UI (`ABSENCE_CONFIG` kernel page, Request Type absence tab), server REST `AssociateController` leave-config endpoints  
**Step 2 plan (deferred):** SERVER `AssociateController` + `request.type.renderers.js` §absenceConfig + optional `rws_leave_config.jsp` legacy path — **not executed** (NEEDS_MORE_INFO gate).

---

## Prior analysis critique

No L1/L2/L3 comment analysis present in ticket.

---

## Evidence trail

**KB citations:** ENT-0043 (Absence Configuration / Modes 1-8); kernel `ABSENCE_CONFIG` page → `/rws/staff/rws_leave_config.jsp`.  
**Code citations:** None (Step 2 skipped).  
**Attachments read:** jira:image-2026-07-21-19-41-17-935.png (skipped — not found under `jira-intake/WFM-141485/attachments/` after fetch).  
**Skipped layers:** DBSCRIPTS_ROOT, I18N_ROOT not in workspace config; ZTA/ShiftBuilding/FWS/Scheduling optional repos not in workspace.

### Confidence flags

- HIGH confidence on **NEEDS_MORE_INFO** verdict — missing minimum reproducible evidence (messages, save outcome, network payload).
- `[INFO: Step 2 skipped per NEEDS_MORE_INFO gate — code paths identified for follow-up only]`
- `[WARN: Jira attachment PNG referenced in JSON but not available locally for visual analysis]`

### Refinement log

| Version | Date | Trigger | Confidence | Notes |
|---|---|---|---|---|
| R1 | 2026-08-11 | `/triage WFM-141485` | HIGH (NEEDS_MORE_INFO) | Initial triage; Jira API hook; Step 2 not run |

---

## Clarifications

_(none)_

---

_Generated by Cursor AI · wst-defect-triage v2.2.0 · R1 · 2026-08-11_

<!-- /SKILL_SECTION -->
