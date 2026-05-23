# Using this repo with Cursor

This project includes a **Cursor project rule** so the Vibe Engineer guidelines apply automatically when you work here.

## In this repository

1. Open the folder in Cursor.
2. The rule [`.cursor/rules/vibe-engineer-guidelines.mdc`](.cursor/rules/vibe-engineer-guidelines.mdc) is committed with `alwaysApply: true`, so no extra installation is needed.
3. In Cursor, confirm under **Settings → Rules**, where `vibe-engineer-guidelines` should appear.

## Use the same guidelines in another project

**Cursor (recommended)**: Copy `.cursor/rules/vibe-engineer-guidelines.mdc` into that project's `.cursor/rules/` directory (create the folders if needed). Merge with existing rules as you like.

**Other AI coding tools**: If a stack only supports a root instruction file, copy [`CLAUDE.md`](CLAUDE.md) into that project instead (or merge its contents into your existing instructions). Most modern AI coding tools (Claude Code, Continue, Cline, Windsurf, Aider) read a root-level instruction file.

## Optional: personal Agent Skills

If you want the same content as a reusable skill under `~/.cursor/skills`, use [`skills/vibe-engineer-guidelines/SKILL.md`](skills/vibe-engineer-guidelines/SKILL.md). Copy or symlink it into your personal skills directory.
