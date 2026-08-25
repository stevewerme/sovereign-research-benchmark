# Coverage metric — retrieval benchmark (Goal 2)

How the 40-question retrieval benchmark was scored. Locked before any retrieval run.

## Definitions

- **Gold key source** — a *fact-level equivalence class*: a canonical URL plus any document
  reporting the same underlying fact, regardless of outlet or publication date. Locked at
  question review (2026-08-16); no new gold facts after lock. Only equivalence-class
  *membership* — "does this retrieved document report the same fact?" — is judged at scoring
  time.
- **Essential vs supporting** — each question carries 1–3 *essential* golds (gating) and
  optionally *supporting* golds (reported as a secondary statistic only).
- **Surfaced** — a member of the gold's equivalence class appears in the sovereign system's
  **raw post-rerank top-10**, or, for the baseline, anywhere in its **full citation list,
  uncapped**. The generosity is deliberate and symmetric in the system's favour being tested:
  the sovereign system does not have to find *the* article, only *an* article carrying the fact.
- **Per-question pass** — every essential gold surfaced.
- **Coverage** — fraction of the locked set passing. **PASS = coverage ≥ 80% AND ≥ the
  baseline's coverage on the same set.**

## Adjudication conventions

Adjudication was human, line by line, to a JSONL scoring record per benchmark run. Registry gaps
count as misses. Every failed question receives a miss cause, adjudicated against the crawl
manifest so "the crawler never saw it" and "retrieval buried it" are distinguishable:

- `registry-gap` — no source in the registry covers the fact
- `crawl-gap` — a registry source covers it, but the crawler could not fetch it
- `retrieval-gap` — the document is in the index and did not surface

## Escape hatch (declared in advance)

A gold found *wrong* at scoring time drops its question from the denominator **for both
systems**, logged; more than ~10% dropped would count as a question-authoring failure and be
reported as such. **Outcome: 0 of 56 golds dropped.**

## Baseline protocol

Perplexity Pro Search, default model, fresh thread per question, memory and personalization off,
verbatim question text in the original language, first answer taken, no retries (technical
failures rerun and logged). Full answer and complete ordered sources panel captured per question.
Both systems ran after gold-lock and the crawl snapshot, inside the same 7-day window.
