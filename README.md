# ionq-data

Daily first-hand market snapshots for the IonQ buy plan. **Data only** — the
generator (`market_data_daily_v1.py`) lives in `~/projects/ionq_data` and is
kept out of this repo on purpose.

- `snapshots/market_snapshot_YYYY-MM-DD_v1.json` — one file per US trading day
- `latest.json` — a copy of the most recent snapshot, for a stable URL

Written by `run_daily.sh` at 08:15 Australia/Sydney, Tuesday–Saturday, after
each US trading day. Every field carries `value`, `as_of`, `source` and
`grade`; a field that could not be fetched first-hand is `null` with an
`error`, never a substituted or stale value.

No credentials are stored here. See `.gitignore`.

## Credit-spread history

- `credit_history.csv` — daily, `date` + `BAMLH0A3HYC, BAMLH0A2HYB, BAMLH0A1HYBB,
  BAMLH0A0HYM2, BAMLC0A4CBBB, BAMLC0A0CM, DGS10` (CCC → B → BB → HY total → BBB
  → IG, then the 10-year yield).
- `credit_history_longrun.csv` — `date` + `BAA10Y, AAA10Y`.
- `credit_episodes.json` — per-episode peak / value at window start / max
  5-observation widening / trading days to peak, plus `current` with each
  series' latest 5-observation change and its percentile against the trailing
  year.

Source: FRED API (`api.stlouisfed.org/fred/series/observations`), fetched
first-hand on every run. Gaps are written as `null` with a reason and are never
filled from another vendor or from memory.

**Known limit, measured 2026-11-22:** FRED serves the six ICE BofA OAS series
only for a rolling ~3-year window (earliest available `2023-11-21`), and ALFRED
holds no vintages for them at all — a vintage request returns *"The series does
not exist in ALFRED"*. Pre-2023 ICE history is therefore **not obtainable** from
FRED/ALFRED. Only the 2025 episode has ICE data; every earlier episode falls
back to `BAA10Y`/`AAA10Y`, which are Moody's **yield spreads, not
option-adjusted spreads**, and so are kept in separate columns and never merged
into an ICE column.

Because FRED's ICE window rolls forward (~1 trading day lost per day), each run
merges fresh observations with what is already committed here, so first-hand
rows once captured are retained. Fresh FRED values win on overlapping dates.
