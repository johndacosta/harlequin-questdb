# Publish `harlequin-questdb` to PyPI

Maintainer runbook for the community `harlequin-questdb` repository. Live PyPI upload is **human-operated only** — never automate `twine upload` from CI or the Ralph loop.

## Before you publish

1. Run the Harlequin fork sync script so staged source matches canonical adapter code:

   ```bash
   ./questdb-loop/scripts/sync-harlequin-questdb-package.sh
   ```

   (From the community repo after extract, skip this step and verify `src/`, `docs/`, and `pyproject.toml` are current.)

2. Confirm all Must items in `V1_SHIP_CHECKLIST.md` are satisfied in the fork loop before tagging.

3. Bump `version` in `pyproject.toml` for the release.

## Build

From the package root (`harlequin-questdb/`):

```bash
uv build
```

Artifacts land in `./dist/` (`harlequin_questdb-*.whl` and `harlequin_questdb-*.tar.gz`).

## Release candidate (local wheel install)

Test the built wheel in a clean environment **without** PyPI credentials:

```bash
python -m venv /tmp/hq-questdb-rc
source /tmp/hq-questdb-rc/bin/activate
pip install ./dist/harlequin_questdb-*.whl
python -c "from harlequin.plugins import load_adapter_plugins; assert 'questdb' in load_adapter_plugins()"
harlequin --adapter questdb --help
```

RC validation uses a local wheel only. Do not embed API tokens or credential examples in this repo.

## Upload to PyPI (human only)

1. Install `twine` in a maintainer environment (not in CI).
2. Authenticate with your PyPI account using your **local** credentials (for example `~/.pypirc` or an environment variable on your machine). **Never** commit tokens, `.pypirc`, or credential snippets to git.
3. Upload:

   ```bash
   twine upload dist/*
   ```

## Never commit to the community repo

- `.env` files or production secrets
- Loop / Ralph credentials or local automation state
- `~/.pypirc`, PyPI API tokens, or upload scripts containing secrets
- `questdb-loop/` paths, loop artifacts, or fork-only directory layout

See also [`EXTRACT.md`](EXTRACT.md) for pushing this tree to GitHub before the first PyPI release.
