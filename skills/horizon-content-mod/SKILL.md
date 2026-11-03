---
name: horizon-content-mod
description: Build, validate, and install Horizon content mods (.hmod) — data-driven mods for the Horizon game (the Rust rewrite of Minecraft Java Edition 26.3). Use when the user wants to create a Horizon mod, add recipes, blocks, items, loot tables, tags, or lang files, fix a mod that silently does not appear in-game, package or check a .hmod, or asks about horizon-mod.toml, capabilities, or the Horizon mod pipeline. Also use for porting Java datapack-style content to Horizon. Even if the user never says "content mod", any "make me a Horizon mod that adds X" request should trigger this skill.
---

# Horizon Content Mods (.hmod)

Horizon content mods are **data and assets only — no code**. A `.hmod` is a zip containing a `horizon-mod.toml` manifest plus optional `assets/` and `data/` directories, layered over the game's base resources at startup. WebAssembly (`mod.wasm`) is dead and rejected. If the user wants executable behavior, that is a native mod — see the `horizon-native-mod` skill instead.

## The working loop

1. **Scaffold** the project folder (manifest + `data/`/`assets/`), or run `cargo horizon-mod new <path> --name "Name"`. If you used the scaffold: **delete its generated example content** (blockstates, items, models, block declarations, loot tables, and anything under `data/minecraft/`) before adding yours — it demonstrates the exact silently-failing patterns described in `references/pitfalls.md`. Keep only the manifest and your own files.
2. **Add content** under the mod's own namespace: `data/<mod-id>/recipe/...`, `assets/<mod-id>/lang/...`. Never write into `data/minecraft/` — put overrides in `data/<mod-id>/`.
3. **Build**: `cargo horizon-mod build` → produces `target/horizon-mod/<id>.hmod`.
4. **Check**: `cargo horizon-mod check target/horizon-mod/<id>.hmod`.
5. **Install**: copy the `.hmod` into the game's `horizon-mods/` directory. Where that is: inside the **Horizon install folder**, next to `horizon-client.exe` and `horizon-mods.txt` (e.g. `...\horizon-windows-x64\horizon-mods\`) — this is where our installs load from. The user-data directory (`%APPDATA%\Roaming\horizon`) is a different location and held no mods folder. Create the folder if a fresh install lacks it.
6. **Enable**: content mods load once at startup — quit the game completely, relaunch, Options → Mods → enable, quit completely again, relaunch, load a world.

After step 3, optionally run `horizon-inspect target/horizon-mod/<id>.hmod --json` (exit 0 = valid) for a security/manifest report if the tool is installed.

## Start from what is proven to work

Data-driven **recipes with vanilla item ids are the verified baseline** — they work with no textures, models, or item registrations:

```json
{
  "type": "minecraft:smelting",
  "cookingtime": 200,
  "experience": 0.35,
  "ingredient": "minecraft:rotten_flesh",
  "result": { "id": "minecraft:leather" }
}
```

(`minecraft:blasting` for blast furnace; `cookingtime` in ticks, 200 = ~10 s.)

**Add one kind of content at a time and retest after each.** Horizon can silently reject content it does not understand, so a big-bang mod is undebuggable.

## Hard-won pitfalls (verified on Horizon 0.2.97)

Read `references/pitfalls.md` before designing anything beyond vanilla-item recipes — the failure modes there are silent and cost hours. Headlines:

- New item/block declarations may **not appear in-game**; the game can reject them without logging errors.
- Item `components` are unsupported on this version.
- Manifest uses `deny_unknown_fields`; mod ids are `[a-z0-9_]` only.

## References — read on demand

- `references/manifest-reference.md` — full `horizon-mod.toml` field reference: api lines, capabilities, side, id rules. Read before writing any manifest.
- `references/data-and-assets.md` — verified data formats (recipes, lang) and asset layout. Read when adding content beyond one smelting recipe.
- `references/pitfalls.md` — the silent-failure catalog and debugging strategy. Read before testing and whenever "the mod doesn't show up".
- `references/validation-and-publishing.md` — inspector/checker usage, registry rules (`index.json`, `revoked.toml`, MOD-POLICY), deterministic packaging. Read before publishing.

## Rules that are not negotiable

- Content mods declare capabilities; never request more than the mod needs — players see these before enabling.
- Java mod **jars do not run** in Horizon; do not try to port `.jar` files, port their *data* instead.
- Horizon contains no Mojang assets; never bundle Minecraft assets into a mod. Derive everything or reference vanilla ids.
