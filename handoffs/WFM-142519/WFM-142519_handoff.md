# JUnit/Mockito Plan — WFM-142519

**Ticket:** WFM-142519 — Roster blank space at bottom when adding new record  
**Skill revision:** T1  
**Verdict:** No JUnit implementation recommended for current fix diff

## PLAN — JUnit/Mockito (wst-junit-mockito)

What changed (in plain English):
On Employee > Roster associate detail tabs, the Add bottom panel was shown before the page recalculated list height, leaving blank space at the bottom. The fix reorders five AngularJS controller methods so `updatePageHeight()` runs before the Add panel visibility flag is set.

Why it could break (developer perspective):
- Add panel may still appear with wrong height if toggle order regresses on any of the five tabs.
- Close path may leave residual whitespace if list height is not restored before hiding the panel.
- Other roster tabs using the same pattern could still show the defect if not updated consistently.

Test cases I will write:

| # | Category    | Scenario (plain English)                         | Expected outcome                |
|---|-------------|--------------------------------------------------|---------------------------------|
| — | Out of scope | No Java production files changed in WFM-142519 fix diff | JUnit/Mockito cannot directly assert AngularJS layout timing |

Coverage of the changed lines (production file:line ranges):
- `WebContent/scripts/roster/roster.payRule.renderers.js:199-214` -> **not Java; no JaCoCo target**
- `WebContent/scripts/roster/roster.timeClocking.renderers.js` (same pattern) -> **not Java**
- `WebContent/scripts/roster/roster.timeCollection.renderers.js` (same pattern) -> **not Java**
- `WebContent/scripts/roster/roster.timeRounding.renderers.js` (same pattern) -> **not Java**
- `WebContent/scripts/roster/roster.alternateWorkLocation.renderers.js` (same pattern) -> **not Java**

Mechanics (one-liners, not for review):
- Tier: **None applicable** — fix is client-side JS only; `./gradlew test` would not exercise changed lines.
- Test class: **none proposed** for this diff.
- Peer mirrored: existing `RosterTimeClockingTest` / `RosterAlternateWorkLocationTest` cover **Java service APIs**, not `showHideContactInfoPanel` layout logic.
- Static mocks: none.
- Recommended validation instead: manual/UI or BAT using handoff cases `TC-NEG-WFM142519-01` through `-05` and regression scope RS-1..RS-4.

>>> DEVELOPER REVIEW <<<
Reply with one of:
  HOLD                          -> recommended; use UI/BAT validation for this ticket
  GO + ADD: <Java class/scenario> -> only if you want unrelated backend regression tests added
  GO                          -> implement only if Java production code for this fix is added first
