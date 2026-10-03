# Initialization for EDMS application

This folder is the single entry point for running EDMS — clone the repo
and run one command.

## Run it

```bash
cd init
docker compose up --build
```

That's it — nothing to configure first, no `.env` file.

EDMS keeps all its data — collections, request/response history, everything
under `storage/` — in a plain folder on your machine, not hidden inside a
Docker volume: by default `../../edms-data`, a folder next to this repo.
It's created automatically the first time you run this — you don't need
to make it yourself.

**Want your data somewhere else?** Open `docker-compose.yml`, find this
line under `webserver: volumes:`, and change the left-hand side to your
own path:

```yaml
- ../../edms-data:/app/edms_root
```

Whatever you set it to gets created automatically the same way — no
manual folder setup either way.

First run takes a few minutes (Rust release build). Once it's up:

- **App:** http://localhost:3911
- **Backend API:** http://localhost:3000 (see `backend/webserver/API_REFERENCE.md`)

## Automatic offline dummy data

On a fresh installation, `docker compose up --build` automatically runs the
`seed` service before the webserver starts. It creates **25** clearly marked
synthetic `example.invalid` endpoints with **50** request/response/header sets,
tags, bookmarks, history, a collection, view previews, and valid compressed
and uncompressed import/export samples. Every required storage folder receives
sample data. No profile or separate seed command is needed.

Docker creates the default `../../edms-data` folder automatically for both
services. No manual directory creation or YAML edit is required. If you choose
a different storage location, update the host path in both services' mounts.

The seed container has no network access and uses EDMS's existing schema,
folder manager, EID formatter, and QP writers. This replaces the old automatic
network demo with synthetic fixtures; no actual endpoint requests are made.
The older network-based `seed.mjs` remains available for manual use.

When storage is populated or `backend/webserver/data/edms.db` exists, seeding
exits successfully without changing anything, allowing normal startup and
repeat runs. Missing directories or generation errors still fail startup.
An exited seed container with status 0 is expected.

With Rust installed, you can also create an empty local folder and run:

```bash
cargo run --manifest-path backend/compute/Cargo.toml --bin synthetic-data -- /absolute/path/to/empty-data
cargo test --manifest-path backend/compute/Cargo.toml --bin synthetic-data
```

For Docker, use the seed container so database paths point to `/app/edms_root`.

## Resetting

```bash
docker compose down -v
rm -rf ../backend/webserver/data
```

Your storage folder (`../../edms-data`, or wherever you pointed it) is
untouched by this — delete it yourself if you want a truly clean slate.
