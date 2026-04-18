# File Ingestion Skill — Partner Data File

## Purpose

This skill defines the exact Python method for reading the partner data file
in the TNGD Campaign Post-Mortem Engine. It covers .csv, .xlsx, and .xls
formats. Follow every rule here exactly. Do not deviate.

---

## Rule 1 — Always Use Python + pandas as the Reading Layer

Never attempt to read the partner file by opening it directly or browsing it
as a preview. Always use Python with pandas as the extraction layer.
This is the only reliable method in the ChatGPT code interpreter environment.

---

## Rule 2 — Detect Format Before Reading

```python
import os
import pandas as pd

def detect_format(filepath):
    ext = os.path.splitext(filepath)[1].lower().strip()
    if ext == '.csv':
        return 'csv'
    elif ext == '.xlsx':
        return 'xlsx'
    elif ext == '.xls':
        return 'xls'
    else:
        raise ValueError(
            f"UNSUPPORTED FORMAT: '{ext}'. "
            f"Accepted: .csv, .xlsx, .xls. "
            f"Stop and report under ACCESS FAILURE RULE."
        )
```

---

## Rule 3 — Use a Resilient Reader With Encoding Fallbacks

Read files one at a time. Do not load everything at once.
Use encoding fallback logic so manual retries are never needed.

### For CSV files:

```python
def read_csv_safe(filepath):
    last_error = None
    for enc in ['utf-8', 'utf-8-sig', 'cp1252', 'latin-1']:
        try:
            df = pd.read_csv(filepath, dtype=str, encoding=enc)
            print(f"CSV read OK: {filepath} | encoding={enc} | "
                  f"{len(df)} rows | {len(df.columns)} columns")
            return df
        except UnicodeDecodeError as e:
            last_error = e
            continue
        except Exception as e:
            raise RuntimeError(
                f"CSV READ FAILED: {filepath} — {str(e)}"
            )
    raise RuntimeError(
        f"CSV READ FAILED after all encoding attempts: {filepath} — {last_error}"
    )
```

### For Excel files:

```python
def read_excel_safe(filepath):
    ext = os.path.splitext(filepath)[1].lower()
    engine = 'openpyxl' if ext == '.xlsx' else 'xlrd'
    try:
        # First inspect available sheets
        xf = pd.ExcelFile(filepath, engine=engine)
        print(f"Excel sheets available: {xf.sheet_names}")
        # Read first sheet by default
        df = pd.read_excel(filepath, sheet_name=0, dtype=str, engine=engine)
        print(f"Excel read OK: {filepath} | sheet='{xf.sheet_names[0]}' | "
              f"{len(df)} rows | {len(df.columns)} columns")
        return df
    except Exception as e:
        raise RuntimeError(
            f"EXCEL READ FAILED: {filepath} — {str(e)}"
        )
```

### Combined reader:

```python
def read_partner_file(filepath):
    fmt = detect_format(filepath)
    if fmt == 'csv':
        return read_csv_safe(filepath)
    else:
        return read_excel_safe(filepath)
```

---

## Rule 4 — Read ALL Columns on First Load, Then Map

Do not try to filter columns during the initial read.
Read the full file first, print the column list, then map logical roles.

```python
df = read_partner_file(partner_filepath)

# Immediately print schema for confirmation
print(f"\nColumns found ({len(df.columns)}):")
for i, col in enumerate(df.columns):
    print(f"  [{i}] {col}")
print(f"\nFirst 3 rows preview:")
print(df.head(3).to_string())
```

This confirmation must be printed before any filtering or aggregation begins.

---

## Rule 5 — Use Fuzzy Column Matching

Partner files use different column names each time.
Never hardcode exact column names. Always use fuzzy matching.

```python
def find_col(df, keywords):
    for col in df.columns:
        col_clean = str(col).strip().lower()
        for kw in keywords:
            if kw.lower() in col_clean:
                return col
    return None

# Map all required logical roles
txn_type_col = find_col(df, ['transaction_type', 'txn_type', 'trans_type', 'type'])
userid_col   = find_col(df, ['wallet_userid', 'user_id', 'userid', 'customer_id'])
txn_id_col   = find_col(df, ['wallet_transaction_id', 'txn_id', 'transaction_id', 'trans_id'])
amount_col   = find_col(df, ['transaction_amount', 'amount', 'txn_amount', '(rm) transaction'])
benefit_col  = find_col(df, ['benefit_amount', 'incentive', 'cashback', '(rm) benefit'])
date_col     = find_col(df, ['transaction_date', 'date', 'txn_date', 'trans_date'])

# Report mapping results
print(f"\nColumn mapping:")
print(f"  transaction_type   → {txn_type_col}")
print(f"  wallet_userid      → {userid_col}")
print(f"  transaction_id     → {txn_id_col}")
print(f"  transaction_amount → {amount_col}")
print(f"  benefit_amount     → {benefit_col}")
print(f"  transaction_date   → {date_col}")

# Flag any unmapped critical columns
critical = {
    'transaction_type':   txn_type_col,
    'wallet_userid':      userid_col,
    'transaction_id':     txn_id_col,
    'transaction_amount': amount_col,
    'benefit_amount':     benefit_col,
}
unmapped = [k for k, v in critical.items() if v is None]
if unmapped:
    print(f"\nWARNING: Could not map: {unmapped}. "
          f"Affected KPIs will be set to MANUAL_INPUT_REQUIRED.")
```

---

## Rule 6 — Clean Data Immediately After Mapping

```python
# Normalise transaction_type to uppercase
if txn_type_col:
    df[txn_type_col] = df[txn_type_col].astype(str).str.upper().str.strip()

# Clean amount columns — strip currency symbols and commas
for col in [amount_col, benefit_col]:
    if col and col in df.columns:
        df[col] = (
            df[col].astype(str)
            .str.replace(r'[RM,\s]', '', regex=True)
            .str.replace(r'[^\d.\-]', '', regex=True)
        )
        df[col] = pd.to_numeric(df[col], errors='coerce')
        null_count = df[col].isna().sum()
        if null_count > 0:
            print(f"WARNING: {null_count} non-numeric values in "
                  f"'{col}' — excluded from aggregation.")

# Drop fully empty rows
df = df.dropna(how='all').reset_index(drop=True)
print(f"\nAfter cleaning: {len(df)} rows remaining.")
```

---

## Rule 7 — Filter and Extract KPIs

```python
if txn_type_col:
    charges = df[df[txn_type_col] == 'CHARGE'].copy()
    refunds = df[df[txn_type_col] == 'REFUND'].copy()
    other   = df[~df[txn_type_col].isin(['CHARGE', 'REFUND'])].copy()
    print(f"\nRow split — CHARGE: {len(charges)} | "
          f"REFUND: {len(refunds)} | OTHER: {len(other)}")
else:
    print("WARNING: transaction_type column not found. "
          "Cannot split CHARGE/REFUND. All KPIs set to MANUAL_INPUT_REQUIRED.")
    charges = pd.DataFrame()
    refunds = pd.DataFrame()

# Primary KPIs
def safe_nunique(df, col):
    if col and col in df.columns and len(df) > 0:
        return int(df[col].nunique())
    return 'MANUAL_INPUT_REQUIRED'

def safe_sum(df, col):
    if col and col in df.columns and len(df) > 0:
        val = pd.to_numeric(df[col], errors='coerce').sum()
        return round(float(val), 2)
    return 'MANUAL_INPUT_REQUIRED'

participants = safe_nunique(charges, userid_col)
tpv          = safe_sum(charges, amount_col)
txn_count    = safe_nunique(charges, txn_id_col)
cpam_cost    = safe_sum(charges, benefit_col)
refund_count = len(refunds)
refund_tpv   = safe_sum(refunds, amount_col)
refund_ben   = safe_sum(refunds, benefit_col)

print(f"\nKPI EXTRACTION RESULTS:")
print(f"  Campaign Participants : {participants}")
print(f"  Campaign TPV (RM)     : {tpv}")
print(f"  Campaign Txn #        : {txn_count}")
print(f"  CPAM Cost (RM)        : {cpam_cost}")
print(f"  Refund Count          : {refund_count}")
print(f"  Refund TPV (RM)       : {refund_tpv}")
print(f"  Refund Benefit (RM)   : {refund_ben}")
```

---

## Rule 8 — Strip PII Before Any Output

```python
pii_cols = [c for c in [userid_col, txn_id_col] if c and c in df.columns]
if pii_cols:
    df = df.drop(columns=pii_cols)
    print(f"PII columns removed: {pii_cols}")
```

Never print individual rows. Only print aggregated results.

---

## Rule 9 — Error Handling

```python
try:
    df = read_partner_file(partner_filepath)
except FileNotFoundError:
    print(f"ACCESS FAILURE: File not found — '{partner_filepath}'. "
          f"Check filename in RUN CONFIGURATION. Stop run.")
    raise
except ValueError as e:
    print(f"ACCESS FAILURE: {str(e)}")
    raise
except RuntimeError as e:
    print(f"ACCESS FAILURE: {str(e)}")
    raise
except Exception as e:
    print(f"ACCESS FAILURE: Unexpected error — {str(e)}")
    raise
```

---

## Rule 10 — Install Check

Run at the start of every session:

```python
def check_deps():
    import importlib
    missing = []
    for lib in ['pandas', 'openpyxl', 'xlrd']:
        if importlib.util.find_spec(lib) is None:
            missing.append(lib)
    if missing:
        import subprocess, sys
        subprocess.check_call([sys.executable, '-m', 'pip', 'install'] + missing)
        print(f"Installed: {missing}")
    else:
        print("Dependencies OK: pandas, openpyxl, xlrd")

check_deps()
```

---

## Common Failures and Fixes

| Symptom | Cause | Fix |
|---|---|---|
| "cannot access uploaded file" | File not read via Python — only browsed | Use read_partner_file() Python function |
| UnicodeDecodeError | Wrong encoding | read_csv_safe() tries 4 encodings automatically |
| Empty dataframe after filter | transaction_type not normalised | .str.upper().str.strip() before filter |
| Amount column is NaN | Currency symbols in numeric field | clean_dataframe() strips RM and commas |
| Zero participants | userid column not found | Use fuzzy find_col() not exact name |
| XLRDError on .xlsx | Wrong engine | Use openpyxl for .xlsx, xlrd for .xls |
| File found but rows not extracted | Tried to read without Python | Always use pandas — never browse directly |
