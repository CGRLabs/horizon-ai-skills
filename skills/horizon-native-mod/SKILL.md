---
name: horizon-native-mod
description: Build Horizon native mods (.hnmod) — Rust mixin bundles compiled into a custom Horizon client binary. Use when the user wants code-level behavior in the Horizon game (camera, input, rendering, physics, gameplay hooks), mentions horizon-mixins, mixin injection, native hooks, NATIVE_HOOKS_API, .hnmod, native-mod staging, or porting a Java mod's mechanics to Horizon. Also use when debugging a native mod that fails to stage, build, or pin correctly. Even if the user just says "Horizon mod that changes how X works", this skill applies — content mods (horizon-content-mod skill) cannot run code.
---

# Horizon Native Mods (.hnmod)

Native mods are **Rust mixin bundles compiled into a patched Horizon client** by the staging tool. They are unsandboxed code running with full user privileges — say so plainly in any UI copy, and treat the bundle as powerful: it can do anything the game can.

Two structural facts drive everything:

1. **Pinning.** A bundle names the exact Horizon commit it was built against (`horizon_revision`, 40-hex) plus the `horizon-mixins` commit. Alternatively it declares `hooks: N` (a `NATIVE_HOOKS_API` level) and then the launcher builds it on **any** Horizon whose hook API is at least N — hook points are only ever added, never removed, so this is what survives Nightlies. Prefer hook points.
2. **Platform.** Staging requires **Linux** (WSL2 works), Rust 1.95.0, Python 3.12+, Git. Runtime unit tests of the bundle's library code run anywhere, including Windows.

## Bundle anatomy

```
<repo>/
  compatibility.json            # pins + hooks level (see references/pipeline.md)
  mixins/<id>/
    horizon-mixin.toml          # mod block + injections (see references/mixin-bundle-format.md)
    runtime.rs                  # #[ used as a library crate injected into the client
    <x>_helper.rs               # one file per injection point (helpers)
  tools/package.py              # builds the deterministic .hnmod
  tools/horizon.py              # stages + builds the patched client (borrow from gyro)
```

One bundle per `.hnmod`; the folder under `mixins/` must be named exactly the mod id; only `.rs` and `.toml` files ship.

## The pipeline (commands verified end-to-end)

```sh
cargo test                                        # flight-maths-style unit tests, run anywhere
python3 tools/horizon.py stage                    # export pinned Horizon source, plan/apply mixins (~3 min)
cargo check  --locked --offline                   # in the staged tree
cargo test   --locked --offline
cargo build  --profile fast --locked --offline    # patched client (~6 min first time)
python3 tools/package.py --check-hooks            # target/<id>-<version>.hnmod + verifies hooks claim
```

`--check-hooks` fetches `native_hooks.rs` at the pinned commit and asserts the declared `hooks` level really exists — never let the number be a guess.

## Where the knowledge lives

- `references/mixin-bundle-format.md` — `horizon-mixin.toml` fields, the three injection kinds (`HEAD`, `RETURN`, `OVERWRITE`), binding syntax, `$value`, the `[[module]]` runtime crate. Read before writing any injection.
- `references/pipeline.md` — `compatibility.json` field-by-field, pin strategy (commit vs hooks), staging internals, deterministic packaging rules, launcher "Add from GitHub" flow. Read before packaging or choosing pins.
- `references/example-gyro.md` — annotated walkthrough of a complete, working, shipped mod (camera roll + input capture across 6 hook points). Read it before designing anything nontrivial; copy its structure.

## Non-negotiables

- Never guess the `hooks` number — verify against the pinned commit (`--check-hooks` does this).
- Keep helper files tiny and single-purpose; put logic in `runtime.rs` where it can be unit-tested.
- Native mods are not reviewed by default: the registry/launcher shows an unsandboxed-code warning, and your README must too.
- Ship no Mojang assets or code; MIT-style licensing consistent with Horizon's.
- The Options → Mods row prints `VERSION` from `runtime.rs`; keep it equal to the manifest version or the check fails.
