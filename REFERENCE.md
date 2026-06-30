# Augur — reference

Disclosed reference for [`SKILL.md`](SKILL.md). Loaded only when a step points here.

## Interview mechanics (Step 6)

**Default mechanic: run the [[grill-me]] skill.** It's built for exactly this — relentless one-question-at-a-time grilling that resolves the decision tree, using the same `AskUserQuestion` UI pattern augur uses everywhere else. It respects user-set pacing.

Confirm with `AskUserQuestion` before launching grill-me. It's intense by design:

```
question: "Ready to start the interview?"
header:   "Interview"
options:
  - "Yes — run grill-me, 5 questions (Recommended)"
      desc: "Five targeted AskUserQuestion grills. Each has 2–4 sharp options with a recommended pick."
  - "Yes — run grill-me, but longer (10+ questions)"
      desc: "Deeper coverage. Use if you want a more thorough read before I write."
  - "Lighter — 2–3 questions, no full grill"
      desc: "Just the highest-leverage gaps from what I already know."
  - "Skip — go straight to synthesis with what we have"
      desc: "Only viable if a credible report was already provided."
```

If grill-me isn't installed in the user's setup, fall back to running the same pattern manually: `AskUserQuestion` per turn, 2–4 options each, recommended-first.

## Worked example — `personality.md` (Step 5)

The whole file, ~20 lines:

```markdown
# Bob — how to work with me

Blunt, fast, ship-biased. Lead with the answer; earn every word after it.

## How to work with me
- Be blunt. No "great question," no softening preamble.
- Recommend one thing, not a menu of three.
- Idea-dense over hand-holding. Don't dumb it down.
- No flowery prose. Function over form.
- Tolerate disorder. Don't demand structure that isn't there.
- Drive to completion. Bias to shipping.
- Don't add caveats I wouldn't add myself.

## Watch for me
- Visual / UI decisions — push back, suggest alternatives I won't reach for.
- External-facing copy — flag when a softer register would land better with the audience.
- Long-running ambiguity — surface structure if it's starting to cost; don't demand it.
```
