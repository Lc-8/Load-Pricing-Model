# Codex Build Prompt — CFA Load Pricing Engine (Excel-Driven)

You are implementing a full-stack quoting app in a NEW empty repository.

**Goal:** replicate the Excel workbook logic deterministically in code, with an “over the top” analytics layer.

## Source-of-truth
Use this workbook as config:
- `Advanced Load Pricing Model.xlsx`

Do NOT attempt to evaluate Excel formulas at runtime. Parse config + re-implement the math.

## Required docs in repo
- `README.md` (already provided)
- `docs/EXCEL_SCHEMA.md` (already generated from workbook)
- `docs/ENGINE_SPEC.md` (Excel formulas mapped to code)
- `docs/API_CONTRACT.md`
- `docs/SCHEMA.json`

## Implement these modules
Backend:
- FastAPI app with endpoints:
  - POST `/api/quote`
  - GET `/api/config/status`
  - POST `/api/config/reload`
  - GET `/api/quotes/recent`
  - GET `/api/health`

Excel config loader:
- Parse defined names + Excel tables (openpyxl)
- Build config objects:
  - Rate cards
  - Fuel surcharge config + schedule
  - Assumptions
  - Accessorials (optional)
- Validate config and emit warnings
- Cache config + compute `config_hash`

Pricing engine:
- Provide 3 modes:
  - Quick Estimate (matches `Quick Estimate` sheet logic)
  - Full Quote (matches `Cost Model` + `Pricing Summary`)
  - Blank6 (matches `B6_LoadsTable` column formulas)
- ALWAYS output: costs, price, profit, ledger, analytics

Analytics engine:
- Sensitivity analysis table
- Scenario ladder
- Break-even metrics
- Risk explanation (score breakdown)

Persistence:
- SQLite quote history: store request + response + config hash + warnings

Frontend:
- Mobile-first React UI:
  - Inputs
  - Results cards
  - Ledger table
  - Analytics accordion
  - Config status panel
  - Copy/paste button

Testing:
- Unit tests for:
  - rate card lookup edge cases
  - fuel surcharge compute
  - end-to-end pipeline with fixed sample input
- Optional “Excel parity test” runner

## Git gotcha
If repo is empty: you MUST create files and commit/push to create `main` branch.
