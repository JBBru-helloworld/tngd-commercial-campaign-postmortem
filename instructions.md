# Instructions: Processing the TNGD Campaign Post-Mortem

## Purpose
These instructions define a **reusable engine** for any campaign post-mortem using the uploaded `campaign_file_TEMPLATE.xlsx` workbook and the static engine files.

## Two-Phase Execution Model

The engine operates in two phases. The AI executes **Phase 1 only**.

**Phase 1 — AI Run:** The AI reads the RFA file and partner raw data file, populates all campaign identity fields (from RFA), provisional KPI fields (from partner CHARGE rows), the MDR rate (from the ignition prompt), and the traffic-light classification. All other fields are marked `PENDING_HUMAN_VALIDATION`.

**Phase 2 — Human Validation:** After receiving the Phase 1 output Excel file, a team member validates the partner file against internal database records and populates: internal KPI reconciliation, all retention data, all finance cost rates, merchant-impact data, CLTV, and the second-review decision.

**AI Boundary Rule:** The AI must never block, error, or stop because internal data, retention files, finance files, or BI files are absent. Mark Phase 2 cells `PENDING_HUMAN_VALIDATION` and continue.

## Step 0: Read the Static Engine Sources First
Before processing any campaign-specific uploads:
- open the current master workbook
- read the rules / notes column
- read all merged input / output ranges
- read all formulas
- read any embedded engine tabs
- read `guardrails.md`
- read `instructions.md`
- read `template_schema.md`
- read `data_validation.md`
- read `placeholder_dictionary.md`
- read `cell_to_source_mapping.md`
- read `engine_runbook.md`
- read `framework_summary.md`

Treat the workbook layout as the fixed target structure for the execution run.

## Step 1: Register the Run Configuration from the Ignition Prompt
Extract the campaign-specific run parameters from the ignition prompt:
- campaign name / run label
- MDR percentage for the run
- MDR source / approver note, if supplied
- current campaign file names for the RFA, internal data, partner raw data, retention, and finance sources

Rules:
- treat MDR as a controlled campaign-specific input
- do not infer MDR from prior campaigns
- if MDR is missing from the prompt and not supplied by an approved source file, mark it `MANUAL_INPUT_REQUIRED`

## Step 2: Parse the Approved RFA (Sage) [Phase 1 Step]
Extract generic campaign metadata into placeholders such as:
- `{{CAMPAIGN_NAME}}`
- `{{APPROVED_RFA_NO}}`
- `{{RFA_STATUS}}`
- `{{RFA_START_DATE}}`
- `{{RFA_END_DATE}}`
- `{{RFA_AMOUNT_RM}}`
- `{{RFA_FUNDING_MECHANISM}}`
- `{{COMMERCIAL_OBJECTIVES_TEXT}}`
- `{{MECHANIC_TEXT}}`
- `{{REQUESTOR_DEPARTMENT}}`
- `{{PROJECT_NAME}}`

Rules:
- only use the RFA number if the document status is **Approved**
- keep campaign naming campaign-specific at execution time
- preserve mechanics wording closely enough for business review

## Step 3: Standardize the Partner Raw File [Phase 1 Step]
Normalize partner/raw exports into canonical fields:
- `transaction_type`
- `campaign_id`
- `campaign_name`
- `merchant_name`
- `wallet_userid`
- `transaction_date`
- `wallet_transaction_id`
- `transaction_amount_rm`
- `benefit_amount_rm`

Then:
1. filter `transaction_type = CHARGE` to produce campaign KPI totals
2. separately summarize `transaction_type = REFUND`
3. group by campaign_id / campaign_name / mechanic where relevant
4. produce generic KPI placeholders:
   - `{{CAMPAIGN_PARTICIPANTS}}`
   - `{{CAMPAIGN_TPV_RM}}`
   - `{{CAMPAIGN_TXN_COUNT}}`
   - `{{PARTNER_INCENTIVE_GROSS_RM}}`
   - `{{REFUND_TXN_COUNT}}`
   - `{{REFUND_TPV_RM}}`
   - `{{REFUND_INCENTIVE_RM}}`

## Step 4: Reconcile Against Internal Data [Phase 2 Step]
**This is a Phase 2 step. The AI must skip population of these cells and mark them `PENDING_HUMAN_VALIDATION`. Do not flag this as an error.**

If internal transaction files are uploaded:
- recalculate TPV, participants, and transactions from internal data
- compare them against partner/raw totals
- treat internal totals as final source-of-truth

If internal files are **not** uploaded:
- populate partner/raw-derived fields where allowed
- set internal verification fields to `DATA_NOT_FOUND`
- write a validation note that independent reconciliation could not be performed

## Step 5: Retention Logic [Phase 2 Step]
**This is a Phase 2 step. The AI must skip population of these cells and mark them `PENDING_HUMAN_VALIDATION`. Do not flag this as an error.**

Retention rows require an uploaded retention source.

For each retention month required by the template:
- identify the matching post-campaign month cohort
- compute `retention_rate = retained_users_for_month / campaign_participants`
- populate the exact month bucket required by the template

If retention source is missing:
- set all retention outputs to `DATA_NOT_FOUND`
- do not backsolve from assumptions in the RFA

## Step 6: Revenue, MDR, and Cost Inputs [Partially Phase 2]
**MDR (`{{MDR_RATE}}`) and CPAM Cost (`{{CPAM_COST_RM}}`) are Phase 1 fields — populate from the ignition prompt and partner file respectively. All finance cost-rate inputs (`{{AVG_RELOAD_COST_PCT}}`, `{{AVG_CLOUD_COST_PER_TXN_RM}}`, `{{AVG_PLSA_COST_PER_TXN_RM}}`) are Phase 2 steps. The AI must mark those cells `PENDING_HUMAN_VALIDATION`. Do not flag this as an error.**

Populate or confirm:
- `{{MDR_RATE}}`
- `{{CPAM_COST_RM}}`
- `{{AVG_RELOAD_COST_PCT}}`
- `{{AVG_CLOUD_COST_PER_TXN_RM}}`
- `{{AVG_PLSA_COST_PER_TXN_RM}}`

Rules:
- populate the workbook MDR input from the **run configuration** unless an approved higher-priority override is explicitly supplied
- validate any file-based MDR against the prompt MDR and flag mismatches
- `Campaign Revenue (RM)` = `Campaign TPV * MDR`
- `Campaign Total Cost (RM)` = `CPAM Cost + Campaign Direct Cost`
- `Campaign Direct Cost (RM)` = `(Campaign TPV * Avg Reload Cost %) + (Campaign Txn # * Avg Cloud Cost / Txn) + (Campaign Txn # * Avg PLSA Cost / Txn)`

If finance or rate inputs are absent:
- preserve the formula rule in the workbook
- set unresolved rate inputs to `DATA_NOT_FOUND` or `MANUAL_INPUT_REQUIRED` as appropriate

## Step 7: Traffic-Light Classification
Populate the template traffic-light result using the campaign MDR rule:
- **Green** if MDR >= 0.47%
- **Yellow** if MDR >= 0.18% and MDR < 0.47%
- **Red** if MDR < 0.18%

Rules:
- if the workbook contains a formula for the traffic-light cell, keep the formula intact
- only populate the MDR input cell and any required source note
- do not overwrite the traffic-light cell with free text if the template already derives it

## Step 8: Merchant-Level Impact Analysis
Populate the merchant-level section using pre/during/post monthly TPV and MTU data from approved BI or internal sources.

If source data is unavailable:
- set the section to `DATA_NOT_FOUND`
- do not reuse sample values from the workbook

## Step 9: Write the Output
Return exactly **2 deliverables only**:
1. the completed Excel file matching the template design
2. a chat summary of findings, discrepancies, missing files, incomplete fields, unresolved manual inputs, and important validation warnings

Never output row-level transaction data. Only output aggregated workbook-ready values.