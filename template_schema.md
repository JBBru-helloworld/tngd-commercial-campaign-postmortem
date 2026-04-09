# Template Schema: Campaign Working File

## Purpose
This schema is **campaign agnostic**. It preserves the logic and structure of the uploaded working file while replacing campaign-specific sample values with placeholders or formula rules.

## Workbook Handling Rule
At execution time, always derive the final schema from the **currently uploaded** workbook.
- Preserve the workbook's actual sheet names, merged cells, labels, formulas, and Rules / Notes column.
- Do not rely on merchant-specific sample labels that may exist in a workbook copy.
- Do not carry forward sample data from a prior campaign.

---

## Section A: Campaign Header / Identity

| Generic Field | Placeholder | Source | Rule | Phase |
|---|---|---|---|---|
| Campaign Name | `{{CAMPAIGN_NAME}}` | Approved RFA | Copy from current RFA only. | Phase 1 |
| RFA No. | `{{APPROVED_RFA_NO}}` | Approved RFA | Populate only if status is Approved. | Phase 1 |
| Partner Name | `{{PARTNER_NAME}}` | Approved RFA / current package | Must reflect current campaign only. | Phase 1 |
| Type of Campaign | `{{TYPE_OF_CAMPAIGN}}` | BU manual / trusted source | If not derivable, use `MANUAL_INPUT_REQUIRED`. | Phase 1 |
| Campaign Period | `{{CAMPAIGN_PERIOD}}` | RFA / BU override | Use current campaign date range only. | Phase 1 |
| Funding Mechanism | `{{FUNDING_MECHANISM}}` | Approved RFA / BU confirmation | Flag if conflicts across sources. | Phase 1 |
| Budget / Approved Amount | `{{RFA_AMOUNT_RM}}` | Approved RFA | Do not infer from spend. | Phase 1 |
| Objectives / Mechanics | `{{COMMERCIAL_OBJECTIVES_TEXT}}`, `{{MECHANIC_TEXT}}` | Approved RFA | Preserve reviewable wording. | Phase 1 |

## Section B: Core Campaign KPI Block

| Generic Field | Placeholder | Source | Rule | Phase |
|---|---|---|---|---|
| Campaign Participants | `{{CAMPAIGN_PARTICIPANTS}}` | Partner raw data (provisional) | Distinct CHARGE users unless net rule is explicitly required. Phase 1: write partner-derived provisional value. Phase 2: overwrite with internal data. | Phase 1 (provisional, partner-derived) |
| Campaign TPV (RM) | `{{CAMPAIGN_TPV_RM}}` | Partner raw data (provisional) | CHARGE-only unless net rule is explicitly required. Phase 1: write partner-derived provisional value. Phase 2: overwrite with internal data. | Phase 1 (provisional, partner-derived) |
| Campaign Txn # | `{{CAMPAIGN_TXN_COUNT}}` | Partner raw data (provisional) | Distinct successful charge transactions. Phase 1: write partner-derived provisional value. Phase 2: overwrite with internal data. | Phase 1 (provisional, partner-derived) |
| Partner Incentive / CPAM Cost | `{{CPAM_COST_RM}}` | Partner raw / finance validation | Validate against policy where needed. Write from partner file if present; otherwise mark `PENDING_HUMAN_VALIDATION`. | Phase 1 (if in partner file) / Phase 2 |
| Internal TPV | `{{INTERNAL_TPV_RM}}` | Internal file | Final source-of-truth when available. Mark `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| Internal Participants | `{{INTERNAL_PARTICIPANTS}}` | Internal file | Final source-of-truth when available. Mark `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| Internal Txn # | `{{INTERNAL_TXN_COUNT}}` | Internal file | Final source-of-truth when available. Mark `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| TPV Variance | `{{TPV_VARIANCE_PCT}}` | Derived | `(Internal - Partner) / Internal` when both exist. Mark `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| Txn Variance | `{{TXN_VARIANCE_PCT}}` | Derived | Same rule. | Phase 2 |
| User Variance | `{{USER_VARIANCE_PCT}}` | Derived | Same rule. | Phase 2 |

## Section C: Retention Block

| Generic Field | Placeholder | Source | Rule | Phase |
|---|---|---|---|---|
| Post 1M Retention | `{{RETENTION_POST_1M}}` | Internal retention file | Cohort aligned to current campaign. Mark `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| Post 2M Retention | `{{RETENTION_POST_2M}}` | Internal retention file | Same rule. | Phase 2 |
| Post 3M Retention | `{{RETENTION_POST_3M}}` | Internal retention file | Same rule. | Phase 2 |
| Post 4M Retention | `{{RETENTION_POST_4M}}` | Internal retention file | Same rule. | Phase 2 |
| Post 5M Retention | `{{RETENTION_POST_5M}}` | Internal retention file | Same rule. | Phase 2 |
| Post 6M Retention | `{{RETENTION_POST_6M}}` | Internal retention file | Same rule. | Phase 2 |
| Post 7M Retention | `{{RETENTION_POST_7M}}` | Internal retention file | Same rule. | Phase 2 |
| Post 8M Retention | `{{RETENTION_POST_8M}}` | Internal retention file | Same rule. | Phase 2 |
| Post 9M Retention | `{{RETENTION_POST_9M}}` | Internal retention file | Same rule. | Phase 2 |
| Post 10M Retention | `{{RETENTION_POST_10M}}` | Internal retention file | Same rule. | Phase 2 |
| Post 11M Retention | `{{RETENTION_POST_11M}}` | Internal retention file | Same rule. | Phase 2 |
| Post 12M Retention | `{{RETENTION_POST_12M}}` | Internal retention file | Same rule. | Phase 2 |

## Section D: Revenue / Cost / ROI Logic

| Generic Field | Placeholder / Formula | Source | Rule | Phase |
|---|---|---|---|---|
| Campaign Revenue (RM) | `= {{CAMPAIGN_TPV_RM}} * {{MDR_RATE}}` | Derived | Keep as formula logic. | Phase 1 (formula depends on Phase 1 inputs) |
| MDR | `{{MDR_RATE}}` | Ignition prompt run parameter / approved finance source | Update per campaign run. Do not reuse prior campaign MDR. | Phase 1 |
| Campaign Total Cost (RM) | `= {{CPAM_COST_RM}} + {{CAMPAIGN_DIRECT_COST_RM}}` | Derived | Keep as formula logic. | Phase 2 (formula depends on Phase 2 cost inputs) |
| Campaign Direct Cost (RM) | `= ({{CAMPAIGN_TPV_RM}} * {{AVG_RELOAD_COST_PCT}}) + ({{CAMPAIGN_TXN_COUNT}} * {{AVG_CLOUD_COST_PER_TXN_RM}}) + ({{CAMPAIGN_TXN_COUNT}} * {{AVG_PLSA_COST_PER_TXN_RM}})` | Derived | Keep as formula logic. | Phase 2 (formula depends on Phase 2 rate inputs) |
| Avg Reload Cost % | `{{AVG_RELOAD_COST_PCT}}` | Finance source | Mark `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| Avg Cloud Cost / Txn (RM) | `{{AVG_CLOUD_COST_PER_TXN_RM}}` | Finance source | Mark `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| Avg PLSA Cost / Txn (RM) | `{{AVG_PLSA_COST_PER_TXN_RM}}` | Finance source | Mark `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| Gross Profit (RM) | `= {{CAMPAIGN_REVENUE_RM}} - {{CAMPAIGN_TOTAL_COST_RM}}` | Derived | Keep as formula logic. | Phase 2 (formula depends on Phase 2 total cost) |
| % ROI (GP / CPAM Cost) | `= {{GROSS_PROFIT_RM}} / {{CPAM_COST_RM}}` | Derived | Keep as formula logic. | Phase 2 (formula depends on Phase 2 gross profit) |

## Section E: Template Rule - Traffic Light (MDR%)
The workbook contains a template rule tied to MDR.

| Generic Field | Placeholder / Formula | Source | Rule | Phase |
|---|---|---|---|---|
| Traffic Light (MDR%) | `{{TRAFFIC_LIGHT_STATUS}}` or formula-driven cell | Derived from `{{MDR_RATE}}` | Keep formula if template calculates it. | Phase 1 |
| Traffic Light Rule | `Green / Yellow / Red` | Derived from `{{MDR_RATE}}` | Green if MDR >= 0.47%; Yellow if MDR >= 0.18% and < 0.47%; Red if MDR < 0.18%. | Phase 1 |

## Section F: Merchant-Level Impact Analysis

| Generic Field | Placeholder | Source | Rule | Phase |
|---|---|---|---|---|
| Pre-Campaign Monthly TPV | `{{PRE_CAMPAIGN_MONTHLY_TPV}}` | BI / internal | Current campaign merchant only. Mark `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| During-Campaign Monthly TPV | `{{DURING_CAMPAIGN_MONTHLY_TPV}}` | BI / internal | Same rule. | Phase 2 |
| Post 1M Monthly TPV | `{{POST_1M_MONTHLY_TPV}}` | BI / internal | Same rule. | Phase 2 |
| Post 2M Monthly TPV | `{{POST_2M_MONTHLY_TPV}}` | BI / internal | Same rule. | Phase 2 |
| Post 3M Monthly TPV | `{{POST_3M_MONTHLY_TPV}}` | BI / internal | Same rule. | Phase 2 |
| Post 6M Monthly TPV | `{{POST_6M_MONTHLY_TPV}}` | BI / internal | Same rule. | Phase 2 |
| Monthly MTU values | `{{PRE_DURING_POST_MTU_VALUES}}` | BI / internal | Use normalized month-level MTU values if required by template. Mark `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| TPV Pre vs During Summary | `{{TPV_PRE_VS_DURING_SUMMARY}}` | Derived narrative | Human-readable output based on current campaign data only. | Phase 2 |
| TPV Pre vs Post 3M Summary | `{{TPV_PRE_VS_POST3M_SUMMARY}}` | Derived narrative | Human-readable output based on current campaign data only. | Phase 2 |
| Proceed to 2nd Review | `{{SECOND_REVIEW_DECISION}}` | Business rule | Follow current template rule only. Mark `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| CPAM Cost / User | `= {{CPAM_COST_RM}} / {{CAMPAIGN_PARTICIPANTS}}` | Derived | Keep as formula logic. | Phase 1 (formula; resolves if CPAM Cost is available from partner file) |
| Est. 12M CLTV | `{{EST_12M_CLTV}}` | BI manual pull | Mark `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |

---

## Generic Validation / Helper Placeholders
Use these placeholders across campaigns when needed:
- `{{REFUND_TXN_COUNT}}`
- `{{REFUND_TPV_RM}}`
- `{{REFUND_INCENTIVE_RM}}`
- `{{VALIDATION_NOTE}}`
- `{{MISSING_INPUT_NOTE}}`
- `{{DATE_ALIGNMENT_NOTE}}`
- `{{SOURCE_CONFLICT_NOTE}}`
- `{{TRAFFIC_LIGHT_STATUS}}`

## Critical Rule
Any value shown in a sample workbook is **not** a framework default. During a live run, every populated value must come from the current upload set, the current run configuration, or remain unresolved.
