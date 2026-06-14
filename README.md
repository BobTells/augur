> [!WARNING]
> **Early prototype.** A personal experiment, shared rough — expect sharp edges and breaking changes.

# augur

A Claude Code skill that tunes the LLM to a user's personality. You hand it evidence — a Big Five report, DISC results, a "user manual for me," a bio — or it interviews you as fallback. It reads the input critically, weights it by credibility, and emits two artifacts:

1. **Behavior preamble** — terse rules that drop into your existing Claude setup (global `CLAUDE.md`, project memory, wherever your durable instructions live). This is what tunes Claude in future sessions.
2. **Personality artifact** — a human-readable document that shows the work: what you claimed, what was observed, why the preamble says what it says.

Both artifacts together are the contract. You see the rules and the evidence. Claude reads the rules and honors them.

## Thesis

You are not having a conversation with your AI, you're working a math problem in English. The personality vector is one of the inputs. Augur makes that input explicit instead of hoping the model guesses.

Long-form write-up: [Human API on rebootpilot.com](https://www.rebootpilot.com/items/augur.html).

## Install

```sh
# clone the skill into your Claude Code skills folder
git clone https://github.com/bobtells/augur ~/.claude/skills/augur
```

Then in any Claude Code session, invoke `/augur`, or paste personality assessment results (Big Five, DISC, 360, etc.), or ask Claude to "tune itself to me." The skill auto-triggers on those patterns.

## What it actually does

- Asks you up front *how* you want this done — one-shot dump, slow walk-through, interview-heavy, whatever.
- Reads whatever evidence you give it (assessment report, doc, paste). Self-report is treated as suspect; behavioral signal in your existing setup is cross-referenced.
- Marks every trait as `claimed`, `observed`, or `both` so the seams stay visible.
- Emits a preamble + an artifact, and asks you to sign off before writing either.
- Adapts to your existing setup. Thin `CLAUDE.md`? Fine. Big one? Fine. Project memory? Skill folder? It works around what you already have.

## Hard rules baked into the skill

- No flattery. Will not tell you your input is smart, insightful, or thoughtful.
- Confirms before writing anything to disk.
- Surfaces blind spots, not just strengths — a low score is a tell, not a compliment.
- One question per turn during decision points.

Full ruleset in [SKILL.md](SKILL.md).

## Companion skills

- [grill-me](https://github.com/mattpocock/skills) — by Matt Pocock. The skill that pulls you into clear thinking when your own idea is still fuzzy. Use before augur if you want to sharpen your own input first.

## Status

Working, and in active refinement. The thesis — personality is a real input to the math, and a written contract is one way to set it — is the part I'm sure about. Whether *this* shape of contract is the right shape is open. PRs and pushback both welcome.

## License

MIT
