# Quartet_data

Per-user **data** for the [Quartet](../Quartet) project, kept deliberately
outside the code repo:

- `historical/` — point-in-time market-data archives (`HistoricalDataStore`
  snapshots), one folder per EOD/SOD capture, plus the `index.csv` registry.
- `historical/fixings/` — the master daily-fixing series (e.g. `SOFR.csv`),
  append-only, shared across all snapshots.
- `historical/Static_skew_quote_set/` — the live VCUB skew quote set (PNGs +
  ingest cache are gitignored; the resolved CSVs are tracked).

## Why it lives here, not in the code repo

- **Releases stay lean.** `release/build_release.py` copies only what
  `git ls-files` reports inside the code repo, so nothing here is ever shipped.
- **Each user owns their own history.** Snapshots are generated on the machine
  that runs Quartet (via the EOD-trigger flow); they are not a shared artifact.
- **These are immutable records.** Keeping them in their own git repo preserves
  versioning/backup without coupling to the code repo's history.

## How the code finds this directory

The code repo resolves the location in `src/mds/historical/paths.py`:

1. `$QUARTET_DATA_DIR` if set (this directory).
2. otherwise `<code-repo-parent>/Quartet_data` — i.e. a sibling of the Quartet
   checkout. Works identically for a deployed release tree (its own sibling).

## Integrity / hashes

Each archive carries a content hash in its `manifest.json` and in the root
`index.csv`; it is **location-independent** (computed over serialized content,
never a path). Moving this directory, or generating archives on a fresh
machine, never affects hash verification — archives self-verify on load.
