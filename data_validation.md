# Data Validation Rules: TNGD Campaign Post-Mortem

## Validation Mode
These rules are **campaign agnostic** and must be applied to the files and run parameters supplied in the current run only.

## 1. Static-Source Read Gate
- Confirm the static engine sources were read before any campaign-specific extraction begins.
- If the workbook rules / notes or mapping assets were not read first, return `MANUAL_REVIEW_REQUIRED`.

## 2. RFA Approval Gate
- The RFA number is only valid if the document status is explicitly **Approved**.
- If status is not Approved, stop the workflow and return `MANUAL_REVIEW_REQUIRED`.

## 3. Date Alignment
Validate all three date views separately:
1. **RFA approved date range**
2. **Actual transaction date range in campaign raw/internal data**
3. **Workbook campaign period values**

Flag a mismatch when any of the following occur:
- transaction dates begin before the declared campaign start date
- transaction dates end materially earlier or later than the declared campaign end date
- workbook dates differ from the RFA and no BU override is provided

## 4. Funding Mechanism Check
- Compare `Funding Mechanism` across the RFA, workbook, and any BU-confirmed source.
- If they differ, output `MANUAL_REVIEW_REQUIRED`.

## 5. KPI Extraction Rule
Primary campaign KPIs must be based on successful campaign charges unless a finance-approved net rule is supplied:
- Campaign Participants = distinct users on CHARGE rows
- Campaign TPV = RM sum on CHARGE rows
- Campaign Txn # = distinct CHARGE transactions

Refunds must be summarized separately and disclosed in the validation output.

## 6. Mechanic / Wave Validation
Where the RFA defines multiple mechanic waves, validate actual observed performance by wave.
- Compare actual performance by campaign_id / mechanic against the RFA-defined caps or limits.
- Flag under-delivery, over-redemption, or unexplainable spillover.

## 7. TPV Reconciliation
If both partner and internal TPV are available:
- `TPV variance % = (Internal TPV - Partner TPV) / Internal TPV`
- If absolute variance > **5%**, raise `High Variance Warning`

If internal TPV is unavailable:
- set internal TPV reconciliation to `DATA_NOT_FOUND`
- do not fabricate a variance %

## 8. Budget Check
Compare:
- Approved Budget from RFA
- Actual CPAM Cost
- Total Campaign Cost

Rules:
- flag if `CPAM Cost > Approved Budget`
- flag if total campaign cost cannot be validated because finance inputs are missing

## 9. MDR Input and Traffic-Light Validation
Validate the campaign-specific MDR input as follows:
- MDR must be supplied either in the ignition prompt run configuration or in an explicitly approved finance / BU source
- MDR must be numeric and stored as a percentage input
- if both prompt and file sources exist, compare them and flag a mismatch
- workbook traffic-light output must follow this rule exactly:
  - **Green** if MDR >= 0.47%
  - **Yellow** if MDR >= 0.18% and MDR < 0.47%
  - **Red** if MDR < 0.18%
- if MDR is missing, mark the MDR input `MANUAL_INPUT_REQUIRED` and treat any dependent revenue or traffic-light result as unresolved

## 10. Direct-Cost Dependency Check
Rows for:
- MDR
- Avg Reload Cost %
- Avg Cloud Cost / Txn
- Avg PLSA Cost / Txn

must be validated against the run configuration, finance source workbook, or explicit BU / finance-provided values as applicable.

If required inputs are missing:
- mark these inputs `DATA_NOT_FOUND` or `MANUAL_INPUT_REQUIRED`
- state that revenue, direct cost, total cost, GP, ROI, and traffic-light output may remain provisional

## 11. Retention Integrity
- Retention must align to the campaign cohort and actual post-campaign months.
- Do not use RFA assumption rates as actual observed retention.
- If retention file is missing, all retention cells must remain unresolved.

## 12. PII Protection
Before output:
- ensure `wallet_userid`, `wallet_transaction_id`, and any row-level identifiers are excluded
- output only campaign-level aggregates and notes
