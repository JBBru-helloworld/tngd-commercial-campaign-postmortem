# Final Engine Runbook

Use this runbook to execute and populate the campaign-agnostic TNGD Campaign Working File template.

## Read order and source hierarchy

The engine must always read inputs in this order:

1. **Knowledge Base / Static Engine Sources first**
   - master template workbook
   - guardrails
   - instructions
   - template schema
   - data validation rules
   - placeholder dictionary
   - cell-to-source mapping
   - engine runbook
   - framework summary
2. **Ignition Prompt Run Configuration second**
   - campaign name / run label
   - MDR percentage for the run
   - MDR source / approver note, if supplied
   - campaign-specific file names for the current run
3. **Campaign-Specific Uploaded Files third**
   - RFA / Sage approval file — **Phase 1 Required**
   - partner raw data file(s) — **Phase 1 Required**
   - internal validation / transaction data file(s) — Phase 2 (human-completed)
   - retention data file(s), if provided — Phase 2 (human-completed)
   - finance rate / cost input file(s), if provided — Phase 2 (human-completed)

The master template workbook is the design source of truth. Preserve the same workbook structure, sheet names, colours, borders, merged cells, formulas, notes, layout, and overall visual design.

## Two-Phase Workflow Overview

The engine runs in two distinct phases. The AI executes Phase 1 only. Phase 2 is completed by a human team member.

### Phase 1 — AI Run
- **Inputs:** RFA file + partner raw data file + ignition prompt parameters (campaign name, MDR).
- **What the AI does:** Populates all fields derivable from the RFA (campaign identity block) and partner raw data (provisional KPIs), writes the MDR, evaluates the traffic-light classification, and writes `PENDING_HUMAN_VALIDATION` to all Phase 2 fields.
- **Output:** A partially completed Excel workbook ready for Phase 2 human validation.

### Phase 2 — Human Validation
- **Who:** A team member receives the Phase 1 output Excel file.
- **What they do:** Validates the partner file against internal database files and fills in all remaining sections: internal KPI reconciliation, internal TPV / participants / transaction counts, all retention data, all finance cost rates, and all merchant-impact / BI data.
- **Inputs for Phase 2:** Internal TXN data, internal retention files, finance rate files, BI / merchant-impact dataset.

### AI Behavior at Phase Boundary
- The AI must **never** block, error, or stop because internal data, retention files, finance files, or BI files are absent.
- If a Phase 2 file is missing, the AI writes `PENDING_HUMAN_VALIDATION` to all cells that depend on that file and continues.
- `PENDING_HUMAN_VALIDATION` signals to the human reviewer that the field is expected to be filled — it is not a missing-data error.
- The AI must **only** stop and return `MANUAL_REVIEW_REQUIRED` if the Phase 1 required files (RFA or partner raw data) are missing or if the RFA is not in Approved status.

## Core execution rules

- Duplicate the master template before population.
- Read the static engine sources before touching the campaign-specific uploads.
- Read the workbook's embedded **Placeholder Dictionary**, **Cell-to-Source Mapping**, and **Engine Runbook** tabs before writing any output cells.
- Register the current-run MDR from the ignition prompt before populating the workbook.
- Only write into cells marked `WRITE_INPUT` or other explicit input actions.
- Never overwrite cells marked `KEEP_FORMULA`.
- Never estimate missing values. Use `DATA_NOT_FOUND` or `MANUAL_INPUT_REQUIRED` for genuinely unresolvable Phase 1 fields. Use `PENDING_HUMAN_VALIDATION` for all Phase 2 fields.
- Use the RFA as the source of truth for approved campaign identity and approved budget fields, only after confirming approval status.
- Internal verification / TXN data is the final KPI source when it conflicts with partner raw data — but this reconciliation is a Phase 2 step.
- Use the prompt-supplied MDR unless an approved higher-priority finance / BU override is explicitly defined for the run.
- Keep all template colours, merges, labels, notes, and formulas unchanged.
- Do not carry over values from prior campaigns or from example/template content.

## Step-by-step workflow

### Step 0 - Create working copy
**Do:** Duplicate the master template before any population. Keep all sheet names, merges, formulas, colours, and notes intact.
**Do not:** Do not write into the original master template or redesign the workbook.
**Checkpoint:** New output workbook created.
**Guardrail:** Template structure is immutable.

### Step 1 - Read static engine sources first
**Do:** Read all static engine sources before processing the campaign uploads. Treat them as the permanent operating rules for structure, formatting, placeholder semantics, mapping logic, rules-column interpretation, validation, reconciliation, write-versus-keep-formula behavior, and final QA.
**Do not:** Do not start extracting campaign values before the engine sources are fully read.
**Checkpoint:** Static engine rule set understood.
**Guardrail:** The knowledge base files define how the run must be executed.

### Step 2 - Register ignition prompt run parameters
**Do:** Capture the campaign name / run label, MDR percentage, MDR source / approver note, and the campaign-specific source file names listed in the ignition prompt.
**Do not:** Do not assume the MDR from prior runs. Do not treat a missing MDR as optional if the workbook depends on it.
**Checkpoint:** Run-parameter inventory completed.
**Guardrail:** MDR is a campaign-specific controlled input.

### Step 3 - Register campaign-specific uploads
**Phase 1 required files (AI run):** RFA / Sage approval file and partner raw data file. These are the only files required for Phase 1. If either is missing, stop and return `MANUAL_REVIEW_REQUIRED`.
**Phase 2 files (human-completed):** Internal verification / TXN data, retention data, and finance rate source files. If these files are not uploaded, do not treat this as a blocking error. Mark all cells that depend on these files `PENDING_HUMAN_VALIDATION` and continue the Phase 1 run.
**Do:** Register any uploaded files in the Source File Template and in the validation log. Note Phase 2 files as "to be completed in Phase 2."
**Do not:** Do not flag the absence of Phase 2 files as an error. Do not infer a source from memory.
**Checkpoint:** Source inventory completed with Phase 1 / Phase 2 status noted.
**Guardrail:** Only Phase 1 files are blocking. Phase 2 files are optional for the AI run.

### Step 4 - Open mapping assets before writing cells
**Do:** Read the Placeholder Dictionary and Cell-to-Source Mapping assets before touching any output cells. Use the Write Action field to decide whether a target cell must be populated, preserved as a formula, or flagged for manual handling.
**Do not:** Do not overwrite any `KEEP_FORMULA` target.
**Checkpoint:** Population plan established.
**Guardrail:** The mapping sheet is the write contract and the placeholder dictionary is the semantic contract.

### Step 5 - Validate the RFA gate
**Do:** Extract campaign identity fields from the RFA only if the document status is Approved. Populate campaign name, RFA ID, dates, mechanics, amount, funding mechanism, and cost centre per the mapping sheet.
**Do not:** Do not populate the RFA ID from an unapproved RFA. Do not let sample values remain if the RFA provides real values.
**Checkpoint:** General-info block populated.
**Guardrail:** Approved RFA is the source of truth for identity and approved budget.

### Step 6 - Normalize partner raw data (Phase 1)
**Do:** Map the partner raw file into canonical fields: qualified transactions, qualified users, TPV, refunds, mechanics / waves, and campaign date coverage. Produce provisional KPI values (campaign participants, TPV, transaction count) from the partner CHARGE rows. Label these as provisional (partner-derived) in the output. Preserve raw partner totals for later variance checks by the human reviewer.
**Do not:** Do not mix refunds into the primary KPI block unless a finance-approved net rule exists.
**Checkpoint:** Partner staging summary completed. Provisional KPIs populated in Phase 1 cells.
**Guardrail:** Partner-derived KPIs are provisional. Internal verification (Phase 2) is the final source of truth.

### Step 7 - Normalize internal verification data (Phase 2 — human-completed)
**This step is a Phase 2 human-completed step. The AI must not attempt to perform this step.**
**Do:** Leave all cells in the internal KPI reconciliation section marked `PENDING_HUMAN_VALIDATION`. Include a note that the team member will populate these from the internal database after the AI run.
**Do not:** Do not populate internal KPI cells from partner data as a substitute. Do not silently prefer partner data over internal data.
**Checkpoint:** Internal KPI cells written as `PENDING_HUMAN_VALIDATION`.
**Guardrail:** Internal data normalization is reserved for Phase 2.

### Step 8 - Reconcile discrepancies (Phase 2 — human-completed)
**This step is a Phase 2 human-completed step. The AI must not attempt to perform this step.**
**Do:** Leave all KPI reconciliation fields (TPV variance, transaction variance, user variance) marked `PENDING_HUMAN_VALIDATION`. Include a note that the human reviewer will compare partner versus internal KPI values and log variances.
**Do not:** Do not average the two sources or choose whichever looks better. Do not flag the absence of internal data as an error.
**Checkpoint:** KPI reconciliation cells written as `PENDING_HUMAN_VALIDATION`.
**Guardrail:** KPI reconciliation is reserved for Phase 2.

### Step 9 - Populate retention (Phase 2 — human-completed)
**This step is a Phase 2 human-completed step. The AI must not attempt to perform this step.**
**Do:** Leave all retention fields marked `PENDING_HUMAN_VALIDATION`. Include a note that the team member will populate these from the internal retention source aligned to the campaign cohort and post-campaign month buckets.
**Do not:** Do not use assumed rates from the RFA as actual retention. Do not flag missing retention files as an error during Phase 1.
**Checkpoint:** All retention cells written as `PENDING_HUMAN_VALIDATION`.
**Guardrail:** Retention population is reserved for Phase 2.

### Step 10 - Populate MDR and finance inputs
**Phase 1 (AI populates):** Write the campaign-specific MDR input from the ignition prompt into the workbook MDR field. If CPAM Cost is available in the partner file, populate it; otherwise write `PENDING_HUMAN_VALIDATION`.
**Phase 2 (human-completed):** Finance cost-rate inputs — Avg Reload Cost %, Avg Cloud Cost/Txn, and Avg PLSA Cost/Txn — are Phase 2 human inputs. The AI must write `PENDING_HUMAN_VALIDATION` to these fields. Do not infer or estimate these rates.
**Do not:** Do not copy sample finance values from the workbook. Do not overwrite the traffic-light cell if it is formula-driven.
**Checkpoint:** MDR written from ignition prompt. Finance rate fields written as `PENDING_HUMAN_VALIDATION`.
**Guardrail:** Finance-controlled rate fields remain Phase 2 human inputs only.

### Step 11 - Apply the MDR traffic-light rule
**Do:** Preserve the template logic for the traffic-light row. The result must follow this rule:
- Green if MDR >= 0.47%
- Yellow if MDR >= 0.18% and MDR < 0.47%
- Red if MDR < 0.18%

**Do not:** Do not replace the formula with hardcoded text if the template already derives the result.
**Checkpoint:** Traffic-light field resolved or appropriately flagged.
**Guardrail:** The template rule must be preserved exactly.

### Step 12 - Populate merchant-impact analysis (Phase 2 — human-completed)
**This step is a Phase 2 human-completed step. The AI must not attempt to perform this step.**
**Do:** Write `PENDING_HUMAN_VALIDATION` to all merchant-impact fields (TPV series, MTU series, growth values, CLTV, second-review decision).
**Do not:** Do not invent baseline growth conventions. Do not flag missing BI data as an error during Phase 1.
**Checkpoint:** All merchant-impact cells written as `PENDING_HUMAN_VALIDATION`.
**Guardrail:** Merchant-impact data requires Phase 2 BI / internal database pulls.

### Step 13 - Leave formulas intact
**Do:** For revenue, total cost, direct cost, gross profit, ROI, CPAM cost per user, and helper-derived deltas, keep the workbook formulas exactly as-is. Only populate their dependency cells.
**Do not:** Do not replace formulas with hardcoded numbers.
**Checkpoint:** Derived rows calculate automatically.
**Guardrail:** Formula cells are part of the template logic.

### Step 14 - Write narratives and decisions last
**Do:** After all Phase 1 numeric sections are validated, write concise narrative summaries where derivable from Phase 1 data. Leave Phase 2 narrative fields marked `PENDING_HUMAN_VALIDATION`.
**Do not:** Do not let narratives contradict the numbers. Do not auto-approve 2nd review without a rule or confirmation.
**Checkpoint:** Phase 1 narrative fields completed where possible.
**Guardrail:** Narratives must be evidence-based.

### Step 15 - Quality assurance and export
**Do:** Run final checks: no PII, Phase 1 fields fully populated, all Phase 2 fields clearly marked `PENDING_HUMAN_VALIDATION`, no broken formulas, date alignment validated, MDR and traffic-light rule validated, and all genuinely unresolved Phase 1 items marked `DATA_NOT_FOUND` or `MANUAL_INPUT_REQUIRED`.
**Do not:** Do not ship a workbook that contains guessed values or overwritten formula cells.
**Checkpoint:** Workbook ready for Phase 2 handoff.
**Guardrail:** Zero-hallucination policy applies to every field.

## Final required outputs

The engine must return exactly **2 outputs only**:

1. **Partially completed Excel file (Phase 1 output)**
   - Partially completed workbook ready for Phase 2 human validation
   - Phase 1 sections (campaign identity, provisional partner KPIs, MDR, traffic light) fully populated
   - All Phase 2 sections clearly marked `PENDING_HUMAN_VALIDATION` for the human reviewer
   - Matches the master template design exactly
   - Preserves formatting, colours, formulas, structure, and worksheet design

2. **Summary in chat**
   - campaign identification details used
   - files successfully read
   - MDR percentage used
   - traffic-light result
   - major findings
   - provisional partner KPI values populated in Phase 1
   - Phase 2 fields awaiting human validation (listed clearly)
   - missing Phase 1 files (if any — these are blocking)
   - incomplete Phase 1 fields
   - unresolved items requiring manual input or review
   - validation warnings

Do not produce a third deliverable.

## Final QA checklist
- Static engine sources read before run parameters and campaign-specific files.
- Prompt MDR captured before workbook population.
- RFA status checked and approved before using the RFA ID.
- Source inventory registered in the helper / source area with Phase 1 / Phase 2 status noted.
- Phase 1 completeness confirmed: campaign identity block fully populated from RFA, provisional partner KPIs populated from CHARGE rows, MDR written from ignition prompt, traffic-light classification resolved.
- All Phase 2 fields confirmed as `PENDING_HUMAN_VALIDATION` (not blank, not `DATA_NOT_FOUND`).
- Required Phase 1 missing sources explicitly flagged (RFA or partner file absence is a blocking stop).
- Retention populated only from observed retention data (Phase 2 human step — must be marked `PENDING_HUMAN_VALIDATION`).
- MDR and finance inputs sourced only from approved inputs; finance rate fields marked `PENDING_HUMAN_VALIDATION`.
- Traffic-light rule checked against MDR.
- Formula cells preserved.
- Narrative cells checked against numbers.
- No PII or row-level IDs in the final output.
- Only 2 deliverables returned: Phase 1 partially completed workbook and chat summary.

---

## Excel Population Technical Runbook

This section gives the AI step-by-step Python/openpyxl instructions for reading the template file and writing the Phase 1 output.

### Required Library

```
Use the openpyxl library exclusively for reading and writing the Excel template.
Do not use xlrd, xlwt, xlsxwriter, or pandas to_excel for the final output file.
Install with: pip install openpyxl
```

### How to Open the Template

```python
import openpyxl

wb = openpyxl.load_workbook("campaign_file_TEMPLATE.xlsx")

# The only sheet you need to populate is the Campaign Template sheet.
# Access it by name:
ws = wb["Campaign Template"]

# Do not access, modify, or delete any other sheet.
# If other sheets do not exist in the file, that is acceptable — work only with Campaign Template.
```

### How to Read Existing Cell Values Before Writing

```python
# Before writing any value, read the cell to check if it contains a formula.
# Cells that begin with "=" are formula cells and must NEVER be overwritten.

def is_formula(cell):
    val = cell.value
    return isinstance(val, str) and val.startswith("=")

# Example check before writing:
cell = ws["C14"]
if not is_formula(cell):
    ws["C14"] = your_value
else:
    # Leave the cell alone. Log it as a preserved formula cell.
    pass
```

### How to Handle Merged Cells

```python
# Many cells in the template are merged (e.g., C2:H2).
# When writing to a merged range, always write to the TOP-LEFT cell of the merged range only.
# Do not iterate over the full merge range — openpyxl will raise an error if you write to a non-anchor merged cell.

# Example: to write the Campaign Name into C2:H2, write only to C2.
ws["C2"] = "My Campaign Name"

# To confirm a cell is the anchor of a merge, you can inspect merged_cells:
for merged_range in ws.merged_cells.ranges:
    print(merged_range)  # e.g., C2:H2
```

### Cell-by-Cell Write Instructions for Phase 1

Write ONLY the following cells during a Phase 1 run. Reference the exact cell addresses from the Cell-to-Source Mapping. For each cell, apply the formula-check from the section above before writing.

```
PHASE 1 WRITE TARGETS — Campaign Template sheet only:

General Info block:
  C2  — CAMPAIGN_NAME          (string, from RFA)
  C3  — RFA_ID                 (string, from RFA, only if Approved)
  C4  — PARTNER_NAME           (string, from RFA)
  C5  — CAMPAIGN_TYPE          (string, from RFA or MANUAL_INPUT_REQUIRED)
  C7  — CAMPAIGN_START_DATE    (date string, from RFA)
  F7  — CAMPAIGN_END_DATE      (date string, from RFA)
  C8  — FUNDING_MECHANISM      (string, from RFA)
  C9  — MECHANIC_PRIMARY       (string, from RFA)
  C11 — MECHANIC_SECONDARY     (string, from RFA or blank)
  C12 — RFA_AMOUNT_RM          (number, from RFA)
  C13 — COST_CENTRE            (string, from RFA)

Core KPIs block (partner-derived, provisional):
  C14 — CAMPAIGN_PARTICIPANTS  (integer, from partner CHARGE rows)
  C15 — CAMPAIGN_TPV_RM        (number, from partner CHARGE rows)
  C16 — CAMPAIGN_TXN_COUNT     (integer, from partner CHARGE rows)

MDR and Revenue:
  C22 — MDR_RATE               (decimal, e.g. 0.0047 for 0.47%, from ignition prompt)
  C24 — CPAM_COST_RM           (number, from partner file if available, else PENDING_HUMAN_VALIDATION)

Phase 2 placeholder writes — write the string "PENDING_HUMAN_VALIDATION" to these cells:
  C18, D18, E18, F18, G18, H18  — Retention Post 1M–6M
  C20, D20, E20, F20, G20, H20  — Retention Post 7M–12M
  C26  — AVG_RELOAD_PCT
  C27  — AVG_CLOUD_COST_RM
  C28  — AVG_PLSA_COST_RM
  C34, D34, E34, F34, G34, H34  — Merchant TPV values
  C35, D35, E35, F35, G35, H35  — Merchant TPV growth values
  C37, D37, E37, F37, G37, H37  — MTU values
  C43  — EST_12M_CLTV
  C41  — SECOND_REVIEW_DECISION

Do not write to any KEEP_FORMULA cells:
  C21, C23, C25, C29, C30, C38, C42
  (and any other cell that begins with "=" when read)
```

### How to Save the Output File

```python
# Always save to a new file. Never overwrite the original template.
output_filename = "campaign_postmortem_[CAMPAIGN_NAME]_phase1.xlsx"
wb.save(output_filename)
print(f"Saved: {output_filename}")
```

### Error Handling Rules

```
1. If openpyxl cannot open the template file, stop immediately and return:
   "ERROR: Template file could not be opened. Confirm the file name matches exactly
   and the file is a valid .xlsx format."

2. If the "Campaign Template" sheet is not found in the workbook:
   - List all available sheet names using wb.sheetnames.
   - Return: "ERROR: Sheet 'Campaign Template' not found. Available sheets: [list].
     Please confirm the correct sheet name and update the cell-to-source mapping."
   - Do not attempt to infer or guess the sheet name.

3. If a write operation fails on a specific cell:
   - Log the cell address and the error.
   - Skip that cell and continue with the remaining cells.
   - Include all failed cells in the chat summary under "Write Errors."

4. If a merged cell conflict occurs (writing to a non-anchor cell):
   - Identify the top-left anchor of the merged range.
   - Retry the write to the anchor cell only.
   - Log the correction in the chat summary.

5. Never raise an unhandled exception that stops the entire run.
   Always complete the save step and produce the output file even if some cells failed to write.
```

### Scope Restriction — Other Tabs

```
The AI engine must only interact with the "Campaign Template" sheet.

Do not read from, write to, or modify any other sheet in the workbook
(e.g., "Source File Template", helper tabs, or any embedded engine tabs).

If the template references other sheets in its formulas (e.g., =SourceFileTemplate!R12),
leave those formulas intact and do not attempt to populate the referenced cells in those
other sheets. The human validator will manage cross-sheet references in Phase 2.

This restriction simplifies Phase 1 execution and prevents accidental corruption of helper sheets.
```
