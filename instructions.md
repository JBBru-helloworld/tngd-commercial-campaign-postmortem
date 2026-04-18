# Instructions: Processing the TNGD Campaign Post-Mortem

## Purpose
These instructions define a **reusable engine** for any campaign post-mortem using the uploaded `campaign_file_TEMPLATE.xlsx` workbook and the static engine files.

## Two-Phase Execution Model

The engine operates in two phases. The AI executes **Phase 2 only**.

**Phase 1 — Human Validation:** Before the AI runs, a team member validates the partner data file against internal database records and populates: internal KPI reconciliation, all retention data, all finance cost rates, merchant-impact data, CLTV, and the second-review decision. The validated values are written into the partner data file or a separate validated data file.

**Phase 2 — AI Run:** The AI reads the validated partner data file and the approved RFA, populates all campaign identity fields (from RFA), final validated KPI fields (from the validated partner data file), the MDR rate (from the ignition prompt), and the traffic-light classification. Fields the human could not confirm during Phase 1 are marked `MANUAL_INPUT_REQUIRED`.

**AI Boundary Rule:** The AI must never block, error, or stop because internal data, retention files, finance files, or BI files are absent. These are Phase 1 human tasks. If a field is not present in the validated partner data file, mark it `MANUAL_INPUT_REQUIRED` and continue.

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
- read `file_ingestion_skill.md`
- read `framework_summary.md`

Treat the workbook layout as the fixed target structure for the execution run.

## Step 1: Register the Run Configuration from the Ignition Prompt
Extract the campaign-specific run parameters from the ignition prompt:
- campaign name / run label
- campaign ID
- MDR percentage for the run
- MDR source / approver note, if supplied
- current campaign file names for the RFA, validated partner data file

Rules:
- treat MDR as a controlled campaign-specific input
- do not infer MDR from prior campaigns
- if MDR is missing from the prompt and not supplied by an approved source file, mark it `MANUAL_INPUT_REQUIRED`

## Step 2: Parse the Approved RFA (Sage) [Phase 2 Step]
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

## Step 3: Read the Validated Partner Data File [Phase 2 Step]
The partner data file has already been validated by the human reviewer in Phase 1.
Read it using the exact method defined in `file_ingestion_skill.md`. Run check_deps() first.

Normalize the file into canonical fields:
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
4. produce final KPI values (these are final validated values, not provisional):
   - `{{CAMPAIGN_PARTICIPANTS}}`
   - `{{CAMPAIGN_TPV_RM}}`
   - `{{CAMPAIGN_TXN_COUNT}}`
   - `{{PARTNER_INCENTIVE_GROSS_RM}}`
   - `{{REFUND_TXN_COUNT}}`
   - `{{REFUND_TPV_RM}}`
   - `{{REFUND_INCENTIVE_RM}}`

If a KPI cannot be extracted, write `MANUAL_INPUT_REQUIRED`.

## Step 4: Read Validated Internal Data from the Partner File [Phase 1 Step]
**This is a Phase 1 human task. The human has already completed internal KPI reconciliation before the AI run.**

The AI reads already-reconciled values from the validated partner data file.
If reconciled internal KPI values are present in the validated partner data file:
- read internal TPV, participants, and transaction counts from the file
- use them as final source-of-truth values

If reconciled values are **not** present in the validated partner data file:
- set internal verification fields to `MANUAL_INPUT_REQUIRED`
- write a validation note that the human reviewer should supply these values

## Step 5: Retention Logic [Phase 1 Step]
**This is a Phase 1 human task. The AI reads validated retention values from the partner data file.**

For each retention month required by the template:
- read the validated retention rate from the partner data file
- populate the exact month bucket required by the template

If retention values are not present in the validated partner data file:
- set all retention outputs to `MANUAL_INPUT_REQUIRED`
- do not backsolve from assumptions in the RFA

## Step 6: Revenue, MDR, and Cost Inputs [Partially Phase 1]
**MDR (`{{MDR_RATE}}`) and CPAM Cost (`{{CPAM_COST_RM}}`) are Phase 2 fields — populate from the ignition prompt and validated partner data file respectively. All finance cost-rate inputs (`{{AVG_RELOAD_COST_PCT}}`, `{{AVG_CLOUD_COST_PER_TXN_RM}}`, `{{AVG_PLSA_COST_PER_TXN_RM}}`) are Phase 1 human inputs. If not present in the validated partner data file, the AI must mark those cells `MANUAL_INPUT_REQUIRED`. Do not flag this as an error.**

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

If finance or rate inputs are absent from the validated partner data file:
- preserve the formula rule in the workbook
- set unresolved rate inputs to `MANUAL_INPUT_REQUIRED`

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
Populate the merchant-level section using pre/during/post monthly TPV and MTU data from the validated partner data file or approved BI / internal sources provided after Phase 1.

If source data is unavailable:
- set the section to `MANUAL_INPUT_REQUIRED`
- do not reuse sample values from the workbook

## Step 9: Write the Output
Return exactly **2 deliverables only**:
1. the completed Excel file matching the template design, named:
   `campaign_postmortem_[CAMPAIGN_ID]_[CAMPAIGN_NAME]_phase2.xlsx`
2. a chat summary of findings, discrepancies, missing fields, incomplete fields, unresolved manual inputs, and important validation warnings

Never output row-level transaction data. Only output aggregated workbook-ready values.
