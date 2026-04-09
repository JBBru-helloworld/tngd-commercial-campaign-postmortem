# Ignition Prompt: Run the TNGD Campaign Post-Mortem Engine

Use this prompt to run a new campaign post-mortem. Update the run-configuration values near the top before each run.

---

## RUN CONFIGURATION — UPDATE THESE VALUES BEFORE EACH RUN

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

### B. Campaign-Specific Manual Inputs for This Run
These values must be updated for each campaign run.

- Campaign name / label: `[UPDATE_CAMPAIGN_NAME]`
- MDR percentage for this run: `[UPDATE_MDR_PERCENTAGE]`
- MDR source / approver note, if applicable: `[UPDATE_MDR_SOURCE_OR_NOTE]`

### C. Campaign-Specific Files for This Run
These are the uploaded files that will change every time this prompt is used.

- RFA / Sage approval file: `[UPDATE_FILENAME_RFA_FILE]`
- Internal validation / transaction data file(s): `[UPDATE_FILENAME_INTERNAL_VALIDATION_FILE_OR_FILES]`
- Partner raw data file(s): `[UPDATE_FILENAME_PARTNER_RAW_FILE_OR_FILES]`
- Internal retention data file(s), if provided: `[UPDATE_FILENAME_RETENTION_FILE_OR_FILES]`
- Finance rate / cost input file(s), if provided: `[UPDATE_FILENAME_FINANCE_FILE_OR_FILES]`

---

## PRIMARY INSTRUCTION

Please initialize and run the **TNGD Campaign Post-Mortem Engine** for the campaign listed above.

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
After the static engine sources are fully read, capture the current-run values from Section B above.

Rules:
- use the MDR percentage from Section B as the campaign-specific MDR input for this run
- do not infer, estimate, or reuse MDR from prior campaigns
- if MDR is missing from Section B and no approved supporting source explicitly provides it, mark the MDR field `MANUAL_INPUT_REQUIRED`
- if both a prompt-provided MDR and a file-provided MDR exist, validate the mismatch and follow the source hierarchy defined in the runbook

### Step 3 — Read the campaign-specific uploaded files
After the static engine sources and manual inputs are fully read, read the campaign-specific uploaded files listed in Section C above.

At minimum, process these files in this priority order:
1. RFA / Sage approval file
2. Internal validation / transaction data file(s)
3. Partner raw data file(s)
4. Retention data file(s), if available
5. Finance rate / cost file(s), if available

### Step 4 — Apply the engine rules
When processing the campaign files:
- confirm the RFA approval status before using RFA values as approved source data
- use the RFA as source of truth for approved campaign identity and approved budget fields
- use internal validation / transaction data as source of truth for final reconciled performance metrics when it conflicts with partner raw data
- use partner raw data as supporting evidence and variance-check source
- use retention files only for actual observed retention values
- use finance input files only for finance-controlled fields
- use the current-run MDR from Section B to populate the workbook MDR input field unless an explicitly approved higher-priority override is defined
- preserve the template traffic-light rule tied to MDR:
  - **Green** if MDR >= 0.47%
  - **Yellow** if MDR >= 0.18% and MDR < 0.47%
  - **Red** if MDR < 0.18%
- obey all template rules, mapping rules, and validation rules from the knowledge base sources
- never redesign or simplify the workbook
- never overwrite cells that should remain formulas
- never estimate missing values
- mark unresolved values clearly using `DATA_NOT_FOUND` or `MANUAL_INPUT_REQUIRED`, whichever is appropriate under the engine rules
- clearly flag discrepancies between partner-reported values and internal values

### Step 5 — Produce the final outputs
Your output must be exactly **2 things only**:

#### Output 1 — Completed Excel file
Create and return the **full completed Excel file** populated for this campaign.

Requirements:
- it must match the master template design exactly
- it must preserve the same formatting, colours, formulas, structure, and worksheet design
- it must be the completed workbook output for the current campaign
- it must reflect all populated fields that can be determined from the provided files and prompt inputs
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

---

## START OF RUN

Campaign to process: `[UPDATE_CAMPAIGN_NAME]`  
MDR for this run: `[UPDATE_MDR_PERCENTAGE]`

Now read all Section A knowledge base files first, then Section B run inputs, then Section C campaign files, then generate the 2 required outputs.