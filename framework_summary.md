# TNGD Campaign Post-Mortem Framework Summary

## Framework status
This package is **campaign agnostic**.
- It does **not** compute or preserve actual campaign numbers.
- Any values previously seen in source files or workbook examples are treated as **sample signals only**.
- Actual values must be computed later from the files uploaded for a specific campaign run.

## What this framework now does
1. Defines a reusable source-of-truth hierarchy.
2. Preserves the working-file structure, rules column, notes, and formula logic.
3. Standardizes partner/raw, internal, retention, finance, and prompt-level campaign inputs.
4. Defines validation rules for reconciliation, funding conflicts, date alignment, MDR handling, and missing dependencies.
5. Enforces placeholder-based output until a real campaign execution run occurs.

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
- read the current partner raw file
- read current internal / retention / finance files
- map only those current-run values into the preserved workbook structure
- leave formulas intact where the template requires formulas
- mark missing values as `DATA_NOT_FOUND` or `MANUAL_INPUT_REQUIRED`