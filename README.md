# Sovereign Research Benchmark

The locked test sets and scoring protocol behind the benchmark write-up,
**[which is published here](https://stevewerme.github.io/sovereign-research-benchmark/)**
— a four-weekend test of whether a fully EU-sovereign pipeline can produce verified research:
EU-hosted open models for the quality-critical verification step, and a curated Nordic/EU crawl
against a whole-web commercial baseline.

The essay and the data it reports on live in the same place on purpose: read the write-up, then
check it against the question set, the labels and the protocol that produced it. Both are CC BY
4.0. Argue with either.

## What's here

| Path | Contents |
|---|---|
| `data/questions.jsonl` | The 40 research questions with gold key sources — **verbatim**, as locked 2026-08-16 |
| `data/judgment-pairs.jsonl` | The 55 (claim, source) judgment pairs — labels, traps and gold rationale intact, article text **redacted to URL + SHA-256** (see below) |
| `protocol/coverage-metric.md` | How retrieval coverage was scored, incl. the equivalence-class gold definition and adjudication conventions |
| `protocol/judge.md` | The verdict taxonomy, the verbatim-quote rule, and the pass bars for the verification eval |
| `protocol/judge-v4-prompt.md` | The frozen judge prompt, exactly as run |
| `protocol/blind-eval.md` | The blind A/B protocol for the synthesis eval |

## Method, in brief

Falsifiability first: **numeric exit criteria were committed before any run produced a number**,
and meeting or missing them were declared equally publishable outcomes in advance.

- **Ground truth is human and was locked.** All labels were authored by hand. The question set's
  gold key sources were locked **2026-08-16**; the judgment pairs were relabelled under the final
  taxonomy the same day and extended with nine true-partial pairs on 2026-08-17, before the final
  runs. After lock, no new ground truth could be added — only equivalence-class membership
  ("does this document report the same fact?") was decided at scoring time, recorded line by line.
- **The escape hatch went unused.** A gold found wrong at scoring time could drop its question
  from the denominator for both systems, logged. **Zero of 56 gold key sources were dropped.**
- **Both systems ran in the same seven-day window**, after gold-lock and after the corpus was
  frozen (crawl snapshot 2026-08-17; sovereign benchmark run 2026-08-17; baseline captured
  2026-08-18). Neither could see the other's results.
- **The frontier model was a comparator, never a participant.** A Claude-class model runs in the
  evaluation harness only; a test fails the build if it ever appears in the pipeline.
- **Every model call carries provenance** — provider, model, timestamp, tokens, cost.

## Why the article text is redacted

Each judgment pair's `source_text` — the extracted article body the judge ruled against — is
replaced by `source_text_sha256` and `source_text_chars`. Several source outlets explicitly
refuse AI training in their terms; republishing 55 article bodies to prove a point about
verification would be a poor way to make it.

Verification is still possible: fetch `source_url`, extract the article text, hash it (SHA-256
over the UTF-8 string), and compare. A match confirms you are judging the same bytes the
benchmark did. Extraction drift and dead links are real — the hash tells you *whether* you have
the same text, not how to reconstruct it.

## What is not published

- **The source registry** (57 curated Nordic/EU sources spanning tech press, business media,
  public broadcasters, government and DPA newsrooms, EU institutions, policy media and analysis
  orgs). It is described in aggregate in the essay; the list itself is retained.
- **The blind-eval packets and assignment map** — published here once the essay is live, so the
  blinding claim is checkable after the fact without being breakable before it.

## License and corrections

Everything here is **CC BY 4.0** (see `LICENSE`), same as the essay. Disputes about the numbers
are welcome as issues on this repo — that is what it is for. Corrections to the essay are made
in place and logged there with dates.
