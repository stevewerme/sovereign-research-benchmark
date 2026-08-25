# Verification judge — taxonomy, quote rule, bars (Goal 1)

How the 55 judgment pairs score a model's ability to decide whether a source supports a claim.

## Verdicts

- **`supports`** — the source states the claim's specifics (who / what / how much / when).
  A related topic is not support.
- **`partial`** — the claim is *accurate in what it states* and the remainder is simply not
  addressed by the source: the stated part is correct and quotable, the excess absent — not
  contradicted, not inflated.
- **`does_not_support`** — the source is readable but does not state the claim — **including
  whenever any specific in the claim is wrong or inflated** relative to the source: a bigger
  number, a stronger role or scope, a completed status the source calls planned, a wrong entity
  or period. A claim with a wrong specific is never `partial`, even if the rest matches. The
  reasoning: "partially supported, quote shows 15%" still lends an invented 25% figure borrowed
  credibility. Verification exists to catch exactly that drift, so the overstated sentence is
  rejected outright; salvage happens by rewriting the claim down and re-judging.
- **`insufficient_content`** — the text itself is unusable (paywall stub, cookie boilerplate,
  bare teaser, truncation). About the state of the text only — never a substitute for
  `does_not_support` on a readable article.

## The verbatim-quote rule

The judge must return a `supporting_quote` copied character-for-character from the source (or
null for rejecting verdicts). The quote is checked programmatically as a substring of the source
text. **A fabricated quote counts as false support regardless of the verdict it accompanies.**
This is the check that catches a model quoting "Operative EBIT improved to EUR −12.4 million" as
evidence a company "returned to profitability" — a stitched, non-verbatim quote supporting a
false claim, invisible to anyone skimming verdicts.

## Bars

Per model, against the human gold labels: **agreement ≥ 90% AND false support ≤ 3%**, where
false support = gold ∈ {does_not_support, insufficient_content} but verdict ∈ {supports,
partial}, plus any fabricated quote. The two bars are asymmetric on purpose: false refutation
costs recall; false support is the trust-breaking error class, so it gets its own ceiling.

The frontier reference judge is a *comparator* to contextualize the gap. Gold is truth;
disagreements between gold and any model were resolved by the human, never by the frontier model.

## Trap composition (55 pairs, 35 sv / 20 en)

Clear support 13 · topically-related non-support 13 · overstatement 9 · accurate-but-incomplete
(true partial) 9 · degraded fetch 6 · wrong entity/period 5.

The frozen prompt is `judge-v4-prompt.md` in this directory, exactly as run.
