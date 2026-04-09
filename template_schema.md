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

| Generic Field | Placeholder | Source | Rule |
|---|---|---|---|
| Campaign Name | `{{CAMPAIGN_NAME}}` | Approved RFA | Copy from current RFA only. |
| RFA No. | `{{APPROVED_RFA_NO}}` | Approved RFA | Populate only if status is Approved. |
| Partner Name | `{{PARTNER_NAME}}` | Approved RFA / current package | Must reflect current campaign only. |
| Type of Campaign | `{{TYPE_OF_CAMPAIGN}}` | BU manual / trusted source | If not derivable, use `MANUAL_INPUT_REQUIRED`. |
| Campaign Period | `{{CAMPAIGN_PERIOD}}` | RFA / BU override | Use current campaign date range only. |
| Funding Mechanism | `{{FUNDING_MECHANISM}}` | Approved RFA / BU confirmation | Flag if conflicts across sources. |
| Budget / Approved Amount | `{{RFA_AMOUNT_RM}}` | Approved RFA | Do not infer from spend. |
| Objectives / Mechanics | `{{COMMERCIAL_OBJECTIVES_TEXT}}`, `{{MECHANIC_TEXT}}` | Approved RFA | Preserve reviewable wording. |

## Section B: Core Campaign KPI Block

| Generic Field | Placeholder | Source | Rule |
|---|---|---|---|
| Campaign Participants | `{{CAMPAIGN_PARTICIPANTS}}` | Partner raw / internal | Distinct CHARGE users unless net rule is explicitly required. |
| Campaign TPV (RM) | `{{CAMPAIGN_TPV_RM}}` | Partner raw / internal | CHARGE-only unless net rule is explicitly required. |
| Campaign Txn # | `{{CAMPAIGN_TXN_COUNT}}` | Partner raw / internal | Distinct successful charge transactions. |
| Partner Incentive / CPAM Cost | `{{CPAM_COST_RM}}` | Partner raw / finance validation | Validate against policy where needed. |
| Internal TPV | `{{INTERNAL_TPV_RM}}` | Internal file | Final source-of-truth when available. |
| Internal Participants | `{{INTERNAL_PARTICIPANTS}}` | Internal file | Final source-of-truth when available. |
| Internal Txn # | `{{INTERNAL_TXN_COUNT}}` | Internal file | Final source-of-truth when available. |
| TPV Variance | `{{TPV_VARIANCE_PCT}}` | Derived | `(Internal - Partner) / Internal` when both exist. |
| Txn Variance | `{{TXN_VARIANCE_PCT}}` | Derived | Same rule. |
| User Variance | `{{USER_VARIANCE_PCT}}` | Derived | Same rule. |

## Section C: Retention Block

| Generic Field | Placeholder | Source | Rule |
|---|---|---|---|
| Post 1M Retention | `{{RETENTION_POST_1M}}` | Internal retention file | Cohort aligned to current campaign. |
| Post 2M Retention | `{{RETENTION_POST_2M}}` | Internal retention file | Same rule. |
| Post 3M Retention | `{{RETENTION_POST_3M}}` | Internal retention file | Same rule. |
| Post 4M Retention | `{{RETENTION_POST_4M}}` | Internal retention file | Same rule. |
| Post 5M Retention | `{{RETENTION_POST_5M}}` | Internal retention file | Same rule. |
| Post 6M Retention | `{{RETENTION_POST_6M}}` | Internal retention file | Same rule. |
| Post 7M Retention | `{{RETENTION_POST_7M}}` | Internal retention file | Same rule. |
| Post 8M Retention | `{{RETENTION_POST_8M}}` | Internal retention file | Same rule. |
| Post 9M Retention | `{{RETENTION_POST_9M}}` | Internal retention file | Same rule. |
| Post 10M Retention | `{{RETENTION_POST_10M}}` | Internal retention file | Same rule. |
| Post 11M Retention | `{{RETENTION_POST_11M}}` | Internal retention file | Same rule. |
| Post 12M Retention | `{{RETENTION_POST_12M}}` | Internal retention file | Same rule. |

## Section D: Revenue / Cost / ROI Logic

| Generic Field | Placeholder / Formula | Source | Rule |
|---|---|---|---|
| Campaign Revenue (RM) | `= {{CAMPAIGN_TPV_RM}} * {{MDR_RATE}}` | Derived | Keep as formula logic. |
| MDR | `{{MDR_RATE}}` | Ignition prompt run parameter / approved finance source | Update per campaign run. Do not reuse prior campaign MDR. |
| Campaign Total Cost (RM) | `= {{CPAM_COST_RM}} + {{CAMPAIGN_DIRECT_COST_RM}}` | Derived | Keep as formula logic. |
| Campaign Direct Cost (RM) | `= ({{CAMPAIGN_TPV_RM}} * {{AVG_RELOAD_COST_PCT}}) + ({{CAMPAIGN_TXN_COUNT}} * {{AVG_CLOUD_COST_PER_TXN_RM}}) + ({{CAMPAIGN_TXN_COUNT}} * {{AVG_PLSA_COST_PER_TXN_RM}})` | Derived | Keep as formula logic. |
| Avg Reload Cost % | `{{AVG_RELOAD_COST_PCT}}` | Finance source | Mark unresolved if missing. |
| Avg Cloud Cost / Txn (RM) | `{{AVG_CLOUD_COST_PER_TXN_RM}}` | Finance source | Mark unresolved if missing. |
| Avg PLSA Cost / Txn (RM) | `{{AVG_PLSA_COST_PER_TXN_RM}}` | Finance source | Mark unresolved if missing. |
| Gross Profit (RM) | `= {{CAMPAIGN_REVENUE_RM}} - {{CAMPAIGN_TOTAL_COST_RM}}` | Derived | Keep as formula logic. |
| % ROI (GP / CPAM Cost) | `= {{GROSS_PROFIT_RM}} / {{CPAM_COST_RM}}` | Derived | Keep as formula logic. |

## Section E: Template Rule - Traffic Light (MDR%)
The workbook contains a template rule tied to MDR.

| Generic Field | Placeholder / Formula | Source | Rule |
|---|---|---|---|
| Traffic Light (MDR%) | `{{TRAFFIC_LIGHT_STATUS}}` or formula-driven cell | Derived from `{{MDR_RATE}}` | Keep formula if template calculates it. |
| Traffic Light Rule | `Green / Yellow / Red` | Derived from `{{MDR_RATE}}` | Green if MDR >= 0.47%; Yellow if MDR >= 0.18% and < 0.47%; Red if MDR < 0.18%. |

## Section F: Merchant-Level Impact Analysis

| Generic Field | Placeholder | Source | Rule |
|---|---|---|---|
| Pre-Campaign Monthly TPV | `{{PRE_CAMPAIGN_MONTHLY_TPV}}` | BI / internal | Current campaign merchant only. |
| During-Campaign Monthly TPV | `{{DURING_CAMPAIGN_MONTHLY_TPV}}` | BI / internal | Same rule. |
| Post 1M Monthly TPV | `{{POST_1M_MONTHLY_TPV}}` | BI / internal | Same rule. |
| Post 2M Monthly TPV | `{{POST_2M_MONTHLY_TPV}}` | BI / internal | Same rule. |
| Post 3M Monthly TPV | `{{POST_3M_MONTHLY_TPV}}` | BI / internal | Same rule. |
| Post 6M Monthly TPV | `{{POST_6M_MONTHLY_TPV}}` | BI / internal | Same rule. |
| Monthly MTU values | `{{PRE_DURING_POST_MTU_VALUES}}` | BI / internal | Use normalized month-level MTU values if required by template. |
| TPV Pre vs During Summary | `{{TPV_PRE_VS_DURING_SUMMARY}}` | Derived narrative | Human-readable output based on current campaign data only. |
| TPV Pre vs Post 3M Summary | `{{TPV_PRE_VS_POST3M_SUMMARY}}` | Derived narrative | Human-readable output based on current campaign data only. |
| Proceed to 2nd Review | `{{SECOND_REVIEW_DECISION}}` | Business rule | Follow current template rule only. |
| CPAM Cost / User | `= {{CPAM_COST_RM}} / {{CAMPAIGN_PARTICIPANTS}}` | Derived | Keep as formula logic. |
| Est. 12M CLTV | `{{EST_12M_CLTV}}` | BI manual pull | Mark `MANUAL_INPUT_REQUIRED` if unavailable. |

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