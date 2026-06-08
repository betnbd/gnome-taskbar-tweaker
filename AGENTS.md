# AGENTS.md

## Purpose

GNOME Shell extension (UUID `gnome-taskbar-tweaker@betnbd`) that reorders supported top-panel `statusArea` items across the left, center, and right sections. Only items in `Main.panel.statusArea` are movable; unsupported actors are left untouched.

## Stack

- GNOME Shell extension JavaScript (GJS), targets Shell `49` and `50`
- GSettings schema (`schemas/org.gnome.shell.extensions.gnome-taskbar-tweaker.gschema.xml`)
- Bash helper scripts under `scripts/`, GNU Make

## Build / Run / Test

```bash
make check                 # compile schemas + node --check (lint skipped if node absent)
make schemas               # glib-compile-schemas only
make install               # rsync into ~/.local/share/gnome-shell/extensions/<uuid> + install schema
make uninstall
make package               # gnome-extensions pack -> dist/ (runs check first)
make install-package       # pack + gnome-extensions install --force
./scripts/install.sh       # wrapper around `make install`
./scripts/smoke-test.sh --install   # install, assert enabled, request live rescan, verify settings
./scripts/manual-test.sh
```

Enable after install (requires a live GNOME session):

```bash
gnome-extensions enable gnome-taskbar-tweaker@betnbd
./scripts/open-prefs.sh
```

Diagnostics (run inside the GNOME session): `scripts/show-items.sh`, `show-layout.sh`, `show-status.sh`, `logs.sh`, `request-sync.sh`, `reset-layout.sh`, `set-layout.sh`.

Required at runtime/build: `gnome-extensions`, `gsettings`, `glib-compile-schemas`, `python3`, `rsync`. `node` is optional (lint only). `smoke-test.sh` and the diagnostic scripts need `DBUS_SESSION_BUS_ADDRESS` set, i.e. a real logged-in GNOME session.

## Layout

- `extension.js` — Shell runtime integration (enable/disable, live panel manipulation)
- `prefs.js` — preferences window (move/reorder/refresh/reset controls, persist toggle)
- `layout.js` — shared layout parsing and movement logic; packed via `--extra-source`
- `schemas/` — GSettings schema; keys: `panel-layout`, `baseline-layout`, `available-items`, `persist-layout`, `layout-version`, `sync-generation`, `last-error`
- `scripts/` — install, package, diagnostics, test helpers (`common.sh` is shared)
- `RELEASING.md` / `CHANGELOG.md` — release checklist and notes; `QA.md` — manual QA matrix

## Gotchas

- Bump the integer `version` field in `metadata.json` before packaging any public build; the release artifact name (`gnome-taskbar-tweaker-v<version>.zip`) is derived from it.
- After a metadata-only compatibility change, GNOME Shell can keep reporting `State: OUT OF DATE` until you log out and back in (stale session cache). `smoke-test.sh` downgrades this to a warning when the installed metadata already advertises the running shell.
- GNOME Shell does not always hot-reload edits; disable/re-enable the extension or start a fresh session to pick up changes.
- Disabling the extension does NOT restore the original panel order — run `reset-layout.sh` (or the Reset button) first if you want the captured baseline back.
- `make install` deletes `schemas/gschemas.compiled` in the install dir and recompiles, and also installs the schema into `~/.local/share/glib-2.0/schemas` so `prefs.js` can read settings outside the extension dir.
- `version` in `metadata.json` is an integer, not semver.
