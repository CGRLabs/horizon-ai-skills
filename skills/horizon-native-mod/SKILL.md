---
name: horizon-native-mod
description: Build Horizon code mods (.hmod) — Rust against the mod_api crate, shipped as WebAssembly (sandboxed) or native source bundles (unsandboxed, launcher-compiled). Use when the user wants code-level behavior in the Horizon game (camera, input, rendering, physics, gameplay hooks), mentions mod_api, Hooks, Registrar, parts, mixins, surface.json, wasm, native-build, or porting a Java mod's mechanics to Horizon. Also use when debugging a code mod that fails to build, fit, or run. Content mods (horizon-content-mod skill) cannot run code.
---

# Horizon Code Mods (.hmod)

As of the 2026-11-18 cutover, code mods ship as `.hmod`, never `.hnmod`. Two kinds, **same source**, one or the other:

| | WebAssembly mod | Native mod |
|---|---|---|
| **Inside the .hmod** | `mod.wasm` + manifest | Your Rust source + `Cargo.toml` |
| **Sandboxed?** | Yes (time + memory budgets) | No — runs with the player's rights |
| **Who compiles** | You, once | Every player, on their own machine |
| **Player install cost** | Drop a file | 20-30 min first build + trust prompt |
| **Reach** | All `mod_api` items tagged `wasm: true` | Everything wasm can do **plus** mixins into engine-private code, entity parts, raw mouse deltas, camera roll beyond the shake limit |

**Prefer WebAssembly.** Reach for native only when the hook you need is not on wasm (mixins, `attach_entity_part`/`register_goals`/`register_behaviours`, player file save/load, chunk generators/biome sources) or the target is an engine-private function.

## Structural facts

1. **One `.hmod` file format** for all three kinds (content, wasm, native). What differs is what is inside the zip.
2. **Pin the SDK.** Code mods pin an exact Horizon engine commit in `[package.metadata.horizon] sdk = ...` (and the manifest). `cargo horizon-mod update` moves the pin. `cargo horizon-mod fits` checks your code against the pinned SDK's `surface.json`.
3. **Windows is first-class.** The SDK publishes `x86_64-pc-windows-gnu` slices; Rust 1.99.1. **No WSL, no Visual Studio.**
4. **No `hooks: N`.** The global `NATIVE_HOOKS_API` counter is gone; hooks are per-declaration in `crates/horizon-mod-vocab/src/hooks/`. A manifest with a `hooks` key is rejected.

## Scaffolding

```sh
# WebAssembly (default choice)
cargo horizon-mod new my_mod --template wasm
cd my_mod
# edit src/lib.rs: implement the mod_api::Hooks methods you need
cargo horizon-mod build --wasm
cargo horizon-mod check target/horizon-mod/my_mod.hmod

# Native (only when wasm cannot reach it)
cargo horizon-mod new my_mod --template native
horizon-launcher native-build . --instance "My Instance" --trust-native-code
```

Install for testing (wasm): copy the `.hmod` into the game install's `horizon-mods/` and add `mod.<id>:true` to `horizon-mods.txt`. No launcher, no trust prompt.

## Finding what the API gives you

```sh
cargo horizon-mod hooks "camera roll"   # search the surface by phrase
cargo horizon-mod fits                  # does my code fit the pinned SDK?
cargo horizon-mod sdk diff              # what moved since my last pin
```

`surface.json` (in the `sdk-<commit>` release) is the versioned contract: 749 hooks, every item tagged with its `wasm` availability. The genuinely wasm-only gaps are the items marked `/// Not on wasm:` in source (18 of them) — compile errors, not runtime surprises.

## Mixins (native only)

Mixins still exist for native mods only: `[[native.mixin]]`, `[[native.module]]`, `[[native.loop]]`, `[[native.bridge]]`, top-level `[[inject]]`, and `internals`. They are the only way to touch an engine-private function (e.g. `game::interface::fov::Game::spinning_matrix` for a camera roll). A mixin bundle pins the exact Horizon commit it builds against.

## Non-negotiables

- Never guess the API surface — check `surface.json` or `cargo horizon-mod hooks`.
- Keep the SDK pin exact; run `cargo horizon-mod fits` before publishing.
- Native mods are unsandboxed: the launcher shows a red warning before every build, and your README must too.
- Ship no Mojang assets or code.
- Prefer wasm; reach for native only when the surface cannot reach what you need or the target is engine-private.

## Where the knowledge lives

- `references/mixin-bundle-format.md` — `horizon-mixin.toml` fields, injection kinds (`HEAD`, `RETURN`, `OVERWRITE`, etc.), binding syntax, `$value`, the `[[module]]` runtime crate. Read before writing any injection.
- `references/pipeline.md` — pin strategy, staging internals, deterministic packaging, launcher "Add from GitHub" flow. Read before packaging or choosing pins.
- `references/example-gyro.md` — annotated walkthrough of a complete shipped native mod (camera roll + input capture). Read before designing anything nontrivial.

> **Note:** The references above describe the pre-cutover `.hnmod` bundle pipeline in places. Treat them as **historical reference for mixin mechanics**, not as current instructions. The current truth is `docs/modding/src/` in the engine at the pinned rev, the registry README, and `MOD-ONBOARDING.md` §4. Prefer those.