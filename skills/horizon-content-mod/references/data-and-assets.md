# Data and Assets — Verified Formats

Everything here was confirmed working on Horizon 0.2.97 with an installed, enabled mod. Recipe schema cross-checked against Horizon source (`crates/horizon-client/src/lan/advancements.rs`).

## Smelting recipe — VERIFIED

`data/<mod-id>/recipe/flesh_to_leather.json`:

```json
{
  "type": "minecraft:smelting",
  "cookingtime": 200,
  "experience": 0.35,
  "ingredient": "minecraft:rotten_flesh",
  "result": { "id": "minecraft:leather" }
}
```

## Blasting recipe — VERIFIED

`data/<mod-id>/recipe/bamboo_to_coal.json`:

```json
{
  "type": "minecraft:blasting",
  "cookingtime": 100,
  "experience": 0.15,
  "ingredient": "minecraft:bamboo",
  "result": { "id": "minecraft:coal" }
}
```

Notes:
- `cookingtime` is in ticks (200 = ~10 seconds).
- `experience` = XP orbs when taking the output.
- `result.id` — vanilla ids work with or without the `minecraft:` prefix in the result; ingredients used with the prefix.
- Other cooking types (`minecraft:smoking`, `minecraft:campfire_cooking`) follow the same shape but were not separately verified.

## Lang file — VERIFIED

`assets/<mod-id>/lang/en_us.json`:

```json
{
  "recipe.flesh_to_leather": "Rotten Flesh to Leather"
}
```

Used for recipe display names.

## Namespace rules — IMPORTANT

- Put files under `data/<your-mod-id>/...`, **never** `data/minecraft/...`. The game layers mod namespaces over base resources; writing into `data/minecraft/` is the wrong layer and behaves unpredictably.
- Mod id must match the namespace directory exactly (`[a-z0-9_]`).

## What was NOT verified (as of Horizon 0.2.97)

- New block/item declarations (with models + blockstates) — did **not** render in-game; see pitfalls.md.
- Item `components` — unsupported; stripped from the working mod.
- Advancement/loot-table JSONs — formats expected to mirror vanilla but not proven in-game yet.

Treat anything in this section as untested: add one file, build, install, test in-game, and only then add the next.
