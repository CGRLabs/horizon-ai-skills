# Pitfalls — The Silent-Failure Catalog

These cost real debugging hours on Horizon 0.2.97. The unifying theme: **Horizon can silently reject content it does not understand, without writing errors to logs.** Assume nothing worked until you saw it work in-game.

## 1. New items/blocks may not appear — at all

The full test mod (2 blocks, 4 items, 19 recipes, models, blockstates) simply did not appear in-game on v0.2.97, while vanilla-item recipes from the same pipeline worked.

- Suspected cause: new declarations need model/blockstate assets the game does not auto-resolve yet.
- Strategy: prove the pipeline with vanilla-item-only recipes first. Then add ONE new block or item, test, and only continue if it appears.
- If a new declaration does not show up, downgrade to a recipe-only approach using vanilla ids.

## 2. Item `components` are unsupported

`components` on items had to be stripped for the mod to work. Do not emit them; re-test on each new Horizon version before relying on them.

## 3. Silent rejection — the debugging playbook

When "the mod doesn't show up":

1. Confirm the `.hmod` is in the game's `horizon-mods/` directory (not the project's `target/`).
2. Confirm the enable toggle: content mods load **once at startup** — the enable dance is quit → relaunch → Options → Mods → enable → **quit completely again** → relaunch → world. Enabling mid-session does nothing.
3. Check `horizon-mods.txt` in the game folder — the mod should appear as `mod.<id>:true`.
4. Run `cargo horizon-mod check <file>` and `horizon-inspect <file> --json`; fix every error (manifest syntax, id rules, unknown fields).
5. Bisect content: rebuild with only the manifest + one recipe, confirm it works, then add files back one at a time.

## 4. Manifest strictness

- `deny_unknown_fields`: any typo'd or speculative key invalidates the manifest. Only send keys from manifest-reference.md.
- Id characters: `[a-z0-9_]` — `ember-forge` is invalid, `ember_forge` is valid.
- Keep one bundle per `.hmod`; the namespace directories must match the id.

## 5. Wrong-layer data files

`data/minecraft/recipe/...` inside your mod is the wrong layer. Use `data/<mod-id>/recipe/...`.

## 6. Java mods do not port by copying

`.jar` files never run in Horizon. Port the *idea*: recreate recipes/data in Horizon's JSON formats with vanilla or your own ids. Native-level behavior (code) is a different skill — see `horizon-native-mod`.
