# Validation, Inspection, and Publishing

## Local validation loop

```sh
cargo horizon-mod build                        # zip the mod -> target/horizon-mod/<id>.hmod
cargo horizon-mod check target/horizon-mod/<id>.hmod   # registry-rule validation
horizon-inspect target/horizon-mod/<id>.hmod --json    # security + manifest report (if installed)
```

- Exit codes: `0` valid, `1` validation failure, `2` tool error — safe to gate scripts/CI on them.
- `horizon-inspector` (https://github.com/CGRLabs/horizon-inspector — clone and `cargo build --release`; not installed by default) checks archive safety (Zip-Slip, zip bombs, duplicate entries), manifest validity, checksums, and optional Minisign signatures, and classifies security tier. It never executes mod code. Its absence does not block building or installing — it is an extra validation gate.
- Warnings worth fixing before publishing: missing description, missing authors, experimental capabilities.

## Registry model (how publishing works)

- The official registry is the `horizon-mods` GitHub repository: `index.json` lists packages with TOML metadata and SHA-256 checksums; `revoked.toml` lists revocations. **It is the source of truth** — aggregator sites (including CGRLabs Horizon Hub) mirror it rather than replace it.
- Submission flows through the registry CLI (`cargo horizon-mod registry`) and is governed by that repo's `MOD-POLICY.md` — read it before submitting.
- Keep releases reproducible: same sources → same bytes → same SHA-256.

## Deterministic packaging tips

- Build from a clean tree; avoid timestamps/absolute paths in file content.
- Pin your tool versions in the README so others reproduce your checksums.
- State the exact Horizon version(s) tested — content mods ride API lines (0.2), but data-format quirks are version-specific (see pitfalls.md).

## Version pinning of these claims

Everything "verified" in this skill was tested on **Horizon 0.2.97 (2026-11)**. Note the user's game version when working; if it is newer, re-verify the formats against the current Horizon source before trusting the details — data-format quirks are version-specific.

## In-game verification handoff

If you cannot launch the game yourself, **do not declare success** — packaging checks prove structure, not behavior. Hand the user the exact smoke-test steps (install → enable dance → trigger each recipe in a furnace/blast furnace → confirm outputs) and ask them to report what they saw. The skill's rule — assume nothing worked until seen in-game — applies to you too.

## Distribution checklist

1. `cargo horizon-mod check` passes with zero errors.
2. `horizon-inspect --json` shows zero errors (warnings addressed or explained) — if the tool is available.
3. Tested in-game on a stated Horizon version — the enable dance completed and the content visibly works (by you or confirmed by the user).
4. README: what it adds, the id, tested-on version, uninstall (delete the `.hmod` from `horizon-mods/`).
5. No Mojang assets bundled; vanilla ids referenced rather than copied.
