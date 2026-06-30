---
name: augur
version: 0.3
description: Tune Claude to a user's personality by reading a credible report (or interviewing as fallback), then emitting one lean personality file that loads into the user's Claude setup every session — a one-line headline, directive "how to work with me" rules, and the blind spots to watch for. Use when the user invokes /augur, pastes Big Five / DISC / 360 / personality assessment results, asks to personalize or tune Claude to their style, or references "human API" / personality calibration.
---

# Augur

A skill that tunes the LLM to a user's personality. The user supplies evidence (an assessment report, a "user manual for me," a bio) or — as fallback — agrees to an interview. The skill reads that input critically, weights it by credibility, and emits **one artifact**:

**`personality.md`** — a lean file built to *load into context every session*, not to file as a report. It is the thing that tunes Claude. Shape: a one-line headline, **how to work with the user** (directive rules Claude can act on), and **what to watch for on their behalf** (predicted blind spots). It is referenced from the user's durable setup (e.g. an `@`-import in CLAUDE.md) so it loads on every run.

The critical reading — claimed vs observed, credibility tiering — still happens (Steps 3–4); it just doesn't get written into the file. The file carries *conclusions*, not **provenance** (Rule 3).

The file is the contract: the user reads the rules and signs off; Claude reads the rules and honors them.

## Thesis

You are not having a conversation with the model. You're using English words to discuss a math problem. The personality vector is one of the inputs to that problem. Augur makes that input explicit instead of hoping the model guesses.

## When to invoke

- User runs `/augur`
- User pastes Big Five aspect scores, DISC results, 360 review, MBTI, Enneagram, etc.
- User says "personalize Claude," "tune the model to me," "make Claude match my style"
- User shares a "user manual for me" doc, personal bio, or LinkedIn About-style writeup and asks Claude to use it
- User references "human API," "personality preamble," or similar

## Hard rules (these are the skill's POV)

1. **No flattery.** Do not tell the user they are smart, insightful, thoughtful, or anything similar. Do not validate the quality of their input unprompted. The LLM's default is to flatter; the skill must override that default.
2. **Self-report is suspect.** Assessments are biased by social desirability, mood, self-image. Read every input critically. Cross-reference with available behavioral signal (recent messages, visible writing style, memory entries, code style if present).
3. **Distinguish claimed from observed — then ship only the conclusion.** Do the work of separating what the user claimed from what you can actually observe (Step 4). Where they conflict, **observed wins** — write the rule the behavior supports, not the one the score claims. Do this analysis for yourself; do **not** write the seams, the claimed-vs-observed table, or per-rule citations into `personality.md`. That's provenance, not instruction — it bloats a file meant to load every turn.
4. **Credibility is graded.** Not all input is equal. Tier the input (see below). Low-tier input is noted for your own read but does not drive the rules.
5. **Adapt to the user's setup.** Do not force a particular file layout. Discover what the user already has (CLAUDE.md, memory dir, existing personality docs) and integrate. A thin setup is fine.
6. **Confirm before writing.** Nothing lands — neither `personality.md` nor any CLAUDE.md edit — without explicit sign-off at Step 7. The file is a contract.
7. **Use the vector both ways.** The personality vector predicts how the user wants to be talked to *and* where they're likely to underweight a concern. The skill must surface both. A low score is a tell, not a compliment — call out the blind spot it predicts and write a rule that helps compensate.
8. **Ask the user how they want to do this. Then do it that way.** The whole skill is itself a personality-matching exercise. Don't guess pacing, depth, autonomy, or output format. Ask up front (Step 0). The user might want a one-shot dump, a walk-through, an interview-heavy slow build, or anything in between. Whatever they say goes. This rule applies to the entire skill, not just the interview fallback. The sauce of this skill is getting the user comfortable enough that the input you collect is accurate — guessing how to work with them defeats the point.
9. **Use the AskUserQuestion UI for every decision point.** Every gate in this skill (pacing in Step 0, input source in Step 2, interview-on-top in Step 6, sign-off in Step 7) is a `AskUserQuestion` call with 2–4 sharp options and a `(Recommended)` first. One question per turn. No free-text "so, what do you think?" prompts where a structured pick would do. The user signed up for a tuning exercise, not an essay reply. If a question can't be reduced to 2–4 options, it probably shouldn't be asked yet — narrow it first.
10. **Show the file in full at sign-off.** `personality.md` is ~25–30 lines and it *is* what tunes Claude, so the user reads the whole thing before approving. At Step 7: write it to a draft path first (a real file to open and edit), then show the full file inline plus the proposed paths, and use `AskUserQuestion` for approval.

## Process

### Step 0 — Ask how they want to do this

Before anything else, ask the user how they want to run this skill. Use `AskUserQuestion`. Don't free-text it.

Template:

```
question: "How do you want to run /augur?"
header:   "Pacing"
options:
  - "Show me each step, I'll approve as we go (Recommended)"
      desc: "I ask you something, you pick, we move. Best for first-time runs. No surprises."
  - "Ask me a bunch of questions first, then write"
      desc: "I run a grill-me interview to learn you better, THEN produce personality.md. Use if you want a deeper read."
  - "Just produce it, I'll review at the end"
      desc: "Fastest. I make the calls based on what you give me and present results for approval. Use if you've done this before."
  - "Brief plan, 2-3 quick questions, then write"
      desc: "I sketch what I'm about to do, ask a few targeted things, then ship the draft. Middle ground."
```

Keep option text concrete — describe *what happens to the user* in plain English, not insider names like "walk-through" or "interview-heavy."

Honor whatever they pick. One-shot → run Steps 1–7 in a single pass. Walk-through → every step is its own turn. Interview-heavy → go to Step 6 even with a report in hand.

### Step 1 — Discover the user's existing setup

Before asking for input, look at what they already have:

- Global CLAUDE.md? (`~/.claude/CLAUDE.md` on Mac/Linux, `C:\Users\<user>\.claude\CLAUDE.md` on Windows)
- Memory dir with MEMORY.md and entries?
- Any existing personality doc (search for `personality.md`, `user-manual.md`, `about-me.md`, etc.)
- Project CLAUDE.md files in the working directory?

**Also check for prior Augur output — this skill may have run before.** Look for an `<!-- AUGUR START -->` … `<!-- AUGUR END -->` block in CLAUDE.md and any existing `personality.md`. That puts the run in one of three states — name which one to the user:

- **Fresh** — no markers, no file. Default path.
- **Re-run** — markers already hold an `@`-import line and `personality.md` is in the current lean format. You're refreshing, not migrating: regenerate the file from the new input, overwrite in place at sign-off; the marker block is already correct, so leave it. Don't append a second import or a second marker block.
- **Migration** — markers hold a *terse inline preamble* (old format) or `personality.md` is an old report (claimed/observed table, citations, caveats). Convert the block to an `@`-import and overwrite the report with the lean file (Step 7).

In every prior-output case, don't silently overwrite — flag the state and fold sign-off in at Step 7.

Report what you found in one or two sentences. This is where the file will eventually integrate.

### Step 2 — Ask for input (gate before interview)

Use `AskUserQuestion`. Don't free-text it.

```
question: "What do you have for me to read?"
header:   "Input"
options:
  - "I'll paste a personality report (Recommended)"
      desc: "Big Five aspect-level (Understand Myself, IPIP-NEO), DISC, 360. High-tier input — drives the rules."
  - "I'll give you a file path to a report or 'user manual for me' doc"
      desc: "I'll read it from disk. Medium or High tier depending on source."
  - "I have nothing — interview me instead"
      desc: "Skip to Step 6. Treated as Medium-tier input."
  - "I want to combine: I'll paste a report AND have you interview me on top"
      desc: "Strongest signal. Report + interview cross-check each other."
```

If they pick paste → wait for content in the next turn.
If they pick file path → ask for the path (one-line free response is fine here, it's just a path).
If they pick nothing → go to Step 6.
If they pick combine → read the report first (Steps 3–4), then run Step 6, then synthesize.

### Step 3 — Grade the input

Tier the input by credibility:

| Tier | Examples | How to weight |
|---|---|---|
| **High** | Big Five aspect-level (Understand Myself, IPIP-NEO), DISC with full profile, 360-degree feedback, validated work-style assessments | Drives the rules. Trust the structure; still distinguish claimed vs observed. |
| **Medium** | Self-written "user manual for me," professional bio, LinkedIn About, blog post about working style | Useful signal, but cross-reference heavily with observed behavior. |
| **Low** | Astrology, BuzzFeed quizzes, fictional archetype (Hogwarts house, D&D alignment, Star Wars character), MBTI four-letter type with no other context | Do not derive rules from it. Lean on observation + interview instead; if the user wants it acknowledged, do so in chat, not the file. |

Know the tier — it governs how hard the input drives the rules. It does **not** get written into `personality.md` (that's provenance); if the user asks how you weighted things, tell them in chat.

### Step 4 — Evaluate critically

Read the input and ask yourself:

- What's claimed? List every behavioral claim.
- What's observed in this session and any available signal (memory, recent messages, code style if present)? List those.
- Where do claimed and observed agree? Where do they diverge?
- Which claims are unsupported by anything you can see?

Done when **every claim is tagged** agree / diverge / unsupported, and every divergence has a resolution (observed wins) ready to become a rule in Step 5. The skeptical lens is the skill — an untagged claim means the step isn't finished.

### Step 5 — Synthesize `personality.md`

One artifact. Built to **load into context every session**, not to file as a report. Cap it at **~25–30 lines** so it's cheap to load every turn. If it grows past that, you're writing a report — cut back to conclusions.

Do the critical analysis (Step 4) in your head: claimed vs observed, where they diverge, which claims nothing supports. Then throw the scaffolding away and write only the rules. Where a self-reported score and observed behavior conflict, write the **observed** rule.

Structure — three parts, nothing else:

```markdown
# <user> — how to work with me

<one-line headline. The single most important thing about working with this user. Not a paragraph.>

## How to work with me
- <directive rule Claude can act on. Imperative voice. "Be blunt," not "User is low on agreeableness.">
- <one rule per line. Behavior, not trait description.>
- <...>

## Watch for me
- <a failure mode to catch on the user's behalf — a blind spot the personality predicts.>
- <imperative: what Claude should DO when it sees the gap, not just naming the gap.>
- <...>
```

Rules for the content:

- **Directive, not descriptive.** Every line is something Claude does or watches for. No trait labels, no percentile scores, no "observed in session 3" footnotes, no self-report caveats. Those are provenance — they don't belong in a load-every-turn file.
- **"How to work with me"** = how the user wants to be talked to and worked with. Pacing, bluntness, depth, autonomy, format.
- **"Watch for me"** = where the user is likely to underweight a concern, written as an action. A low score is a tell, not a compliment — name the blind spot *and* what to do about it.

Example shape (this is the whole file, ~20 lines):

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

This is what tunes Claude, and it's also the human-readable thing. Strangers reading it should understand it cold.

### Step 6 — Interview (fallback, or supplement)

Use this when:
- Step 2 returned no input, **or**
- The user picked "interview-heavy" or "combine" in Step 0 / Step 2.

**Default mechanic: run the [[grill-me]] skill for the interview.** Grill-me is built for exactly this — relentless one-question-at-a-time grilling that resolves the decision tree, using the same `AskUserQuestion` UI pattern this skill uses everywhere else. It respects user-set pacing.

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

Do not impose a fixed question set. **Each question should narrow based on the prior answer.** Pre-planning all questions defeats the point. The instruction is: conduct an interview of a length, depth, and style appropriate to this user.

After the interview, treat the transcript as Medium-tier input and run Steps 3–5.

### Step 7 — Place the file (with confirmation)

The file is short and it's what tunes Claude, so the user sees it in full before it's placed (Rule 10).

Order of operations:

1. **Write `personality.md` to a draft path first** — e.g. `~/.claude/augur/personality.draft.md`. This gives the user a real file to open and edit *while* you're asking for sign-off.
2. **Show inline**: (a) the full file (~25–30 lines), (b) the path you propose for it, (c) the exact `@`-import line you'll add to CLAUDE.md.
3. **Use `AskUserQuestion` for the sign-off:**

```
question: "Approve and place personality.md?"
header:   "Sign-off"
options:
  - "Approve — place it and wire the import (Recommended)"
      desc: "Renames the draft to personality.md and adds the @-import line to CLAUDE.md so it loads every session."
  - "Place the file, but don't touch CLAUDE.md"
      desc: "Writes personality.md. You wire the import yourself later. Use if your setup is non-standard."
  - "Hold — I want to edit the draft first, then re-run sign-off"
      desc: "Nothing is placed. Draft stays at personality.draft.md. Ping me when ready."
  - "Reject — start over"
      desc: "Discard the draft. Restart from Step 0 or wherever you want."
```

**Location.** Default `~/.claude/augur/personality.md`. If Step 1 found an existing personality doc or a non-standard setup, ask where it should live instead of assuming — the user might want it alongside their existing docs.

**Wiring it to load every session.** A plain text pointer in CLAUDE.md does *not* load the file's content — only an `@`-import does. Place a single import line inside the re-runnable marker block:

```markdown
<!-- AUGUR START -->
@~/.claude/augur/personality.md
<!-- AUGUR END -->
```

Integrate based on the Step 1 state:
- **Fresh, has global CLAUDE.md** → write the marker block with the `@`-import line (path adjusted to wherever the file landed).
- **Re-run** → overwrite `personality.md` in place; the marker block already holds the right import, so leave it untouched. One file changes, nothing else.
- **Migration** → *replace* the old inline preamble between the markers with the `@`-import line, and overwrite the old report-format `personality.md` with the lean one. Tell the user the format changed: rules now live in `personality.md`, CLAUDE.md just imports it. Never leave both an inline preamble and an import — that double-loads the rules.
- **Has memory system** → write a reference memory pointing at the file's path so future sessions can find it.
- **No CLAUDE.md** → suggest creating `~/.claude/CLAUDE.md` with the marker block + import as the seed. If they decline, leave `personality.md` on disk and tell them it won't auto-load until something imports it.

After writing, confirm in ≤3 lines what landed where.

## What this skill will NOT do

- Flatter the user.
- Trust self-report uncritically.
- Force a personality.md file format on top of an existing setup.
- Treat astrology, MBTI-type-only, or fictional-archetype input as load-bearing.
- Generate a personality from thin air. If there is no input and no interview, the skill stops.
- Write `personality.md` without sign-off.
- Skip Step 0. Pacing is *always* the user's call, never Claude's guess.
- Write a report. `personality.md` carries conclusions — directive rules — not the claimed-vs-observed scaffolding, scores, citations, or caveats. If it grows past ~30 lines, cut it back.
- Ask gating questions in free-text prose when AskUserQuestion will do. The UI is the mechanic.

## Sharing and portability notes

This skill is intended to be shareable. The `personality.md` it produces is designed to be:

- **Cheap to load** — ~25–30 lines, so it can sit in context every session without weight.
- **Readable by strangers** — directive rules, no internal jargon, no scores to decode.
- **Editable by the user** — the file is theirs, they own it, they can rewrite.

If you (Claude, running this skill) catch yourself adding flourish, hedging, padding, or provenance to the file: stop, cut it, ship the lean version.
