# Unit Turn Schedule and Budget Tool

A self-contained, offline-capable HTML tool for scheduling and budgeting apartment/unit turns. No server, no login, no build step, and no network calls once loaded — every library it needs (jsPDF, jsPDF AutoTable, SheetJS) is bundled inline. All project and settings data is stored in the browser's local storage, scoped per community.

## Running it

Open `index.html` directly in a browser, or serve this repo with GitHub Pages and visit the published URL. Either way works offline; nothing is fetched over the network.

## Repository structure

- `index.html` — the tool itself. Single file, self-contained.
- `spec/` — the governing specification and build/edit prompts used to produce the tool:
  - `Unit_Turn_Scheduling_Tool_Spec_Rev5_updated.docx` — source-of-truth spec
  - `Unit_Turn_Tool_Build_Prompt_v1.md` — original build prompt (v1.0)
  - `unit_turn_tool_edit_prompt.md`, `unit_turn_tool_edit_prompt_v2.md`, `unit_turn_tool_edit_prompt_v3.md` — follow-up edit prompts
- `samples/` — example generated output (e.g. a sample Unit Turn Request PDF)

## Data and branding

Community identity (name, key, resources, finish options) is configured from the Settings tab inside the running tool — never hardcoded. Visual branding (colors, fonts, spacing) is driven from a single shared `BRAND` block in `index.html`; both the on-screen UI and generated PDFs read from it.
