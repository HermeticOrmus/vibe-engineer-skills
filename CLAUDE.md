# CLAUDE.md

Behavioral guidelines for the **Vibe Engineer** — the human directing AI codegen. Merge with project-specific instructions as needed.

**The companion to [andrej-karpathy-skills](https://github.com/HermeticOrmus/andrej-karpathy-skills)**: Karpathy's principles govern how Claude should behave; these govern how you should behave when directing Claude.

**Tradeoff**: These guidelines bias toward verification over velocity. For trivial tasks, use judgment.

## 1. Hypothesis before help

Form a hypothesis about the bug's shape BEFORE asking the AI to fix it.

- Trace data flow yourself first.
- Read the error, then the line, then the call site.
- State your hypothesis even if uncertain: "I think `X` retrieved on line N is never used downstream — root cause?"
- A sharp hypothesis produces a sharp diagnosis. A vague request produces vague code.

The cost of skipping this step compounds: the AI's diff matches your prompt's resolution; vague prompts get plausible-looking patches that hide the real bug.

## 2. Scoped prompts

File paths. Line numbers. Variable names. Specific symptoms. Never "fix this."

Bad: "the auth thing isn't working"
Good: "`AuthSession.refresh` at `auth/session.ts:142` returns `null` when the refresh token is valid but the access token is expired — expected behavior or bug?"

Scope reduces hallucination surface. The AI can only reason about what you put in context. Vague prompts make the AI guess at scope; guesses are often wrong.

## 3. Validate before accepting

For every AI-proposed diff, ask three questions:

1. **Does it over-engineer?** New abstraction, new file, new dependency that the task didn't require?
2. **Does it miss an edge case?** Null input, empty list, race condition, error path?
3. **Does it match the root cause I identified?** Or does it patch the symptom?

If yes to (1) or (2), or no to (3): reject and iterate, don't accept.

The AI doesn't know your codebase's invariants. You do. Validation isn't second-guessing — it's the job.

## 4. Reject working-but-wrong

Solutions that pass tests by obscuring the bug are worse than no solution.

Symptoms of working-but-wrong:

- The fix wraps the real bug in a try/catch that swallows the failure mode
- The fix adds a defensive check that masks the upstream contract violation
- The fix renames the symptom variable instead of fixing the root cause
- The fix mocks the broken dependency instead of fixing it
- The fix asserts the bug away (`assert x is not None` where `x` should never be None)

A solution that hides the bug propagates it. Reject and iterate even when tests pass.

## 5. AI output is a draft, not a verdict

Every AI diff is a hypothesis about what you want. Treat it as a draft:

- Read the diff before staging.
- Run it. Then read what it did, not just whether it succeeded.
- Compare against your hypothesis from step 1. Mismatch is data.
- The AI is fast and confident. You are slow and skeptical. Both are needed.

Precision. Skepticism. Verification. The model is your draftsperson; you are the engineer.

---

## Trigger this teaching when

- A request omits the hypothesis → prompt for one before drafting
- A diff is accepted without being read → flag it
- A prompt is vague → suggest a sharper rewrite
- The proposed fix misses the root cause → call it out and self-correct
- A bug gets solved → name the *category* and the catch-it-faster-next-time lesson

---

**Source**: This file packages the Vibe Engineer discipline from a wider system documented at [`hitchhikers-guide-to-vibe-engineering`](https://github.com/HermeticOrmus/hitchhikers-guide-to-vibe-engineering) and informed by [Karpathy's December 2025 observation](https://x.com/karpathy/status/2015883857489522876) that programming is being dramatically refactored. Google now reports ~3 of 4 lines of new code are AI-generated; the Vibe Engineer discipline is the new core competency.

**License**: MIT — use it, fork it, merge it into your own.
