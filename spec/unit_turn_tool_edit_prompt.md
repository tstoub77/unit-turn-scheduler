# Unit Turn Scheduling Tool: Edit Prompt for Claude Code

You are working on an existing browser-based HTML tool. It is a single HTML file using browser local storage for persistence. No server, no login. The tool manages unit turn projects across senior living communities.

The working directory contains the full project: the existing HTML tool file and the working specification document (Unit Turn Scheduling Tool Spec Rev5). Read both before writing any code. The spec is the authoritative reference for intended behavior; the HTML file is the current build state. Where the two conflict, surface the discrepancy before resolving it. Do not restructure or rename anything not listed below. Apply only the changes described. Preserve all existing functionality not explicitly touched.

Work in phases. Complete each phase before beginning the next. After each phase, confirm what was done and what is next.

---

## Phase 0: Branding configuration (do this first, everything else depends on it)

The tool currently has organization name and brand colors scattered or hardcoded. This phase consolidates all of it into one place before any other changes are made.

**0.1 Single shared BRAND object**
Locate or create a single configuration object named BRAND at the top of the file, before any other logic. It must be the sole source of truth for every color, font, spacing value, and the organization name used anywhere in the tool interface or the PDF output. The object must match this structure exactly:

```javascript
const BRAND = {
  name: "Franciscan Ministries",
  logo: null,                   // set to embedded base64 string at build; null is acceptable for now
  colors: {
    primary: "#1B3A6B",         // navy: section headers, table header fills, primary borders
    accent:  "#C9A84C",         // gold: heading underlines, dividers, secondary accents
    text:    "#1A1A1A",
    surface: "#FFFFFF",
    muted:   "#6B7280"
  },
  status: {                     // fixed semantic meaning; not part of rebranding
    gray:   "#9CA3AF",
    green:  "#2F855A",
    yellow: "#D69E2E",
    red:    "#C53030",
    blue:   "#2B6CB0"
  },
  fonts: {
    base:    "Helvetica, Arial, sans-serif",
    heading: "Helvetica, Arial, sans-serif",
    sizeBase: 11,
    sizeHeading: 14
  },
  spacing: {
    unit: 4,
    sectionGap: 16,
    tablePadding: 6
  }
};
```

**0.2 Interface consumes BRAND via CSS custom properties**
At load time, write each BRAND value into a CSS custom property on the document root. The stylesheet must contain no literal color, font, or spacing values of its own. Every style rule must reference a CSS variable (for example `var(--color-primary)`). Audit the existing stylesheet and replace all hardcoded values.

**0.3 PDF generation consumes the same BRAND object**
Every fill, text color, font, and spacing value used in PDF generation must be read from the BRAND object directly. There must be no parallel color or font definitions in the PDF generation code.

**0.4 Organization name from BRAND.name**
Every place the organization name appears in the interface or the PDF must read from `BRAND.name`. No string literal of any organization name may appear anywhere else in the code.

**0.5 Verify the swap works**
After implementing, confirm that changing `BRAND.name` to a different string and `BRAND.colors.primary` and `BRAND.colors.accent` to two different hex values produces a correctly rebranded interface and PDF with no additional edits needed anywhere else in the file.

---

## Phase 1: UI cleanup and placeholder behavior

**1.1 Remove example placeholder text**
The opening screen shows "Marian Village" as a prefilled example. Remove it. No community name should be pre-populated anywhere. The field should be visually blank so the user can configure at will.

**1.2 Apartment type dropdown**
Replace the value "Home" with "Garden Home" in the Apartment Type dropdown. This is a label change only; update display text everywhere this value appears including any PDF output.

**1.3 Resource name prefill**
The Name field in the resource settings panel is prefilled with "New resource." The user must delete this before typing. Change it to placeholder text styled in gray that disappears on click or keypress. Do not prefill the input value.

**1.4 Version label**
Remove "Version 1.0" from the tool header. It should not appear anywhere in the interface.

**1.5 Organization name color in header**
In the header, render the organization name (from BRAND.name, per Phase 0) using BRAND.colors.accent. Do not use a hardcoded hex value here or anywhere else.

---

## Phase 2: Resource panel behavior

**2.1 Trades list as multi-select dropdown**
The Trades field under Add Resource is currently a text input or static list. Convert it to a dropdown with checkboxes so the user can select all that apply. The user should see a summary of selected trades when the dropdown is closed.

**2.2 Resource list collapse**
The resource list grows long. Collapse each resource entry to a single summary row showing Name, Type, and Priority Level only. Expand on click. Only one resource entry should be open at a time. Clicking a collapsed entry closes any currently open entry before opening the selected one.

**2.3 Add new resource placement**
Currently a new resource is added at the top via a button, but new entries appear at the bottom of the list. Fix this so the flow is consistent. Place a persistent "Add New Resource" row at the bottom of the list, always visible, styled distinctly (dashed border or muted row). The button at the top can remain if it scrolls to or opens a new entry at the bottom. New entries should appear immediately above the persistent Add New row.

**2.4 Multiple resource assignment per task**
Currently only one resource can be assigned to a task. Update the task resource assignment UI to allow multiple resources. Punchlist Repairs may need several trades assigned; Inspection may need more than one person. The task detail view should support selecting and displaying multiple assigned resources.

---

## Phase 3: Project setup and step behavior

**3.1 Vacancy notice date remains editable**
Saving the project header currently locks the Vacancy Notice Date field. Remove that lock. The field must remain editable after save.

**3.2 Step collapse behavior**
Each step in the project setup should collapse to a single summary row when not actively being worked. A toggled-off step shows one compact row. A toggled-on step expands to show full detail. Only the active step needs to be fully visible. This applies during project setup, not just during execution.

**3.3 Tab key behavior in task text fields**
Tab behaves unexpectedly when moving through text inputs while building tasks in a project. Audit and fix tab order so it moves logically through fields within a task without triggering unintended actions or focus jumps.

**3.4 Requires Material Order visibility**
The "Requires a Material Order" checkbox blends too much with the header below it and is easily missed. Make it stand out. One option is to display the checkbox label and its surrounding area in red or a red-tinted background when unchecked. Another option is a bold bordered callout box. Pick the approach that is most readable without being visually jarring. Apply it consistently to all task panels that include this field.

**3.5 Manual override default state**
The manual override section for each task currently opens by default. Change it so manual override is collapsed by default. Add a small toggle or checkbox labeled something like "Manual Override" that expands the section only when the user actively selects it.

---

## Phase 4: Step logic and countertop dependency

**4.1 Countertop dependency chain**
Countertop Installation must be tied to Countertop Measuring. The rules are:

- Cabinets must be installed before countertops can be measured
- Countertop Measuring must be completed before Countertop Installation can be scheduled
- Countertop Installation does not depend on Millwork Installation
- The two countertop steps must be scheduled separately but cannot exist independently

Enforce this in the dependency logic. Reorder step numbers if needed to reflect this chain correctly. Do not break any existing dependency rules for other steps.

**4.2 Countertop color option**
Move the Countertop Color selection field from the Countertop Installation step to the Countertop Measuring step. It belongs at measurement time, not installation time.

---

## Phase 5: Projects tab dashboard

**5.1 Add Completion Date column**
Add a Completion Date column to the Projects tab. This should display the date the project was marked complete. Leave blank for in-progress projects.

**5.2 Add Time to Turn column**
Add a Time to Turn column. This calculates the number of calendar days from Demo Day 1 through the Completion Date, inclusive of both days. Display only for completed projects.

**5.3 Add Total Project Cost column**
Add a Total Project Cost column showing the sum of all task and resource costs for the project. Display for all projects. This turns the Projects tab into a functional dashboard.

---

## Phase 6: Calendar tab improvements

**6.1 Date labels on timeline**
The calendar timeline does not display dates. Add date markers to the timeline axis so the user can orient tasks to actual dates. At minimum show week or day markers depending on the visible range.

**6.2 Calendar task link to project**
When a task on the calendar is yellow (approaching late) or red (overdue), the user currently must manually navigate to the correct project and task. Add click-through behavior: clicking a colored task on the calendar view opens or navigates to that specific task within its project. Manual navigation back to the calendar view is acceptable.

**6.3 Color cascade investigation**
When a single task is not on time, all tasks in the full project timeline view turn red or yellow, but the per-row expanded view shows correct colors. Determine whether this color cascade in the full timeline is intentional or a bug. If it is a bug, fix it so the full timeline reflects per-task status accurately. If it is by design, note it clearly in a code comment so future changes do not inadvertently alter it.

---

## Phase 7: PDF output

**7.1 Generate PDF button placement**
Move the Generate Unit Turn Request PDF button into the project header area. It should appear next to the Save Header Changes button. Display it only after the project status becomes "Scheduled." Style the button using BRAND.colors.accent.

**7.2 PDF layout restructure: two pages only**
Restructure the PDF so it fits cleanly on two pages with no page 3.

Page 1: Project header information and Finish Specifications.
Page 2: Cost Summary followed by Scope of Work Summary.

The Scope of Work Summary box on page 2 should be sized to fit below the cost summary without breaking to a third page. If content risks overflow, reduce font size or padding before allowing a third page.

**7.3 Cost summary column header alignment**
Format the Summary of Costs table so each column header is right-aligned above its corresponding dollar amount column. Dollar amounts should stack cleanly beneath their headers.

**7.4 Cost summary total order**
Within the Summary of Costs section, display in this order:
1. Summary of Costs subtotal
2. Custom Costs Paid by Resident (as a deduction)
3. Total Request Amount (bolded), calculated as subtotal minus custom resident costs

**7.5 Remove generated-by footer**
Remove the footer line that begins with "Generated by..." from all PDF pages.

**7.6 Apartment type label**
Confirm "Garden Home" from Phase 1.2 is reflected correctly in PDF output wherever Apartment Type appears.

---

## Standing constraints

- All colors, fonts, spacing, and the organization name live in the BRAND object established in Phase 0. No literal values anywhere else.
- No community names hardcoded anywhere in the tool. Community identity is configured in the Settings Tab per instance.
- Browser local storage is the only persistence layer.
- No server, no login, no external dependencies beyond what is already bundled.
- All changes must be backward compatible with data already saved in local storage. Do not alter existing storage keys or schema structure without a documented migration.

After completing all phases, do a final pass to confirm no regressions in existing functionality, particularly: project creation flow, scheduling engine, step dependency logic, and existing PDF generation.
