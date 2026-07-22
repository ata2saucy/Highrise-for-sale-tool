# High-Rise Sale-Comp & Price-Triangulation Tool

A reusable, deterministic toolchain for building **sale-comp workbooks** for high-rise
developments (resale condo *or* pre-construction) and triangulating the price a subject should
achieve. You assemble verified comp data into a single JSON input; a generator emits a fully
formatted 6-sheet Excel workbook; a validator proves it is correct and reproducible.

The process is **subject-agnostic** — point it at any building by supplying that building's
`comp_data.json`. This repo is the **for-sale port** of
[highrise-rent-comp-tool](https://github.com/ata2saucy/highrise-rent-comp-tool): the method docs
here are complete; the generator/validator scripts and a worked example land next, following the
same pattern.

## How it works (any subject)

```bash
pip install -r requirements.txt

# 1. Assemble <subject>.json conforming to comp_data.schema.json  (VERIFIED comps only)
# 2. Generate the workbook:
python build_workbook.py comp_data.json "<Subject> Sale Comps _vACTIVE.xlsx"
# 3. Validate (structure, zero-error recalc, numpy tie-out, SF coverage):
python validate_workbook.py "<Subject> Sale Comps _vACTIVE.xlsx" comp_data.json
```

- **`build_workbook.py`** is the single source of truth for workbook structure and every formula.
  It reads a `comp_data.json` (the input contract) and writes the `.xlsx` deterministically.
- **`validate_workbook.py`** asserts the 6-sheet structure, recalculates every formula to **zero
  errors**, ties the averages/blocks to an independent numpy recomputation, and checks **SF
  coverage** (every sold-in-window unit has a verified SF or a documented exhaustion record).
  A green **PASS** means the workbook reproduces exactly from its inputs.
- **`comp_data.schema.json`** is the input contract; **`comp_data.example.json`** is a filled
  template. **`CLAUDE.md`** is the full operating method (triangulation logic, SF-verification
  rules, output spec); **`condo_sqft_verification_method_1.md`** details the SF methodology.

## Core (reusable tool)

| File | Purpose |
|---|---|
| `build_workbook.py` | Workbook generator — owns all structure & formulas *(to be ported)* |
| `validate_workbook.py` | Validator — structure, zero-error recalc, numpy tie-out, SF coverage *(to be ported)* |
| `comp_data.schema.json` | Input contract *(to be ported)* |
| `comp_data.example.json` | Filled example input (template) *(to be ported)* |
| `CLAUDE.md` | Operating method / triangulation + verification rules + output spec |
| `condo_sqft_verification_method_1.md` | SF-verification methodology |
| `v2 Format Conformance — what was missing + target spec.md` | Format spec |
| `building_memory/` | Cross-session memory system (stable building facts, never reportable SF) |
| `requirements.txt` | `openpyxl`, `formulas`, `numpy` *(to be ported)* |

## Worked example — to be added

A complete worked example (a real subject with verified **sold** comps: input JSON → generated
workbook → green-PASS validation → provenance log) will be added once the scripts are ported.
Until then, the rent-comp repo's **10 Lower Spadina** example shows the exact end-to-end pattern
this tool follows.

## Notes & disclaimers

- **Redactions:** internal SharePoint coordinates for the originating organization were removed
  from `CLAUDE.md` (`[REDACTED-*]`), and unrelated projects were excluded from this repo.
- **Data:** example comp records, when added, will be factual sold-transaction data derived from
  condos.ca for analysis; re-use is subject to the source site's terms. Provided for
  reproducibility of the analysis, not redistribution of a listing service.
