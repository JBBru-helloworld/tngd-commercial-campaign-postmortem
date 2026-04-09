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

## 2. Mandatory Stop / Flag Conditions
Return `DATA_NOT_FOUND` or `MANUAL_REVIEW_REQUIRED` when any of the following apply:
- RFA status is not explicitly **Approved**.
- A required source file is missing.
- Campaign dates in source files do not align and no BU override is provided.
- Funding mechanism conflicts across trusted sources and no BU confirmation is provided.
- Required finance rates are unavailable.
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