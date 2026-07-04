# Unit Turn Schedule and Budget Tool: Edit Prompt for Claude Code (Round 2)

You are working on an existing single file browser based HTML tool (`unit-turn-tool-v1.html`). It uses browser local storage for persistence. No server, no login, no runtime network calls. The jsPDF, jsPDF autoTable, and SheetJS libraries are bundled inline and must stay that way.

Read the current HTML file in full before writing any code. The current build is the authoritative reference for how things work now. The working specification document (Unit Turn Scheduling Tool Spec) is available for intent; where the two conflict, surface the discrepancy before resolving it.

Apply only the changes described below. Do not restructure, rename, or reflow anything not listed. Preserve every existing behavior not explicitly touched, including the scheduling engine, dependency walk up, baseline capture, status logic, JSON and Excel export, and the offline promise.

The function and variable names in each phase are anchors to help you locate the code. Verify each against the current file before editing rather than trusting the name alone.

Work in phases. Complete and self verify each phase before starting the next. After each phase, state what changed and what is next. Write the completed file to disk at the close of each phase rather than waiting until the end.

Standing rule for any user facing text you add: no em dashes and no hyphens in prose shown to the user. Technical strings such as CSS values, filenames, and date format strings are exempt.

---

## Phase 0: Backward compatibility and migration scaffold (do this first)

Several phases add new fields to the project and step data shape. Before touching anything else, locate the existing project normalize and import shaping routine (the function that fills fresh defaults into an imported or loaded project so every step and trade slot exists in the shape the app expects) and the schema version handling.

Every new field introduced in the phases below must be added to that normalize routine with a safe default, so a project saved by the current build loads cleanly with the new fields present. Do not change existing storage keys. If you bump the schema version integer, add a migration entry that fills the new defaults for older data rather than rejecting it. A first run browser with no saved projects must also work.

---

## Phase 1: Remove the initial launch screen

The first time the tool opens in a browser it shows a launch screen to collect community name and community key before the app is usable. Remove that screen. The same community information is already editable in the Settings tab, so the launch step is redundant.

Anchors: the `firstrun` view gate in `route()` (`if (!STATE.settings) { STATE.view = 'firstrun'; ... }`), `renderFirstRun()`, `completeFirstRun()`, `newSettings(name, key)`, and `ACTIVE_KEY_STORAGE`.

Required behavior:

1. On first load with no saved settings, initialize a default settings object automatically instead of routing to the launch screen. Build it through the same `newSettings` path so the resource list and the finish option lists are all present. Community name starts blank. Community key starts as a stable default value so local storage has a namespace to write into.
2. Land the user on the Settings tab on that first load, where they fill in community name and community key.
3. Community key is the local storage namespace and is stamped into every JSON export, PDF filename, and Excel export filename. So when the user sets or changes the community key in Settings, re point the active storage namespace to the new key and carry the current settings and any projects across to it, then update `ACTIVE_KEY_STORAGE`. On a fresh install there are no projects yet, so this is a rename of an effectively empty namespace, but implement it so it also holds if a key is changed later.
4. Delete `renderFirstRun` and `completeFirstRun` and the `firstrun` branches in `route()` and `renderApp()`, and remove `completeFirstRun` from the exposed `App` object. Do not leave a dead reference.
5. Never hardcode any real community name anywhere. The default community name stays blank until the user types one.

Verify: opening the tool in a clean browser goes straight into the app on the Settings tab with blank community name, no launch screen, and no console error. Setting a community key in Settings produces correct PDF and export filenames.

---

## Phase 2: Split Demo and Framing into separate trades

Today step 1 is a single step labeled "Demo and wall relocation" and MEPFP rough in waits on it. Framing and wall relocation must become its own trade sitting between demo and MEPFP, and MEPFP must trigger on the framing task.

Anchor: the `STEP_DEFS` array. Current relevant entries:
- `demo` number 1, label "Demo and wall relocation", `alwaysOn: true`, formCategory 'Demo'
- `mepfp_rough` number 2, waitsOn `['demo']`

Required changes:

1. Change the `demo` step label to "Demo". Keep it `alwaysOn` and keep formCategory 'Demo'.
2. Insert a new step `framing` immediately after `demo` in the array, labeled "Framing and wall relocation", `waitsOn: ['demo']`. It is a normal toggleable step, not `alwaysOn`, so it is present only when the turn actually involves framing or wall relocation. Give it its own form category 'Framing' (see decision note below).
3. Change `mepfp_rough` to `waitsOn: ['framing']` instead of `['demo']`.
4. Renumber the `number` field on every step so the sequence reads 1 through the new total in array order. Array order is the topological order the engine relies on, so inserting `framing` directly after `demo` keeps it valid.
5. The toggle off walk up already substitutes a step's own predecessors when it is inactive. Confirm that with `framing` toggled off, MEPFP rough in resolves its start against `demo` exactly as it does today. This should fall out of the existing walk up with no special casing. Verify it by hand.
6. If you add the 'Framing' form category, add it to `FORM_CATEGORIES` and the Form Category Mapping so a framing cost line flows into the cost table and totals. Keep the vendor, labor, and materials column rules consistent with the other trade steps.
7. Migration: existing saved projects have no `framing` step. In the normalize routine, add a `framing` step defaulted to inactive (toggled off) so old projects keep their current effective schedule.

Decision notes, defaults chosen, change with one line if you want otherwise:
- Framing carries its own 'Framing' cost category rather than folding into 'Demo', so its cost reads as a distinct trade line.
- Framing is a single step, not a multi trade line step like the MEPFP steps.
- Shower conversion still waits on `demo`, since the instruction named only MEPFP. Say the word if shower should also wait on framing.

Verify: a project with framing off behaves identically to the current build. A project with framing on schedules framing after demo and MEPFP rough in after framing, and a Framing cost line appears in the cost summary.

---

## Phase 3: Room checklists on LVP and Carpet

The LVP and Carpet flooring steps need a room checklist so the DPO can check off which rooms receive that flooring.

Anchors: `STEP_DEFS` entries `lvp` (finishField 'lvpColor') and `carpet` (finishField 'carpetColor'), `renderStepCard`, and `FINISH_FIELD_LABELS`.

Required behavior:

1. Define a single constant near the top of the app code listing the rooms in this order: Kitchen, Foyer, Living Room, Primary Bedroom, Primary Bathroom, Bedroom 2, Bathroom 2. Keep it in one place so the list stays portable and editable.
2. On the LVP step card and the Carpet step card only, render that room list as a set of checkboxes the DPO can toggle. Store the checked rooms in that step's data as a new field. Default is no rooms checked.
3. Reflect the checked rooms in the PDF where that flooring finish spec appears, appending the selected rooms to the finish specification line for LVP and for Carpet, so the request shows which rooms each flooring covers.
4. Migration: add the room field defaulted to empty for both steps in the normalize routine so older projects load cleanly.

Verify: checking rooms on LVP and Carpet persists across reload and appears on the generated PDF. No other step shows a room checklist.

---

## Phase 4: Projects tab completion date and time to turn from the schedule

On the Projects tab the Completion Date and Time to Turn columns are blank until a project is fully complete, because they read only the actual completion stamps. They should instead fill in from the schedule and adjust as the schedule changes.

Anchors: `renderProjectListView` row build (the `completionDate` and `timeToTurn` cells), `projectCompletionDate(project)` (actual completion, returns null until Completed), `projectedCompletionDate(project)` (schedule based, next working day after `final_clean.plannedEnd`), and `timeToTurnDays(project, completionDate)`.

Required behavior:

1. Completion Date column shows the projected schedule completion date from `projectedCompletionDate` while the project is in progress, so it is populated as soon as the project is scheduled and updates whenever the schedule recalculates. When the project is actually Completed, show the real completion date from `projectCompletionDate`.
2. Time to Turn is computed from Demo planned start through whichever completion date is shown in that row, using the existing `timeToTurnDays` calendar day math, and adjusts along with the schedule.
3. Leave both cells blank only when the project is not yet scheduled and no completion date can be computed.

Verify: a scheduled but in progress project shows a projected completion date and a time to turn, both of which move when a step slips. A completed project shows the actual completion date.

---

## Phase 5: Resource dropdown closes when moving on

In the resource configuration, selecting the resource name for a trade leaves the dropdown open when the user moves to the next text box. It should close when the user moves on.

Anchors: `STATE.openDropdown`, `renderStepResourceDropdown`, `toggleStepResourceDropdown`, and the generic dropdown open state used elsewhere in the resource and settings panels.

Required behavior:

1. When focus leaves an open dropdown, or the user clicks or tabs into another field, close the open dropdown by clearing `STATE.openDropdown`. Add a close on blur and a close on outside interaction so only one action is needed to move on.
2. Apply the same close behavior to any dropdown driven by `STATE.openDropdown`, so both the resource configuration dropdown and the step resource dropdown close consistently.
3. Do not change what the dropdown selects or how selections are stored. This is only about when it closes.

Verify: pick a resource, then click or tab into the next field. The dropdown closes on its own with no second click.

---

## Phase 6: Calendar fixes

Anchors: `renderTimelineAxis`, `renderCalendarView`, `renderByUnit`, `renderByProject`, `renderByResource`, `renderProjectStepRows`, `renderDaySegmentBar`, `PX_PER_DAY`, `globalDateRange`, `ganttTrackWidth`, and the `accordion-body` wrapper.

### 6.1 Axis date labels overlap on short or single projects

`renderTimelineAxis` labels every day when the visible span is 21 days or fewer. At `PX_PER_DAY` of 16 pixels, a daily label like "Mon 7/6" is wider than its slot, so on a short span or a single short project the labels overlap and become illegible. On a 4 to 5 week span it looks fine.

Fix the label density so labels never overlap. Compute a minimum pixel gap that fits the rendered label width and only place a label when there is room for it since the last placed label, stepping the interval up as needed. Always keep the range first visible day labeled. The bars, the today marker, and the track width must not change. Only the axis label placement changes.

### 6.2 Expanded rows must line up with the summary row and axis

When a By Unit or By Resource row is expanded, the per task bars below do not line up with the summary row bar above them or with the axis date marks. Determine why the expanded track origin is offset from the summary row and axis, most likely the `accordion-body` wrapper adding horizontal padding or a difference in the row label column width, and make the expanded rows share the exact same range, `PX_PER_DAY`, row label column width, and left origin as the summary row and the axis. After the fix, a task due on a given date sits directly under that date on the axis and under the matching segment of the summary bar.

### 6.3 Calendar task rows link regardless of status

In `renderProjectStepRows` and in the expanded rows of `renderByResource`, a task row is only clickable when its status is yellow or red (`var clickable = r.status.color === 'yellow' || r.status.color === 'red';`). Make every task row clickable so it opens that task in its project no matter the status, so the user can open a task and adjust its date manually at any time. Keep the existing `goToTask` handler and the landing on the opened step. Apply this in both places that set `clickable`.

Verify: axis labels are readable on a single short project and on a 4 to 5 week project. Expanded task bars align with the summary bar and axis. Every task row on the calendar opens its task.

---

## Phase 7: PDF fixes

Anchor: `generateUnitTurnPdf`. The current layout is page one header block plus Finish Specifications, then page two Cost Summary table plus the subtotal, resident deduction, and total, then the Scope of Work Summary.

### 7.1 Cost rows on page one

Confirmed from current output: the page one header block is a single autoTable (anchor `headerRows`) whose rows run Community, Care level, Unit number, Apartment type, Turn type, Deposit on unit, Requested by, Date submitted, Custom costs paid by incoming resident, Vacancy notice date, Unit available date, Projected schedule completion date. The only cost figure on page one is the one row "Custom costs paid by incoming resident".

That single cost line must become a contiguous three row cost group in this order:
1. Total summary costs, the sum of all costs, above.
2. Custom costs paid by incoming resident, the existing row, in the middle.
3. Net request total cost, below, bolded, equal to total summary costs minus the custom resident costs.

Implementation: move all three cost rows together as one contiguous group to the bottom of the `headerRows` autoTable, immediately below Projected schedule completion date and just above the Finish Specifications section. This means removing the existing Custom costs paid by incoming resident row from its current mid block position and placing it as the middle row of the group, with Total summary costs directly above it and Net request total cost directly below it. Bold both the label and the amount on the Net request total cost row (a per row style through the autoTable, not a global change).

Use the existing figures with no new math: `totalProjectCost` for Total summary costs, the existing custom resident cost field for the middle row, and `totalRequestAmount` for Net request total cost. On the sample unit these resolve to Total summary costs $23,150.00, Custom costs paid by incoming resident $200.00, and Net request total cost $22,950.00.

Keep the two new row labels as written above. Leave the existing middle row label as it is and leave the page two labels (Summary of Costs subtotal, Custom Costs Paid by Resident, Total Request Amount) unchanged. The per category Cost Summary table stays on page two.

Placement, locked: the three row cost group reads as a clean summary at the bottom of the header block, just above Finish Specifications, not buried among the date fields. Do not leave a Custom costs row in the old mid block position.

### 7.2 Finish Specifications header

Give the Finish Specifications header more vertical space above it and make it more pronounced, larger and heavier than the surrounding lines so it reads as a clear section break.

### 7.3 Page two top margin

The page two Cost Summary needs more margin above it, more space between the top of the page and where the cost content starts.

### 7.4 Scope of work spacing

The Scope of Work Summary narrative needs more space above its heading and more space between the summary cost table and the narrative, so the narrative is not crowded against the totals above it. Keep the existing safeguard that shrinks the narrative before it would ever spill onto a third page.

Verify: page one shows the three cost rows in the stated order with the net request total bolded. The Finish Specifications header stands out with clear space above it. Page two cost content sits lower from the top, and the scope narrative has clear separation above it and below the cost table. The PDF still fits on two pages and still downloads with no browser print dialog.

---

## Standing constraints

- One self contained HTML file. Libraries stay bundled inline. No server, no login, no runtime network calls.
- Browser local storage is the only persistence layer. All changes stay backward compatible with data already saved, per Phase 0.
- No community names hardcoded anywhere. Community identity is configured in Settings.
- All colors, fonts, and spacing continue to read from the single BRAND object and its CSS custom properties. Add no literal color, font, or spacing values elsewhere.
- Do not use em dashes or hyphens in any user facing text you add.

After all phases, do a final pass confirming no regression in project creation, the scheduling engine and dependency walk up, status colors, PDF generation, and JSON and Excel export. Confirm the file runs correctly opened directly from disk with no network. Save the output to the working directory.
