# API Contract — CFA Load Pricing Engine

## Base URL
- Local: `http://localhost:8000`
- Replit: Replit-assigned URL

---

## POST `/api/quote`
Runs the pricing pipeline and returns a full quote response.

### Request body (JSON)
See `docs/SCHEMA.json` for full JSON Schema. High-level fields:

- `equipment_type` (string, required)
- `weight_lbs` (number, required)
- `loaded_miles` (number, required)
- `deadhead_miles` (number, optional, default 0)
- `other_flat` (number, optional, default 0)
- optional: origin/destination/pickup_date for risk + metadata
- optional overrides: `diesel_price_override`, `target_margin_override`, `contingency_override`, `urgency_override`

### Response body (JSON)
Must include:
- totals (miles, days)
- market (rate, minimum, linehaul)
- fuel (surcharge per mile + total)
- costs (itemized + subtotals)
- risk (score + contingency)
- price (cost floor, market ref, min, recommended, final)
- profit (dollars + percent)
- ledger (rows + subtotals + net)
- analytics (sensitivity, scenario ladder, breakeven)
- copy/paste line
- warnings list

---

## GET `/api/config/status`
Returns config status and metadata.

Returns:
- `config_hash`
- `loaded_at`
- `source` (dropbox vs cache)
- counts (rate cards rows, fuel rows, accessorial rows)
- parsing warnings/errors

---

## POST `/api/config/reload`
Forces re-download and re-parse of Excel workbook.

---

## GET `/api/health`
Simple health check.

---

## GET `/api/quotes/recent?limit=50`
Returns recent quote history (stored in SQLite).

---

## Errors
All endpoints should return:
- `error_code`
- `message`
- `details` (optional)
- `trace_id` (optional)
