# Vibe Engineer Skills

> A single `CLAUDE.md` for the human directing AI codegen. Companion to [andrej-karpathy-skills](https://github.com/HermeticOrmus/andrej-karpathy-skills).

The Karpathy guidelines tell Claude how to behave. **These tell you how to behave when working with Claude.** Both are needed.

## The problem

Google now reports ~3 of 4 lines of new code are AI-generated. Their new [Code Comprehension Interview](https://velocode.ai/blog/google-code-comprehension-interview) tests exactly the bottleneck: not whether you can write code, but whether you can **catch when AI writes the wrong thing**.

The discipline of directing AI codegen is now the core competency. It is its own skill set with its own failure modes:

- Vague prompts that the AI completes with plausible-looking but wrong code
- Accepting diffs without reading them
- Letting the AI pick an interpretation silently
- Approving "working-but-wrong" solutions that hide the real bug
- Treating AI output as a verdict instead of a draft

## The five principles

| Principle | Addresses |
|---|---|
| **Hypothesis before help** | Vague prompts produce vague diagnoses |
| **Scoped prompts** | Hallucination surface scales with unspecified context |
| **Validate before accepting** | The AI doesn't know your invariants; you do |
| **Reject working-but-wrong** | Symptom-fixes propagate bugs |
| **AI output is a draft, not a verdict** | Confidence ≠ correctness |

Full content: [`CLAUDE.md`](CLAUDE.md). Worked examples: [`EXAMPLES.md`](EXAMPLES.md).

## Install

### As a project CLAUDE.md

Drop [`CLAUDE.md`](CLAUDE.md) at the root of your repository. Claude Code picks it up automatically. Merge with existing project instructions if any.

```bash
curl -o CLAUDE.md https://raw.githubusercontent.com/HermeticOrmus/vibe-engineer-skills/main/CLAUDE.md
```

### As a Claude Code skill

The same content is packaged as a skill under [`skills/vibe-engineer-guidelines/`](skills/vibe-engineer-guidelines/) for `~/.claude/skills/`. See the `SKILL.md` inside for installation.

### In Cursor

See [`CURSOR.md`](CURSOR.md) for the Cursor-rule equivalent at [`.cursor/rules/vibe-engineer-guidelines.mdc`](.cursor/rules/vibe-engineer-guidelines.mdc).

### In other AI coding tools

If your tool reads a single instruction file at the project root, copy `CLAUDE.md` to whatever name your tool expects (`AGENTS.md`, `INSTRUCTIONS.md`, etc.).

## Why this exists

In 2025, Karpathy observed:

> *"I've never felt this much behind as a programmer. The profession is being dramatically refactored."*

The refactor has two sides. Karpathy's principles cover Claude's side — be cautious, be simple, be surgical, be goal-driven. This repo covers the **human's side**: be specific, be skeptical, be hypothesis-driven, reject working-but-wrong.

A Vibe Engineer is the human who:

- Directs AI codegen with scoped, specific prompts
- Validates output against domain invariants the AI cannot see
- Catches the difference between a working diff and a correct diff
- Treats AI confidence as a draft signal, not a verdict

For the long-form treatment of Vibe Engineering as a discipline, see the companion repo: [`hitchhikers-guide-to-vibe-engineering`](https://github.com/HermeticOrmus/hitchhikers-guide-to-vibe-engineering).

## Five principles in detail

### 1. Hypothesis before help

State your hypothesis about the bug's shape BEFORE asking the AI to fix it. The cost of skipping this step compounds — vague prompts get plausible-looking patches that hide the real bug.

### 2. Scoped prompts

File paths, line numbers, variable names, specific symptoms. Scope reduces hallucination surface. The AI can only reason about what you put in context.

### 3. Validate before accepting

Three questions per AI-proposed diff: does it over-engineer? does it miss an edge case? does it match the root cause? The AI doesn't know your codebase's invariants; validation is the job.

### 4. Reject working-but-wrong

Solutions that hide the bug propagate it. Wrapped try/catch, defensive masking checks, renamed symptom variables, mocked broken dependencies — reject and iterate even when tests pass.

### 5. AI output is a draft, not a verdict

The AI is fast and confident. You are slow and skeptical. Both are needed. Read the diff before staging. Compare against your hypothesis. Mismatch is data.

## See also

- [`andrej-karpathy-skills`](https://github.com/HermeticOrmus/andrej-karpathy-skills) — the companion: how Claude should behave (Karpathy's four principles)
- [`hitchhikers-guide-to-vibe-engineering`](https://github.com/HermeticOrmus/hitchhikers-guide-to-vibe-engineering) — the long-form book: Vibe Engineering as a discipline, styled after Douglas Adams
- [Karpathy's original X post](https://x.com/karpathy/status/2015883857489522876) — the prompt for this work
- [Google's Code Comprehension Interview](https://velocode.ai/blog/google-code-comprehension-interview) — empirical signal that "catch AI mistakes" is now the bottleneck

## Contributing

PRs welcome — especially for additional worked examples in [`EXAMPLES.md`](EXAMPLES.md), translations of the README, and adaptations of `CURSOR.md` for other AI coding tools (Windsurf, Cline, Aider, Continue, etc.).

## License

MIT — use it, fork it, merge it into your own CLAUDE.md.
