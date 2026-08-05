# WFM-142519 — Handoff Document

*Multi-skill handoff document. Each section below represents a different skill output.*

<!-- SKILL_SECTION: wst-defect-triage R1 -->
## TRIAGE (R1)
*Updated: 2026-08-05 23:33*

﻿---
jira_key: WFM-142519
summary: "Dev story - WFM-130904 - v45_COPPEL_SB CORP (RWS) : Employee > Roster > Blank space is displayed at bottom of the page while adding new record"
triage_status: GENUINE_BUG
confidence: HIGH
version: R1
status: Draft
last_updated: 2026-08-05T18:02:00Z
step_reached: "Step 2 (code analysis)"
input_form: "Jira API (hook)"
wfm_module: "Roster"
customer: "v45_COPPEL_SB"
affects_version: "45.1.22.0"
blast_radius: multi-customer
next_owner: DEVELOPER
sla: P2_3_DAYS
attachments_read: null
triage_duration: "~6m"
model: claude-4.5-sonnet
---

# Triage: WFM-142519

**Summary:** Dev story - WFM-130904 - v45_COPPEL_SB CORP (RWS) : Employee > Roster > Blank space is displayed at bottom of the page while adding new record

**Triage status:** GENUINE_BUG  
**Confidence:** HIGH â€” via Step 2 (code analysis)  
**Blast radius:** multi-customer  
**Next owner:** DEVELOPER  
**SLA:** P2_3_DAYS

---

## TL;DR

**Bug:** Bootstrap grid row incomplete (9 of 12 columns) after Mobile Carrier field removal causes blank space at bottom of Contact Info tab and 5 other affected roster detail tabs (Pay Rule, Time Clocking, Alternate work location, Time Rounding, Time Collection).

**Fix:** Add 3 empty placeholder columns to complete the 12-column Bootstrap grid in `summaryContactInfo.html` row 3 (IM / City / State), OR redistribute existing columns proportionally to span full 12 units.

**Action:** Apply Option 1 (quick fix - add `<td class="ws-header-cell col-md-1"><span></span></td>` + `<td class="ws-header-cell col-md-2" colspan="2"></td>`) after IM field, before City field; OR apply Option 2 (better fix - change all labels from `col-md-1` to `col-md-2` and remove explicit `col-md-2` wrappers, redistributing columns evenly).

---

## Probable cause

**Symptom:** Blank space (equivalent to 3 Bootstrap grid columns) displayed at the bottom of the Contact Info tab when adding or editing employee records. User reports the same issue affects 5 additional tabs: Pay Rule, Time Clocking, Alternate work location, Time Rounding, Time Collection.

**Fault:** Git commit `905729ce` (WFM-117570 - "Getting 'Invalid home phone no.' error on saving the contact info. after edit") removed the Mobile Carrier field (label + value = 3 Bootstrap grid columns: `col-md-1` + `col-md-2`) from `WebContent/ngtemplates/roster/summary/summaryContactInfo.html` without adjusting the Bootstrap 12-column grid layout. Row 3 (lines 68-75) now contains only 9 columns (IM: 3, City: 3, State: 3), leaving 3 empty columns that Bootstrap renders as blank space.

**Trigger:** Opening any roster detail tab in edit/add mode. The incomplete grid in the Contact Info summary template affects the shared `entityDetailsContentHeight` calculation used by all roster tabs that use the same edit panel layout mechanism (`showAddTemplateTab`, `updatePageHeight()`).

**Consequence:** Visual layout defect (blank space) appears at the bottom of the affected tabs' edit panels. The incomplete 12-column row causes Bootstrap to leave the remaining 3 columns empty, creating visible whitespace below the last populated field row.

**What raises confidence to HIGH:** Direct code evidence from git diff showing the exact removed HTML columns (`<td class="ws-header-cell col-md-1 rs-card-label">` for Mobile Carrier label and `<td class="ws-header-cell col-md-2" colspan="2">` for Mobile Carrier value), combined with git log confirmation that commit `905729ce` (WFM-117570) was the last modification to `roster.contactInfo.renderers.js`, which also removed the `mobileCarrierMap` and `onMobileCarrierSelect` controller logic. All 6 affected tabs share the same `entityDetailsContentHeight` calculation pattern in their renderers.

---

## Probable fix

### Pattern B â€” Code/template patch

**File(s) to change:**
- `WebContent/ngtemplates/roster/summary/summaryContactInfo.html` (lines 68-75, row 3)

**Option 1 â€” Quick fix (add placeholder columns):**

Insert 3 empty placeholder columns to complete the 12-column grid. Add after the IM field (before City field):

```html
<tr class="ws-header-row" data-href="#">
    <td class="ws-header-cell col-md-1 rs-card-label" colspan="1">
        <span ng-if="summCtrl.consolidatedUICMap['rosterContactInfoUICConfig']['imId']">
            {{summCtrl.i18nFn('IM')}}
        </span>
    </td>
    <td class="ws-header-cell col-md-2" colspan="2">
        <span ng-if="summCtrl.consolidatedUICMap['rosterContactInfoUICConfig']['imId']">
            {{summCtrl.selectedAssociateAllDetailsMap['contactDetails'].imId}}
        </span>
    </td>
    <!-- ADD THESE 3 EMPTY COLUMNS TO COMPLETE 12-COLUMN GRID -->
    <td class="ws-header-cell col-md-1 rs-card-label" colspan="1"></td>
    <td class="ws-header-cell col-md-2" colspan="2"></td>
    <!-- END PLACEHOLDER COLUMNS -->
    <td class="ws-header-cell col-md-1 rs-card-label" colspan="1">
        <span ng-if="summCtrl.consolidatedUICMap['rosterContactInfoUICConfig']['city']">
            {{summCtrl.i18nFn('City')}}
        </span>
    </td>
    <td class="ws-header-cell col-md-2" colspan="2">
        <span ng-if="summCtrl.consolidatedUICMap['rosterContactInfoUICConfig']['city']">
            {{summCtrl.selectedAssociateAllDetailsMap['contactDetails'].city}}
        </span>
    </td>
    <td class="ws-header-cell col-md-1 rs-card-label" colspan="1">
        <span ng-if="summCtrl.consolidatedUICMap['rosterContactInfoUICConfig']['state']">
            {{summCtrl.i18nFn('State')}}
        </span>
    </td>
    <td class="ws-header-cell col-md-2" colspan="2">
        <span ng-if="summCtrl.consolidatedUICMap['rosterContactInfoUICConfig']['state']">
            {{summCtrl.stateDescMap[summCtrl.selectedAssociateAllDetailsMap['contactDetails'].stateCd]}}
        </span>
    </td>
</tr>
```

**Option 2 â€” Better fix (redistribute columns):**

Change the existing 3 field pairs (IM, City, State) to each span 4 columns (2 label + 2 value), distributing the 12-column grid evenly:

```html
<tr class="ws-header-row" data-href="#">
    <td class="ws-header-cell col-md-2 rs-card-label" colspan="2">
        <span ng-if="summCtrl.consolidatedUICMap['rosterContactInfoUICConfig']['imId']">
            {{summCtrl.i18nFn('IM')}}
        </span>
    </td>
    <td class="ws-header-cell col-md-2" colspan="2">
        <span ng-if="summCtrl.consolidatedUICMap['rosterContactInfoUICConfig']['imId']">
            {{summCtrl.selectedAssociateAllDetailsMap['contactDetails'].imId}}
        </span>
    </td>
    <td class="ws-header-cell col-md-2 rs-card-label" colspan="2">
        <span ng-if="summCtrl.consolidatedUICMap['rosterContactInfoUICConfig']['city']">
            {{summCtrl.i18nFn('City')}}
        </span>
    </td>
    <td class="ws-header-cell col-md-2" colspan="2">
        <span ng-if="summCtrl.consolidatedUICMap['rosterContactInfoUICConfig']['city']">
            {{summCtrl.selectedAssociateAllDetailsMap['contactDetails'].city}}
        </span>
    </td>
    <td class="ws-header-cell col-md-2 rs-card-label" colspan="2">
        <span ng-if="summCtrl.consolidatedUICMap['rosterContactInfoUICConfig']['state']">
            {{summCtrl.i18nFn('State')}}
        </span>
    </td>
    <td class="ws-header-cell col-md-2" colspan="2">
        <span ng-if="summCtrl.consolidatedUICMap['rosterContactInfoUICConfig']['state']">
            {{summCtrl.stateDescMap[summCtrl.selectedAssociateAllDetailsMap['contactDetails'].stateCd]}}
        </span>
    </td>
</tr>
```

**Why Option 2 is preferred:** Maintains proportional field widths and eliminates the need for placeholder columns. The redistributed layout is more semantically correct and future-proof (any future field removals will be more obvious).

**Verification:**
1. Open Employee > Roster > Contact Info
2. Click "Edit" or "Add" to open the detail panel
3. Verify no blank space appears at the bottom of the Contact Info tab
4. Repeat for Pay Rule, Time Clocking, Alternate work location, Time Rounding, Time Collection tabs
5. Confirm all rows sum to 12 Bootstrap grid columns using browser DevTools

---

## Test gap

| Gap ID | Description | Why it wasn't caught | Recommended TC |
|--------|-------------|---------------------|----------------|
| `TC-NEG-ROSTER-LAYOUT-001` | Bootstrap grid completeness validation across roster detail tabs | No SIT test case validates that all roster summary panel rows have complete 12-column Bootstrap grids; visual layout checks are manual-only | Add automated UI test: "Verify all roster detail tab summary panels (Contact Info, Pay Rule, Time Clocking, Alternate work location, Time Rounding, Time Collection) have complete 12-column Bootstrap grid rows with no blank space at bottom of edit panels" |
| `TC-NEG-ROSTER-WFM117570-001` | Regression test for WFM-117570 Mobile Carrier field removal | Original fix (WFM-117570) did not include layout regression checks after field removal | Add test case: "After removing Mobile Carrier field from Contact Info, verify grid layout remains complete (12 columns) and no blank space appears in Contact Info or shared edit panel tabs" |

---

## Evidence trail

### KB citations
- None (Step 1 returned INCONCLUSIVE; no KB match for Bootstrap grid layout defects)

### Code citations
- `WebContent/ngtemplates/roster/summary/summaryContactInfo.html:68-75` â€” incomplete row (9 columns)
- `WebContent/scripts/roster/roster.contactInfo.renderers.js:9-10` â€” removed `mobileCarrierMap` and `mobileCarrierList`
- `WebContent/scripts/roster/roster.contactInfo.renderers.js:19-21` â€” removed `onMobileCarrierSelect` function
- `git commit 905729ce` â€” WFM-117570 last modified Contact Info renderer
- `WebContent/scripts/roster/roster.payRule.renderers.js`, `roster.timeClocking.renderers.js`, `roster.timeRounding.renderers.js`, `roster.timeCollection.renderers.js`, `roster.alternateWorkLocation.renderers.js` â€” all share `entityDetailsContentHeight` calculation pattern

### TER evidence
Not triggered

### Skipped layers
None (all core repos available)

---

## Open questions

None

---

## Provenance

### Confidence flags
- [INFO] Step 1 KB triage: INCONCLUSIVE â€” no KB match for Bootstrap grid layout defects
- [INFO] Step 2 code analysis: GENUINE_BUG â€” direct code evidence from git diff + commit history
- [INFO] Step 2 included WebContent template layer â€” HTML/AngularJS grid structure analysis
- [INFO] Multi-tab scope confirmed by user: "the other affected tabs (Pay Rule, Time Clocking, Alternate work location, Time Rounding, Time Collection) have the blank space issue"

### Refinement log

| Version | Date | Triggered by | Change summary |
|---------|------|--------------|----------------|
| R1 | 2026-08-05T18:02:00Z | /wst-defect-triage (initial) | Triage complete â€” GENUINE_BUG at HIGH confidence via Step 2 |

---

## Ticket snapshot

**JIRA_KEY:** WFM-142519  
**Linked issues:** WFM-130904  
**Affects version:** 45.1.22.0  
**WFM Module:** Roster  
**Customer:** v45_COPPEL_SB CORP (RWS)  
**Application type:** Not specified  
**SIT flavour:** Not specified

**Original input:** [NOT APPLICABLE]

**Description:**
Dev story created from bug WFM-130904. In dev branch v45_COPPEL_SB on CORP > Employee > Roster: Blank space is displayed at bottom of the page while adding new record.

**Steps to Replicate:**
1. Login with CORP Profile
2. Click on Employee > Roster
3. Select any associate
4. Click on Contact Information
5. Click on Edit icon on top right
6. Scroll down
7. Click on Mobile carrier
8. Select any value
9. Observe blank space is displayed at bottom of the page (Ã—)

**Build:** WFM.45.1.22.0.20260416.U003851

**Attachments read:** none

---

## Prior analysis

None

---

## Clarifications

<!-- Append-only. Each entry records substantial follow-up context that changed
     one or more sections. Q&A answers that fit on one line go into Open questions
     above; lengthy context or multi-section impacts go here. Oldest first. -->

**2026-08-05 (User clarification):**
User confirmed the blank space issue affects 5 additional tabs beyond Contact Info: Pay Rule, Time Clocking, Alternate work location, Time Rounding, Time Collection. This multi-tab scope indicates the root cause is in a shared layout component or template structure, not tab-specific logic. Analysis confirmed all affected tabs share the same `entityDetailsContentHeight` calculation pattern in their renderer controllers.

<!-- REFINE: paste new context, Q&A answers, or follow-up findings below this line.
     Then attach this file in a chat with the wst-defect-triage skill to trigger refinement. -->

---

<!-- /SKILL_SECTION -->
