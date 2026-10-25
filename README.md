# Creative Flows — Peckboard Plugin Registry

A catalog of installable [Peckboard](https://github.com/PeckBoard/peckboard)
WASM plugins for creative work: writing, stories, visual novels. Same format
as the main [PeckBoard/plugins](https://github.com/PeckBoard/plugins)
registry, so Peckboard reads it the same way.

It is a **single static file**, [`registry.json`](registry.json). There is no
service here. Each plugin's `.wasm` is built and published from its **own**
repository as a GitHub release asset. This registry only points at those
assets and records the SHA-256 that Peckboard checks the download against.

## Plugins

| Plugin | Version | What it does |
| ------ | ------- | ------------ |
| [Story Writer](https://github.com/firestar/peckboard-story-writer) (`story-writer`) | 0.1.2 | Per-project Story tab: setup wizard and a card-driven writing pipeline (records → pilot → cold read → owner gate → parallel chapters → review passes → export). |

## Use this registry

1. In Peckboard open **Settings → Plugin Registry**, tab **Repositories**.
2. Add `PeckBoard/creative-flows`. Peckboard resolves it to
   `https://raw.githubusercontent.com/PeckBoard/creative-flows/main/registry.json`.
3. Switch to **Browse**, find the plugin, click **Install**, then approve its
   hooks in **Settings → Plugins**.

Or, for a host you configure by environment, set
`PECKBOARD_PLUGIN_REGISTRY_URL` to the raw URL above.

## How install works (the trust model)

1. Peckboard fetches `registry.json` and lists the plugins.
2. You pick one. Peckboard downloads its `url`, computes the SHA-256, and
   **rejects the file if it doesn't match `sha256`** in the index.
3. The verified `.wasm` is saved as `<id>.wasm` in your data dir and loaded
   **inert**: none of its hooks run yet.
4. Peckboard asks you to approve the exact set of `hooks` the plugin declares.
   Nothing the plugin declared runs until you approve.

## `registry.json` format

Validated by [`schema/registry.schema.json`](schema/registry.schema.json)
(JSON Schema draft 2020-12), copied from PeckBoard/plugins.

| Field           | Required | Notes |
| --------------- | -------- | ----- |
| `id`            | yes      | Stable id. **Must equal the `.wasm` file stem** Peckboard saves it under. `^[a-z0-9_-]+$`. |
| `name`          | yes      | Display name. |
| `description`   | yes      | Short summary. |
| `author`        | yes      | Publisher. |
| `homepage`      | no       | Source/docs URL (`https://`). |
| `version`       | yes      | Semantic version of the published `.wasm`. |
| `hooks`         | yes      | Every hook the plugin declares: what the operator approves. Non-empty, unique. |
| `url`           | yes      | `https://` download URL of the `.wasm`. |
| `sha256`        | yes      | Lowercase hex SHA-256 of the bytes at `url`. |
| `min_peckboard` | no       | Lowest Peckboard version the plugin supports. |
| `tags`, `category` | no    | Browse filters. |

## Adding or updating a plugin

1. Publish the plugin `.wasm` as a release asset in its own repo.
2. Compute its checksum:
   ```bash
   curl -sL -o plugin.wasm "<release asset url>"
   sha256sum plugin.wasm
   ```
3. Add or update the entry in `registry.json`. Keep `id` stable across
   versions; bump `version`, `url`, `sha256` and `hooks` together.
4. Validate, then open a PR:
   ```bash
   python scripts/validate.py                     # structure
   python scripts/validate.py --verify-checksums  # downloads + checks sha256
   ```
