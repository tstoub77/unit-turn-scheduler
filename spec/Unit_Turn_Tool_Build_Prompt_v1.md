# This build runs in Claude Code. Write the completed file to disk at the close of each phase. Do not wait until Phase 5 to commit output.

# Build Prompt: Unit Turn Schedule and Budget Tool, Version 1.0

## Your role

You are the senior build engineer and subject matter expert for this project. You own the full implementation end to end. You make every technical and design decision yourself at an expert standard. You do not ask the requester any questions during the build. If the specification is silent on a detail, you choose the correct professional answer, note the decision in your build log, and keep moving. The only acceptable reasons to stop before the tool is finished are a genuine internal contradiction in the specification that cannot be resolved by reading the whole document, or a factual impossibility.

## Source of truth and naming

The attached Revision 5 specification is the authoritative source for every behavior, rule, field, and output. Build exactly to it. Two explicit overrides sit above the spec text:

* The tool is named **Unit Turn Schedule and Budget Tool**. Use this name in the interface header, the browser title, and anywhere the product names itself. The spec title "Unit Turn Scheduling Tool" is superseded.
* The tool is **version 1.0**. Display the version in the interface and stamp it into the code.

Where the spec marks something out of scope for version one, honor that. Regional and ED rollup visibility (M1) and templates by turn type (M2) are not built. The tool is DPO only.

## How you work

Build in five sequential phases, listed below. Do not begin a phase until the previous phase has passed its verification gate. At the end of each phase you stop coding, run the verification gate for that phase against the relevant spec sections, fix every discrepancy you find before advancing, and record the result in a running build log. The build log travels with you across all five phases and appears in your final summary. Do not surface partial work or questions to the requester between phases. Proceed through all five phases in one continuous effort and present the finished tool at the end.

You determine the best build method yourself: file structure, how you bundle libraries, how you test the scheduling engine, how you verify the PDF and Excel output. You are the expert. Choose the approach that produces a correct, maintainable, single file deliverable.

## Non negotiable architecture constraints

These come straight from the spec and cannot be traded away for convenience.

* One self contained HTML file. Everything ships inside it: markup, styles, script, and every third party library.
* No server, no login, no build step required to run it. A DPO opens the file in a browser and it works.
* No runtime network calls of any kind. The PDF library (jsPDF recommended, pdf lib acceptable if you need finer layout control) and the Excel library (SheetJS) are bundled inline in the file at build time, not loaded from a CDN. The offline promise holds without exception.
* Persistence is browser local storage only. Keys are scoped by community key so a shared origin cannot cross contaminate data between communities. Each project save is a single atomic write. Every write and every export is stamped with a single integer schema version, with a migration chain for older data and a hard refusal for data newer than the running tool.
* This tool is not compatible with a claude.ai artifact preview, because browser storage is blocked there. Do not deliver it as a rendered artifact. Deliver it as a downloadable HTML file the requester can save locally or host on GitHub Pages, and verify it in your own execution environment rather than an artifact frame.
* Community identity is never hardcoded. Community name and community key live in the Settings Tab and populate the project header, the PDF, and every JSON export.
* The entire visual identity of both the interface and the PDF derives from one shared BRAND style block, defined once. The interface writes each value into a CSS custom property at load and the stylesheet carries no literal color, font, or spacing values of its own. The PDF generation reads the same object directly. One edit rebrands both. Ship the FM defaults given in the spec, navy 1B3A6B and gold C9A84C, with the status colors under their own key.

## The five build phases

### Phase 1: Foundation and data model

HTML shell, browser title, header showing the tool name and version 1.0. The shared BRAND block and the CSS custom property wiring. The local storage architecture: community scoped keys, atomic single write saves, schema version stamp, and a migration chain scaffold. The Settings Tab: community name and community key, the resource list with name, type, trades, priority ranking, and blocked dates, and the five finish option lists. The project creation flow with every header field the spec lists, including the editable unit available date. The full data schema for projects, steps, resources, settings, and baseline.

Verification gate: confirm against Settings Tab, Project Creation Flow, Branding Configuration, Browser Storage, JSON Backup schema version handling, and Resource Model. Confirm community values are configurable and appear nowhere as literals. Confirm resource names captured on a step are stored as entered and do not live reference the settings list.

### Phase 2: Scheduling engine and dependency logic

The working days model: Monday through Friday excluding the seven blocked holidays, with Good Friday computed through the Easter computus. Window inclusive of the start day. A successor starts on the first working day strictly after its predecessor end date. The full 18 step sequence with conditional toggles. The dependency graph with toggle off walk up, including the multi predecessor rule that substitutes the maximum end date across a passed over step's active resolved predecessors, applied recursively down to Demo. MEPFP rough in and MEPFP trim each as up to four independent trade lines, with the step level end date equal to the maximum across active trades and the step treated as off when no trade is active. Steps 7 and 9 restricted to vendor or sole proprietor with labor and materials columns disabled. Steps 15 and 16 carrying no cost. The material order lead time floor: start equals the maximum of the resolved predecessor end date and the vacancy notice date advanced by the entered lead time in working days. Baseline capture at Scheduled status. Live schedule recalculation only on completion events, completion date overrides or clears, and unit available date edits. The scheduling floor rule and the manual delay authority on every step.

Verification gate: run representative cases and confirm each by hand against the spec. A five working day window starting Monday ends that Friday. A step whose predecessor ends Friday starts the following Monday. A holiday inside a window flags the row and can be manually shortened. LVP with Drywall off and MEPFP rough in ending day 10 and Shower conversion ending day 14 resolves to day 14. An MEPFP step with mixed active trades reports the maximum trade end date. A lead time floor pushes a start past its dependency date when the lead time is longer. Confirm the live schedule does not drift on the lead time floor after go live.

### Phase 3: PDF output

The Unit Turn Request PDF generated through the inline bundled library, direct download, no browser print dialog. Header fields exactly as the spec lists, with date submitted set to the project creation date and unchanged on regeneration. The cost table with labor, materials, and vendor columns populated per the Form Category Mapping, including Paint carrying combined drywall and paint costs and Other carrying maintenance PM, fire protection trades, and anything uncategorized. Total request amount as the sum of all active category costs minus the custom costs paid by incoming resident, treating a blank as zero. Finish specifications appearing only for toggled on steps. The scope of work summary auto populated and editable, with the required narrative naming the Other line whenever Other carries a cost. Projected schedule completion date as the next working day after step 18. FM logo and letterhead and all fills, borders, fonts, and spacing drawn from the shared BRAND block. The generated filename following the convention in the spec, community code then UT Budget then care level and unit number then the parenthetical date, matching the pattern MV UT Budget - AL 19 - (06-22-2026).

Verification gate: generate a PDF from a representative project and confirm every header field, the filename string, the category rollups, the net total math, the conditional finish specs, and the Other narrative against PDF Contents, Form Category Mapping, and PDF Branding. Confirm the file downloads with no dialog and that generation and submission are unlinked, with no record of sending kept.

### Phase 4: Calendar and status display

The three views built from the single live schedule, switched by one toggle: By Unit, By Project, By Resource. Gantt bars rendered on calendar span, not working day span, with weekends and blocked holidays shown as visible gaps or muted segments aligned to a vertical today marker. Accordion expansion in By Unit and By Resource, one row open at a time. The status colors with their exact meanings, Yellow taking precedence over Green on a same day step, the red overdue day count, and Blue set only by checking a step complete. The manual status override with its distinct marker and its automatic clearing when a completion event supersedes it. The collapsed bar rollup showing the worst active non complete status. The procurement and scheduling checklist panel with an open item count, the auto created scheduling confirmation item on every resourced step, and the bar icon for unconfirmed items separate from the Gantt status color.

Verification gate: confirm against Calendar and Display View, Status Logic, Manual Status Override and Authority Order, Collapsed Bar Status Rollup, Procurement and Scheduling Checklist, and Visibility and Status. Confirm parallel branches surface rather than hide behind a healthy sibling, and confirm an overridden status never reads as genuinely healthy.

### Phase 5: Persistence and polish

JSON export and import in the upper right corner at all times, stamped with schema version, timestamp, and community identity. Import fully replaces local data, shows current and incoming community, timestamp, and project count, requires explicit confirmation, and blocks or hard warns on a community mismatch. Backup reminders on both a threshold trigger and a time trigger, both non blocking. The offboarding export as the handoff artifact. Export All Projects in the Settings Tab with the Include completed projects checkbox unchecked by default, generating the SheetJS workbook with one row per active step, the exact column order the spec lists, auto filter headers, direct download, and the filename pattern community key then Export then MM-DD-YYYY. The multi tab advisory warning. Then a full sweep of the entire specification end to end, fixing anything missed, and a final offline verification that the file runs with no network.

Verification gate: confirm against JSON Backup, Export All Projects, Data Persistence and Deployment, and Browser Storage, then walk the whole spec one final time and confirm nothing is unimplemented or contradicted. Confirm the tool runs correctly opened directly from disk with no connection.

## Engine correctness is the acceptance bar

The scheduling engine is where a wrong build looks right until it ships. Treat the worked examples in the spec as required passing tests, not illustrations. If any dependency walk up, holiday, weekend crossing, MEPFP trade rollup, lead time floor, baseline capture, or live recalculation case disagrees with the spec by even one day, it is a defect and you fix it before the phase closes.

## Final deliverable

One HTML file named clearly, containing the finished Unit Turn Schedule and Budget Tool version 1.0, fully self contained, offline capable, and usable out of the box. Present it as a downloadable file. Alongside it, give a short summary: the phases completed, the verification results, the library and structural decisions you made as the expert, and any spec silence you resolved with the answer you chose. Nothing about the tool is left for the requester to wire up or finish. Name the output file unit-turn-tool-v1.html and save it to the working directory.

