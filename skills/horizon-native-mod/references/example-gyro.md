# Example: Gyro — a complete, shipped native mod

Repo: https://github.com/CGRLabs/gyro (MIT). Gyro makes elytra flight fly like an aircraft: the mouse rolls/pitches you, `A`/`D` are rudder, and the gliding avatar stays locked to the camera. Use it as the structural template for any native mod.

## What it touches and why

| Hook (target) | Kind | Helper | Purpose |
|---|---|---|---|
| `native_hooks::Game::native_mods` | `HEAD` | `list_helper.rs` | The mod's row in Options → Mods |
| `native_hooks::Game::native_key` | `HEAD` | `key_helper.rs` | Capture rudder key state |
| `native_hooks::Game::native_frame` | `HEAD` | `frame_helper.rs` | Per-frame tick: level-out, rudder yaw |
| `game::fov::Game::spinning_matrix` | `RETURN` | `spin_helper.rs` | Camera roll: rotate the render matrix; capture view state |
| `game::Game::apply_mouse_look` | `OVERWRITE` | `turn_helper.rs` | Reinterpret mouse deltas as roll/pitch while gliding |
| `mobs::render::base_transform` | `RETURN` | `model_roll_helper.rs` | Pin the player's own avatar to the camera roll |

Design lessons baked into this table:

- **Observe via `HEAD`, transform via `RETURN`, replace via `OVERWRITE` only when nothing else works.** Gyro overwrites exactly one function (mouse look) because that behavior *is* a replacement; everything else rides observation/return hooks and stays Nightly-proof via `hooks: 3`.
- **State lives in `runtime.rs`** (roll physics + atomics), injected as a `pub` module `horizon_client::gyro`, and is unit-tested (`cargo test`) without a game. Helpers are dumb adapters: read bindings, call the module, return.
- **Client-side only:** the roll never hits the network; movement side effects behave like any mouse-sensitivity mod. Know which of your effects the server can see.

## The math sketch (for camera-style mods)

Camera roll rides the projection: `clip = projection · (bob · spin) · view · model`, where `spin` gets a Z-rotation. Because the roll is in the projection, *everything* tilts — so the avatar is un-tilted by left-multiplying `base_transform` with `view⁻¹ · Rz(-roll) · view` (a rotation about the render camera's look axis). The yaw/pitch/camera-mob state it needs is captured in the spin hook, which runs right after `camera_view`, so spectator/third-person cameras stay correct.

You do not need this exact math — you need the *shape*: find the render/input seam, hook its smallest stable point, keep a runtime module of pure functions, and test the math standalone before fighting the live game.

## Its proving loop (copy this)

1. `cargo test` — flight maths as pure functions.
2. Stage + build the patched client (see pipeline.md).
3. `python3 proof.py --resources <prepared .local> --game <patched client>` — drives the real game on throwaway data dirs, asserts a horizontal mouse move rolls the frame *without* turning the player, that rolling back restores the same picture, and that ground mouse-look recovers. Screenshots to `target/proof/`.
4. `python3 tools/package.py --check-hooks` — deterministic `.hnmod` with a verified hooks claim.

Only after all four did it ship. Steal all four steps.
