# Extract `harlequin-questdb` to a community repository

This staging tree is meant to become a standalone GitHub repository (for example `harlequin-questdb`) before PyPI publish. Run the sync script from the Harlequin fork root whenever canonical adapter source changes:

```bash
./questdb-loop/scripts/sync-harlequin-questdb-package.sh
```

## Push to a new remote (human steps)

1. Copy or clone this directory (`questdb-loop/package/harlequin-questdb/`) to a clean working tree.
2. Initialize git (do **not** copy loop credentials or `.env` files):

   ```bash
   cd harlequin-questdb
   git init
   git add .
   git commit -m "Initial harlequin-questdb package"
   ```

3. Create an empty GitHub repository and add it as `origin`:

   ```bash
   git remote add origin git@github.com:<org>/harlequin-questdb.git
   git branch -M main
   git push -u origin main
   ```

4. Tag releases from that repository when ready for PyPI (`pypi-publish` slice).

Never commit production credentials, `.env` files, or `questdb-loop/` loop state into the community repo.
