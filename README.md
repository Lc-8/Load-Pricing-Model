# CFA Load Pricing Engine — “Badass” Quick Estimate + Full Quote App (Excel Source-of-Truth)

This repo is a **mobile-first load quoting system** for *hotshot / box truck / light-duty dray* loads.  
It is built around one core idea:

> **The Excel workbook is the source-of-truth for pricing configuration.**  
> Code loads the workbook, parses the config tabs, and runs a deterministic pricing engine that matches the workbook logic (and can exceed it with analytics).

---

## What you get (outputs)

Given shipper-provided inputs, the engine returns:

1) **Operating cost estimate (expenses)** — itemized + subtotaled  
2) **Recommended customer charge (price)** — profit-built + market-aware  
3) **Profit $ and profit %** (+ optional “cash profit” view)  
4) **Mini-ledger** (debits, credits, subtotals, final net)  
5) **Copy/paste quote line** (SMS/email ready)  
6) **Analytics** (sensitivity, scenario ladder, break-even, risk explanation)  

---

## Workbook used by this repo (source-of-truth)

**Workbook file:** `Advanced Load Pricing Model.xlsx`

The engine parses the workbook using **Excel Tables** (preferred) and **Defined Names** (named ranges).  
See the full schema map here:

- `docs/EXCEL_SCHEMA.md`
- `docs/EXCEL_SCHEMA.json` (machine-readable)

### Sheet inventory (all sheets in the workbook)
- **Quick Estimate**
- **README**
- **Assumptions**
- **Accessorials**
- **Rate Cards**
- **Fuel Surcharge**
- **Load Input**
- **Cost Model**
- **Pricing Summary**
- **Blank 6 Read Me**
- **Blank 6 Assumptions**
- **Blank 6 Rate Card**
- **Blank 6 Loads**
- **Blank 6 Load Quote**
- **Blank 6 Dashboard**

---

## Three engine “modes” mapped to the workbook

### Mode 1 — Quick Estimate (sheet: `Quick Estimate`)
Fast quoting, minimal inputs, outputs a rough estimate + mini-ledger.

**Primary dependencies:**
- `Assumptions` (global settings)
- `Rate Cards` (market reference)
- `Fuel Surcharge` (FSC logic + schedule table)

This is what you use when a shipper calls/texts and you need a quote in 30 seconds.

### Mode 2 — Full Quote (sheets: `Load Input`, `Cost Model`, `Pricing Summary`, `Quote Copy`)
More detailed quote logic:
- richer accessorial handling
- more explicit cost model
- printable quote output (“Quote Copy”)

This is what you use when the load is complex or high value.

### Mode 3 — “Blank 6” Batch Quoting (sheets: `Blank 6 Loads`, `Blank 6 Load Quote`, `Blank 6 Dashboard`, `Blank 6 Invoices`, plus B6 config sheets)
A batch workflow for quoting/storing multiple loads using a table-driven engine:
- `Blank 6 Loads` has a structured table (`B6_LoadsTable`) with computed pricing columns
- `Blank 6 Load Quote` renders a single selected load
- `Blank 6 Dashboard` is a summary view
- `Blank 6 Invoices` provides invoice formatting
- `Blank 6 Assumptions` and `Blank 6 Rate Card` hold B6 configuration

---

## “C2” definition (Excel loaded from Dropbox)
C2 = **Excel workbook is loaded from Dropbox at runtime** and treated as config.

- Backend downloads workbook from Dropbox.
- Parses config (tables + named ranges).
- Caches parsed config + hash/version.
- If Dropbox is down, uses **last known good** cache.

Environment variables:
- `DROPBOX_ACCESS_TOKEN`
- `DROPBOX_WORKBOOK_PATH` (e.g. `/Apps/CFA/Advanced Load Pricing Model.xlsx`)
- `CONFIG_CACHE_TTL_SECONDS` (e.g. `300`)

---

## Tech stack

### Backend
- Python 3.11+
- FastAPI
- Pydantic
- openpyxl
- SQLite (quote history / audit)

### Frontend
- React + Vite (mobile-first)
- Tailwind or plain CSS
- iPhone-friendly inputs + big tap targets

### Deployment
- Replit (easy deploy)
- GitHub (source control)

---

## Repo layout (expected)

```
backend/
  app/
    main.py
    engine/
      excel/
        schema.py
        parser.py
        validators.py
      pricing/
        pipeline.py
        rate_cards.py
        fuel_surcharge.py
        costs.py
        margin.py
        risk.py
        minimums.py
        ledger.py
        rounding.py
      analytics/
        sensitivity.py
        breakeven.py
        scenario_ladder.py
      persistence/
        db.py
        models.py
frontend/
  src/
    components/
docs/
  EXCEL_SCHEMA.md
  ENGINE_SPEC.md
  API_CONTRACT.md
```

---

## Deterministic math (non-negotiable)
**We do not rely on Excel to calculate formulas on the server.**  
The backend:
1) parses config inputs from Excel  
2) re-implements the workbook algorithms in code  
3) verifies with golden tests against known workbook cases  

That makes the engine:
- repeatable
- testable
- auditable
- fast

---

## “Badass” analytics layer (what makes this more than Excel)
Every quote also outputs:
- Sensitivity table (diesel, margin, deadhead)
- Scenario ladder (aggressive / target / premium)
- Break-even charge + $/mile views
- Risk score explanation (why contingency moved)

Quote history is stored to create a dataset for ML later (acceptance prediction, rate tuning, lane priors).

---

## Quick start (local)
1) `cd backend && pip install -r requirements.txt`
2) `uvicorn app.main:app --reload`
3) `cd frontend && npm i && npm run dev`

---

## GitHub + Codex gotcha (IMPORTANT)
If Codex says:
> “This repository is empty. Create a default branch (e.g. main) by pushing an initial commit, then retry.”

It means **there is no `main` branch yet**. Fix by pushing an initial commit (see `docs/GITHUB_BOOTSTRAP.md`).

---

## License
Private / proprietary (owner-operator internal tool)
