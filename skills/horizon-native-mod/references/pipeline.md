# Pipeline — Pins, Staging, Packaging, Distribution

## compatibility.json — the pin file

```json
{
  "game": "26.3",
  "hooks": 3,
  "horizon_revision": "ba6344a59cb9e755b730dcf02285fc0d33b97d3c",
  "source_sha256": {
    "crates/horizon-client/src/game.rs": "043c78..."
  },
  "horizon_repository": "https://github.com/horizon-rust/horizon",
  "horizon_mixins_repository": "https://github.com/horizon-rust/horizon-mixins",
  "horizon_mixins_revision": "d9c83d7096b7ea5b83678da2efdfd6161f74997e",
  "mixin_schema": 6
}
```

| Field | Meaning |
|---|---|
| `game` | Target game version ("26.3"). |
| `hooks` | The `NATIVE_HOOKS_API` level your bundle needs. **The survival strategy**: the launcher then builds on any Horizon with `NATIVE_HOOKS_API >= hooks` instead of only the pinned commit. Hook points are only ever added. Verify with `--check-hooks` (below), never guess. |
| `horizon_revision` | 40-hex commit the mod was tested on. |
| `source_sha256` | Checksums of staged files you rely on (optional but catches drift). |
| `horizon_mixins_revision` | 40-hex commit of the mixin engine. |
| `mixin_schema` | Mixin plan schema version (6 in current tooling). |

`horizon-mixin.toml`'s `[compatibility].horizon_revision` must equal `horizon_revision` here — the packager enforces it.

## Environment

- Linux or WSL2 (staging exports and builds there). Rust 1.95.0+, Python 3.12+, Git.
- Unit tests of `runtime.rs` logic run on any OS — keep logic there and helpers thin, so most development needs no WSL round-trip.
- Windows staging notes (from gyro): `tools/horizon.py` runs with a no-op lock (no `fcntl`) and exports with `core.autocrlf=false` so source hashes match the Linux pin; don't let the extensionless Linux `horizon-mixins` binary shadow the `.exe` build.

## The commands

```sh
cargo test                                   # your logic, standalone — run this constantly
python3 tools/horizon.py stage               # export pinned source + plan/apply mixins (~3 min)
cargo check  --locked --offline              # inside the staged tree
cargo test   --locked --offline
cargo build --profile fast --locked --offline   # patched client (~6 min first build, 66 MB ELF)
python3 tools/package.py --check-hooks       # target/<id>-<version>.hnmod
```

`--check-hooks` fetches `crates/horizon-client/src/game/native_hooks.rs` at the pinned commit and asserts `NATIVE_HOOKS_API` is at least your declared `hooks` — the number in `compatibility.json` must never be a guess.

Borrow `tools/horizon.py` and `tools/package.py` from the Gyro repo rather than rewriting them; they encode several hard-won fixes (CRLF, locking, shadowing).

## Deterministic packaging rules (enforced by gyro's package.py)

- Exactly one `mixins/<id>/horizon-mixin.toml` per `.hnmod`; folder name = mod id; id is `[a-z0-9_]` lowercase.
- Archive contains `horizon-native.json` (generated: `schema=1, id, version, game, horizon, mixins, hooks`) + `mixins/<id>/**` (.rs/.toml only).
- Fixed zip entry dates (2026-01-01) and modes (0644), sorted entries → same sources produce identical bytes and SHA-256. Keep it that way; the registry keys off checksums.
- The Options → Mods row must show the bundle's version (VERSION string check in `runtime.rs`), and `list_helper.rs` must take its text from `runtime.rs` (`DESCRIPTION`), not duplicate it.

## Distribution

- **Launcher (preferred):** Mods → Add from GitHub → `<repo URL>`, then Settings → Game → Native mods (experimental) → Build. The launcher downloads its own Rust and compiles against the user's install; first build 20–30 min, later incremental. Users must be shown the unsandboxed-code warning.
- **Release binary:** attach a patched `horizon-client-<modid>.exe` to a GitHub release for launcher-less users; state the exact commit it was built for — it will not match newer installs.
- Registry submission is governed by the `horizon-mods` repo's MOD-POLICY.md; native mods get extra scrutiny and an explicit red warning by design.

## Testing beyond unit tests

Gyro ships `proof.py`: starts the patched game on throwaway data folders (`HORIZON_HOME` etc. pointed at scratch dirs), performs the modded behavior via `horizon-ctl`, and asserts on results — screenshots land in `target/proof/`. Copy that pattern: a native mod that "compiles" has proven nothing; prove the behavior.
