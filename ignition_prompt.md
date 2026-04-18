# TNGD Campaign Post-Mortem Engine — Ignition Prompt

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✏️ RUN CONFIGURATION — THE ONLY SECTION YOU EDIT EACH RUN
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## HOW TO USE THIS SECTION

For each campaign you want to process, fill in one CAMPAIGN BLOCK below.
Copy and paste additional blocks if you have more than one campaign.
Each block must have a unique CAMPAIGN ID (just a number: 1, 2, 3...).
The CAMPAIGN ID is how the engine knows which RFA belongs to which partner file.

For each campaign you must also declare the MDR.
MDR is written as a percentage (e.g. 1.30%) — the engine converts it automatically.

---

### CAMPAIGN BLOCK 1

```
CAMPAIGN ID:        1
CAMPAIGN NAME:      [UPDATE — e.g. 241122 COE RM107.5K Trip.com Launch Campaign]
MDR PERCENTAGE:     [UPDATE — e.g. 1.30%]
RFA FILE NAME:      [UPDATE — e.g. RFA-230752_Tripcom.pdf]
PARTNER DATA FILE:  [UPDATE — e.g. Trip AMA Nov 2024.xlsx]
```

### CAMPAIGN BLOCK 2

```
CAMPAIGN ID:        2
CAMPAIGN NAME:      [UPDATE — e.g. 241201 COE RM50K Grab Launch Campaign]
MDR PERCENTAGE:     [UPDATE — e.g. 0.80%]
RFA FILE NAME:      [UPDATE — e.g. RFA-231001_Grab.pdf]
PARTNER DATA FILE:  [UPDATE — e.g. Grab AMA Dec 2024.csv]
```

### CAMPAIGN BLOCK 3

```
CAMPAIGN ID:        3
CAMPAIGN NAME:      [UPDATE or DELETE THIS BLOCK IF NOT NEEDED]
MDR PERCENTAGE:     [UPDATE or DELETE THIS BLOCK IF NOT NEEDED]
RFA FILE NAME:      [UPDATE or DELETE THIS BLOCK IF NOT NEEDED]
PARTNER DATA FILE:  [UPDATE or DELETE THIS BLOCK IF NOT NEEDED]
```

### CAMPAIGN BLOCK 4

```
CAMPAIGN ID:        4
CAMPAIGN NAME:      [UPDATE or DELETE THIS BLOCK IF NOT NEEDED]
MDR PERCENTAGE:     [UPDATE or DELETE THIS BLOCK IF NOT NEEDED]
RFA FILE NAME:      [UPDATE or DELETE THIS BLOCK IF NOT NEEDED]
PARTNER DATA FILE:  [UPDATE or DELETE THIS BLOCK IF NOT NEEDED]
```

> Delete any unused campaign blocks before running.
> For a single campaign run, keep only CAMPAIGN BLOCK 1.
> The engine processes one campaign at a time, in order, and produces
> one Excel file per campaign before moving to the next.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ENGINE LOGIC BELOW — DO NOT EDIT ANYTHING BELOW THIS LINE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

---

## PHASE DEFINITIONS

### Phase 1 — Human Data Validation (happens BEFORE this engine runs)

The team member validates the partner data file against internal database records.
They confirm KPIs, reconcile TPV, verify transaction counts, and add any internal
data (retention rates, finance cost rates, merchant impact data) directly into
the partner data file or a separate validated data file before uploading.

The AI engine does NOT run during Phase 1. Phase 1 is entirely human-led.

### Phase 2 — AI Engine Run (this prompt)

The AI reads the validated partner data file and the approved RFA, then builds
the fully populated Excel post-mortem workbook. Because the data has already
been validated by the human in Phase 1, the AI populates KPIs as final values
rather than provisional estimates.

Fields that the human could not validate or that require further BU input are
marked MANUAL_INPUT_REQUIRED in the output.

---

## ACCESS FAILURE RULE

If any RFA or partner data file listed in the RUN CONFIGURATION cannot be
accessed, read, or parsed, stop that campaign immediately and report:

- Campaign ID that failed
- Exact filename that failed
- Whether the failure was file not found, unreadable format, or parse error

Move to the next campaign block if one exists.
Do not skip the failure silently.
Do not continue building the Excel file for a campaign with unread source files.

---

## ENGINE CONFIGURATION — FIXED, DO NOT EDIT

### Knowledge Base Files (Static Engine Sources)

The engine must read all of these before processing any campaign:

- `framework_summary.md`
- `guardrails.md`
- `instructions.md`
- `template_schema.md`
- `data_validation.md`
- `placeholder_dictionary.md`
- `cell_to_source_mapping.md`
- `engine_runbook.md`
- `file_ingestion_skill.md`

CONFIRMATION REQUIRED:
After reading all knowledge base files, output this exact line before proceeding:

"Knowledge base loaded: framework_summary.md, guardrails.md, instructions.md,
template_schema.md, data_validation.md, placeholder_dictionary.md,
cell_to_source_mapping.md, engine_runbook.md, file_ingestion_skill.md"

Do not begin processing any campaign until this confirmation is output.

---

## MULTI-CAMPAIGN PROCESSING RULES

- Process campaigns one at a time, in CAMPAIGN ID order (1, 2, 3, 4...)
- Complete all chunks for Campaign 1 before starting Campaign 2
- Produce and save the Excel file for each campaign before moving to the next
- If one campaign fails, log the failure and continue to the next campaign
- Do not mix data between campaigns under any circumstance
- After all campaigns are processed, produce one combined chat summary
  covering all campaigns run in the session

---

## EXECUTION — FOR EACH CAMPAIGN

Repeat these chunks for every campaign block in the RUN CONFIGURATION.
Clearly label each chunk with the Campaign ID being processed.
Example: "CAMPAIGN 1 — CHUNK 1", "CAMPAIGN 1 — CHUNK 2", etc.

---

### CHUNK 1 — RFA Validation

_(Label: CAMPAIGN [ID] — CHUNK 1)_

Read the RFA file for this campaign.
Extract and report:

- Document status — must be Approved. If not, stop this campaign and move to next.
- Campaign name
- RFA ID / number
- Partner name
- Campaign start date and end date
- Funding mechanism
- Approved amount (RM)
- Cost centre
- Primary mechanic text
- Secondary mechanic text (if present)

Rules:

- Only use the RFA if status is explicitly Approved
- Do not carry over any field from a prior campaign block
- Confirm the RFA filename matches the one declared in this campaign's block

Report as:
"CAMPAIGN [ID] CHUNK 1 COMPLETE — RFA: [name], ID: [rfa_id],
Status: Approved, Dates: [start] to [end], Amount: RM[x]"

---

### CHUNK 2 — Partner Data File Ingestion and KPI Extraction

_(Label: CAMPAIGN [ID] — CHUNK 2)_

Read the partner data file for this campaign using Python + pandas.
Follow file_ingestion_skill.md exactly. Run check_deps() first.

File format detection:

- .csv → read with read_csv_safe()
- .xlsx → read with read_excel_safe(), engine='openpyxl'
- .xls → read with read_excel_safe(), engine='xlrd'

After reading, print the column list and first 3 rows for confirmation.
Use fuzzy column matching (find_col()) — never hardcode column names.

Extract from CHARGE rows only:

- Campaign Participants = distinct userid count
- Campaign TPV = sum of transaction_amount
- Campaign Txn # = distinct transaction_id count
- CPAM Cost = sum of benefit_amount

Summarise REFUND rows separately:

- Refund count, refund TPV, refund benefit amount

Strip all PII before any output.
Never print individual rows.

If the partner file contains additional validated fields that the human added
during Phase 1 (e.g. retention rates, finance cost rates, merchant impact data),
extract those as well and map them to the appropriate template fields.

Report as:
"CAMPAIGN [ID] CHUNK 2 COMPLETE — Participants: [n], TPV: RM[x],
Txn #: [n], CPAM Cost: RM[x]. Refunds: [n] txns, RM[x] TPV."

---

### CHUNK 3 — MDR and Traffic-Light Classification

_(Label: CAMPAIGN [ID] — CHUNK 3)_

Read the MDR from the campaign's block in the RUN CONFIGURATION.

MDR CONVERSION RULE:
Convert percentage to decimal before writing.
Example: 1.30% → 0.013
Never write the percentage string. Always write the decimal float.

Traffic-light classification:

- Green if MDR >= 0.0047
- Yellow if MDR >= 0.0018 and MDR < 0.0047
- Red if MDR < 0.0018
- Missing → mark C22 MANUAL_INPUT_REQUIRED, traffic light unresolved

Traffic-light font colour for cell C38:

- Green → FF00B050
- Yellow → FFFFC000
- Red → FFFF0000
- Missing → FFFF0000 (default)

Apply font colour at write time using openpyxl Font color.
Never hardcode a static colour on C38.

Report as:
"CAMPAIGN [ID] CHUNK 3 COMPLETE — MDR: [x]% (0.0[x]),
Traffic Light: [Green/Yellow/Red]"

---

### CHUNK 4 — Excel Build and Population

_(Label: CAMPAIGN [ID] — CHUNK 4)_

Only begin after Chunks 1, 2, and 3 are confirmed complete for this campaign.

Follow the Excel Population Technical Runbook in engine_runbook.md exactly.

Build rules:

- Use Workbook() to create a fresh file. Never use load_workbook().
- Apply all column widths, row heights, merge ranges, static labels, notes,
  formatting, and formula strings exactly as specified in the runbook.
- Set rows 41 and 43 as hidden:
  ws.row_dimensions[41].hidden = True
  ws.row_dimensions[43].hidden = True
- Write all fields that have validated data from Chunks 1 and 2 as final values.
- Write MANUAL_INPUT_REQUIRED for any field the human could not validate in Phase 1.
- Write the MDR decimal value from Chunk 3 to cell C22.
- Apply the traffic-light font colour from Chunk 3 to cell C38.
- Never call SpreadsheetArtifact, artifact_tool, artifact.recalculate(),
  artifact.export(), or artifact.render().
- Never overwrite formula cells: C21, C23, C25, C29, C30, C38, C42.
- Never write MDR as a percentage string.

Output filename format:

```
campaign_postmortem_[CAMPAIGN_ID]_[CAMPAIGN_NAME]_phase2.xlsx
```

Example: `campaign_postmortem_1_241122_COE_Trip.com_phase2.xlsx`

Sanitise filename: replace spaces and special characters with underscores.
Call wb.save(output_filename) once. Print the saved filename. Stop.

Report as:
"CAMPAIGN [ID] CHUNK 4 COMPLETE — Excel saved: [filename]"

---

### CHUNK 5 — Campaign Summary

_(Label: CAMPAIGN [ID] — CHUNK 5)_

After the Excel file for this campaign is confirmed saved, produce a brief
per-campaign summary covering:

- Campaign ID and name
- RFA ID and approval status
- Partner file used
- MDR and traffic-light result
- Fields successfully populated from validated data
- Fields marked MANUAL_INPUT_REQUIRED and why
- Refund summary
- Any data warnings or errors encountered

Then move to the next campaign block if one exists.

---

## FINAL COMBINED SUMMARY

After all campaigns in the session are processed, produce one combined summary:

- Total campaigns attempted
- Total Excel files successfully produced
- List of campaigns that failed and the failure reason
- Any cross-campaign observations (e.g. same merchant appearing in multiple campaigns)
- Any validation warnings that apply across campaigns

---

## EXECUTION GUARDRAILS

- Never carry data, values, or column mappings from one campaign to the next
- Never guess or estimate missing values — use MANUAL_INPUT_REQUIRED
- Never use sample values from prior campaigns or template examples
- Never include PII in any output
- Never call wb.save() more than once per campaign
- Never use artifact_tool or SpreadsheetArtifact
- Never use load_workbook() — always build fresh with Workbook()
- Never overwrite formula cells
- If conflicting values exist in the partner file, use the most recently
  validated value and note the conflict in the campaign summary
- One Excel file per campaign. One chat summary per session covering all campaigns.

### Excel Population Guardrails

- Build from scratch using Workbook() only
- Never use load_workbook()
- Never call SpreadsheetArtifact, artifact_tool, artifact.recalculate(),
  artifact.export(), or artifact.render()
- Always write to top-left anchor cell of merged ranges only
- Never overwrite formula cells: C21, C23, C25, C29, C30, C38, C42
- Write MDR as decimal float only — never as a percentage string
- Set rows 41 and 43 hidden after row height step
- Call wb.save() once per campaign. Print path. Stop.

---

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
START
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Read all knowledge base files first.
Output the confirmation line.
Then process each campaign block in order, Chunks 1 through 5.
Do not start a new campaign until the previous one is fully complete.
After all campaigns, produce the combined summary.
