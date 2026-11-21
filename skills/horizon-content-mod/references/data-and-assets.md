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

## Latest content surfaces (upstream, 2026-11)

The notes above describe Horizon 0.2.97. Upstream `origin/main` (`docs/mods.md`) now documents
working data-driven content far beyond recipes. An AI following this skill should read the live doc
before assuming a surface is unavailable:

```sh
git -C repositories/horizon show origin/main:docs/mods.md
```

Surfaces now documented (all data-only, in `data/<id>/horizon/...`):
- **Blocks** (`block/*.json`): template + properties, state properties, `random_ticks`, `state_light`,
  `render_shape`, `occludes`, `shapes`/`collision_shapes`, `connects_to` pipes, `thickness`, sign blocks
  (`wood_type`, `wall_block`), `place_block` on items.
- **Items** (`item/*.json`): components patch (`minecraft:<type>` sets, `!minecraft:<type>` removes),
  `crafting_remainder`, `place_block`.
- **Creative tabs** (`creative_tab/*.json`), **mob effects** (`mob_effect/*.json` + 18x18 icon),
  **fluids** (`fluid/*.json`, water/lava templates), **entity types** (`entity_type/*.json`: thrown via
  snowball template, living via zombie/villager/wandering_trader with `attributes`), **menu types**
  (`menu/*.json` with optional crafting grids and linked slots), **damage types**.
- **Tags** merge like data packs; `horizon:add_to` for structure sets/pools/processor lists;
  `horizon:enhanced_terrain_adaptation`.
- **Hot reload**: asset/data-only mods reload in-game like `/reload`; block/item/entity mods still need
  a restart.

Keep the "one change at a time, test in-game" discipline — silent rejection still happens.
