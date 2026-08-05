# Triage: FREE-TIMECLOCKING-001

- Verdict: GENUINE_BUG (MEDIUM)
- Scope: Roster > Time Clocking web UI
- Symptom: Excess blank area shown below the add panel in Time Clocking screen (as highlighted in screenshot).
- Probable cause: Bottom panel visibility state and page height recalculation are not synchronized in `showHideContactInfoPanel`, causing stale container height rendering.
- Evidence:
  - `WebContent/ngtemplates/roster/timeClocking/timeClocking.html` uses nested containers bound to `timeClockCtrl.pageContentHeight`.
  - `WebContent/scripts/roster/roster.timeClocking.renderers.js` controls panel state via `showHideContactInfoPanel()`.
  - Local diff shows explicit fix pattern: set `showAddTemplateTab` after `updatePageHeight(...)` calls, matching other roster modules.
- Test gap: No UI regression test validating layout after Add/Close toggles for Time Clocking panel.
- Needed follow-up: confirm on 1366x768 and 1920x1080 with browser zoom 100% and 125%.
