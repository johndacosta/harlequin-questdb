# Merge proposal: consolidating `harlequin-questdb`

**From:** John DaCosta ([johndacosta](https://github.com/johndacosta))  
**To:** Rik Huygen ([rhuygen](https://github.com/rhuygen))  
**Date:** 2026-07-25  
**Draft PR branch:** `consolidate/enhanced-adapter-v0.4.0` on [johndacosta/harlequin-questdb](https://github.com/johndacosta/harlequin-questdb)

## Summary

Rik shipped the first `harlequin-questdb` on PyPI (0.3.x) and opened the correct upstream integration path ([harlequin #975](https://github.com/tconbeer/harlequin/pull/975), [harlequin-web #146](https://github.com/tconbeer/harlequin-web/pull/146)). John developed a deeper implementation in parallel on a Harlequin fork, extracted to a standalone package tree, and opened a competing docs PR ([#147](https://github.com/tconbeer/harlequin-web/pull/147)).

**Proposal:** keep **`rhuygen/harlequin-questdb`** and PyPI distribution name as canonical; merge John's enhancements via PR; release **0.4.0**; close duplicate upstream PRs; land one docs PR.

---

## File-by-file comparison

### Package layout

| Path | Rik (`0.3.3`) | John (draft `0.4.0-rc`) | Merge action |
|------|---------------|-------------------------|--------------|
| `src/harlequin_questdb/__init__.py` | Exports `QuestDBAdapter` | Exports `HarlequinQuestDbAdapter` | **Keep both** via aliases (draft branch) |
| `src/harlequin_questdb/adapter.py` | ~218 lines, single file | ~527 lines, connection pool | **Replace** with John's; add 0.3.x API aliases |
| `src/harlequin_questdb/cli_options.py` | 4 options (`username`, default password) | 7 options (`user`, `dbname`, `sslmode`, …) | **Merge** — superset + `username` kwarg compat |
| `src/harlequin_questdb/catalog.py` | — (flat catalog in adapter) | 238 lines, interactive tree | **Add** |
| `src/harlequin_questdb/completions.py` | — | 91 lines, dialect + `functions()` | **Add** |
| `src/harlequin_questdb/interactions.py` | — | 68 lines, catalog sidebar actions | **Add** |
| `tests/test_adapter.py` | Template tests (online, needs live DB) | Offline catalog/contract tests + fixtures | **Expand** |
| `README.md` | Install via `uv tool install --with` | PGWire, pinned 8.2.2, auth warnings | **Merge** best of both |
| `docs/USER_GUIDE.md` | — | Full user guide | **Add** |
| `docs/KNOWN_LIMITATIONS.md` | — | Pin-scoped limitations matrix | **Add** |
| `docker-compose.yml` | — | Pinned `questdb/questdb:8.2.2` | **Add** |
| `CHANGELOG.md` | Present | Release notes for 0.4.0 | **Update** |
| `LICENSE` | MIT | MIT | No change |
| `pyproject.toml` | `psycopg[binary]`, entry `harlequin_questdb:QuestDBAdapter` | `psycopg[pool]`, `harlequin>=2.5` | **Merge** deps; keep Rik's entry-point path |

### `adapter.py` — behavioral differences

| Capability | Rik `0.3.3` | John `0.4.0` draft |
|------------|-------------|-------------------|
| Connection model | Single `psycopg.connect`, manual reconnect in `get_catalog` | `psycopg_pool.ConnectionPool`, keepalive, auto-reconnect on execute |
| Catalog shape | Flat list of tables → columns | `db → schema → relation → column` lazy tree |
| Partition hints | Not shown | `t·DAY`, `mv·MONTH` type labels |
| Designated timestamp | Not shown | `ts★` on column type label |
| Relation kinds | All labeled `table` | table / view / materialized view |
| Catalog interactions | None | Preview, insert columns, SHOW COLUMNS/PARTITIONS |
| Completions | None | `SAMPLE BY`, `ASOF JOIN`, … + bounded `functions()` |
| Query cancel | Not implemented | `IMPLEMENTS_CANCEL = True`, `cancel_safe()` |
| `table_columns()` SQL | f-string with **unescaped** table name | Escaped literal via `_escape_quest_literal()` |
| `connection_id` | libpq conninfo string | `host:port/dbname` |
| Default password | `quest` in CLI default | Default `quest` in pool kwargs (draft compat shim) |
| CLI user field | `username` (`-u`) | `user` (`-U`/`-u`); accepts `username` kwarg for compat |

### Tests

| Suite | Rik | John (Harlequin fork) | Port to community repo? |
|-------|-----|----------------------|-------------------------|
| Plugin discovery | `tests/test_adapter.py` | `test_questdb_packaging.py` | Yes |
| Contract / catalog offline | — | `test_questdb.py` (~27 tests) | Yes (subset in draft) |
| Completions | — | `test_questdb_completions.py` | Yes |
| Docs paths | — | `test_questdb_docs.py` | Yes |
| Online Docker smoke | Implicit in template tests | `test_questdb_online.py` (5 tests) | Optional CI job |
| Packaging gates | — | `test_questdb_packaging.py` | Adapt paths for standalone repo |

John's Harlequin fork has **~63 test functions** across 5 files; the draft branch ports the highest-value offline tests.

### Upstream PRs (who owns what)

| PR | Author | Status | After merge |
|----|--------|--------|-------------|
| [harlequin #975](https://github.com/tconbeer/harlequin/pull/975) | Rik | Open — Tom asked for docs + PyPI repo link | **Refresh** `uv.lock` to 0.4.0, re-request review |
| [harlequin-web #146](https://github.com/tconbeer/harlequin-web/pull/146) | Rik | Vercel fork auth failure | **Base** for final docs PR (+ John's content, `repoMap`) |
| [harlequin-web #147](https://github.com/tconbeer/harlequin-web/pull/147) | John | Vercel fork auth failure | **Close** when #146 updated |

Tom's blocker on #975 (2026-06-30): official docs + verifiable PyPI source/license. Rik addressed PyPI URLs in 0.3.3; docs still pending.

---

## Breaking changes to manage in 0.4.0

| Area | Risk | Mitigation in draft |
|------|------|---------------------|
| Entry point class path | Low | `QuestDBAdapter` alias in `__init__.py`; keep `harlequin_questdb:QuestDBAdapter` |
| CLI option `username` → `user` | Medium | Accept `username=` kwarg in adapter `__init__` |
| Password required vs defaulted | Low | `pool_kwargs.setdefault("password", "quest")` |
| Connection pooling | Low | Transparent to users; may change error timing |
| Catalog UI structure | Low | Improvement; tree replaces flat list |

---

## Proposed release plan

1. **Review** draft branch `consolidate/enhanced-adapter-v0.4.0` on John's fork (or open PR against `rhuygen/harlequin-questdb`).
2. **Agree** co-maintainer roles and contributor credits (`pyproject.toml` authors, `adapters.md`).
3. **Merge** to `rhuygen/harlequin-questdb` `main`.
4. **Publish** `harlequin-questdb==0.4.0` on PyPI.
5. **Refresh** harlequin #975 (`uv lock`); Tom re-reviews.
6. **Update** harlequin-web #146 (or maintainer cherry-pick); close #147.
7. **John** removes in-tree adapter from Harlequin fork; stops duplicate upstream work.

---

## What John is offering

- PR(s) against `rhuygen/harlequin-questdb` with enhanced source, tests, and docs
- Close harlequin-web #147; help merge doc content into #146
- Co-maintain releases and QuestDB pin updates (currently **8.2.2**)
- No competing PyPI package name

---

## Open questions for Rik

1. Comfortable adding John as co-maintainer on repo + PyPI?
2. Prefer one large 0.4.0 PR or staged PRs (catalog → completions → pool)?
3. Keep class names `QuestDB*` (your convention) or `HarlequinQuestDb*` (Harlequin adapter convention)?
4. Who drives the maintainer cherry-pick for harlequin-web (Vercel fork auth)?
