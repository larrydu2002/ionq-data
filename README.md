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

## Stored state

- `state/ndx_reference_peak.json` — the Nasdaq-100 **reference closing high**
  that B-list item 6 anchors to: `peak_date`, `peak_close`, `source`, `set_at`,
  `set_as_of_data_date` and a `history` array of every peak it has replaced.

Before 2026-11-23 that anchor was recomputed on every run as the maximum of a
rolling 260-row ^NDX window, which meant the anchor silently depended on when
the script happened to run: once the window's start date rolled past the real
peak, the window's own first row became the "peak". Lei ruled on 2026-11-23
that it must be stored state instead.

It moves only when ^NDX closes **strictly above** `peak_close`; the old entry is
appended to `history` and item 6's state clears by construction, because only
breadth readings dated after the anchor count. Every run cross-checks the
stored close against first-hand yfinance data. If the file is missing or
corrupt, if the fetched window does not reach `peak_date`, or if the stored
close disagrees with the data by more than 0.1%, item 6 is written as
`triggered: null` with an `error` and listed in `missing`. **The script never
re-creates or repairs this file** — doing so would reset item 6 without anyone
noticing. Initialise it by hand and re-run.

## FINRA margin debt — observation item F

- `finra_margin_history.csv` — monthly, `month` (YYYY-MM), `debit_mn`,
  `free_credit_cash_mn`, `free_credit_margin_mn`, in $ millions, every month the
  source provides (currently 1997-01 onward).
- `finra_raw/margin-statistics_YYYY-MM-DD.xlsx` — a dated copy of exactly what
  FINRA served on that snapshot date, kept so any number here can be audited
  back to the file it came from.
- `fields.finra_margin` in each snapshot and in `latest.json` carries
  `latest_month`, `debit_mn`, `mom_pct`, `yoy_pct`, `peak_month`,
  `peak_debit_mn`, `from_peak_pct`, `net_credit_mn` (plus its `_mom_mn`,
  `_yoy_mn`, `_min_mn`, `_min_month` and `_rule`), the two flags
  `flag_mom_drop_gt_10` / `flag_yoy_negative`, and
  `new_month_since_last_snapshot`.

**Net credit balance.** `net_credit_mn = free_credit_cash_mn +
free_credit_margin_mn − debit_mn`, in $ millions, so a **negative** number is
net leverage. Lei replaced `debit_to_cash_ratio` with it on 2026-11-23 because
this is the standard published measure. Any month missing a component is `null`
rather than a partial sum, and FINRA only reports `free_credit_margin_mn` from
2010-02, so 1997-01 … 2010-01 carry no net credit at all. Its month-on-month
and year-on-year figures are absolute changes in $ millions, not percentages:
the series is signed and currently far below zero, which makes a percentage
change meaningless. That choice is the script's (`脚本口径`) and is flagged for
Lei to confirm.

**Source.** FINRA's own margin statistics page and the workbook linked on it:
<https://www.finra.org/rules-guidance/key-topics/margin-accounts/margin-statistics>.
FINRA states it does not distribute this data outside that page and offers no
feed, so nothing else is consulted — no news site, no YCharts, no aggregator.
If both the workbook and the page table fail, the block is written as `null`
with an `error`; a value is never estimated or carried over.

**Publication timing.** FINRA publishes the previous month around the third week
of the following month, so `latest_month` normally trails the current month by
one to two months. That lag is expected, not staleness.

**It is an observation item only** (`in_force: false`, note
`观察项 F，只记录，不触发`). It feeds no light, no batch and no trigger, and the
snapshot validator asserts that it has not leaked into any of them.
