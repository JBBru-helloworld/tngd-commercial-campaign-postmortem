# Ignition Prompt: Run the TNGD Campaign Post-Mortem Engine

## ✏️ RUN CONFIGURATION — UPDATE THESE BEFORE EVERY RUN

CAMPAIGN NAME:     [UPDATE — e.g. 241122 COE RM107.5K Trip.com Launch Campaign]
MDR PERCENTAGE:    [UPDATE — e.g. 1.30%]
RFA FILE NAME:     [UPDATE — e.g. RFA-230752_Tripcom.pdf]
PARTNER DATA FILE: [UPDATE — e.g. Trip AMA Nov 2024.csv or Trip AMA Nov 2024.xlsx]

> These are the only four things you need to change each run.
> Everything below this line is fixed engine logic — do not edit it.

---

Use this prompt to run a new campaign post-mortem. The four run-configuration values above are the only fields that change between runs.

---

## ENGINE CONFIGURATION — FIXED, DO NOT EDIT

### A. Knowledge Base / Static Engine Sources
These are the engine reference files that the AI must read **first** before reading the run configuration or campaign-specific uploads. Update these names only if your stored engine files have been renamed.

- Master template workbook: `campaign_file_TEMPLATE.xlsx`
- Framework summary file: `framework_summary.md`
- Guardrails file: `guardrails.md`
- Instructions file: `instructions.md`
- Template schema file: `template_schema.md`
- Data validation rules file: `data_validation.md`
- Placeholder dictionary file: `placeholder_dictionary.md`
- Cell-to-source mapping file: `cell_to_source_mapping.md`
- Engine runbook file: `engine_runbook.md`
- File ingestion skill file: `file_ingestion_skill.md`

### B. Campaign-Specific Manual Inputs for This Run
These values are read from the RUN CONFIGURATION block at the top of this prompt.

- Campaign name / label: Read campaign name from the RUN CONFIGURATION block at the top of this prompt.
- MDR percentage for this run: Read MDR percentage from the RUN CONFIGURATION block at the top of this prompt.
- MDR source / approver note, if applicable: `[UPDATE_MDR_SOURCE_OR_NOTE]`

### C. Campaign-Specific Files for This Run (Phase 1 Required + Phase 2 Optional)
These are the uploaded files that will change every time this prompt is used.

- RFA / Sage approval file: Read RFA file name from the RUN CONFIGURATION block at the top of this prompt. — **[REQUIRED — Phase 1]**
- Partner raw data file(s): Read partner data file name from the RUN CONFIGURATION block at the top of this prompt. The file may be .csv, .xlsx, or .xls — detect the format from the file extension and read it with the appropriate method. If the extension is not .csv, .xlsx, or .xls, stop immediately and report under the ACCESS FAILURE RULE. — **[REQUIRED — Phase 1]**
- Internal validation / transaction data file(s): `[UPDATE_FILENAME_INTERNAL_VALIDATION_FILE_OR_FILES]` — **[OPTIONAL — Phase 2, human-completed]**
- Internal retention data file(s), if provided: `[UPDATE_FILENAME_RETENTION_FILE_OR_FILES]` — **[OPTIONAL — Phase 2, human-completed]**
- Finance rate / cost input file(s), if provided: `[UPDATE_FILENAME_FINANCE_FILE_OR_FILES]` — **[OPTIONAL — Phase 2, human-completed]**

---

## PRIMARY INSTRUCTION

Please initialize and run the **TNGD Campaign Post-Mortem Engine** for the campaign listed in the RUN CONFIGURATION block at the top of this prompt.

You must execute the work in the following order:

### Step 1 — Read the Knowledge Base / Static Engine Sources first
Before touching any campaign-specific data, read and absorb all of the static engine sources listed in Section A above.

These files are the permanent operating rules for the engine. Treat them as the source of truth for:
- workbook structure
- formatting preservation
- placeholder definitions
- cell-to-source logic
- rules-column interpretation
- validation rules
- reconciliation rules
- write-versus-keep-formula behavior
- final QA checks

The master template workbook must be treated as the design source of truth. Preserve the same:
- workbook structure
- worksheet names
- colours
- borders
- merged cells
- formulas
- notes
- layout
- overall visual design

Read the workbook tabs and embedded engine tabs first, including any helper, placeholder, mapping, or runbook tabs contained inside the workbook.

### Step 2 — Read the campaign-specific manual inputs from this prompt
After the static engine sources are fully read, capture the current-run values from the RUN CONFIGURATION block at the top of this prompt.

Rules:
- use the MDR percentage from the RUN CONFIGURATION block as the campaign-specific MDR input for this run
- do not infer, estimate, or reuse MDR from prior campaigns
- if MDR is missing from the RUN CONFIGURATION block and no approved supporting source explicitly provides it, mark the MDR field `MANUAL_INPUT_REQUIRED`
- if both a prompt-provided MDR and a file-provided MDR exist, validate the mismatch and follow the source hierarchy defined in the runbook

MDR CONVERSION RULE:
The MDR in the RUN CONFIGURATION block is written as a percentage (e.g. 1.30%).
Before writing it to the Excel file, convert it to a decimal by dividing by 100.
Example: 1.30% → write 0.013 to cell C22.
Never write the percentage string directly. Always write the decimal float.

### Step 3 — Read the campaign-specific uploaded files

MANDATORY READING METHOD:

Before writing any code to process the partner file, read file_ingestion_skill.md
from the knowledge base. The entire partner file ingestion must follow the method
defined in that skill file exactly.

The partner file must be read using Python + pandas via the read_partner_file()
function. Never attempt to open, browse, or preview the file directly.
Never attempt to read it as plain text or markdown.

Run check_deps() first to confirm all required libraries are installed.
Then follow Rules 1 through 10 in file_ingestion_skill.md in order.

After the static engine sources and manual inputs are fully read, read the campaign-specific uploaded files listed in Section C above.

**Only the RFA and partner raw file are required for Phase 1.** If Phase 2 files (internal validation data, retention data, finance rate files) are not uploaded, do not treat this as a blocking error. Mark all cells that depend on Phase 2 files `PENDING_HUMAN_VALIDATION` and continue the Phase 1 run.

Process uploaded files in this priority order:
1. RFA / Sage approval file — **[REQUIRED — Phase 1]**
2. Partner raw data file(s) — **[REQUIRED — Phase 1]**
3. Internal validation / transaction data file(s) — [OPTIONAL — Phase 2, human-completed]
4. Retention data file(s), if available — [OPTIONAL — Phase 2, human-completed]
5. Finance rate / cost file(s), if available — [OPTIONAL — Phase 2, human-completed]

### Step 4 — Apply the engine rules
When processing the campaign files:
- confirm the RFA approval status before using RFA values as approved source data
- use the RFA as source of truth for approved campaign identity and approved budget fields
- use internal validation / transaction data as source of truth for final reconciled performance metrics when it conflicts with partner raw data
- use partner raw data as supporting evidence and variance-check source
- use retention files only for actual observed retention values
- use finance input files only for finance-controlled fields
- use the current-run MDR from the RUN CONFIGURATION block to populate the workbook MDR input field unless an explicitly approved higher-priority override is defined
- preserve the template traffic-light rule tied to MDR:
  - **Green** if MDR >= 0.47%
  - **Yellow** if MDR >= 0.18% and MDR < 0.47%
  - **Red** if MDR < 0.18%

TRAFFIC-LIGHT FONT COLOUR RULE:
The font colour of cell C38 must match the traffic-light result.
After determining the MDR classification, apply the correct font colour:

  if MDR >= 0.0047  → font colour FF00B050  (Green)
  if MDR >= 0.0018  → font colour FFFFC000  (Yellow)
  if MDR <  0.0018  → font colour FFFF0000  (Red)
  if MDR missing    → font colour FFFF0000  (Red, default)

This must be applied at write time using openpyxl Font color.
Do not hardcode a static red font on C38.
The colour must reflect the actual MDR value for each run.

- obey all template rules, mapping rules, and validation rules from the knowledge base sources
- never redesign or simplify the workbook
- never overwrite cells that should remain formulas
- never estimate missing values
- mark unresolved values clearly using `DATA_NOT_FOUND` or `MANUAL_INPUT_REQUIRED`, whichever is appropriate under the engine rules
- clearly flag discrepancies between partner-reported values and internal values

### Step 5 — Produce the final outputs
Your output must be exactly **2 things only**:

#### Output 1 — Partially completed Excel file (Phase 1 output)
Create and return the **partially completed Excel file** for this campaign, ready for Phase 2 human validation.

Requirements:
- Phase 1 sections must be fully populated: campaign identity block (from RFA), provisional partner KPIs (from CHARGE rows), MDR rate (from ignition prompt), and traffic-light classification
- All Phase 2 sections must be clearly marked `PENDING_HUMAN_VALIDATION` for the human reviewer
- it must match the master template design exactly
- it must preserve the same formatting, colours, formulas, structure, and worksheet design
- it must preserve formula-driven cells wherever the template requires formulas to remain intact

#### Output 2 — Summary in chat
Provide a concise but complete summary in chat that includes:
- campaign identification details used
- files successfully read
- MDR percentage used for the run
- whether MDR came from the prompt, a supporting file, or both
- traffic-light result
- major findings
- reconciled KPI observations
- discrepancies found between partner and internal data
- missing files
- incomplete fields
- unresolved items requiring manual input or review
- any important validation warnings

Do not output a separate third deliverable. The response must only consist of:
1. the completed Excel file
2. the chat summary

---

## EXECUTION GUARDRAILS

- Do not use sample values from template examples as actual campaign results.
- Do not carry over values from prior campaigns.
- Do not guess or backfill missing fields without a valid source.
- Do not change row order, headers, formulas, worksheet names, workbook structure, or the rules / notes logic.
- Do not overwrite the template traffic-light formula if the workbook already derives it from MDR.
- Do not include PII in the final workbook summary output unless the template explicitly requires a non-PII business identifier.
- If a required file is missing, continue where possible, but clearly flag the gap in the chat summary and in the workbook where applicable under engine rules.
- If conflicting values are found, prioritize internal validated data for final KPI outputs and log the variance in the summary.
- If MDR is missing or conflicting, explicitly flag it in the chat summary.

### Excel Population Guardrails

These rules apply to the Python script that populates the Excel output file.
They must be followed exactly. Any deviation will produce a blank or corrupted file.

- Build the output from scratch using `Workbook()`. Do not open any template file.
  Never use `openpyxl.load_workbook()` anywhere in the script.
- Never call `SpreadsheetArtifact`, `artifact_tool`, `artifact.recalculate()`,
  `artifact.export()`, or `artifact.render()`. These destroy the output file.
- Never delete any sheet from the workbook.
- Never set fill, font, border, number_format, or alignment on any cell.
- Always write to the top-left anchor cell of merged ranges only.
- Always check `is_formula()` before writing. Never overwrite a formula cell.
- Write MDR as a plain decimal (e.g. `0.013`), never as a percentage string.
- Call `wb.save(output_path)` once. Print the path. End the script.

---

## START OF RUN

Campaign to process: Read campaign name from the RUN CONFIGURATION block at the top of this prompt.
MDR for this run: Read MDR percentage from the RUN CONFIGURATION block at the top of this prompt.

Now read all Section A knowledge base files first, then Section B run inputs, then Section C campaign files, then generate the 2 required outputs.
