# Data Validation Rules: TNGD Campaign Post-Mortem

## Validation Mode
These rules are **campaign agnostic** and must be applied to the files and run parameters supplied in the current run only.

## 1. Static-Source Read Gate [Phase 2]
- Confirm the static engine sources were read before any campaign-specific extraction begins.
- If the workbook rules / notes or mapping assets were not read first, return `MANUAL_REVIEW_REQUIRED`.

## 2. RFA Approval Gate [Phase 2]
- The RFA number is only valid if the document status is explicitly **Approved**.
- If status is not Approved, stop the workflow and return `MANUAL_REVIEW_REQUIRED`.

## 3. Date Alignment [Phase 2]
Validate all three date views separately:
1. **RFA approved date range**
2. **Actual transaction date range in validated partner data file**
3. **Workbook campaign period values**

Flag a mismatch when any of the following occur:
- transaction dates begin before the declared campaign start date
- transaction dates end materially earlier or later than the declared campaign end date
- workbook dates differ from the RFA and no BU override is provided

Note: In Phase 2, date validation is performed using the RFA dates and the validated partner data file transaction date range. Internal data date verification is a Phase 1 human step.

## 4. Funding Mechanism Check [Phase 2]
- Compare `Funding Mechanism` across the RFA, workbook, and any BU-confirmed source.
- If they differ, output `MANUAL_REVIEW_REQUIRED`.

## 5. KPI Extraction Rule [Phase 2 — AI-Populated from Validated Partner Data File]
Primary campaign KPIs must be based on successful campaign charges unless a finance-approved net rule is supplied:
- Campaign Participants = distinct users on CHARGE rows
- Campaign TPV = RM sum on CHARGE rows
- Campaign Txn # = distinct CHARGE transactions

The partner data file may be provided as .csv, .xlsx, or .xls. Detect the file format from the file extension and read it with the appropriate method (pd.read_csv for .csv; pd.read_excel for .xlsx or .xls). Do not assume the file is always CSV.

In Phase 2, extract KPIs from the validated partner data file CHARGE rows. These values are treated as final validated values because Phase 1 human validation has already been completed. If a value cannot be extracted, write `MANUAL_INPUT_REQUIRED`.

Refunds must be summarized separately and disclosed in the validation output.

## 6. Mechanic / Wave Validation [Phase 1]
This rule requires internal data and is a Phase 1 human validation step. The AI must skip this rule and mark dependent fields `MANUAL_INPUT_REQUIRED` if values are not present in the validated partner data file.

Where the RFA defines multiple mechanic waves, validate actual observed performance by wave.
- Compare actual performance by campaign_id / mechanic against the RFA-defined caps or limits.
- Flag under-delivery, over-redemption, or unexplainable spillover.

## 7. TPV Reconciliation [Phase 1]
This rule requires internal data and is a Phase 1 human validation step completed before the AI run.

The human reviewer reconciles partner TPV against internal database records in Phase 1 and writes the reconciled values into the validated partner data file.
- If reconciled internal TPV values are present in the validated partner data file, the AI reads and uses them.
- The AI does not perform this reconciliation itself.
- If reconciled values are absent from the validated partner data file, set internal TPV reconciliation to `MANUAL_INPUT_REQUIRED`.

If both partner and internal TPV are available in the validated file:
- `TPV variance % = (Internal TPV - Partner TPV) / Internal TPV`
- If absolute variance > **5%**, raise `High Variance Warning`

## 8. Budget Check [Phase 1]
This rule requires finance inputs and is a Phase 1 human validation step. The AI must skip this rule and mark dependent fields `MANUAL_INPUT_REQUIRED` if not present in the validated partner data file.

Compare:
- Approved Budget from RFA
- Actual CPAM Cost
- Total Campaign Cost

Rules:
- flag if `CPAM Cost > Approved Budget`
- flag if total campaign cost cannot be validated because finance inputs are missing

## 9. MDR Input and Traffic-Light Validation [Phase 2]
Validate the campaign-specific MDR input as follows:
- MDR must be supplied either in the ignition prompt run configuration or in an explicitly approved finance / BU source
- MDR must be numeric and stored as a percentage input
- if both prompt and file sources exist, compare them and flag a mismatch
- workbook traffic-light output must follow this rule exactly:
  - **Green** if MDR >= 0.47%
  - **Yellow** if MDR >= 0.18% and MDR < 0.47%
  - **Red** if MDR < 0.18%
- if MDR is missing, mark the MDR input `MANUAL_INPUT_REQUIRED` and treat any dependent revenue or traffic-light result as unresolved

## 10. Direct-Cost Dependency Check [Phase 1]
This rule requires finance inputs and is a Phase 1 human validation step. The AI must mark all direct-cost rate inputs `MANUAL_INPUT_REQUIRED` if not present in the validated partner data file.

Rows for:
- MDR — Phase 2 (populated from ignition prompt)
- Avg Reload Cost % — Phase 1 (mark `MANUAL_INPUT_REQUIRED` if absent)
- Avg Cloud Cost / Txn — Phase 1 (mark `MANUAL_INPUT_REQUIRED` if absent)
- Avg PLSA Cost / Txn — Phase 1 (mark `MANUAL_INPUT_REQUIRED` if absent)

must be validated against the run configuration, finance source workbook, or explicit BU / finance-provided values as applicable.

If Phase 1 rate inputs are not present in the validated partner data file:
- mark these inputs `MANUAL_INPUT_REQUIRED`
- state that direct cost, total cost, GP, ROI, and dependent results will remain unresolved until Phase 1 values are supplied

## 11. Retention Integrity [Phase 1]
This rule requires the internal retention source and is a Phase 1 human validation step. The AI must mark all retention cells `MANUAL_INPUT_REQUIRED` if values are not present in the validated partner data file.

- Retention must align to the campaign cohort and actual post-campaign months.
- Do not use RFA assumption rates as actual observed retention.
- If retention values are absent from the validated partner data file, all retention cells must be marked `MANUAL_INPUT_REQUIRED`.

## 12. PII Protection [All Phases]
Before output:
- ensure `wallet_userid`, `wallet_transaction_id`, and any row-level identifiers are excluded
- output only campaign-level aggregates and notes
