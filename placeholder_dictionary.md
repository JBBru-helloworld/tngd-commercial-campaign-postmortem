# Placeholder Dictionary

This dictionary is campaign agnostic. It defines what each placeholder means, where it lives, and how it should be populated. Formula-backed placeholders must keep their workbook formulas intact.

## General Info

| Placeholder | Field Name | Target Cell(s) | Write Action | Primary Source | Definition | Phase |
|---|---|---|---|---|---|---|
| `CAMPAIGN_NAME` | Campaign Name | `C2:H2` | `WRITE_INPUT` | Approved RFA (Sage) | Official approved campaign name used as the campaign identifier. | Phase 1 |
| `RFA_ID` | Approved RFA ID | `C3:H3` | `WRITE_INPUT` | Approved RFA (Sage) | Approved request-for-approval number. | Phase 1 |
| `PARTNER_NAME` | Partner Name | `C4:H4` | `WRITE_INPUT` | Approved RFA (Sage) | Merchant or partner name associated with the campaign. | Phase 1 |
| `CAMPAIGN_TYPE` | Type of Campaign | `C5:H5` | `WRITE_INPUT` | BU / Template Owner confirmation | Business classification such as CMS or Non CMS. | Phase 1 |
| `CAMPAIGN_START_DATE` | Campaign Start Date | `C7:E7` | `WRITE_INPUT` | Approved RFA (Sage) / BU override | Approved or BU-confirmed campaign live date. | Phase 1 |
| `CAMPAIGN_END_DATE` | Campaign End Date | `F7:H7` | `WRITE_INPUT` | Approved RFA (Sage) / BU override | Approved or BU-confirmed campaign end date. | Phase 1 |
| `FUNDING_MECHANISM` | Funding Mechanism | `C8:H8` | `WRITE_INPUT` | Approved RFA (Sage) / BU confirmation | Funding arrangement for the campaign. | Phase 1 |
| `MECHANIC_PRIMARY` | Campaign Mechanic | `C9:H10` | `WRITE_INPUT` | Approved RFA (Sage) | Primary campaign offer mechanics shown in one or more lines. | Phase 1 |
| `MECHANIC_SECONDARY` | Campaign Mechanic | `C11:H11` | `WRITE_INPUT` | Approved RFA (Sage) / BU confirmation | Secondary mechanic line where required by the campaign. | Phase 1 |
| `RFA_AMOUNT_RM` | RFA Approved Amount | `C12:H12` | `WRITE_INPUT` | Approved RFA (Sage) | Approved campaign budget or amount in RM. | Phase 1 |
| `COST_CENTRE` | Cost Centre | `C13:H13` | `WRITE_INPUT` | Approved RFA (Sage) | Owning cost centre for the campaign. | Phase 1 |

## Core KPIs

| Placeholder | Field Name | Target Cell(s) | Write Action | Primary Source | Definition | Phase |
|---|---|---|---|---|---|---|
| `CAMPAIGN_PARTICIPANTS` | Campaign Participants | `C14:H14` | `WRITE_INPUT` | Partner raw data (Phase 1 provisional) / Internal verification / TXN data (Phase 2 final) | Distinct qualified users in the campaign cohort. Phase 1: write partner-derived provisional value from CHARGE rows. Phase 2: overwrite with internal data. | Phase 1 (provisional, partner-derived) / Phase 2 (final, internal) |
| `CAMPAIGN_TPV_RM` | Campaign TPV | `C15:H15` | `WRITE_INPUT` | Partner raw data (Phase 1 provisional) / Internal verification / TXN data (Phase 2 final) | Total payment value for qualified campaign transactions. Phase 1: write partner-derived provisional value from CHARGE rows. Phase 2: overwrite with internal data. | Phase 1 (provisional, partner-derived) / Phase 2 (final, internal) |
| `CAMPAIGN_TXN_COUNT` | Campaign Transaction Count | `C16:H16` | `WRITE_INPUT` | Partner raw data (Phase 1 provisional) / Internal verification / TXN data (Phase 2 final) | Qualified campaign transaction count. Phase 1: write partner-derived provisional value from CHARGE rows. Phase 2: overwrite with internal data. | Phase 1 (provisional, partner-derived) / Phase 2 (final, internal) |

## Retention

| Placeholder | Field Name | Target Cell(s) | Write Action | Primary Source | Definition | Phase |
|---|---|---|---|---|---|---|
| `RET_P1` | Retention Post 1M | `C18` | `WRITE_INPUT` | Internal retention data | Observed campaign-user retention for post month 1. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `RET_P2` | Retention Post 2M | `D18` | `WRITE_INPUT` | Internal retention data | Observed campaign-user retention for post month 2. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `RET_P3` | Retention Post 3M | `E18` | `WRITE_INPUT` | Internal retention data | Observed campaign-user retention for post month 3. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `RET_P4` | Retention Post 4M | `F18` | `WRITE_INPUT` | Internal retention data | Observed campaign-user retention for post month 4. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `RET_P5` | Retention Post 5M | `G18` | `WRITE_INPUT` | Internal retention data | Observed campaign-user retention for post month 5. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `RET_P6` | Retention Post 6M | `H18` | `WRITE_INPUT` | Internal retention data | Observed campaign-user retention for post month 6. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `RET_P7` | Retention Post 7M | `C20` | `WRITE_INPUT` | Internal retention data | Observed campaign-user retention for post month 7. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `RET_P8` | Retention Post 8M | `D20` | `WRITE_INPUT` | Internal retention data | Observed campaign-user retention for post month 8. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `RET_P9` | Retention Post 9M | `E20` | `WRITE_INPUT` | Internal retention data | Observed campaign-user retention for post month 9. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `RET_P10` | Retention Post 10M | `F20` | `WRITE_INPUT` | Internal retention data | Observed campaign-user retention for post month 10. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `RET_P11` | Retention Post 11M | `G20` | `WRITE_INPUT` | Internal retention data | Observed campaign-user retention for post month 11. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `RET_P12` | Retention Post 12M | `H20` | `WRITE_INPUT` | Internal retention data | Observed campaign-user retention for post month 12. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |

## Revenue / Cost / ROI

| Placeholder | Field Name | Target Cell(s) | Write Action | Primary Source | Definition | Phase |
|---|---|---|---|---|---|---|
| `CAMPAIGN_REVENUE_RM` | Campaign Revenue | `C21:H21` | `KEEP_FORMULA` | Workbook formula logic | Derived from Campaign TPV x MDR. | Phase 1 (KEEP_FORMULA) |
| `MDR_RATE` | MDR Rate | `C22:H22` | `WRITE_INPUT` | Ignition prompt run parameter / approved finance or BU source | Campaign-specific MDR percentage used for campaign revenue and traffic-light logic. | Phase 1 |
| `CAMPAIGN_TOTAL_COST_RM` | Campaign Total Cost | `C23:H23` | `KEEP_FORMULA` | Workbook formula logic | Derived from CPAM Cost + Direct Cost. | Phase 2 (KEEP_FORMULA — resolves when Phase 2 inputs are populated) |
| `CPAM_COST_RM` | CPAM Cost | `C24:H24` | `WRITE_INPUT` | Partner file (Phase 1) / Internal verification / finance validation (Phase 2) | Confirmed CPAM or funded incentive cost attributable to the campaign. Phase 1: write from partner file if available; otherwise write `PENDING_HUMAN_VALIDATION`. | Phase 1 (if in partner file) / Phase 2 |
| `DIRECT_COST_RM` | Campaign Direct Cost | `C25:H25` | `KEEP_FORMULA` | Workbook formula logic | Derived from TPV and transaction-based finance cost rates. | Phase 2 (KEEP_FORMULA — resolves when Phase 2 rate inputs are populated) |
| `AVG_RELOAD_PCT` | Average Reload Cost % | `C26:H26, R12` | `WRITE_INPUT` | Finance cost-rate source / finance workbook | Average reload cost rate applied to TPV direct-cost calculation. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `AVG_CLOUD_COST_RM` | Average Cloud Cost per Txn | `C27:H27, R13` | `WRITE_INPUT` | Finance cost-rate source / finance workbook | Average cloud cost per transaction applied to direct-cost calculation. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `AVG_PLSA_COST_RM` | Average PLSA Cost per Txn | `C28:H28, R14` | `WRITE_INPUT` | Finance cost-rate source / finance workbook | Average PLSA cost per transaction applied to direct-cost calculation. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `GROSS_PROFIT_RM` | Gross Profit | `C29:H29` | `KEEP_FORMULA` | Workbook formula logic | Derived from Campaign Revenue - Campaign Total Cost. | Phase 2 (KEEP_FORMULA — resolves when Phase 2 cost inputs are populated) |
| `ROI_PCT` | ROI % | `C30:H30` | `KEEP_FORMULA` | Workbook formula logic | Derived from Gross Profit / CPAM Cost. | Phase 2 (KEEP_FORMULA — resolves when Phase 2 inputs are populated) |

## Merchant-Level Impact Analysis

| Placeholder | Field Name | Target Cell(s) | Write Action | Primary Source | Definition | Phase |
|---|---|---|---|---|---|---|
| `TPV_PRE_T` | Merchant Impact TPV Display - Pre-Campaign | `C32` | `WRITE_INPUT` | Internal BI / merchant-impact dataset | Presentation-layer TPV value for the Pre-Campaign bucket in the main template. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `TPV_DUR_T` | Merchant Impact TPV Display - During Campaign | `D32` | `WRITE_INPUT` | Internal BI / merchant-impact dataset | Presentation-layer TPV value for the During Campaign bucket in the main template. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `TPV_P1_T` | Merchant Impact TPV Display - Post 1M | `E32` | `WRITE_INPUT` | Internal BI / merchant-impact dataset | Presentation-layer TPV value for the Post 1M bucket in the main template. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `TPV_P2_T` | Merchant Impact TPV Display - Post 2M | `F32` | `WRITE_INPUT` | Internal BI / merchant-impact dataset | Presentation-layer TPV value for the Post 2M bucket in the main template. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `TPV_P3_T` | Merchant Impact TPV Display - Post 3M | `G32` | `WRITE_INPUT` | Internal BI / merchant-impact dataset | Presentation-layer TPV value for the Post 3M bucket in the main template. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `TPV_P6_T` | Merchant Impact TPV Display - Post 6M | `H32` | `WRITE_INPUT` | Internal BI / merchant-impact dataset | Presentation-layer TPV value for the Post 6M bucket in the main template. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `GR_PRE_T` | Merchant Impact TPV Growth Display - Pre-Campaign | `C33` | `WRITE_INPUT` | Internal BI / merchant-impact dataset | Presentation-layer TPV growth value for the Pre-Campaign bucket. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `GR_DUR_T` | Merchant Impact TPV Growth Display - During Campaign | `D33` | `WRITE_INPUT` | Internal BI / merchant-impact dataset | Presentation-layer TPV growth value for the During Campaign bucket. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `GR_P1_T` | Merchant Impact TPV Growth Display - Post 1M | `E33` | `WRITE_INPUT` | Internal BI / merchant-impact dataset | Presentation-layer TPV growth value for the Post 1M bucket. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `GR_P2_T` | Merchant Impact TPV Growth Display - Post 2M | `F33` | `WRITE_INPUT` | Internal BI / merchant-impact dataset | Presentation-layer TPV growth value for the Post 2M bucket. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `GR_P3_T` | Merchant Impact TPV Growth Display - Post 3M | `G33` | `WRITE_INPUT` | Internal BI / merchant-impact dataset | Presentation-layer TPV growth value for the Post 3M bucket. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `GR_P6_T` | Merchant Impact TPV Growth Display - Post 6M | `H33` | `WRITE_INPUT` | Internal BI / merchant-impact dataset | Presentation-layer TPV growth value for the Post 6M bucket. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `TPV_PRE_V` | Merchant Impact TPV Value - Pre-Campaign | `C34` | `WRITE_INPUT` | Internal BI / merchant-impact dataset | Numeric merchant-level TPV value for the Pre-Campaign bucket. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `TPV_DUR_V` | Merchant Impact TPV Value - During Campaign | `D34, R4` | `WRITE_INPUT` | Internal BI / merchant-impact dataset | Numeric merchant-level TPV value for the During Campaign bucket. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `TPV_P1_V` | Merchant Impact TPV Value - Post 1M | `E34, S4` | `WRITE_INPUT` | Internal BI / merchant-impact dataset | Numeric merchant-level TPV value for the Post 1M bucket. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `TPV_P2_V` | Merchant Impact TPV Value - Post 2M | `F34, T4` | `WRITE_INPUT` | Internal BI / merchant-impact dataset | Numeric merchant-level TPV value for the Post 2M bucket. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `TPV_P3_V` | Merchant Impact TPV Value - Post 3M | `G34, U4` | `WRITE_INPUT` | Internal BI / merchant-impact dataset | Numeric merchant-level TPV value for the Post 3M bucket. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `TPV_P6_V` | Merchant Impact TPV Value - Post 6M | `H34` | `WRITE_INPUT` | Internal BI / merchant-impact dataset | Numeric merchant-level TPV value for the Post 6M bucket. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `GR_PRE_V` | Merchant Impact TPV Growth Value - Pre-Campaign | `C35` | `WRITE_INPUT` | Internal BI / merchant-impact dataset | Numeric TPV growth amount for the Pre-Campaign bucket. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `GR_DUR_V` | Merchant Impact TPV Growth Value - During Campaign | `D35` | `WRITE_INPUT` | Internal BI / merchant-impact dataset | Numeric TPV growth amount for the During Campaign bucket. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `GR_P1_V` | Merchant Impact TPV Growth Value - Post 1M | `E35` | `WRITE_INPUT` | Internal BI / merchant-impact dataset | Numeric TPV growth amount for the Post 1M bucket. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `GR_P2_V` | Merchant Impact TPV Growth Value - Post 2M | `F35` | `WRITE_INPUT` | Internal BI / merchant-impact dataset | Numeric TPV growth amount for the Post 2M bucket. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `GR_P3_V` | Merchant Impact TPV Growth Value - Post 3M | `G35` | `WRITE_INPUT` | Internal BI / merchant-impact dataset | Numeric TPV growth amount for the Post 3M bucket. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `GR_P6_V` | Merchant Impact TPV Growth Value - Post 6M | `H35` | `WRITE_INPUT` | Internal BI / merchant-impact dataset | Numeric TPV growth amount for the Post 6M bucket. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `GR_PRE_%` | Merchant Impact TPV Growth % - Pre-Campaign | `C36` | `WRITE_INPUT` | Internal BI / merchant-impact dataset | Optional percentage TPV growth metric for the Pre-Campaign bucket. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `GR_DUR_%` | Merchant Impact TPV Growth % - During Campaign | `D36` | `WRITE_INPUT` | Internal BI / merchant-impact dataset | Optional percentage TPV growth metric for the During Campaign bucket. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `GR_P1_%` | Merchant Impact TPV Growth % - Post 1M | `E36` | `WRITE_INPUT` | Internal BI / merchant-impact dataset | Optional percentage TPV growth metric for the Post 1M bucket. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `GR_P2_%` | Merchant Impact TPV Growth % - Post 2M | `F36` | `WRITE_INPUT` | Internal BI / merchant-impact dataset | Optional percentage TPV growth metric for the Post 2M bucket. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `GR_P3_%` | Merchant Impact TPV Growth % - Post 3M | `G36` | `WRITE_INPUT` | Internal BI / merchant-impact dataset | Optional percentage TPV growth metric for the Post 3M bucket. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `GR_P6_%` | Merchant Impact TPV Growth % - Post 6M | `H36` | `WRITE_INPUT` | Internal BI / merchant-impact dataset | Optional percentage TPV growth metric for the Post 6M bucket. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `MTU_PRE_V` | Monthly MTU - Pre-Campaign | `C37` | `WRITE_INPUT` | Internal BI / merchant-impact dataset | Monthly transacting users for the Pre-Campaign bucket. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `MTU_DUR_V` | Monthly MTU - During Campaign | `D37` | `WRITE_INPUT` | Internal BI / merchant-impact dataset | Monthly transacting users for the During Campaign bucket. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `MTU_P1_V` | Monthly MTU - Post 1M | `E37` | `WRITE_INPUT` | Internal BI / merchant-impact dataset | Monthly transacting users for the Post 1M bucket. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `MTU_P2_V` | Monthly MTU - Post 2M | `F37` | `WRITE_INPUT` | Internal BI / merchant-impact dataset | Monthly transacting users for the Post 2M bucket. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `MTU_P3_V` | Monthly MTU - Post 3M | `G37` | `WRITE_INPUT` | Internal BI / merchant-impact dataset | Monthly transacting users for the Post 3M bucket. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `MTU_P6_V` | Monthly MTU - Post 6M | `H37` | `WRITE_INPUT` | Internal BI / merchant-impact dataset | Monthly transacting users for the Post 6M bucket. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `TRAFFIC_LIGHT_STATUS` | Traffic Light (MDR%) | `C38:H38` | `KEEP_FORMULA` | Derived from MDR_RATE using template rule | Template-derived traffic-light classification. Green if MDR >= 0.47%; Yellow if MDR >= 0.18% and < 0.47%; Red if MDR < 0.18%. | Phase 1 (KEEP_FORMULA) |
| `TPV_PRE_VS_DURING_SUMMARY` | TPV Pre vs During Summary | `C39:H39` | `WRITE_INPUT` | Derived narrative from merchant-impact values | Human-readable summary of pre-campaign versus during-campaign TPV movement. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `TPV_PRE_VS_POST_3M_SUMMARY` | TPV Pre vs Post 3M Summary | `C40:H40` | `WRITE_INPUT` | Derived narrative from merchant-impact values | Human-readable summary of pre-campaign versus post-3M TPV movement. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `SECOND_REVIEW_DECISION` | Proceed to 2nd Review | `C41:G41` | `WRITE_INPUT` | BU / Template Owner confirmation | Go / no-go / review decision for second review. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `CPAM_COST_PER_USER_RM` | CPAM Cost per User | `C42:H42` | `KEEP_FORMULA` | Workbook formula logic | Derived from CPAM Cost / Campaign Participants. | Phase 1 (KEEP_FORMULA — resolves if CPAM Cost available from partner file) |
| `EST_12M_CLTV` | Estimated 12M CLTV | `C43:G43` | `WRITE_INPUT` | Internal BI / merchant-impact dataset | Approved 12-month customer lifetime value estimate for the relevant campaign cohort. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |

## Source Helper - TPV Impact

| Placeholder | Field Name | Target Cell(s) | Write Action | Primary Source | Definition | Phase |
|---|---|---|---|---|---|---|
| `SOURCE_LINK_OR_FILE_REFERENCE` | Source Registration Reference | `B1` | `WRITE_INPUT` | Uploaded file package | Reference to the uploaded source file or link used for the helper sheet. | Phase 1 |
| `TPV_M2_V` | Helper TPV Value - Month 2 source bucket | `P4` | `WRITE_INPUT` | Internal BI / merchant-impact dataset | Helper-sheet TPV input for pre-campaign month bucket. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `TPV_M1_V` | Helper TPV Value - Month 1 source bucket | `Q4` | `WRITE_INPUT` | Internal BI / merchant-impact dataset | Helper-sheet TPV input for pre-campaign month bucket. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `GR_M2_RM` | Helper TPV Growth RM - Month 2 source bucket | `P5` | `WRITE_INPUT` | Internal BI / merchant-impact dataset | Helper-sheet growth value for the pre-campaign month bucket. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `GR_M1_RM` | Helper TPV Growth RM - Month 1 source bucket | `Q5` | `WRITE_INPUT` | Internal BI / merchant-impact dataset | Helper-sheet growth value for the pre-campaign month bucket. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `GR_DUR_RM` | Helper TPV Growth RM - During Campaign | `R5` | `KEEP_FORMULA` | Workbook formula logic | Formula-driven helper growth value for the During bucket. | Phase 2 (KEEP_FORMULA) |
| `GR_P1_RM` | Helper TPV Growth RM - Post 1M | `S5` | `KEEP_FORMULA` | Workbook formula logic | Formula-driven helper growth value for the Post 1M bucket. | Phase 2 (KEEP_FORMULA) |
| `GR_P2_RM` | Helper TPV Growth RM - Post 2M | `T5` | `KEEP_FORMULA` | Workbook formula logic | Formula-driven helper growth value for the Post 2M bucket. | Phase 2 (KEEP_FORMULA) |
| `GR_P3_RM` | Helper TPV Growth RM - Post 3M | `U5` | `KEEP_FORMULA` | Workbook formula logic | Formula-driven helper growth value for the Post 3M bucket. | Phase 2 (KEEP_FORMULA) |
| `TPV_DUR_RM` | Helper TPV During vs Pre | `Q7` | `KEEP_FORMULA` | Workbook helper formula | Derived helper-sheet delta between During and Pre 1M. | Phase 2 (KEEP_FORMULA) |
| `TPV_POST_RM` | Helper TPV Post vs Pre | `Q8` | `KEEP_FORMULA` | Workbook helper formula | Derived helper-sheet delta between average Post 1M-3M and Pre 1M. | Phase 2 (KEEP_FORMULA) |

## Source Helper - Direct Cost

| Placeholder | Field Name | Target Cell(s) | Write Action | Primary Source | Definition | Phase |
|---|---|---|---|---|---|---|
| `COST_M1` | Finance Rate Month Label | `P11` | `WRITE_INPUT` | Finance cost-rate source / finance workbook | Month labels or source-period tags used for direct-cost helper inputs. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `COST_M2` | Finance Rate Month Label | `Q11` | `WRITE_INPUT` | Finance cost-rate source / finance workbook | Month labels or source-period tags used for direct-cost helper inputs. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `RLD_M1` | Reload Cost Rate - Month 1 | `P12` | `WRITE_INPUT` | Finance cost-rate source / finance workbook | Finance helper input for reload cost rate for month 1. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `RLD_M2` | Reload Cost Rate - Month 2 | `Q12` | `WRITE_INPUT` | Finance cost-rate source / finance workbook | Finance helper input for reload cost rate for month 2. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `CLD_M1` | Cloud Cost per Txn - Month 1 | `P13` | `WRITE_INPUT` | Finance cost-rate source / finance workbook | Finance helper input for cloud cost per txn for month 1. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `CLD_M2` | Cloud Cost per Txn - Month 2 | `Q13` | `WRITE_INPUT` | Finance cost-rate source / finance workbook | Finance helper input for cloud cost per txn for month 2. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `PLSA_M1` | PLSA Cost per Txn - Month 1 | `P14` | `WRITE_INPUT` | Finance cost-rate source / finance workbook | Finance helper input for PLSA cost per txn for month 1. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |
| `PLSA_M2` | PLSA Cost per Txn - Month 2 | `Q14` | `WRITE_INPUT` | Finance cost-rate source / finance workbook | Finance helper input for PLSA cost per txn for month 2. Write `PENDING_HUMAN_VALIDATION` in Phase 1. | Phase 2 |

## System Placeholders

| Placeholder | Field Name | Target Cell(s) | Write Action | Primary Source | Definition | Phase |
|---|---|---|---|---|---|---|
| `PENDING_HUMAN_VALIDATION` | Pending Human Validation | Multiple Phase 2 cells | `WRITE_INPUT` | Phase 2 human input | This field requires human input in Phase 2. It will be filled by the team member validating the partner file against internal database records. Do not treat as a missing-data error. | Phase 2 |
