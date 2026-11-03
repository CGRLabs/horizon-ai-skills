# Horizon AI Skills

Agent Skills that teach AI assistants (ZCode, Claude Code, Cursor, or any agent that reads SKILL.md files) how to build mods for **Horizon** — the free, open-source Rust rewrite of Minecraft: Java Edition 26.3.

People increasingly build mods *with* an AI. But an AI that doesn't know Horizon's exact manifest rules, verified JSON formats, and silent failure modes will confidently produce mods that never appear in-game. These skills encode what we learned by **shipping real mods** — a data-driven content mod and a camera/input native mod — including every pitfall that cost us debugging hours.

## The skills

| Skill | What it covers |
|---|---|
| [`horizon-content-mod`](skills/horizon-content-mod/SKILL.md) | Data-driven `.hmod` mods: `horizon-mod.toml` manifest rules, verified recipe/lang JSON formats, the build → check → install → enable loop, and the silent-failure catalog (new blocks not rendering, unsupported components, id rules). |
| [`horizon-native-mod`](skills/horizon-native-mod/SKILL.md) | Rust mixin bundles (`.hnmod`): `horizon-mixin.toml` injection format (`HEAD`/`RETURN`/`OVERWRITE`), pin strategy (`horizon_revision` vs `hooks` API level), the WSL staging pipeline, deterministic packaging, and a walkthrough of a shipped 6-hook camera mod. |

## Install

Copy the skill folders into your agent's skill directory:

**ZCode / generic agents**
```sh
git clone https://github.com/CGRLabs/horizon-ai-skills.git
cp -r horizon-ai-skills/skills/horizon-content-mod ~/.agents/skills/
cp -r horizon-ai-skills/skills/horizon-native-mod  ~/.agents/skills/
```
(For project-only use, put them in `<project>/.agents/skills/` instead.)

**Claude Code**
```sh
cp -r horizon-ai-skills/skills/horizon-content-mod ~/.claude/skills/
cp -r horizon-ai-skills/skills/horizon-native-mod  ~/.claude/skills/
```

Or just point your agent at this repository and ask it to read `skills/<name>/SKILL.md` before working on a Horizon mod.

## What "verified" means here

Everything marked **verified** was tested against a real installed game:

- Content-mod manifest + smelting/blasting recipes + lang files — installed, enabled, and confirmed working in Horizon 0.2.97 (2026-11).
- Native-mod pipeline — staged, compiled, tested, packaged, and shipped end-to-end against Horizon commit `ba6344a5` (Nightly 2026-10-30), using only documented hook points so it survives newer Nightlies.

Everything marked **unverified** is flagged as such in the reference files. We would rather tell you what we proved than pad the docs.

## Contributing

The value is *operational knowledge from shipped mods*, not speculation. PRs should add formats you ran in a real game, pitfalls you actually hit, or version bumps with re-test notes. See each skill's `references/pitfalls.md` for the format.

## Related

- [`CGRLabs/horizon-inspector`](https://github.com/CGRLabs/horizon-inspector) — offline security inspection for `.hmod`/`.hnmod`/`.hpack`/`.hshader`
- [`CGRLabs/horizon-mod-toolkit`](https://github.com/CGRLabs/horizon-mod-toolkit) — scaffold/build/check CLI
- [`CGRLabs/gyro`](https://github.com/CGRLabs/gyro) — the reference native mod
- [Horizon](https://github.com/horizon-rust/horizon) — the game

## License

MIT — same as Horizon. Contains none of Mojang's code, assets, or data.
