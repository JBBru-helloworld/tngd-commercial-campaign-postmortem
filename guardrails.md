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
5. **Partner raw data** for campaign performance extraction when internal files are absent, incomplete, or used as a first-pass operational source.
6. **Campaign Working File template** for structure, formulas, merged-cell layout, labels, and cell-level business rules.

If two sources disagree:
- Use the **Internal Data File** as the final KPI source.
- Use the **prompt-supplied MDR** unless an approved finance / BU source is explicitly designated as the higher-priority override for that run.
- Preserve the conflicting value in a **Validation Note**.
- Never silently overwrite a discrepancy.

## Phase Boundary Rule

The engine operates in two distinct phases. The AI executes Phase 1 only.

### Phase 1 — AI-Only Fields
The AI populates these fields during a Phase 1 run:
- Campaign identity fields from the RFA (name, ID, dates, mechanics, amount, funding mechanism, cost centre)
- Partner-derived KPIs extracted from CHARGE rows (campaign participants, TPV, transaction count) — labeled as provisional
- MDR rate (from ignition prompt)
- CPAM Cost (from partner file if present; otherwise `PENDING_HUMAN_VALIDATION`)
- Traffic-light classification (derived from MDR)

### Phase 2 — Human-Only Fields
A team member completes these fields after receiving the Phase 1 output:
- Internal KPI reconciliation (internal TPV, internal participants, internal transaction count, all variance cells)
- All retention cells (post-month 1 through 12)
- All finance cost-rate inputs (Avg Reload Cost %, Avg Cloud Cost/Txn, Avg PLSA Cost/Txn)
- Merchant-impact TPV series, MTU series, and growth values
- CLTV (Est. 12M CLTV)
- Second-review decision
- Any field that depends on internal database records, BI pulls, or finance rate workbooks

### AI Behavior at the Phase Boundary
- The AI must **never** substitute estimated values for Phase 2 fields.
- Use `PENDING_HUMAN_VALIDATION` instead of `DATA_NOT_FOUND` for Phase 2 fields specifically. This signals to the human reviewer that the field is expected to be filled — not that the data is permanently unavailable.
- Do not treat absent Phase 2 files as blocking errors. Only the RFA and partner raw data file are required for Phase 1.

## 2. Mandatory Stop / Flag Conditions
Return `DATA_NOT_FOUND` or `MANUAL_REVIEW_REQUIRED` when any of the following apply:
- RFA status is not explicitly **Approved**.
- A Phase 1 required source file is missing (RFA or partner raw data file). If either Phase 1 file is absent, stop and return `MANUAL_REVIEW_REQUIRED`. Phase 2 files (internal TXN data, retention data, finance rate files) are not blocking — mark their dependent cells `PENDING_HUMAN_VALIDATION` and continue.
- Campaign dates in source files do not align and no BU override is provided.
- Funding mechanism conflicts across trusted sources and no BU confirmation is provided.
- MDR is required but missing from both the prompt run configuration and any approved supporting source.
- Template logic depends on an external Phase 1 workbook that has not been uploaded.

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