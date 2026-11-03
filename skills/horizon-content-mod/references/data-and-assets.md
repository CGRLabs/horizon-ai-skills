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
- Use the `minecraft:` prefix on both `ingredient` and `result.id` — that is what the verified examples do. (The result id was also observed working without the prefix; there is no reason to deviate.)
- Other cooking types (`minecraft:smoking`, `minecraft:campfire_cooking`) follow the same shape but were not separately verified.

## Schema limits (unverified territory)

- Verified fields: `type`, `ingredient` (single item id string), `result.id`, `cookingtime`, `experience`. A `result.count` for >1 output is plausible but **unverified** — ship 1:1 ratios first and test before relying on it.
- Shaped/shapeless crafting, tag ingredients, and multi-ingredient formats: **not verified** on 0.2.97. If you must attempt them, read the recipe handling in the Horizon source first (`crates/horizon-client/src/lan/advancements.rs` in the pinned checkout) and add one file at a time per the testing loop.

## Lang file — VERIFIED

`assets/<mod-id>/lang/en_us.json`:

```json
{
  "recipe.flesh_to_leather": "Rotten Flesh to Leather"
}
```

Used for recipe display names; the verified key pattern is `recipe.<file-stem>`. Where exactly it renders and which other keys exist is unverified — stick to this pattern.

## Namespace rules — IMPORTANT

- Put files under `data/<your-mod-id>/...`, **never** `data/minecraft/...`. The game layers mod namespaces over base resources; writing into `data/minecraft/` is the wrong layer and behaves unpredictably.
- Mod id must match the namespace directory exactly (`[a-z0-9_]`).

## What was NOT verified (as of Horizon 0.2.97)

- New block/item declarations (with models + blockstates) — did **not** render in-game; see pitfalls.md.
- Item `components` — unsupported; stripped from the working mod.
- Advancement/loot-table JSONs — formats expected to mirror vanilla but not proven in-game yet.

Treat anything in this section as untested: add one file, build, install, test in-game, and only then add the next.
