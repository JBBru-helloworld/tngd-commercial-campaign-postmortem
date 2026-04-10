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
   - RFA / Sage approval file — **Phase 1 Required**
   - partner raw data file(s) — **Phase 1 Required**
   - internal validation / transaction data file(s) — Phase 2 (human-completed)
   - retention data file(s), if provided — Phase 2 (human-completed)
   - finance rate / cost input file(s), if provided — Phase 2 (human-completed)

The master template workbook is the design source of truth. Preserve the same workbook structure, sheet names, colours, borders, merged cells, formulas, notes, layout, and overall visual design.

## Two-Phase Workflow Overview

The engine runs in two distinct phases. The AI executes Phase 1 only. Phase 2 is completed by a human team member.

### Phase 1 — AI Run
- **Inputs:** RFA file + partner raw data file + ignition prompt parameters (campaign name, MDR).
- **What the AI does:** Populates all fields derivable from the RFA (campaign identity block) and partner raw data (provisional KPIs), writes the MDR, evaluates the traffic-light classification, and writes `PENDING_HUMAN_VALIDATION` to all Phase 2 fields.
- **Output:** A partially completed Excel workbook ready for Phase 2 human validation.

### Phase 2 — Human Validation
- **Who:** A team member receives the Phase 1 output Excel file.
- **What they do:** Validates the partner file against internal database files and fills in all remaining sections: internal KPI reconciliation, internal TPV / participants / transaction counts, all retention data, all finance cost rates, and all merchant-impact / BI data.
- **Inputs for Phase 2:** Internal TXN data, internal retention files, finance rate files, BI / merchant-impact dataset.

### AI Behavior at Phase Boundary
- The AI must **never** block, error, or stop because internal data, retention files, finance files, or BI files are absent.
- If a Phase 2 file is missing, the AI writes `PENDING_HUMAN_VALIDATION` to all cells that depend on that file and continues.
- `PENDING_HUMAN_VALIDATION` signals to the human reviewer that the field is expected to be filled — it is not a missing-data error.
- The AI must **only** stop and return `MANUAL_REVIEW_REQUIRED` if the Phase 1 required files (RFA or partner raw data) are missing or if the RFA is not in Approved status.

## Core execution rules

- Duplicate the master template before population.
- Read the static engine sources before touching the campaign-specific uploads.
- Read the workbook's embedded **Placeholder Dictionary**, **Cell-to-Source Mapping**, and **Engine Runbook** tabs before writing any output cells.
- Register the current-run MDR from the ignition prompt before populating the workbook.
- Only write into cells marked `WRITE_INPUT` or other explicit input actions.
- Never overwrite cells marked `KEEP_FORMULA`.
- Never estimate missing values. Use `DATA_NOT_FOUND` or `MANUAL_INPUT_REQUIRED` for genuinely unresolvable Phase 1 fields. Use `PENDING_HUMAN_VALIDATION` for all Phase 2 fields.
- Use the RFA as the source of truth for approved campaign identity and approved budget fields, only after confirming approval status.
- Internal verification / TXN data is the final KPI source when it conflicts with partner raw data — but this reconciliation is a Phase 2 step.
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
**Phase 1 required files (AI run):** RFA / Sage approval file and partner raw data file. These are the only files required for Phase 1. If either is missing, stop and return `MANUAL_REVIEW_REQUIRED`.
**Phase 2 files (human-completed):** Internal verification / TXN data, retention data, and finance rate source files. If these files are not uploaded, do not treat this as a blocking error. Mark all cells that depend on these files `PENDING_HUMAN_VALIDATION` and continue the Phase 1 run.
**Do:** Register any uploaded files in the Source File Template and in the validation log. Note Phase 2 files as "to be completed in Phase 2."
**Do not:** Do not flag the absence of Phase 2 files as an error. Do not infer a source from memory.
**Checkpoint:** Source inventory completed with Phase 1 / Phase 2 status noted.
**Guardrail:** Only Phase 1 files are blocking. Phase 2 files are optional for the AI run.

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

### Step 6 - Normalize partner raw data (Phase 1)
**Do:** Map the partner raw file into canonical fields: qualified transactions, qualified users, TPV, refunds, mechanics / waves, and campaign date coverage. Produce provisional KPI values (campaign participants, TPV, transaction count) from the partner CHARGE rows. Label these as provisional (partner-derived) in the output. Preserve raw partner totals for later variance checks by the human reviewer.
**Do not:** Do not mix refunds into the primary KPI block unless a finance-approved net rule exists.
**Checkpoint:** Partner staging summary completed. Provisional KPIs populated in Phase 1 cells.
**Guardrail:** Partner-derived KPIs are provisional. Internal verification (Phase 2) is the final source of truth.

### Step 7 - Normalize internal verification data (Phase 2 — human-completed)
**This step is a Phase 2 human-completed step. The AI must not attempt to perform this step.**
**Do:** Leave all cells in the internal KPI reconciliation section marked `PENDING_HUMAN_VALIDATION`. Include a note that the team member will populate these from the internal database after the AI run.
**Do not:** Do not populate internal KPI cells from partner data as a substitute. Do not silently prefer partner data over internal data.
**Checkpoint:** Internal KPI cells written as `PENDING_HUMAN_VALIDATION`.
**Guardrail:** Internal data normalization is reserved for Phase 2.

### Step 8 - Reconcile discrepancies (Phase 2 — human-completed)
**This step is a Phase 2 human-completed step. The AI must not attempt to perform this step.**
**Do:** Leave all KPI reconciliation fields (TPV variance, transaction variance, user variance) marked `PENDING_HUMAN_VALIDATION`. Include a note that the human reviewer will compare partner versus internal KPI values and log variances.
**Do not:** Do not average the two sources or choose whichever looks better. Do not flag the absence of internal data as an error.
**Checkpoint:** KPI reconciliation cells written as `PENDING_HUMAN_VALIDATION`.
**Guardrail:** KPI reconciliation is reserved for Phase 2.

### Step 9 - Populate retention (Phase 2 — human-completed)
**This step is a Phase 2 human-completed step. The AI must not attempt to perform this step.**
**Do:** Leave all retention fields marked `PENDING_HUMAN_VALIDATION`. Include a note that the team member will populate these from the internal retention source aligned to the campaign cohort and post-campaign month buckets.
**Do not:** Do not use assumed rates from the RFA as actual retention. Do not flag missing retention files as an error during Phase 1.
**Checkpoint:** All retention cells written as `PENDING_HUMAN_VALIDATION`.
**Guardrail:** Retention population is reserved for Phase 2.

### Step 10 - Populate MDR and finance inputs
**Phase 1 (AI populates):** Write the campaign-specific MDR input from the ignition prompt into the workbook MDR field. If CPAM Cost is available in the partner file, populate it; otherwise write `PENDING_HUMAN_VALIDATION`.
**Phase 2 (human-completed):** Finance cost-rate inputs — Avg Reload Cost %, Avg Cloud Cost/Txn, and Avg PLSA Cost/Txn — are Phase 2 human inputs. The AI must write `PENDING_HUMAN_VALIDATION` to these fields. Do not infer or estimate these rates.
**Do not:** Do not copy sample finance values from the workbook. Do not overwrite the traffic-light cell if it is formula-driven.
**Checkpoint:** MDR written from ignition prompt. Finance rate fields written as `PENDING_HUMAN_VALIDATION`.
**Guardrail:** Finance-controlled rate fields remain Phase 2 human inputs only.

### Step 11 - Apply the MDR traffic-light rule
**Do:** Preserve the template logic for the traffic-light row. The result must follow this rule:
- Green if MDR >= 0.47%
- Yellow if MDR >= 0.18% and MDR < 0.47%
- Red if MDR < 0.18%

**Do not:** Do not replace the formula with hardcoded text if the template already derives the result.
**Checkpoint:** Traffic-light field resolved or appropriately flagged.
**Guardrail:** The template rule must be preserved exactly.

### Step 12 - Populate merchant-impact analysis (Phase 2 — human-completed)
**This step is a Phase 2 human-completed step. The AI must not attempt to perform this step.**
**Do:** Write `PENDING_HUMAN_VALIDATION` to all merchant-impact fields (TPV series, MTU series, growth values, CLTV, second-review decision).
**Do not:** Do not invent baseline growth conventions. Do not flag missing BI data as an error during Phase 1.
**Checkpoint:** All merchant-impact cells written as `PENDING_HUMAN_VALIDATION`.
**Guardrail:** Merchant-impact data requires Phase 2 BI / internal database pulls.

### Step 13 - Leave formulas intact
**Do:** For revenue, total cost, direct cost, gross profit, ROI, CPAM cost per user, and helper-derived deltas, keep the workbook formulas exactly as-is. Only populate their dependency cells.
**Do not:** Do not replace formulas with hardcoded numbers.
**Checkpoint:** Derived rows calculate automatically.
**Guardrail:** Formula cells are part of the template logic.

### Step 14 - Write narratives and decisions last
**Do:** After all Phase 1 numeric sections are validated, write concise narrative summaries where derivable from Phase 1 data. Leave Phase 2 narrative fields marked `PENDING_HUMAN_VALIDATION`.
**Do not:** Do not let narratives contradict the numbers. Do not auto-approve 2nd review without a rule or confirmation.
**Checkpoint:** Phase 1 narrative fields completed where possible.
**Guardrail:** Narratives must be evidence-based.

### Step 15 - Quality assurance and export
**Do:** Run final checks: no PII, Phase 1 fields fully populated, all Phase 2 fields clearly marked `PENDING_HUMAN_VALIDATION`, no broken formulas, date alignment validated, MDR and traffic-light rule validated, and all genuinely unresolved Phase 1 items marked `DATA_NOT_FOUND` or `MANUAL_INPUT_REQUIRED`.
**Do not:** Do not ship a workbook that contains guessed values or overwritten formula cells.
**Checkpoint:** Workbook ready for Phase 2 handoff.
**Guardrail:** Zero-hallucination policy applies to every field.

## Final required outputs

The engine must return exactly **2 outputs only**:

1. **Partially completed Excel file (Phase 1 output)**
   - Partially completed workbook ready for Phase 2 human validation
   - Phase 1 sections (campaign identity, provisional partner KPIs, MDR, traffic light) fully populated
   - All Phase 2 sections clearly marked `PENDING_HUMAN_VALIDATION` for the human reviewer
   - Matches the master template design exactly
   - Preserves formatting, colours, formulas, structure, and worksheet design

2. **Summary in chat**
   - campaign identification details used
   - files successfully read
   - MDR percentage used
   - traffic-light result
   - major findings
   - provisional partner KPI values populated in Phase 1
   - Phase 2 fields awaiting human validation (listed clearly)
   - missing Phase 1 files (if any — these are blocking)
   - incomplete Phase 1 fields
   - unresolved items requiring manual input or review
   - validation warnings

Do not produce a third deliverable.

## Final QA checklist
- Static engine sources read before run parameters and campaign-specific files.
- Prompt MDR captured before workbook population.
- RFA status checked and approved before using the RFA ID.
- Source inventory registered in the helper / source area with Phase 1 / Phase 2 status noted.
- Phase 1 completeness confirmed: campaign identity block fully populated from RFA, provisional partner KPIs populated from CHARGE rows, MDR written from ignition prompt, traffic-light classification resolved.
- All Phase 2 fields confirmed as `PENDING_HUMAN_VALIDATION` (not blank, not `DATA_NOT_FOUND`).
- Required Phase 1 missing sources explicitly flagged (RFA or partner file absence is a blocking stop).
- Retention populated only from observed retention data (Phase 2 human step — must be marked `PENDING_HUMAN_VALIDATION`).
- MDR and finance inputs sourced only from approved inputs; finance rate fields marked `PENDING_HUMAN_VALIDATION`.
- Traffic-light rule checked against MDR.
- Formula cells preserved.
- Narrative cells checked against numbers.
- No PII or row-level IDs in the final output.
- Only 2 deliverables returned: Phase 1 partially completed workbook and chat summary.

---

## Excel Population Technical Runbook — Fresh Build

## Overview

The engine builds the output Excel file from scratch on every run. It does NOT open
or modify the template file. It creates a new openpyxl Workbook, applies all
formatting explicitly, writes all formulas as strings, then populates campaign values.

## Required Library

```python
from openpyxl import Workbook
from openpyxl.styles import Font, PatternFill, Alignment, Border, Side
from openpyxl.utils import get_column_letter
```

Install with: `pip install openpyxl`

Do not use xlrd, xlwt, xlsxwriter, pandas.to_excel, SpreadsheetArtifact,
artifact_tool, artifact.recalculate(), or artifact.export() anywhere in the script.

---

## Step 1 — Create the Workbook and Sheet

```python
wb = Workbook()
ws = wb.active
ws.title = "Campaign Template"
```

---

## Step 2 — Set Column Widths

Apply these exact column widths:

```python
col_widths = {
    'A': 2.63,
    'B': 35.18,
    'C': 16.54,
    'D': 15.09,
    'E': 15.82,
    'F': 13.82,
    'G': 16.27,
    'H': 13.00,
    'I': 80.00,
    'L': 9.09,
    'M': 10.54,
    'N': 9.09,
    'P': 9.09,
}
for col, width in col_widths.items():
    ws.column_dimensions[col].width = width
```

---

## Step 3 — Set Row Heights

Apply these exact row heights:

```python
row_heights = {
    2: 72.0, 3: 20.5, 4: 20.5, 5: 20.5, 6: 18.65, 7: 18.65,
    8: 18.5, 9: 18.5, 10: 18.5, 11: 58.0, 12: 21.0, 13: 21.0,
    14: 18.65, 15: 18.65, 16: 18.65, 17: 18.65, 18: 18.65,
    19: 18.65, 20: 18.65, 21: 18.65, 22: 18.65, 23: 18.65,
    24: 18.65, 25: 18.65, 26: 18.65, 27: 18.65, 28: 18.65,
    29: 18.65, 30: 18.65, 31: 19.75, 32: 21.0, 33: 21.0,
    34: 21.0, 35: 21.0, 36: 21.0, 37: 22.0, 38: 37.75,
    39: 55.25, 40: 55.25, 41: 26.4, 42: 26.4, 43: 26.4,
}
for row, height in row_heights.items():
    ws.row_dimensions[row].height = height
```

---

## Step 4 — Define Reusable Style Helpers

```python
FONT_NAME = 'Aptos Narrow'

# Fill colours
FILL_YELLOW_LIGHT = PatternFill('solid', fgColor='FFFFFFCC')  # light yellow input cells
FILL_AMBER        = PatternFill('solid', fgColor='FFFFC000')  # amber/orange input cells
FILL_NONE         = None

# Fonts
def font(size=11, bold=False, color='FF000000', name=FONT_NAME):
    return Font(name=name, size=size, bold=bold, color=color)

FONT_14           = font(14)
FONT_14_BOLD      = font(14, bold=True)
FONT_14_RED       = font(14, color='FFFF0000')
FONT_14_BOLD_RED  = font(14, bold=True, color='FFFF0000')
FONT_14_BOLD_DKRED= font(14, bold=True, color='FFC00000')
FONT_14_BOLD_BLUE = font(14, bold=True, color='FF0070C0')
FONT_11_RED       = font(11, color='FFFF0000')
FONT_12           = font(12)

# Alignments
ALIGN_CENTER      = Alignment(horizontal='center', vertical='center')
ALIGN_CENTER_WRAP = Alignment(horizontal='center', vertical='center', wrap_text=True)
ALIGN_LEFT        = Alignment(horizontal='left', vertical='center')
ALIGN_LEFT_WRAP   = Alignment(horizontal='left', vertical='center', wrap_text=True)

# Borders
THIN = Side(style='thin')
MED  = Side(style='medium')

def border(left=True, right=True, top=True, bottom=True):
    s = THIN
    return Border(
        left=s if left else Side(),
        right=s if right else Side(),
        top=s if top else Side(),
        bottom=s if bottom else Side()
    )

BORDER_ALL  = border()
BORDER_NONE = Border()

# Helper to apply style to a cell
def style(ws, ref, fill=None, fnt=None, align=None, bdr=None, num_fmt=None):
    c = ws[ref]
    if fill:    c.fill       = fill
    if fnt:     c.font       = fnt
    if align:   c.alignment  = align
    if bdr:     c.border     = bdr
    if num_fmt: c.number_format = num_fmt
```

---

## Step 5 — Apply Merge Ranges

Apply all merged cell ranges exactly as defined:

```python
merges = [
    'C2:H2',  'C3:H3',  'C4:H4',   'C5:H5',
    'C6:E6',  'F6:H6',  'C7:E7',   'F7:H7',
    'C8:H8',  'C9:H10', 'C11:H11', 'C12:H12',
    'C13:H13','C14:H14','C15:H15', 'C16:H16',
    'C21:H21','C22:H22','C23:H23', 'C24:H24',
    'C25:H25','C26:H26','C27:H27', 'C28:H28',
    'C29:H29','C30:H30','C38:H38', 'C39:H39',
    'C40:H40','C41:G41','C42:H42', 'C43:G43',
]
for m in merges:
    ws.merge_cells(m)
```

---

## Step 6 — Write Static Labels (Column B and Header Rows)

These are the fixed row labels. Write them exactly as shown:

```python
labels = {
    'B2':  'Campaign Name',
    'B3':  'RFA No.',
    'B4':  'Partner Name',
    'B5':  'Type of Campaign',
    'B6':  'Campaign Period',
    'B8':  'Funding Mechanism',
    'B9':  'Mechanic',
    'B12': 'RFA Amt',
    'B13': 'Cost Centre',
    'B14': 'Campaign Participants',
    'B15': 'Campaign TPV (RM)',
    'B16': 'Campaign Txn #',
    'B17': 'Campaign User Retention Rate',
    'B21': 'Campaign Revenue (RM)',
    'B22': 'MDR',
    'B23': 'Campaign Total Cost (RM)',
    'B24': 'CPAM Cost (RM)',
    'B25': 'Campaign Direct Cost (RM)',
    'B26': 'Avg Reload Cost %',
    'B27': 'Avg Cloud Cost / Txn (RM)',
    'B28': 'Avg PLSA Cost / Txn (RM)',
    'B29': 'Gross Profit (RM)',
    'B30': '% ROI (GP / CPAM Cost)',
    'B31': 'Merchant-Level Impact Analysis',
    'B32': 'Monthly TPV (RM)',
    'B33': 'Avg Monthly TPV Growh (RM)',
    'B34': 'Monthly TPV (RM)',
    'B35': 'Avg Monthly TPV Growh (RM)',
    'B36': 'Avg Monthly TPV Growh (%)',
    'B37': 'Monthly MTU',
    'B38': 'Usecases',
    'B39': 'TPV Pre- vs During Campaign',
    'B40': 'TPV Pre- vs Post 3M Campaign',
    'B41': 'Proceed to 2nd Review',
    'B42': 'CPAM Cost/User (RM)',
    'B43': 'Est. 12M CLTV',
    # Sub-headers
    'C6':  'Starting Date',
    'F6':  'Ending Date',
    # Retention column headers row 17
    'C17': 'Post 1M', 'D17': 'Post 2M', 'E17': 'Post 3M',
    'F17': 'Post 4M', 'G17': 'Post 5M', 'H17': 'Post 6M',
    # Retention column headers row 19
    'C19': 'Post 7M',  'D19': 'Post 8M',  'E19': 'Post 9M',
    'F19': 'Post 10M', 'G19': 'Post 11M', 'H19': 'Post 12M',
    # Merchant impact column headers row 31
    'C31': 'Pre-Campaign', 'D31': 'During',  'E31': 'Post 1M',
    'F31': 'Post 2M',      'G31': 'Post 3M', 'H31': 'Post 6M',
}
for ref, val in labels.items():
    ws[ref] = val
```

---

## Step 7 — Write Notes Column (Column I)

These are the notes/instructions visible in column I. Write them exactly:

```python
notes = {
    'I2':  "Get from RFA pdf copy under 'Campaign Name'",
    'I3':  "Get from RFA pdf copy on 2nd Line from Top and Status need to be 'Approved'. Don't grab the RFA number its not approved",
    'I4':  "Get from RFA pdf copy under 'Campaign Name'",
    'I5':  'CMS or Non CMS. Ivy to get from BU',
    'I7':  'Campaign dates in Finance PR file may not be accurate as BU may defer the date. Ivy to update manually after checking with BU',
    'I8':  'Ivy to update manually after checking with BU',
    'I9':  'Get from RFA pdf copy under Purpose/Mechanics or Ivy to update manually after checking with BU',
    'I12': "Get from RFA pdf copy under 'Amount'",
    'I13': "Get from RFA pdf copy under 'Campaign Name'",
    'I14': 'Populate from reconciled Partner Raw Data and Internal Verification Data.',
    'I15': 'Populate from reconciled Partner Raw Data and Internal Verification Data.',
    'I16': 'Populate from reconciled Partner Raw Data and Internal Verification Data.',
    'I17': 'This is not actual number, just a dummy number to show my requirement',
    'I18': 'Populate with month-on-month retention percentages for Post 1M to Post 6M based on campaign participants.',
    'I19': 'This is not actual number, just a dummy number to show my requirement',
    'I20': 'Populate with month-on-month retention percentages for Post 7M to Post 12M based on campaign participants.',
    'I21': 'Auto-calculates when Campaign TPV and MDR are numeric; otherwise shows placeholder.',
    'I22': 'Update from the ignition prompt MDR input for this campaign, or from an explicitly approved finance / BU override. Do not reuse prior campaign MDR.',
    'I23': 'Auto-calculates when CPAM Cost and Campaign Direct Cost are numeric; otherwise shows placeholder.',
    'I24': 'Populate from reconciled Partner Raw Data and Internal Verification Data.',
    'I25': 'Auto-calculates when TPV, Txn, and direct-cost rate inputs are numeric; otherwise shows placeholder.',
    'I26': 'Populate with the applicable average reload cost percentage from Finance or approved cost source.',
    'I27': 'Populate with the applicable average cloud cost per transaction from Finance or approved cost source.',
    'I28': 'Populate with the applicable average PLSA cost per transaction from Finance or approved cost source.',
    'I29': 'Auto-calculates when Campaign Revenue and Campaign Total Cost are numeric; otherwise shows placeholder.',
    'I30': 'Auto-calculates when Gross Profit and CPAM Cost are numeric; otherwise shows placeholder.',
    'I31': 'Pre-campaign mthly tpv is 30days before campaign date',
    'I33': 'Populate if this metric is required for the campaign; otherwise leave blank per business rule.',
    'I34': 'Populate with actual monthly TPV values from BI/internal data source.',
    'I35': 'Populate with actual average monthly TPV growth values.',
    'I36': 'Populate if this metric is required for the campaign; otherwise leave blank per business rule.',
    'I38': 'Traffic Light (MDR%) rule: Green if MDR >= 0.47%; Yellow if MDR >= 0.18% and < 0.47%; Red if MDR < 0.18%. Keep formula intact and update MDR only.',
    'I42': 'Auto-calculates when CPAM Cost and Campaign Participants are numeric; otherwise shows placeholder.',
    'I43': 'Populate only when sourced from the approved CLTV data owner/system.',
}
for ref, val in notes.items():
    ws[ref] = val
```

---

## Step 8 — Write Formulas

Write these formulas exactly as strings into the anchor cells shown.
These are the self-calculating cells. Do not hardcode their values.

```python
ws['C21'] = '=IF(COUNT(C15,C22)=2,C15*C22,"{{CAMPAIGN_REVENUE_RM}}")'
ws['C23'] = '=IF(COUNT(C24,C25)=2,C24+C25,"{{CAMPAIGN_TOTAL_COST_RM}}")'
ws['C25'] = '=IF(COUNT(C15,C16,C26,C27,C28)=5,(C15*C26)+(C16*C27)+(C16*C28),"{{DIRECT_COST_RM}}")'
ws['C29'] = '=IF(COUNT(C21,C23)=2,C21-C23,"{{GROSS_PROFIT_RM}}")'
ws['C30'] = '=IF(COUNT(C29,C24)=2,C29/C24,"{{ROI_PCT}}")'
ws['C38'] = '=IF(ISNUMBER(C22),IF(C22>=0.0047,"Green",IF(C22>=0.0018,"Yellow","Red"))&" (MDR% = "&TEXT(C22,"0.00%")&")","{{TRAFFIC_LIGHT_STATUS}}")'
ws['C42'] = '=IF(COUNT(C24,C14)=2,C24/C14,"{{CPAM_COST_PER_USER_RM}}")'
```

---

## Step 9 — Apply All Cell Formatting

Apply formatting to every cell as extracted from the original template.
Use the style helper defined in Step 4.

```python
# ── ROW 2: Campaign Name ──────────────────────────────────────────
style(ws,'B2', fnt=font(14,bold=True), align=ALIGN_LEFT_WRAP,   bdr=BORDER_ALL)
style(ws,'C2', fnt=font(14,bold=True), align=ALIGN_CENTER_WRAP, bdr=BORDER_ALL)
for col in ['D','E','F','G','H']:
    style(ws, f'{col}2', fnt=FONT_14, bdr=BORDER_ALL)

# ── ROW 3: RFA No. ───────────────────────────────────────────────
style(ws,'B3', fnt=FONT_14, align=ALIGN_LEFT_WRAP,   bdr=BORDER_ALL)
style(ws,'C3', fnt=FONT_14, align=ALIGN_CENTER_WRAP, bdr=BORDER_ALL)
for col in ['D','E','F','G','H']:
    style(ws, f'{col}3', fnt=FONT_14, bdr=BORDER_ALL)

# ── ROWS 4–5: Partner Name, Type of Campaign ──────────────────────
for row in [4, 5]:
    style(ws, f'B{row}', fnt=FONT_14, align=ALIGN_LEFT_WRAP,   bdr=BORDER_ALL)
    style(ws, f'C{row}', fnt=FONT_14, align=ALIGN_CENTER_WRAP, bdr=BORDER_ALL)
    for col in ['D','E','F','G','H']:
        style(ws, f'{col}{row}', fnt=FONT_14, bdr=BORDER_ALL)

# ── ROWS 6–7: Campaign Period ─────────────────────────────────────
style(ws,'B6', fnt=FONT_14, bdr=BORDER_ALL)
style(ws,'C6', fnt=FONT_14, align=ALIGN_CENTER, bdr=BORDER_ALL, num_fmt='mm-dd-yy')
style(ws,'F6', fnt=FONT_14, align=ALIGN_CENTER, bdr=BORDER_ALL, num_fmt='mm-dd-yy')
for col in ['D','E','G','H']:
    style(ws, f'{col}6', fnt=FONT_14, bdr=BORDER_ALL)
style(ws,'I6', fnt=FONT_11_RED)

style(ws,'B7', fnt=FONT_14, bdr=BORDER_ALL)
style(ws,'C7', fnt=FONT_14, align=ALIGN_CENTER, bdr=BORDER_ALL, num_fmt='mm-dd-yy')
style(ws,'F7', fnt=FONT_14, align=ALIGN_CENTER, bdr=BORDER_ALL, num_fmt='mm-dd-yy')
for col in ['D','E','G','H']:
    style(ws, f'{col}7', fnt=FONT_14, bdr=BORDER_ALL)

# ── ROW 8: Funding Mechanism ──────────────────────────────────────
style(ws,'B8', fnt=FONT_14, bdr=BORDER_ALL)
style(ws,'C8', fnt=FONT_14, align=ALIGN_CENTER, bdr=BORDER_ALL)
for col in ['D','E','F','G','H']:
    style(ws, f'{col}8', fnt=FONT_14, bdr=BORDER_ALL)

# ── ROWS 9–11: Mechanic ───────────────────────────────────────────
style(ws,'B9',  fnt=FONT_14, bdr=BORDER_ALL)
style(ws,'C9',  fnt=FONT_14, align=ALIGN_LEFT_WRAP, bdr=BORDER_ALL)
for col in ['D','E','F','G','H']:
    style(ws, f'{col}9', fnt=FONT_14, bdr=BORDER_ALL)
style(ws,'B10', fnt=FONT_14, bdr=BORDER_ALL)
for col in ['C','D','E','F','G','H']:
    style(ws, f'{col}10', fnt=FONT_14, bdr=BORDER_ALL)
style(ws,'B11', fnt=FONT_14, bdr=BORDER_ALL)
style(ws,'C11', fnt=FONT_14, align=ALIGN_LEFT_WRAP, bdr=BORDER_ALL)
for col in ['D','E','F','G','H']:
    style(ws, f'{col}11', fnt=FONT_14, bdr=BORDER_ALL)

# ── ROW 12: RFA Amount ────────────────────────────────────────────
style(ws,'B12', fnt=FONT_14, align=ALIGN_LEFT_WRAP,  bdr=BORDER_ALL)
style(ws,'C12', fnt=FONT_14, align=ALIGN_CENTER_WRAP,bdr=BORDER_ALL, num_fmt='"RM"#,##0;[Red]\\-"RM"#,##0')
for col in ['D','E','F','G','H']:
    style(ws, f'{col}12', fnt=FONT_14, bdr=BORDER_ALL)

# ── ROW 13: Cost Centre ───────────────────────────────────────────
style(ws,'B13', fnt=FONT_14, align=ALIGN_LEFT_WRAP,  bdr=BORDER_ALL)
style(ws,'C13', fnt=FONT_14, align=ALIGN_CENTER_WRAP,bdr=BORDER_ALL, num_fmt='"RM"#,##0;[Red]\\-"RM"#,##0')
style(ws,'H13', fnt=FONT_14, bdr=BORDER_ALL)

# ── ROW 14: Campaign Participants (yellow input) ───────────────────
style(ws,'B14', fill=FILL_YELLOW_LIGHT, fnt=FONT_14, bdr=BORDER_ALL)
style(ws,'C14', fill=FILL_YELLOW_LIGHT, fnt=FONT_14, align=ALIGN_CENTER, bdr=BORDER_ALL, num_fmt='#,##0')
for col in ['D','E','F','G','H']:
    style(ws, f'{col}14', fnt=FONT_14, bdr=BORDER_ALL)
style(ws,'I14', fnt=FONT_11_RED)

# ── ROW 15: Campaign TPV (yellow input) ───────────────────────────
style(ws,'B15', fill=FILL_YELLOW_LIGHT, fnt=FONT_14, bdr=BORDER_ALL)
style(ws,'C15', fill=FILL_YELLOW_LIGHT, fnt=FONT_14, align=ALIGN_CENTER, bdr=BORDER_ALL, num_fmt='#,##0.00')
style(ws,'H15', fnt=FONT_14, bdr=BORDER_ALL)
style(ws,'I15', fnt=FONT_11_RED)

# ── ROW 16: Campaign Txn (yellow input) ───────────────────────────
style(ws,'B16', fill=FILL_YELLOW_LIGHT, fnt=FONT_14, bdr=BORDER_ALL)
style(ws,'C16', fill=FILL_YELLOW_LIGHT, fnt=FONT_14, align=ALIGN_CENTER, bdr=BORDER_ALL, num_fmt='#,##0')
for col in ['D','E','F','G','H']:
    style(ws, f'{col}16', fnt=FONT_14, bdr=BORDER_ALL)
style(ws,'I16', fnt=FONT_11_RED)

# ── ROW 17: Retention header row (yellow, bold headers) ────────────
style(ws,'B17', fill=FILL_YELLOW_LIGHT, fnt=FONT_14, bdr=BORDER_ALL)
for col in ['C','D','E','F','G','H']:
    style(ws, f'{col}17', fill=FILL_YELLOW_LIGHT, fnt=FONT_14_BOLD, align=ALIGN_CENTER, bdr=BORDER_ALL)
style(ws,'I17', fnt=FONT_11_RED)

# ── ROW 18: Retention P1–P6 (yellow input, percentage format) ──────
style(ws,'B18', fill=FILL_YELLOW_LIGHT, fnt=FONT_14, bdr=BORDER_ALL)
for col in ['C','D','E','F','G','H']:
    style(ws, f'{col}18', fill=FILL_YELLOW_LIGHT, fnt=FONT_14, align=ALIGN_CENTER, bdr=BORDER_ALL, num_fmt='0%')
style(ws,'I18', fnt=FONT_11_RED)

# ── ROW 19: Retention header row 2 ────────────────────────────────
style(ws,'B19', fill=FILL_YELLOW_LIGHT, fnt=FONT_14, bdr=BORDER_ALL)
for col in ['C','D','E','F','G','H']:
    style(ws, f'{col}19', fill=FILL_YELLOW_LIGHT, fnt=FONT_14_BOLD, align=ALIGN_CENTER, bdr=BORDER_ALL)
style(ws,'I19', fnt=FONT_11_RED)

# ── ROW 20: Retention P7–P12 (yellow input, percentage format) ──────
style(ws,'B20', fill=FILL_YELLOW_LIGHT, fnt=FONT_14, bdr=BORDER_ALL)
for col in ['C','D','E','F','G','H']:
    style(ws, f'{col}20', fill=FILL_YELLOW_LIGHT, fnt=FONT_14, align=ALIGN_CENTER, bdr=BORDER_ALL, num_fmt='0%')
style(ws,'I20', fnt=FONT_11_RED)

# ── ROW 21: Campaign Revenue (formula row) ────────────────────────
style(ws,'B21', fnt=FONT_14, bdr=BORDER_ALL)
style(ws,'C21', fnt=FONT_14, align=ALIGN_CENTER, bdr=BORDER_ALL, num_fmt='#,##0')
for col in ['D','E','F','G','H']:
    style(ws, f'{col}21', fnt=FONT_14, bdr=BORDER_ALL)

# ── ROW 22: MDR (amber input) ─────────────────────────────────────
style(ws,'B22', fill=FILL_AMBER, fnt=FONT_14, bdr=BORDER_ALL)
style(ws,'C22', fill=FILL_AMBER, fnt=FONT_14, align=ALIGN_CENTER, bdr=BORDER_ALL, num_fmt='0.00%')
style(ws,'H22', fnt=FONT_14, bdr=BORDER_ALL)

# ── ROW 23: Campaign Total Cost (formula row) ─────────────────────
style(ws,'B23', fnt=FONT_14, bdr=BORDER_ALL)
style(ws,'C23', fnt=FONT_14, align=ALIGN_CENTER, bdr=BORDER_ALL, num_fmt='#,##0')
style(ws,'H23', fnt=FONT_14, bdr=BORDER_ALL)

# ── ROW 24: CPAM Cost (yellow input) ──────────────────────────────
style(ws,'B24', fill=FILL_YELLOW_LIGHT, fnt=FONT_14, bdr=BORDER_ALL)
style(ws,'C24', fill=FILL_YELLOW_LIGHT, fnt=FONT_14, align=ALIGN_CENTER, bdr=BORDER_ALL, num_fmt='#,##0')
for col in ['D','E','F','G','H']:
    style(ws, f'{col}24', fnt=FONT_14, bdr=BORDER_ALL)
style(ws,'I24', fnt=FONT_11_RED)

# ── ROW 25: Campaign Direct Cost (formula row) ────────────────────
style(ws,'B25', fnt=FONT_14, bdr=BORDER_ALL)
style(ws,'C25', fnt=FONT_14, align=ALIGN_CENTER, bdr=BORDER_ALL, num_fmt='#,##0')
for col in ['D','E','F','G','H']:
    style(ws, f'{col}25', fnt=FONT_14, bdr=BORDER_ALL)

# ── ROW 26: Avg Reload Cost % (amber input) ───────────────────────
style(ws,'B26', fill=FILL_AMBER, fnt=FONT_14, align=ALIGN_LEFT, bdr=BORDER_ALL)
style(ws,'C26', fill=FILL_AMBER, fnt=FONT_14, align=ALIGN_CENTER, bdr=BORDER_ALL, num_fmt='0.0000%')
for col in ['D','E','F','G','H']:
    style(ws, f'{col}26', fnt=FONT_14, bdr=BORDER_ALL)
style(ws,'I26', fnt=FONT_11_RED)

# ── ROW 27: Avg Cloud Cost / Txn (amber input) ────────────────────
style(ws,'B27', fill=FILL_AMBER, fnt=FONT_14, align=ALIGN_LEFT, bdr=BORDER_ALL)
style(ws,'C27', fill=FILL_AMBER, fnt=FONT_14, align=ALIGN_CENTER, bdr=BORDER_ALL, num_fmt='#,##0.0000')
for col in ['D','E','F','G','H']:
    style(ws, f'{col}27', fnt=FONT_14, bdr=BORDER_ALL)
style(ws,'I27', fnt=FONT_11_RED)

# ── ROW 28: Avg PLSA Cost / Txn (amber input) ─────────────────────
style(ws,'B28', fill=FILL_AMBER, fnt=FONT_14, align=ALIGN_LEFT, bdr=BORDER_ALL)
style(ws,'C28', fill=FILL_AMBER, fnt=FONT_14, align=ALIGN_CENTER, bdr=BORDER_ALL, num_fmt='#,##0.0000')
for col in ['D','E','F','G','H']:
    style(ws, f'{col}28', fnt=FONT_14, bdr=BORDER_ALL)
style(ws,'I28', fnt=FONT_11_RED)

# ── ROW 29: Gross Profit (formula, bold red font) ─────────────────
style(ws,'B29', fnt=FONT_14, bdr=BORDER_ALL)
style(ws,'C29', fnt=FONT_14_BOLD_RED, align=ALIGN_CENTER, bdr=BORDER_ALL, num_fmt='#,##0')
for col in ['D','E','F','G','H']:
    style(ws, f'{col}29', fnt=FONT_14, bdr=BORDER_ALL)

# ── ROW 30: % ROI (formula, bold red font) ────────────────────────
style(ws,'B30', fnt=FONT_14, bdr=BORDER_ALL)
style(ws,'C30', fnt=FONT_14_BOLD_RED, align=ALIGN_CENTER, bdr=BORDER_ALL, num_fmt='0.00%')
for col in ['D','E','F','G','H']:
    style(ws, f'{col}30', fnt=FONT_14, bdr=BORDER_ALL)

# ── ROW 31: Merchant-Level Impact Analysis header ──────────────────
for col in ['B','C','D','E','F','G','H']:
    style(ws, f'{col}31', fnt=FONT_14_BOLD, align=ALIGN_CENTER, bdr=BORDER_ALL)
style(ws,'I31', fnt=FONT_11_RED)

# ── ROWS 32–35: TPV display/value rows (yellow, millions format) ───
for row in [32, 33, 34, 35]:
    style(ws, f'B{row}', fill=FILL_YELLOW_LIGHT, fnt=FONT_14, bdr=BORDER_ALL)
    for col in ['C','D','E','F','G','H']:
        style(ws, f'{col}{row}', fill=FILL_YELLOW_LIGHT, fnt=FONT_14, align=ALIGN_CENTER,
              bdr=BORDER_ALL, num_fmt='#,##0.00,,')
    if row in [33, 34, 35]:
        style(ws, f'I{row}', fnt=FONT_11_RED)

# ── ROW 36: TPV Growth % (yellow, percentage format) ──────────────
style(ws,'B36', fill=FILL_YELLOW_LIGHT, fnt=FONT_14, bdr=BORDER_ALL)
for col in ['C','D','E','F','G','H']:
    style(ws, f'{col}36', fill=FILL_YELLOW_LIGHT, fnt=FONT_14, align=ALIGN_CENTER,
          bdr=BORDER_ALL, num_fmt='0.0%')
style(ws,'I36', fnt=FONT_11_RED)

# ── ROW 37: Monthly MTU (yellow, integer format) ──────────────────
style(ws,'B37', fill=FILL_YELLOW_LIGHT, fnt=FONT_14, bdr=BORDER_ALL)
for col in ['C','D','E','F','G']:
    style(ws, f'{col}37', fill=FILL_YELLOW_LIGHT, fnt=FONT_14, align=ALIGN_CENTER,
          bdr=BORDER_ALL, num_fmt='#,##0')
style(ws,'H37', fill=FILL_YELLOW_LIGHT, fnt=FONT_14, align=ALIGN_CENTER,
      bdr=BORDER_ALL, num_fmt='#,##0.00,,')

# ── ROW 38: Usecases / Traffic Light (yellow, red font, wrap) ──────
style(ws,'B38', fill=FILL_YELLOW_LIGHT, fnt=FONT_14, bdr=BORDER_ALL)
style(ws,'C38', fill=FILL_YELLOW_LIGHT, fnt=FONT_14_RED, align=ALIGN_CENTER_WRAP, bdr=BORDER_ALL)
for col in ['D','E','F','G','H']:
    style(ws, f'{col}38', fnt=FONT_14, bdr=BORDER_ALL)
style(ws,'I38', fnt=FONT_11_RED)

# ── ROWS 39–40: TPV summaries (yellow, bold red, wrap) ─────────────
for row in [39, 40]:
    style(ws, f'B{row}', fill=FILL_YELLOW_LIGHT, fnt=FONT_14, bdr=BORDER_ALL)
    style(ws, f'C{row}', fill=FILL_YELLOW_LIGHT, fnt=FONT_14_BOLD_RED,
          align=ALIGN_CENTER_WRAP, bdr=BORDER_ALL)
    for col in ['D','E','F','G','H']:
        style(ws, f'{col}{row}', fnt=FONT_14, bdr=BORDER_ALL)

# ── ROW 41: Proceed to 2nd Review (bold dark red, wrap) ─────────────
style(ws,'B41', fnt=FONT_14, bdr=BORDER_ALL)
style(ws,'C41', fnt=FONT_14_BOLD_DKRED, align=ALIGN_CENTER_WRAP, bdr=BORDER_ALL)
for col in ['D','E','F','G']:
    style(ws, f'{col}41', fnt=FONT_14, bdr=BORDER_ALL)
style(ws,'H41', fnt=FONT_14_BOLD_DKRED, align=ALIGN_CENTER_WRAP, bdr=BORDER_ALL)

# ── ROW 42: CPAM Cost/User (formula, RM format) ────────────────────
style(ws,'B42', fnt=FONT_14, bdr=BORDER_ALL)
style(ws,'C42', fnt=FONT_14, align=ALIGN_CENTER_WRAP, bdr=BORDER_ALL, num_fmt='"RM"#,##0.00')
for col in ['D','E','F','G','H']:
    style(ws, f'{col}42', fnt=FONT_14, bdr=BORDER_ALL)

# ── ROW 43: Est. 12M CLTV (bold blue, RM format) ──────────────────
style(ws,'B43', fnt=FONT_14_BOLD_BLUE, bdr=BORDER_ALL)
style(ws,'C43', fnt=FONT_14_BOLD_BLUE, align=ALIGN_CENTER_WRAP,
      bdr=BORDER_ALL, num_fmt='"RM"#,##0.00')
for col in ['D','E','F','G']:
    style(ws, f'{col}43', fnt=FONT_14, bdr=BORDER_ALL)
style(ws,'H43', fnt=FONT_14_BOLD_BLUE, align=ALIGN_CENTER_WRAP, num_fmt='"RM"#,##0.00')
style(ws,'I43', fnt=FONT_11_RED)
```

---

## Step 10 — Populate Phase 1 Campaign Values

After the structure is fully built, write the campaign-specific values.
Use the exact cell references below. Write only to anchor cells.

```python
# ── PHASE 1 INPUT VALUES ──────────────────────────────────────────
# Replace these variables with values derived from the RFA and partner CSV

ws['C2']  = campaign_name         # string — from RFA
ws['C3']  = rfa_id                # string — from RFA, only if Approved
ws['C4']  = partner_name          # string — from RFA
ws['C5']  = campaign_type         # string — from RFA or 'MANUAL_INPUT_REQUIRED'
ws['C7']  = campaign_start_date   # string DD/MM/YYYY — from RFA
ws['F7']  = campaign_end_date     # string DD/MM/YYYY — from RFA
ws['C8']  = funding_mechanism     # string — from RFA or 'MANUAL_INPUT_REQUIRED'
ws['C9']  = mechanic_primary      # string — from RFA
ws['C11'] = mechanic_secondary    # string — from RFA or blank string
ws['C12'] = rfa_amount            # number — from RFA e.g. 107500.00
ws['C13'] = cost_centre           # string — from RFA

# Partner-derived provisional KPIs
ws['C14'] = participants          # integer — distinct users on CHARGE rows
ws['C15'] = tpv                   # float   — sum of transaction_amount on CHARGE rows
ws['C16'] = txn_count             # integer — distinct transaction IDs on CHARGE rows

# MDR — write as plain decimal, e.g. 0.013 for 1.3%
ws['C22'] = mdr_rate              # float   — from ignition prompt

# CPAM Cost — from partner file benefit_amount sum, or PENDING_HUMAN_VALIDATION
ws['C24'] = cpam_cost             # float or string 'PENDING_HUMAN_VALIDATION'

# ── PHASE 2 PLACEHOLDERS ─────────────────────────────────────────
# Write PENDING_HUMAN_VALIDATION string into all Phase 2 cells

PHV = 'PENDING_HUMAN_VALIDATION'

phase2_cells = [
    'C18','D18','E18','F18','G18','H18',   # Retention P1–P6
    'C20','D20','E20','F20','G20','H20',   # Retention P7–P12
    'C26','C27','C28',                      # Finance cost rates
    'C32','D32','E32','F32','G32','H32',   # Merchant TPV display
    'C33','D33','E33','F33','G33','H33',   # Merchant TPV growth display
    'C34','D34','E34','F34','G34','H34',   # Merchant TPV values
    'C35','D35','E35','F35','G35','H35',   # Merchant TPV growth values
    'C36','D36','E36','F36','G36','H36',   # Merchant TPV growth %
    'C37','D37','E37','F37','G37','H37',   # Monthly MTU
    'C39','C40',                            # TPV narrative summaries
    'C41',                                  # 2nd Review decision
    'C43',                                  # Est. 12M CLTV
]
for ref in phase2_cells:
    ws[ref] = PHV
```

---

## Step 11 — Save the Output File

```python
output_filename = f"campaign_postmortem_{campaign_name}_phase1.xlsx"
# Sanitise filename — remove characters not safe for filenames
import re
output_filename = re.sub(r'[\\/*?:"<>|]', '_', output_filename)

wb.save(output_filename)
print("Saved:", output_filename)
```

Save once. To a new filename. End the script immediately after.
Do not call SpreadsheetArtifact, artifact_tool, recalculate, export, or render.

---

## Step 12 — Error Handling

```
1. If partner CSV cannot be read:
   Stop and return:
   "ERROR: Partner CSV file could not be read. Check filename and format."

2. If CHARGE rows cannot be found in the partner CSV:
   Set participants, tpv, txn_count to 'PENDING_HUMAN_VALIDATION'
   Note this in the chat summary under "Data Warnings."

3. If any individual cell write fails:
   Log the cell address and error.
   Skip that cell and continue.
   Include all failed cells in the chat summary under "Write Errors."

4. Never raise an unhandled exception that stops the run.
   Always reach wb.save() even if some cells failed.
   A partial output is better than no output.

5. After wb.save(), print the filename and stop.
   No further operations.
```

---

## ABSOLUTE PROHIBITIONS — ENFORCE ON EVERY RUN

```
NEVER use: SpreadsheetArtifact, artifact_tool, artifact.recalculate(),
           artifact.export(), artifact.render()

NEVER use: openpyxl.load_workbook() — do not open any file, build fresh only

NEVER delete sheets

NEVER write MDR as a string — always write as decimal float e.g. 0.013

NEVER hardcode calculated values — use formulas as strings in formula cells

NEVER call wb.save() more than once
```
