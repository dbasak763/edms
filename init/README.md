# Run EDMS with synthetic storage

## Configure storage first

1. Create an **empty directory outside this checkout** for EDMS data:

   ```bash
   mkdir -p "$HOME/edms-data"
   ```

2. Edit `init/docker-compose.yml`. Replace
   `/REPLACE_WITH_YOUR_EDMS_DATA_PATH` in `webserver.volumes.source` with
   the absolute path you just created (for example `/home/alex/edms-data`).
   Use the expanded path, not `~`. The seed service uses the same YAML anchor.
   Docker refuses to create this folder automatically.

3. Only after configuring the path, run:

   ```bash
   cd init
   docker compose build
   docker compose --profile demo run --rm seed
   docker compose up
   ```

Open http://localhost:3911 for the app, or http://localhost:3000 for the API.
Python, Node, and a host Rust installation are not required. The initial image
build downloads dependencies; **generating the data makes no network requests**.

## Dummy data

The initializer uses EDMS's current SQLite schema, folder manager, EID
formatter, and request/response/header writers. It creates four explicitly
synthetic `example.invalid` endpoints (GET, POST, PUT, DELETE), eight QP pairs,
metadata, history, tags, bookmarks, and a collection membership database.
The allocation watermark is advanced so subsequent endpoints get fresh IDs.

Every required storage folder receives a sample: account workspace metadata,
history JSON, global EQP JSON, collection SQLite, RepoView Markdown, WebView
HTML, takeout Markdown, and valid compressed/uncompressed import and export
bundles. Legacy active, repo, session-backup, exports, temp, and docs folders
also contain demo Markdown. These are preview fixtures, not authenticated
accounts or claims that an endpoint was really tested. WebView bundles contain
no SQLite files. The webview/repoview catalogs keep the application's current
null backing-file convention.

The initializer only accepts an existing, completely empty directory. A second
run fails without modifying existing data. It is optional: skip the seed command
for a clean installation. Stop EDMS before initializing any demo directory.

Compose keeps the main database at its existing host location,
`backend/webserver/data/edms.db`. The initializer refuses to overwrite that
file too, even if the storage folder is empty. Collection membership databases
live inside the configured storage folder.

The older `seed.mjs` remains available as a separate manual API demo; it contacts
a public API and is no longer run automatically by Compose.

## Local initializer and checks

With Rust installed, initialize a fresh folder without Docker:

```bash
cargo run --manifest-path backend/compute/Cargo.toml --bin synthetic-data -- /absolute/path/to/empty-data
cargo test --manifest-path backend/compute/Cargo.toml --bin synthetic-data
```

For a local webserver, point `storage.root` in `backend/webserver/config.yaml`
at that existing directory (relative to the webserver working directory), and
set `EDMS_DB_PATH` to its `edms.db`. Generate Docker fixtures through the seed
container so SQLite catalog paths use `/app/edms_root`, not host paths.

`docker compose down` stops the stack and leaves your storage directory intact.
To try another dataset, create another empty directory and update the YAML path.
