# Unit Turn Schedule and Budget Tool: Edit Prompt for Claude Code (Round 3)

You are working on an existing single file browser based HTML tool (`unit-turn-tool-v1.html`). It uses browser local storage for persistence. No server, no login, no runtime network calls. The jsPDF, jsPDF autoTable, and SheetJS libraries are bundled inline and must stay that way.

Read the current HTML file in full before writing any code. The current build is the authoritative reference for how things work now. The working specification document (Unit Turn Scheduling Tool Spec) is available for intent; where the two conflict, surface the discrepancy before resolving it.

Apply only the changes described below. Do not restructure, rename, or reflow anything not listed. Preserve every existing behavior not explicitly touched, including the scheduling engine, dependency walk up, baseline capture, status logic, JSON and Excel export, and the offline promise.

The function and variable names in each phase are anchors to help you locate the code. Verify each against the current file before editing rather than trusting the name alone.

Work in phases, in the order listed. Complete and self verify each phase before starting the next. After each phase, state what changed and what is next. Write the completed file to disk at the close of each phase rather than waiting until the end. The cleanup phase runs last, after every functional change, so nothing gets regenerated behind it.

Standing rule for any user facing text you add: no em dashes and no hyphens in prose shown to the user. Technical strings such as CSS values, filenames, and date format strings are exempt.

---

## Phase 1: Trade card material purchasing label

The trade card checkbox currently reads "FM purchasing materials directly." This tool is not FM specific, and the label should point at the community rather than at FM.

Anchor: the checkbox markup around `sd.materialsDirect` in the step card render (`'<div class="field field-check"><label style="text-transform:none;font-size:13px;">FM purchasing materials directly</label>...`).

Required change:

1. Change the visible label text to "Community purchasing materials directly."
2. Do not touch the underlying field name `materialsDirect`, its stored value, or any logic that reads it. This is a display text change only, not a data shape change. No migration entry is needed.
3. Search the rest of the file, including the PDF generation code, for any other place the words "FM purchasing" or a similar FM branded phrase describe this same concept, and update those too so the wording is consistent everywhere it appears.

Verify: the trade card checkbox reads "Community purchasing materials directly" with no leftover FM branded phrasing anywhere the concept appears, and the checkbox still stores and reads `materialsDirect` exactly as before.

---

## Phase 2: Move Scope of Work Summary to page one

Anchor: `generateUnitTurnPdf`. Current layout is page one header block (including the three row cost group from the prior round) plus Finish Specifications, then page two Cost Summary table plus the subtotal, resident deduction, and total, then the Scope of Work Summary.

The Scope of Work Summary narrative needs to move to page one, positioned below the three row cost group and above Finish Specifications. The per category Cost Summary table and its totals stay on page two exactly as they are now; only the Scope of Work Summary narrative block moves.

Required behavior:

1. New page one order, top to bottom: header block through the three row cost group ending in Net request total cost, then the Scope of Work Summary heading and narrative, then Finish Specifications.
2. New page two order: Cost Summary table, then Summary of Costs subtotal, Custom Costs Paid by Resident, and Total Request Amount, with nothing else below them.
3. Carry forward the existing safeguard that shrinks the Scope of Work narrative before it would spill onto a third page. That safeguard now has to account for page one carrying more content than before (header block, cost group, scope narrative, and finish specifications all on one page), so re verify the shrink behavior against a project with a long scope narrative and a full finish specification list to confirm it still holds to two pages total.
4. Keep the existing spacing decisions from the prior round: extra clearance above the Finish Specifications header so it still reads as a section break, and clear space above and below the Scope of Work heading so it does not crowd the cost group above it or the Finish Specifications below it.
5. Do not change the wording of any label on either page, and do not change the Cost Summary table or its totals in any way beyond removing the Scope of Work Summary from below them.

Verify: page one shows header block, three row cost group, Scope of Work Summary, then Finish Specifications, in that order. Page two shows only the Cost Summary table and its three totals. Test against a project with a short scope narrative and a project with a long one to confirm the shrink safeguard still holds the output to two pages with no browser print dialog.

---

## Phase 3: Calendar By Resource view

Anchors: `renderByResource`, `STATE.expandedRow`, `expandRow`, `selectResourceForByResource`, `renderDaySegmentBar`, `accordion-body`.

Today the By Resource view shows one collapsed summary row per project, with a single rollup bar spanning that project's committed steps for the selected resource. Clicking the row expands it to show each task with its own bar, indented beneath.

Required behavior:

1. When the By Resource view renders, every project group is expanded by default. The DPO should see every task for every project on that resource without clicking anything.
2. Remove the project level rollup bar from the group header row. The header row becomes a label only, no timeline bar, since the task rows below it already carry the real timeline. This is what the instruction "project timeline is not necessary, just timeline per row per task" means: only the individual task rows carry a bar, not the project group itself.
3. Task rows stay clearly indented under their project header so the grouping still reads at a glance even without the collapsed summary bar to visually separate groups.
4. Keep the header row clickable so a DPO can still collapse a project group they are not currently interested in, since defaulting to expanded is about the starting state, not about removing the ability to collapse. Collapsing one group must not affect the expanded state of the others.
5. Switching the selected resource, or leaving and returning to the By Resource view, resets every group back to expanded by default.
6. Do not change By Unit or By Project. This phase only touches By Resource.

Verify: opening By Resource for a resource with committed steps across several projects shows every project already expanded with its tasks indented beneath it and a real timeline bar on every task row, no bar on any project header row. Clicking a project header collapses only that project. Switching resources returns to fully expanded.

---

## Phase 4: Code and comment cleanup

This phase runs last. Its purpose is a lean, current file: remove anything that no longer serves the file as it exists today, including narration left over from the build process itself.

Required actions:

1. Remove the build log banner at the top of the file (the phase by phase completion status, verification gate results, and build decision narration). None of it is required for the tool to run, and it no longer reflects the current state of the code after three rounds of edits. Replace it with a short header comment stating what the file is, that it is a single file offline tool with no server or network dependency, and where brand configuration and community settings live. A few lines, not a log.
2. Sweep the rest of the file for inline comments that reference a phase number, a verification pass, or a "build decision" explanation tied to a specific historical round of work. Where the comment explains something a future editor genuinely needs to know to avoid breaking the logic, keep the explanation but drop the phase number and the framing as a historical decision, so it reads as a plain statement of how the code works now. Where the comment only restates what the adjacent line already makes obvious, or only exists to log that something was tested, remove it outright.
3. Search for any function, variable, CSS class, or STATE field that is no longer referenced anywhere in the file as a result of this round or prior rounds of edits, including the removed launch screen path from the last round, and remove it. Confirm with a search before deleting each one, since some names are reused or referenced dynamically through string concatenation and are easy to miss with a plain text search.
4. Do not remove or shorten any comment that documents a non obvious rule the scheduling engine depends on, such as the working day model, the dependency walk up, or the baseline freeze logic. Those stay in full. This cleanup targets narration about the build, not documentation of how the logic works.
5. Do not change any functional behavior in this phase. If a cleanup candidate is ambiguous, whether something is truly dead or still reachable, leave it in place and flag it in your summary rather than guessing.

Verify: the file opens and runs with identical behavior to before this phase across project creation, scheduling, PDF generation, calendar views, and export. No console errors. Confirm the total line count dropped and state by how much.

---

## Standing constraints

- One self contained HTML file. Libraries stay bundled inline. No server, no login, no runtime network calls.
- Browser local storage is the only persistence layer. Nothing in this round changes the stored data shape, so no new migration entries are expected. If you find one is unavoidable, say so before proceeding.
- No community names hardcoded anywhere. Community identity is configured in Settings.
- All colors, fonts, and spacing continue to read from the single BRAND object and its CSS custom properties. Add no literal color, font, or spacing values elsewhere.
- Do not use em dashes or hyphens in any user facing text you add.

After all phases, do a final pass confirming no regression in project creation, the scheduling engine and dependency walk up, status colors, PDF generation, and JSON and Excel export. Confirm the file runs correctly opened directly from disk with no network. Save the output to the working directory.
