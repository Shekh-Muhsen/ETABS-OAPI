# ETABS Python Automation via COM API

Python Jupyter notebooks for automating **ETABS structural model editing** using the ETABS OAPI (COM API) via `comtypes`. No plugins required — connects directly to a running ETABS instance.

---

## Files

| File | Description |
|------|-------------|
| `create_EXxx_load_pattern.ipynb` | Creates a new ASCE 7-05 auto-seismic load pattern `EXxx`, cloned from `EX` with importance factor **I changed from 1.25 → 1.0** |
| `create_BNBC2020_1_rs_function.ipynb` | Creates a new response spectrum function `BNBC2020-1`, cloned from `BNBC 2020`, intended for use with **I = 1.0** (SF = 9.81 m/s² in RS load case) |

---

## Requirements

- ETABS 21 or later (tested on ETABS 23.2.0)
- Python 3.x (64-bit, matching ETABS bitness)
- `comtypes` library

```
pip install comtypes
```

---

## How to Run

1. Open ETABS and load your model
2. Open the `.ipynb` in VS Code
3. Select kernel: `Ctrl+Shift+P` → **Python: Select Interpreter**
4. Press **Shift+Enter** to run the single cell

> Do **not** use the Code Runner (▶) button for `.ipynb` files — use the Jupyter cell run button or Shift+Enter.

---

## How It Works

Both scripts follow the same pattern since no direct COM method exists for creating load patterns or RS functions via `comtypes`:

```
Read table  →  Clone row(s)  →  Modify fields  →  Stage  →  Apply  →  Verify
```

| Step | Method |
|------|--------|
| Read existing data | `model.DatabaseTables.GetTableForDisplayArray()` |
| Add pattern shell | `model.LoadPatterns.Add()` *(load pattern script only)* |
| Stage edited table | `model.DatabaseTables.SetTableForEditingArray()` |
| Commit to model | `model.DatabaseTables.ApplyEditedTables()` |

> **Important:** `SetTableForEditingArray` requires **tuples**, not lists, for the field names and flat data arguments when called via `comtypes`. Passing lists will silently fail.

---

## Model Used for Development

- **Project:** TEP-MC1-DYNAMIC-304.EDB (ETABS 23.2.0)
- **Seismic code:** ASCE 7-05 (mapped to BNBC 2020 site parameters)
- **Site:** Class F — Ss=0.70g, S1=0.28g, Fa=1.35, Fv=2.70, TL=4s
- **Units:** kN_m throughout

---

## Notes

- `SetModelIsLocked(False)` **clears all analysis results**. Re-run analysis in ETABS (F5) after running any script that unlocks the model.
- The importance factor `I` is **not stored in the RS function** — it is applied as a scale factor in the RS load case:
  - I = 1.25 → SF = 1.25 × 9.81 = **12.2625 m/s²**
  - I = 1.00 → SF = 1.00 × 9.81 = **9.81 m/s²**
- Scripts connect to whichever ETABS instance is currently open. If multiple instances are running, the first one found is used.

---

## Author

**Muhsen**
