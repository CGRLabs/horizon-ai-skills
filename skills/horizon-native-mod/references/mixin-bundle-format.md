# Mixin Bundle Format — horizon-mixin.toml and Injections

Reference implementation: the Gyro mod (https://github.com/CGRLabs/gyro), shipped and verified. The mixin engine is `horizon-mixins` (https://github.com/horizon-rust/horizon-mixins) — its `docs/` directory covers scopes, bridges, effects, and SpongePowered feature gaps; consult it for advanced features.

## File layout

```
mixins/<id>/
  horizon-mixin.toml     # the plan: module + injections
  runtime.rs             # your logic, injected as a library crate (unit-testable)
  list_helper.rs ...     # one helper file per injection (thin adapters)
```

Only `.rs` and `.toml` files are packaged. The directory name must equal the mod id.

## The [mod] and [compatibility] blocks

```toml
[mod]
id = "gyro"                # [a-z0-9_], must match the mixins/ folder name
version = "1.4.1"          # must appear as VERSION = "<same>" in runtime.rs
priority = 100

[compatibility]
horizon_revision = "ba6344a59cb9e755b730dcf02285fc0d33b97d3c"   # 40-hex, tested commit
```

- The Options → Mods row shows `VERSION` from `runtime.rs`; the package script enforces that it matches `version` above.
- `runtime.rs` should export `DESCRIPTION` (and helpers take their text from it) rather than repeating strings.

## The [[module]] block — your code crate

```toml
[[module]]
id = "gyro"
file = "crates/horizon-client/src/lib.rs"   # where the module gets injected in the staged tree
name = "gyro"                                # crate name -> horizon_client::gyro
source = "runtime.rs"
visibility = "pub"
```

Put real logic here (state, physics, pure functions) so `cargo test` can exercise it without a running game. Helpers only translate between the hook site and the module.

## [[injection]] — the three kinds

```toml
[[injection]]
id = "spin-roll"                                   # unique name for the injection
kind = "RETURN"                                    # HEAD | RETURN | OVERWRITE
target = "horizon_client::game::fov::Game<'_>::spinning_matrix"
helper = "spin_helper.rs"
bindings = [
  { name = "game",  ty = "&crate::game::Game<'_>", value = "self" },
  { name = "value", ty = "glam::Mat4",             value = "$value" },
]
```

| Kind | Meaning |
|---|---|
| `HEAD` | Your helper code runs **before** the function body (e.g. observe args, record state). Gyro uses it for `native_mods` (mod list row), `native_key` (input), `native_frame` (per-frame tick). |
| `RETURN` | Runs at return; `$value` binds the return value so you can read/transform it. Gyro uses it for `spinning_matrix` (camera roll) and `base_transform` (avatar lock). |
| `OVERWRITE` | Replaces the function body. Gyro uses it for `apply_mouse_look` (reinterpret mouse input). Destructive — prefer `HEAD`/`RETURN` when possible so updates to the vanilla body are not lost. |

Binding rules:
- `name` = the parameter name your helper receives; `ty` = its Rust type (must match the target's signature — read the staged source to confirm).
- `value` = expression at the injection site: plain names reference existing locals/params, `self`/`&mut *self` for the receiver, `$value` for the return value in `RETURN` injections.

## Finding hook points

- The stable surface is documented hook points (`docs/native-mods.md` upstream; `NATIVE_HOOKS_API` level). Write against hook points, not arbitrary private functions, or every Nightly can break you.
- `crates/horizon-client/src/game/native_hooks.rs` defines the game's native-mod entry points (`native_mods`, `native_key`, `native_frame`, ...) — these are the sanctioned seams.
- To explore beyond: read the staged source under the pinned commit, but expect to own the breakage.

## helpers receive bindings as function parameters

```rust
// spin_helper.rs — receives the bindings declared above
pub fn spin_roll(game: &crate::game::Game<'_>, value: glam::Mat4) -> glam::Mat4 {
    // compute, then return the (possibly rotated) matrix
    gyro::roll_matrix(game, value)
}
```
