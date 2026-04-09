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
   - RFA / Sage approval file
   - internal validation / transaction data file(s)
   - partner raw data file(s)
   - retention data file(s), if provided
   - finance rate / cost input file(s), if provided

The master template workbook is the design source of truth. Preserve the same workbook structure, sheet names, colours, borders, merged cells, formulas, notes, layout, and overall visual design.

## Core execution rules

- Duplicate the master template before population.
- Read the static engine sources before touching the campaign-specific uploads.
- Read the workbook’s embedded **Placeholder Dictionary**, **Cell-to-Source Mapping**, and **Engine Runbook** tabs before writing any output cells.
- Register the current-run MDR from the ignition prompt before populating the workbook.
- Only write into cells marked `WRITE_INPUT` or other explicit input actions.
- Never overwrite cells marked `KEEP_FORMULA`.
- Never estimate missing values. Use `DATA_NOT_FOUND` or `MANUAL_INPUT_REQUIRED`.
- Use the RFA as the source of truth for approved campaign identity and approved budget fields, only after confirming approval status.
- Internal verification / TXN data is the final KPI source when it conflicts with partner raw data.
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
**Do:** Confirm the uploaded package includes the RFA, partner raw data, internal verification / TXN data, retention data (if required), and finance rate source (if required). Register each file in the Source File Template and in the validation log.  
**Do not:** Do not assume a missing file exists. Do not infer a source from memory.  
**Checkpoint:** Source inventory completed.  
**Guardrail:** Missing critical sources must be flagged immediately.

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

### Step 6 - Normalize partner raw data
**Do:** Map the partner raw file into canonical fields: qualified transactions, qualified users, TPV, refunds, mechanics / waves, and campaign date coverage. Preserve raw partner totals for variance checks only.  
**Do not:** Do not mix refunds into the primary KPI block unless a finance-approved net rule exists.  
**Checkpoint:** Partner staging summary completed.  
**Guardrail:** Partner data is supporting evidence, not the final KPI source when internal data exists.

### Step 7 - Normalize internal verification data
**Do:** Build the internal KPI view for campaign participants, TPV, transaction count, and merchant-impact time buckets. Use this dataset as the final source for KPI cells when available.  
**Do not:** Do not silently prefer partner data over internal data.  
**Checkpoint:** Internal KPI staging completed.  
**Guardrail:** Internal data wins on final reconciled KPI values.

### Step 8 - Reconcile discrepancies
**Do:** Compare partner versus internal KPI values. Write final KPI cells from internal data, and log variances separately. If the variance threshold or business tolerance is breached, raise a warning.  
**Do not:** Do not average the two sources or choose whichever looks better.  
**Checkpoint:** KPI reconciliation complete.  
**Guardrail:** Every conflict must be visible in the validation output.

### Step 9 - Populate retention
**Do:** Populate retention fields only from the internal retention source aligned to the campaign cohort and post-campaign month buckets.  
**Do not:** Do not use assumed rates from the RFA as actual retention.  
**Checkpoint:** Retention block completed or `DATA_NOT_FOUND`.  
**Guardrail:** Observed retention only.

### Step 10 - Populate MDR and finance inputs
**Do:** Write the campaign-specific MDR input from the ignition prompt into the workbook MDR field, unless an explicitly approved override source is designated. Enter finance month labels and month-level rate inputs into the Source File Template helper section, then populate main-sheet finance inputs such as CPAM cost, average reload / cloud / PLSA rates, and CLTV if supplied.  
**Do not:** Do not copy sample finance values from the workbook. Do not infer missing rates. Do not overwrite the traffic-light cell if it is formula-driven.  
**Checkpoint:** MDR and finance inputs completed or flagged.  
**Guardrail:** Finance-controlled fields remain manual / approved-source only.

### Step 11 - Apply the MDR traffic-light rule
**Do:** Preserve the template logic for the traffic-light row. The result must follow this rule:
- Green if MDR >= 0.47%
- Yellow if MDR >= 0.18% and MDR < 0.47%
- Red if MDR < 0.18%

**Do not:** Do not replace the formula with hardcoded text if the template already derives the result.  
**Checkpoint:** Traffic-light field resolved or appropriately flagged.  
**Guardrail:** The template rule must be preserved exactly.

### Step 12 - Populate merchant-impact analysis
**Do:** Populate the helper TPV series first, then fill the main merchant-impact section. Keep date windows consistent across pre, during, post 1M / 2M / 3M / 6M buckets and preserve optional percentage cells only if requested.  
**Do not:** Do not invent baseline growth conventions for ambiguous pre-period cells.  
**Checkpoint:** Merchant-impact section completed.  
**Guardrail:** Use one consistent windowing logic across all buckets.

### Step 13 - Leave formulas intact
**Do:** For revenue, total cost, direct cost, gross profit, ROI, CPAM cost per user, and helper-derived deltas, keep the workbook formulas exactly as-is. Only populate their dependency cells.  
**Do not:** Do not replace formulas with hardcoded numbers.  
**Checkpoint:** Derived rows calculate automatically.  
**Guardrail:** Formula cells are part of the template logic.

### Step 14 - Write narratives and decisions last
**Do:** After all numeric sections are validated, write concise narrative summaries and the 2nd-review decision using documented rules or reviewer input.  
**Do not:** Do not let narratives contradict the numbers. Do not auto-approve 2nd review without a rule or confirmation.  
**Checkpoint:** Narrative fields completed.  
**Guardrail:** Narratives must be evidence-based.

### Step 15 - Quality assurance and export
**Do:** Run final checks: no PII, no missing required sources hidden, no broken formulas, date alignment validated, budget and variance checks logged, MDR and traffic-light rule validated, and all unresolved items marked `DATA_NOT_FOUND` or `MANUAL_INPUT_REQUIRED`.  
**Do not:** Do not ship a workbook that contains guessed values or overwritten formula cells.  
**Checkpoint:** Workbook ready for handoff.  
**Guardrail:** Zero-hallucination policy applies to every field.

## Final required outputs

The engine must return exactly **2 outputs only**:

1. **Completed Excel file**
   - full completed workbook for the current campaign
   - matches the master template design exactly
   - preserves formatting, colours, formulas, structure, and worksheet design
   - populates all fields that can be determined from the uploaded files and current run parameters

2. **Summary in chat**
   - campaign identification details used
   - files successfully read
   - MDR percentage used
   - traffic-light result
   - major findings
   - reconciled KPI observations
   - discrepancies between partner and internal data
   - missing files
   - incomplete fields
   - unresolved items requiring manual input or review
   - validation warnings

Do not produce a third deliverable.

## Final QA checklist
- Static engine sources read before run parameters and campaign-specific files.
- Prompt MDR captured before workbook population.
- RFA status checked and approved before using the RFA ID.
- Source inventory registered in the helper / source area.
- Required missing sources explicitly flagged.
- Partner vs internal KPI variance reviewed and logged.
- Retention populated only from observed retention data.
- MDR and finance inputs sourced only from approved inputs.
- Traffic-light rule checked against MDR.
- Formula cells preserved.
- Narrative cells checked against numbers.
- No PII or row-level IDs in the final output.
- Only 2 deliverables returned: completed workbook and chat summary.