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
