# harlequin-questdb 0.1.0

First public release of the Harlequin adapter for [QuestDB](https://questdb.io/) over **PGWire**.

## Install

```bash
pip install harlequin harlequin-questdb
```

Or with uv:

```bash
uv tool install harlequin
uv tool install harlequin-questdb
```

## Quick start

Start QuestDB (pinned **8.2.2**):

```bash
docker compose up -d
```

Connect with Harlequin:

```bash
harlequin --adapter questdb \
  --host 127.0.0.1 \
  --port 8812 \
  --user admin \
  --password quest \
  --dbname qdb \
  --sslmode disable
```

## Highlights

- **PGWire** connection via `psycopg` (default port **8812**)
- **QuestDB-native catalog** from `tables()` / `table_columns()` — partition strategy (`t·DAY`) and designated timestamp (`ts★`) in the sidebar
- **Time-series completions** — `SAMPLE BY`, `ASOF JOIN`, `LATEST ON`, `INTERPOLATE`, `LT JOIN`, `LATEST BY`, plus live `functions()` names
- **Guide-level docs** — install, auth, PGWire limits, SAMPLE BY / ASOF / LATEST ON recipes
- **Honest limitations doc** — views, indexes, export/copy, cancel behavior

## Documentation

- [User guide](docs/USER_GUIDE.md)
- [Known limitations](docs/KNOWN_LIMITATIONS.md)
- [Publish runbook](PUBLISH.md)

## Requirements

- Python 3.10+
- Harlequin `>=2.5,<3`
- QuestDB **8.2.2** recommended (see `docker-compose.yml`)

## Known limitations (v1)

- No Harlequin export/copy pipeline
- No custom QuestDB syntax lexer
- ILP ingestion is outside Harlequin
- Best-effort query cancel

Full matrix: [docs/KNOWN_LIMITATIONS.md](docs/KNOWN_LIMITATIONS.md).
