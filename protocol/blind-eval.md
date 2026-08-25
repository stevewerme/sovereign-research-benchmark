# Blind synthesis eval — protocol (Goal 3)

How the cited briefs were compared against the baseline's answers, blind.

## Design

- **n external evaluators**, each judging a subset of the 8 questions. Invariant: every question
  receives exactly **k judgments, k odd and ≥ 3** — odd so the per-question majority always
  resolves. Per-evaluator load flexes by at most one question with roster size. The benchmark
  author is not an evaluator (structurally unblindable); he runs the protocol and does not vote.
- **Both systems render identically**: prose followed by a numbered source list. Inline citation
  markers are stripped from *both* — five of eight baseline answers carry no inline citations
  while the sovereign brief marks every claim, so marker density alone would identify the system
  and bias toward whichever answer *looks* better-sourced. One shared rendering function
  produces both, so they cannot drift apart. Provenance-tagged renderings exist separately and
  are not what evaluators see.
- **Balanced A/B**: which system appears as "A" is balanced to an exact global 50/50 across the
  assignment (per question, ceil(k/2)/floor(k/2)), not independently coin-flipped — a skewed
  split would convert evaluator position bias into a systematic advantage.
- **Choice**: "Which answer is more useful for answering this question — A, B, or tie?" Ties are
  real answers.

## Scoring

Each judgment reduces to a binary — was the sovereign brief *usable or better* (it won, or it
tied)? The per-question majority is taken over that binary, which always resolves for odd k.

**Pass = usable-or-better majority on ≥ 6 of 8 questions.** Reported alongside it, always: the
**strict-win count** (majorities that preferred the sovereign brief outright, ties excluded) —
because ties count toward the bar, a question can pass having never been preferred, and "6 of 8"
must not be read as six wins. Questions left short of k judgments are reported as shortfalls,
never silently scored.

## Publication

The per-evaluator packets and the assignment map (which system was "A", per evaluator × question)
are published in this repo **after** the essay is live — the map is the blinding secret while the
eval runs, and publishing it afterwards is what makes the blinding claim checkable rather than
asserted. Evaluator identities stay pseudonymous throughout.
