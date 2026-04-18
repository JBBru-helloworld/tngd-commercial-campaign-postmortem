# TNGD Campaign Post-Mortem Framework Summary

## Framework status
This package is **campaign agnostic**.
- It does **not** compute or preserve actual campaign numbers.
- Any values previously seen in source files or workbook examples are treated as **sample signals only**.
- Actual values must be computed later from the files uploaded for a specific campaign run.

## What this framework now does
1. Defines a reusable two-phase execution model: **Phase 1** (human data validation) followed by **Phase 2** (AI engine run).
2. Preserves the working-file structure, rules column, notes, and formula logic.
3. Standardizes partner/raw, internal, retention, finance, and prompt-level campaign inputs.
4. Defines validation rules for reconciliation, funding conflicts, date alignment, MDR handling, and missing dependencies.
5. Enforces placeholder-based output for any field that cannot be confirmed during the AI run.
6. **Supports processing multiple campaigns in a single session.** Each campaign is identified by a CAMPAIGN ID that links its RFA file to its partner data file. Campaigns are processed sequentially.

## Phase Structure

**Phase 1 — Human Data Validation (before the AI runs)**
The team member validates the partner data file against internal database records.
They confirm KPIs, reconcile TPV, verify transaction counts, and add any internal data directly into the partner data file or a separate validated data file.
The AI does not run during Phase 1. It is entirely human-led.

**Phase 2 — AI Engine Run**
The AI reads the validated partner data file and the approved RFA, then builds the fully populated Excel post-mortem workbook.
Because Phase 1 validation has already occurred, the AI populates KPIs as final values, not provisional estimates.
Fields the human could not validate are marked `MANUAL_INPUT_REQUIRED`.
**The AI engine runs in Phase 2 only, after human data validation is complete.**

## New MDR update
The framework now treats **MDR percentage** as a **campaign-specific run parameter** that must be updated in the ignition prompt for each run.
- MDR must populate the workbook MDR input field.
- The workbook traffic-light output must follow the template rule:
  - **Green**: MDR >= 0.47%
  - **Yellow**: MDR >= 0.18% and < 0.47%
  - **Red**: MDR < 0.18%
- If MDR is missing, the engine must mark the field `MANUAL_INPUT_REQUIRED`.
- If a finance or BU-confirmed source is also uploaded, the engine must validate it against the prompt input and flag any mismatch.

## What this framework does not do yet
- It does not populate final campaign numbers by itself.
- It does not assume any merchant, campaign window, funding model, or KPI value.
- It does not treat any sample workbook value as a reusable default.

## Execution principle
When a future campaign package is uploaded, the engine should:
- read all static engine sources first
- read the campaign-specific run parameters from the ignition prompt
- read the current RFA
- read the validated partner data file (produced after Phase 1 human validation)
- map only those current-run values into the preserved workbook structure
- leave formulas intact where the template requires formulas
- mark missing values as `DATA_NOT_FOUND` or `MANUAL_INPUT_REQUIRED`
