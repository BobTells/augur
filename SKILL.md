---
name: augur
version: 0.1
description: Tune Claude to a user's personality by reading a credible report (or interviewing as fallback), then emitting two artifacts — a terse behavior preamble that drops into the user's existing Claude setup, and a human-readable personality artifact that shows the work. Use when the user invokes /augur, pastes Big Five / DISC / 360 / personality assessment results, asks to personalize or tune Claude to their style, or references "human API" / personality calibration.
---

# Augur

A skill that tunes the LLM to a user's personality. The user supplies evidence (an assessment report, a "user manual for me," a bio) or — as fallback — agrees to an interview. The skill reads that input critically, weights it by credibility, and emits two artifacts:

1. **Behavior preamble** — terse rules that drop into the user's existing Claude setup (global CLAUDE.md or wherever they keep durable instructions). This is what tunes Claude. Includes both **how to work with the user** and **what to watch for on their behalf** (predicted blind spots).
2. **Personality artifact** — human-readable doc that shows the work: what was claimed, what was observed, why the preamble says what it says. This is what makes the preamble defensible and what the user can share, edit, screenshot, or audit.

Both artifacts together are the contract: the user sees the rules and the evidence; Claude reads the rules and honors them.

## Thesis

You are not having a conversation with the model. You're using English words to discuss a math problem. The personality vector is one of the inputs to that problem. Augur makes that input explicit instead of hoping the model guesses.

## When to invoke

- User runs `/augur`
- User pastes Big Five aspect scores, DISC results, 360 review, MBTI, Enneagram, etc.
- User says "personalize Claude," "tune the model to me," "make Claude match my style"
- User shares a "user manual for me" doc, personal bio, or LinkedIn About-style writeup and asks Claude to use it
- User references "human API," "personality preamble," or similar

## Hard rules (these are the skill's POV)

These are non-negotiable. They are why this skill is opinionated, not generic.

1. **No flattery.** Do not tell the user they are smart, insightful, thoughtful, or anything similar. Do not validate the quality of their input unprompted. The LLM's default is to flatter; the skill must override that default.
2. **Self-report is suspect.** Assessments are biased by social desirability, mood, self-image. Read every input critically. Cross-reference with available behavioral signal (recent messages, visible writing style, memory entries, code style if present).
3. **Distinguish claimed from observed.** In the artifact, mark traits as `claimed`, `observed`, or `both`. The reader should be able to see the seams.
4. **Credibility is graded.** Not all input is equal. Tier the input (see below). Low-tier input is recorded for color but does not drive the preamble.
5. **Adapt to the user's setup.** Do not force a particular file layout. Discover what the user already has (CLAUDE.md, memory dir, existing personality docs) and integrate. A thin setup is fine.
6. **Confirm before writing.** Show the user the preamble and the artifact before placing them. The artifact is a contract — the user must sign off.
7. **Use the vector both ways.** The personality vector predicts how the user wants to be talked to *and* where they're likely to underweight a concern. The skill must surface both. A low score is a tell, not a compliment — call out the blind spot it predicts and write a rule that helps compensate.
8. **Ask the user how they want to do this. Then do it that way.** The whole skill is itself a personality-matching exercise. Don't guess pacing, depth, autonomy, or output format. Ask up front (Step 0). The user might want a one-shot dump, a walk-through, an interview-heavy slow build, or anything in between. Whatever they say goes. This rule applies to the entire skill, not just the interview fallback. The sauce of this skill is getting the user comfortable enough that the input you collect is accurate — guessing how to work with them defeats the point.
9. **Use the AskUserQuestion UI for every decision point.** Every gate in this skill (pacing in Step 0, input source in Step 2, interview-on-top in Step 6, sign-off in Step 7) is a `AskUserQuestion` call with 2–4 sharp options and a `(Recommended)` first. One question per turn. No free-text "so, what do you think?" prompts where a structured pick would do. The user signed up for a tuning exercise, not an essay reply. If a question can't be reduced to 2–4 options, it probably shouldn't be asked yet — narrow it first.
10. **Never paste the full personality artifact into chat.** The artifact is a document. Documents are read in editors. At sign-off (Step 7), write it to a draft path first, show *only the paths + the short preamble*, and use `AskUserQuestion` for the approval decision. Pasting the whole artifact inline floods the conversation, makes it un-editable in place, and turns the user's review into scrolling instead of reading. This is the single most common failure mode for this skill — guard against it.

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
      desc: "I run a grill-me interview to learn you better, THEN produce the artifacts. Use if you want a deeper read."
  - "Just produce it, I'll review at the end"
      desc: "Fastest. I make the calls based on what you give me and present results for approval. Use if you've done this before."
  - "Brief plan, 2-3 quick questions, then write"
      desc: "I sketch what I'm about to do, ask a few targeted things, then ship the draft. Middle ground."
```

**Why these labels are concrete, not jargon:** first-time users don't know what "walk-through" or "interview-heavy" means in this skill. The option text must describe *what happens to them* in plain English. Don't drift back to insider names.

Honor whatever they pick. If they pick one-shot, run Steps 1–7 in a single pass. If they pick walk-through, every step is its own turn. If they pick interview-heavy, go to Step 6 even with a report in hand.

This step exists because the skill is itself an exercise in matching the user's style. Guessing here makes the rest of the input less trustworthy — a user who got steamrolled at Step 0 will give shorter, less honest answers downstream.

### Step 1 — Discover the user's existing setup

Before asking for input, look at what they already have:

- Global CLAUDE.md? (`~/.claude/CLAUDE.md` on Mac/Linux, `C:\Users\<user>\.claude\CLAUDE.md` on Windows)
- Memory dir with MEMORY.md and entries?
- Any existing personality doc (search for `personality.md`, `user-manual.md`, `about-me.md`, etc.)
- Project CLAUDE.md files in the working directory?

Report what you found in one or two sentences. This is where the artifacts will eventually integrate.

### Step 2 — Ask for input (gate before interview)

Use `AskUserQuestion`. Don't free-text it.

```
question: "What do you have for me to read?"
header:   "Input"
options:
  - "I'll paste a personality report (Recommended)"
      desc: "Big Five aspect-level (Understand Myself, IPIP-NEO), DISC, 360. High-tier input — drives the preamble."
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
| **High** | Big Five aspect-level (Understand Myself, IPIP-NEO), DISC with full profile, 360-degree feedback, validated work-style assessments | Drives the preamble. Trust the structure; still distinguish claimed vs observed. |
| **Medium** | Self-written "user manual for me," professional bio, LinkedIn About, blog post about working style | Useful signal. Note in artifact, but cross-reference heavily with observed behavior. |
| **Low** | Astrology, BuzzFeed quizzes, fictional archetype (Hogwarts house, D&D alignment, Star Wars character), MBTI four-letter type with no other context | Record in artifact as user-identified-as. Do not derive behavior rules from it. Lean on observation + interview instead. |

State the tier explicitly in the artifact. Do not hide it.

### Step 4 — Evaluate critically

Read the input and ask yourself:

- What's claimed? List every behavioral claim.
- What's observed in this session and any available signal (memory, recent messages, code style if present)? List those.
- Where do claimed and observed agree? Where do they diverge?
- Which claims are unsupported by anything you can see?

Do not skip this step. The skeptical lens is the skill.

### Step 5 — Synthesize the two artifacts

#### A. Personality artifact (human-readable, defensible)

File: ask the user where it goes. Default suggestion: `~/.claude/augur/personality.md` or under their existing project structure.

Structure:

```markdown
# <user> — Personality artifact

**Source:** <name of report / interview transcript>
**Tier:** <high | medium | low>
**Captured:** <date>

## Headline

<short summary — one paragraph at most>

## What was claimed

<bulleted list of behavioral claims from the input, with source>

## What was observed

<bulleted list of behaviors observed in available signal, with where>

## Reconciliation

<table or list: claim | observed | both | unclear>

## Behavioral implications — how to work with this user

<one rule per implication. Each rule cites the trait/score/observation it came from. Populates the "How to work with me" section of the preamble.>

## Predicted blind spots — what to watch for on their behalf

<one rule per likely gap. Each rule cites the score/observation. The user is competent enough to know these exist; the skill names them so Claude can compensate without being asked. Populates the "Watch for me" section of the preamble.>

Example pattern:
- "Low aesthetics → push back on visual decisions, suggest alternatives. User won't naturally reach for them."
- "Low compassion → flag when external comms need a softer register. User won't see it."
- "Low orderliness → offer structure when ambiguity is starting to cost. Don't demand it; surface it."

## Caveats

<self-report bias, sample size, anything else>
```

This is the doc the user can read, edit, and share. It is the contract.

#### B. Behavior preamble (terse, what Claude reads)

A short block — aim for under 20 lines. Bulleted rules only. No explanation, no source citations — those live in the artifact.

Example (drawn from the artifact's "Behavioral implications" section):

```markdown
# Personality preamble

## How to work with me
- Be blunt. No "great question," no softening preamble.
- Recommend one thing, not a menu of three.
- Idea-dense over hand-holding. Don't dumb it down.
- No flowery prose. Function over form.
- Tolerate disorder. Don't demand structure that isn't there.
- Drive to completion. Bias to shipping.
- Don't add caveats the user wouldn't add themselves.

## Watch for me (blind spots)
- Visual / UI decisions — push back, suggest alternatives I won't reach for.
- External-facing copy — flag when a softer register would land better with the audience.
- Long-running ambiguity — surface structure if it's starting to cost; don't demand it.
```

This is what tunes Claude. Strangers reading the preamble should understand it cold.

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
      desc: "Deeper coverage. Use if you want the artifact to be exhaustive."
  - "Lighter — 2–3 questions, no full grill"
      desc: "Just the highest-leverage gaps from what I already know."
  - "Skip — go straight to synthesis with what we have"
      desc: "Only viable if a credible report was already provided."
```

If grill-me isn't installed in the user's setup, fall back to running the same pattern manually: `AskUserQuestion` per turn, 2–4 options each, recommended-first.

Do not impose a fixed question set. **Each question should narrow based on the prior answer.** Pre-planning all questions defeats the point. The instruction is: conduct an interview of a length, depth, and style appropriate to this user.

After the interview, treat the transcript as Medium-tier input and run Steps 3–5.

### Step 7 — Place the artifacts (with confirmation)

**Hard rule, restating Rule 10: do not paste the full personality artifact into chat.** The artifact is a document — it lives in a file, the user opens it in their editor. Pasting it inline is the failure mode this skill exists to prevent.

Order of operations:

1. **Write the artifact to a draft path first** — e.g. `~/.claude/augur/personality.draft.md`. This gives the user a real file to open and edit *while* you're still asking for sign-off.
2. **Show inline only**: (a) the two file paths you propose to use, (b) the preamble in full (it's ≤20 lines and it's what actively changes Claude's behavior, so the user needs to see it before approving), (c) a one-sentence summary of the artifact (not the artifact itself).
3. **Use `AskUserQuestion` for the sign-off:**

```
question: "Approve and place the artifacts?"
header:   "Sign-off"
options:
  - "Approve — place both, update memory (Recommended)"
      desc: "Renames the draft to personality.md, appends preamble between <!-- AUGUR START --> markers in CLAUDE.md, writes reference memory."
  - "Approve preamble only — I'll edit the artifact myself first"
      desc: "Places the preamble. Leaves personality.draft.md for you to edit; you can rename when ready."
  - "Hold — I want to edit the draft first, then re-run sign-off"
      desc: "Nothing is placed yet. Draft stays at personality.draft.md. Ping me when ready."
  - "Reject — start over"
      desc: "Discard the draft. Restart from Step 0 or wherever you want."
```

Then integrate based on the setup you found in Step 1:
- **Has global CLAUDE.md** → append the preamble as a delimited block between `<!-- AUGUR START -->` and `<!-- AUGUR END -->` markers so it's safely re-runnable. Subsequent /augur runs overwrite between the markers, leaving the rest of CLAUDE.md untouched.
- **Has memory system** → write a reference memory pointing at the artifact's path so future sessions can find it.
- **Has neither** → suggest creating `~/.claude/CLAUDE.md` and drop the preamble there as the seed.
- **Artifact location** → default `~/.claude/augur/personality.md`. Ask only if the user has a non-standard setup.

After writing, confirm in ≤3 lines what landed where. Do not summarize the artifact's content — they can open it.

## What this skill will NOT do

- Flatter the user.
- Trust self-report uncritically.
- Force a personality.md file format on top of an existing setup.
- Treat astrology, MBTI-type-only, or fictional-archetype input as load-bearing.
- Generate a personality from thin air. If there is no input and no interview, the skill stops.
- Write the artifacts without sign-off.
- Skip Step 0. Pacing is *always* the user's call, never Claude's guess.
- Paste the full personality artifact into chat. It's a document. Documents live in files.
- Ask gating questions in free-text prose when AskUserQuestion will do. The UI is the mechanic.

## Sharing and portability notes

This skill is intended to be shareable. The artifacts it produces are designed to be:

- **Readable by strangers** — preamble has no internal jargon; artifact shows its work.
- **Editable by the user** — the artifact is theirs, they own it, they can rewrite.
- **Auditable** — any rule in the preamble can be traced back to a source in the artifact.

If you (Claude, running this skill) catch yourself adding flourish, hedging, or padding to the artifacts: stop, cut it, ship the lean version.
