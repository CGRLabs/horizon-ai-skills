# horizon-mod.toml — Field Reference

Source of truth: `crates/horizon-mods/src/manifest.rs` in the Horizon repo (verify against the current checkout — the manifest is schema-strict with `deny_unknown_fields`, so an unknown or misspelled key breaks the whole mod).

## Verified example

```toml
id = "ember_forge"
name = "Ember Forge"
version = "0.1.0"
description = "Test mod: rotten flesh to leather, bamboo to coal."
authors = ["test"]
api = "0.2"
side = "both"
capabilities = ["registries"]
```

## Fields

| Field | Required | Rules |
|---|---|---|
| `id` | yes | `[a-z0-9_]` only, 1–64 chars. **Dashes are rejected.** Must match the namespace used under `data/` and `assets/`. |
| `name` | yes | Display name. A value of `"\\"` or empty is a red flag — checkers flag missing/malformed names. |
| `version` | yes | Semver-style string, e.g. `0.1.0`. |
| `description` | recommended | Empty/missing triggers a quality warning in horizon-inspector. |
| `authors` | recommended | List of strings. Missing triggers a warning. |
| `api` | yes | See API lines below. Content mods need `"0.2"`. |
| `side` | no | `client` (default) \| `server` \| `both`. |
| `capabilities` | yes for content | Content mods require `"registries"`. |

## API lines

| Line | Meaning |
|---|---|
| `0.1` | code mods (legacy) |
| `0.2` | content mods: assets/data, blocks/items/effects — **use this** |
| `0.3` | server mods (experimental) |

## Capabilities

Observed set: `game-read`, `chat`, `hud`, `storage`, `command`, `registries`, plus experimental `ui`, `render`, `input`, `players`, `world`, `recipes`.

Rules:
- Content mods (api `0.2`) require `registries`.
- Experimental capabilities require explicit opt-in; expect a warning from inspectors and a prompt for players.
- Request the minimum set — the player sees capability descriptions before enabling the mod.

## Package layout

```
<mod-id>/
  horizon-mod.toml
  data/
    <mod-id>/
      recipe/*.json ...
  assets/
    <mod-id>/
      lang/en_us.json
```

Built with `cargo horizon-mod build` into `target/horizon-mod/<id>.hmod` (a zip of the above). The bundle must not contain `mod.wasm` — WASM mods no longer run and inspection rejects the package.
