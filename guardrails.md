# Guardrails: TNGD Campaign Post-Mortem Engine

## Engine Mode
This framework is **campaign agnostic**.
- Do **not** hardcode any campaign name, merchant name, date, budget, TPV, participant count, funding model, MDR, or sample KPI from prior files.
- Treat any values seen during framework design as **sample data only**.
- Only compute actual values during a future **execution run** when campaign-specific source files and run parameters are supplied.

## 1. Source-of-Truth Hierarchy
1. **Static engine sources** for workbook structure, formula preservation, mapping rules, rules-column interpretation, and validation logic.
2. **Ignition prompt run parameters** for campaign-specific manual inputs explicitly supplied for the current run, including MDR if provided there.
3. **Approved RFA (Sage)** for campaign identity, approved budget, official mechanics, stated objectives, and approved campaign date range.
4. **Internal transaction / retention files** for KPI reconciliation, TPV verification, retention, and any value used for financial validation.
5. **Partner data file** for campaign performance extraction — used after Phase 1 human validation has been completed.
6. **Campaign Working File template** for structure, formulas, merged-cell layout, labels, and cell-level business rules.

If two sources disagree:
- Use the **Internal Data File** as the final KPI source.
- Use the **prompt-supplied MDR** unless an approved finance / BU source is explicitly designated as the higher-priority override for that run.
- Preserve the conflicting value in a **Validation Note**.
- Never silently overwrite a discrepancy.

## Phase Boundary Rule

The engine operates in two distinct phases. The AI executes **Phase 2 only**.

### Phase 1 — Human-Only Fields
A team member completes these fields **before** the AI runs:
- Internal KPI reconciliation (internal TPV, internal participants, internal transaction count, all variance cells)
- All retention cells (post-month 1 through 12)
- All finance cost-rate inputs (Avg Reload Cost %, Avg Cloud Cost/Txn, Avg PLSA Cost/Txn)
- Merchant-impact TPV series, MTU series, and growth values
- CLTV (Est. 12M CLTV)
- Second-review decision
- Any field that depends on internal database records, BI pulls, or finance rate workbooks

### Phase 2 — AI-Only Fields
The AI populates these fields during a Phase 2 run:
- Campaign identity fields from the RFA (name, ID, dates, mechanics, amount, funding mechanism, cost centre)
- Validated KPIs from the partner data file (campaign participants, TPV, transaction count) — treated as final validated values
- MDR rate (from ignition prompt)
- CPAM Cost (from partner data file if present; otherwise `MANUAL_INPUT_REQUIRED`)
- Traffic-light classification (derived from MDR)
- All formula-driven cells

**KPIs written by the AI in Phase 2 are treated as final validated values because Phase 1 human validation has already been completed.**

### AI Behavior at the Phase Boundary
- The AI must **never** substitute estimated values for Phase 1 fields.
- Use `MANUAL_INPUT_REQUIRED` for any Phase 1 field not present in the validated partner data file. This signals to the human reviewer that the field requires manual follow-up — not that the data is permanently unavailable.
- Do not treat absent Phase 1 files as blocking errors. Only the RFA and validated partner data file are required for Phase 2.

## 2. Mandatory Stop / Flag Conditions
Return `DATA_NOT_FOUND` or `MANUAL_REVIEW_REQUIRED` when any of the following apply:
- RFA status is not explicitly **Approved**.
- A Phase 2 required source file is missing (RFA or validated partner data file). If either Phase 2 file is absent, stop and return `MANUAL_REVIEW_REQUIRED`. Phase 1 files (internal TXN data, retention data, finance rate files) are human-completed tasks — if their outputs are not present in the validated partner data file, mark dependent cells `MANUAL_INPUT_REQUIRED` and continue.
- Campaign dates in source files do not align and no BU override is provided.
- Funding mechanism conflicts across trusted sources and no BU confirmation is provided.
- MDR is required but missing from both the prompt run configuration and any approved supporting source.
- Template logic depends on an external workbook that has not been uploaded.

## 3. Transaction Handling Rules
- Default campaign KPI extraction to **successful campaign charges only** unless finance explicitly requests net-of-refund reporting.
- Extract and retain a separate **refund summary**:
  - refund transaction count
  - refund TPV
  - refund incentive amount
- Do not mix refund rows into the primary campaign participant / TPV / transaction counts unless the template specifically asks for net values.
- Never expose `wallet_userid`, `wallet_transaction_id`, or any other row-level identifiers in the final post-mortem.

## 4. Template Preservation Rules
- Do not change:
  - sheet names in the uploaded working file
  - merged-cell structure
  - row order
  - section labels
  - formulas that are part of the template logic
- Treat the **Rules / Notes** column in the template as operational instructions.
- Preserve the **Traffic Light (MDR%)** rule embedded in the template.
- Use `{{PLACEHOLDER_NAME}}` for all variable inputs in the schema documentation.
- Replace every sample value with a placeholder or formula rule.

## 5. MDR / Traffic-Light Rule
- MDR is a **campaign-specific input** and must be updated for each run.
- Do not infer MDR from prior campaigns.
- Use the current-run MDR to populate the workbook MDR input field.
- The template traffic-light result must be determined exactly as follows:
  - **Green** if MDR >= 0.47%
  - **Yellow** if MDR >= 0.18% and MDR < 0.47%
  - **Red** if MDR < 0.18%
- If the workbook already contains a formula for this field, keep the formula intact and only populate the required MDR input.
- If MDR is missing, mark the input `MANUAL_INPUT_REQUIRED` and leave any dependent result unresolved.

## 6. Manual-Override Fields
The following fields remain manual or BU / Finance confirmed unless a trustworthy source is uploaded:
- Type of Campaign
- Campaign Period when BU overrides official dates
- Funding Mechanism
- MDR source note / approver note
- Finance direct-cost rates
- Est. 12M CLTV
- Any business classification field not derivable from source data

Mark these as `MANUAL_INPUT_REQUIRED` if no source is provided.

## 7. Privacy / Security
- Treat TPV, retention, cost rates, partner incentive economics, and MDR as confidential business inputs.
- Remove or suppress all PII and row-level identifiers from the final output.
- Output only aggregated campaign-level figures.

## 8. Calculation Discipline
- Never estimate missing values.
- If a formula depends on a missing input, keep the formula rule in the schema and set the unresolved input to `DATA_NOT_FOUND` or `MANUAL_INPUT_REQUIRED` as applicable.
- Record every unresolved dependency in a validation log or chat summary.

## Excel Output Guardrails

The following rules apply to every Excel output produced by the engine.
Violation of any rule will produce a blank, corrupted, or visually broken file.

**G-XL-1 — artifact_tool is prohibited.**
Never use SpreadsheetArtifact, artifact_tool, artifact.recalculate(), artifact.export(),
or artifact.render(). These overwrite the saved file with a blank internal format.
The only permitted save call is wb.save(output_path).

**G-XL-2 — Never use load_workbook(). Build fresh only.**
Never open any template file with openpyxl.load_workbook(). Always build the output
workbook from scratch using Workbook(). The template file is a visual reference only —
all structure, formatting, and formulas are rebuilt programmatically in the script.

**G-XL-3 — Never delete sheets.**
All sheets must remain in the workbook. Deleting sheets breaks cross-sheet formula
references in Campaign Template.

**G-XL-4 — Never modify formatting.**
Do not set fill, font, border, number_format, or alignment on any cell.
The template formatting must be preserved exactly as loaded.

**G-XL-5 — Always write to merged cell anchor only.**
Every input row is a merged range. Write only to the top-left anchor cell.
Use the safe_write helper defined in the Excel Population Technical Runbook.

**G-XL-6 — Never overwrite formula cells.**
Check every target cell with is_formula() before writing.
Known formula cells: C21, C23, C25, C29, C30, C38, C42.

**G-XL-7 — Write MDR as a decimal.**
C22 must receive a decimal value (e.g. 0.013). Never write a percentage string.

**G-XL-8 — Save once, then stop.**
Call wb.save() once. Print the output path. End the script.
No render, recalculate, preview, or export calls after save.

**G-XL-9 — Always build from Workbook() — never load_workbook().**
The output file must always be constructed by creating a new Workbook() and applying
all structure, merges, formatting, formulas, and campaign values programmatically.
Never use openpyxl.load_workbook() for any purpose in the population script.

## File Reading Guardrails

**G-FR-1 — Always use Python + pandas to read partner files.**
Never open, browse, or preview the partner data file directly.
The only accepted reading method is the read_partner_file() function
defined in file_ingestion_skill.md.

**G-FR-2 — Always run check_deps() before the first file read.**
If a required library is missing, install it before proceeding.

**G-FR-3 — Always print column names and row count after reading.**
Confirm the schema before any filtering or aggregation.

**G-FR-4 — Always use fuzzy column matching.**
Never hardcode exact column names from a prior run.
Partner column names change between files.

**G-FR-5 — Always clean amount columns before aggregation.**
Strip RM symbols and commas. Convert to numeric with errors='coerce'.

**G-FR-6 — If the file cannot be read after all fallback attempts, stop.**
Report the exact filename, extension detected, and error type.
Do not continue to Excel generation with unread partner data.

## Multi-Campaign Guardrails

**G-MC-1 — Never carry data between campaigns.**
Column mappings, KPI values, RFA fields, and MDR from Campaign 1
must never influence Campaign 2 or any subsequent campaign.
Re-run all ingestion and mapping steps fresh for each campaign.

**G-MC-2 — Always use Campaign ID to link files.**
The Campaign ID declared in the RUN CONFIGURATION is the authoritative
link between an RFA file and a partner data file. Never infer pairings
from merchant names, dates, or filenames.

**G-MC-3 — Process sequentially, not simultaneously.**
Complete all chunks for one campaign before starting the next.

**G-MC-4 — Label all outputs with Campaign ID.**
Every chunk confirmation, every Excel filename, and every summary section
must include the Campaign ID so outputs are never mixed up.

**G-MC-5 — If one campaign fails, continue to the next.**
Log the failure clearly and process remaining campaigns.
Do not abort the entire session because one campaign failed.
