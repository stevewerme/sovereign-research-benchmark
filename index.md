---
layout: default
title: "The binding constraint on sovereign AI isn't model quality — it's corpus access."
---

# The binding constraint on sovereign AI isn't model quality — it's corpus access.

*Four weekends, three falsifiable goals, two of them failed. Here are the numbers.*

---

Over four weekends I built a research pipeline that runs entirely inside the EU: European
inference, European database, European hosting, no US API in the data path. Not as a product —
as a test. I wanted to know which part of the "sovereign AI" pitch is real engineering and which
part is a slide.

I had a prediction going in. The risky part, obviously, would be the models. EU-hosted open
weights against frontier American models, on the one task where being wrong is unforgivable:
deciding whether a source actually supports a claim. That's where I expected to lose.

I was wrong about which part was hard. The verification step passed — three of the four European
open models I tested cleared the bar, two of them matching a frontier model exactly at a fifth of
the cost (the fourth failed, instructively, and I'll come back to it). What failed were the two
parts I had treated as plumbing: getting hold of the documents in the first place, and writing an
answer someone actually wants to read. Neither failure has anything to do with sovereignty, AI, or
model quality.

Here are the numbers, including the ones that don't flatter me — and there are more of those than
I expected.

## How the benchmark was built so that it could fail

Most published AI benchmarks have one structural problem: the person reporting the result chose
the threshold after seeing it. I tried to make that impossible for myself.

- **Thresholds were written down first.** Each stage got a spec with numeric exit criteria and a
  hard stop, committed before any run produced a number. Meeting them and missing them were
  declared equally publishable outcomes in advance.
- **Ground truth is human, and it was locked.** The 55 judgment pairs and the 40 research
  questions were labelled by hand — by me, not by a model — and the answer keys were frozen at a
  dated lock *before* the retrieval system was pointed at them. After lock, no new ground truth
  could be added. Only membership questions ("does this document report the same fact?") were
  decided at scoring time, and those were recorded line by line.
- **The escape hatch was declared in advance and went unused.** The protocol allowed a question to
  be dropped from the denominator if its answer key turned out to be wrong — with the drop logged,
  and >10% dropped counted as a failure of my question-writing, not of the system. Zero of 56
  answer keys were dropped.
- **The frontier model was a spectator, never a participant.** A Claude-class model runs in the
  evaluation harness as a comparator only. It never appears anywhere in the pipeline, and a unit
  test fails the build if it ever does. The point of a sovereignty claim is that it survives
  contact with an actual check.
- **Every model call carries a receipt.** Provider, model, timestamp, latency, token cost, on
  every call, written to disk. The cost tables below aren't estimates.
- **Nothing was rerun after a bad result.** The retrieval benchmark failed its bar. The numbers in
  this post are that run.

Both systems — mine and the commercial baseline — ran inside the same seven-day window, after the
answer keys were locked and after the corpus was frozen. Neither could see the other's results.

Greenfield throughout: no code reused from anything I've built before, nothing borrowed from any
existing product. That was a constraint I gave myself for unrelated reasons, but it had a useful
side effect — nothing in the result is propped up by infrastructure a reader can't see.

## Part one: can a European open model verify a claim?

This is the load-bearing question. A research tool that cites sources is only worth anything if
something checks that the sources say what the text claims they say. Retrieval quality, prose
quality, latency — all recoverable. A system that confidently attaches a real URL to a claim the
URL doesn't support is worse than useless, because it manufactures credibility.

So: 55 (claim, source) pairs, hand-built from real Nordic and EU articles, 35 Swedish and 20
English. The mix is adversarial on purpose:

| Trap class | Count | What it tests |
|---|---|---|
| Clear support | 13 | baseline competence |
| Topically related non-support | 13 | the lazy-similarity trap |
| Claim overstates the source | 9 | the dangerous class |
| Claim adds facts the source omits | 9 | genuine partial support |
| Degraded fetch (paywall stub, cookie wall) | 6 | must abstain, not guess |
| Wrong entity / wrong period | 5 | precision on specifics |

Each pair gets one of four verdicts: `supports`, `does_not_support`, `partial`,
`insufficient_content`. Structured output, temperature 0, and one mechanical check that turns out to
matter a lot: the model must return a supporting quote, verified programmatically to be a real
substring of the source. A fabricated quote counts as false support whatever verdict accompanies
it.

The bar, set in advance: **≥ 90% agreement with the human labels, and ≤ 3% false support.** Those
two are not symmetric. False refutation costs recall — you lose a usable sentence. False support
is the error that breaks the product's reason to exist, so it gets its own ceiling.

**Results, EU-hosted open models via Berget:**

| Model | Judged | Agreement with human labels | False support | Fabricated quotes | Cost / 1k judgments | Mean latency |
|---|---|---|---|---|---|---|
| GLM-5.2 | 54 | **100.0%** | 0 | 0 | €2.64 | 21.5 s |
| Kimi-K3 | 55 | **100.0%** | 0 | 0 | €7.09 | — |
| Kimi-K2.6 | 53 | 94.3% | 0 | 0 | €4.67 | 11.4 s |
| Mistral Medium 3.5 | 55 | 89.1% | **7.3% (4)** | **2** | €2.02 | 4.4 s |
| *claude-opus-5 (comparator, not in pipeline)* | 55 | 100.0% | 0 | 0 | €11.97 | 4.8 s |

Kimi-K3 entered as a post-hoc candidate after the first three were specified; it is scored on the
same locked set, the same frozen prompt and the same rules as the rest.

**Three of the four cleared the bar.** Two of them scored exactly what the frontier comparator
scored, on a set built specifically to catch overclaiming — at **a fifth and a half of the cost**.
Not "close enough for the price". Identical, on this set.

The fourth failed. That's the more useful half of this table.

### How the failing model failed

Mistral Medium missed the agreement bar by a single pair — 89.1% against a 90% threshold. If
agreement were the only criterion, I'd have spent a paragraph on rounding and moved on.

It missed the false-support ceiling by 2.4×. Four claims it called supported were not — and zero
errors ran the other way. Three were the overstatement class: a wrong figure or inflated status
soft-labelled as partially backed. The fourth was a flat `supports` on a claim the source
contradicts.

Then the part I'd never have caught by reading verdicts. Twice, Mistral returned a *fabricated*
supporting quote — text presented as verbatim that does not appear in the source, both stitched from
non-contiguous fragments spliced with an ellipsis. The second is the most instructive data point in
the project. The claim: a company "returned to profitability." The verdict: partially supported.
The evidence, offered as a direct quote:

> Operative EBIT improved to EUR -12.4 million from EUR -26.4 million

That is a loss — smaller than last year's, still a loss — quoted as proof of profit, in a quotation
that was never in the document. A reader skimming verdicts sees "partial, quote attached" and moves
on; the substring check catches it in a millisecond. That is the whole argument for making the check
mechanical.

And the finding that should worry anyone shortcutting this evaluation: **Mistral was the cheapest
and fastest model I tested.** €2.02 per thousand judgments against GLM's €2.64, and 4.4 seconds
against 21.5 — quicker than the model I shipped, and quicker than the frontier comparator. Choose a
verification model on price and latency, the two numbers a dashboard shows by default, and you
choose the one that fabricates evidence.

One wrinkle that looks like a contradiction: Mistral scored **9 out of 9** on the pairs where the
answer genuinely *is* "partially supported", equal to the best. It reaches for `partial` more
readily than the others — right when a claim is merely incomplete, catastrophic when it's inflated.

The honest caveats, in order of importance:

**It's 55 pairs.** A perfect score on 55 pairs is consistent with a true error rate of a few
percent. It is enough to rule out "European open models can't do this", which was the question. It
is not enough to quote a reliability figure at a customer.

**The taxonomy did more work than the model choice.** Every residual disagreement among the passing
models — and the frontier comparator — landed on one boundary: what to do with a claim that
overstates its source. "Grew 25%" where the source says 15%. "Has adopted" where the source says "is
developing". My instinct was to call these partially supported. That's wrong, and it took an
adjudication round to see why: "partially supported, here's a quote showing 15%" still lends the 25%
figure borrowed credibility. So a wrong specific is now flatly `does_not_support`; `partial` is
reserved for claims *accurate in what they state* that simply say more than the source addresses.
Rewriting the rule moved every model's score — and it is exactly the line Mistral could not hold.
Most of the achievable accuracy here sits in defining the task precisely, not in picking the model.

**The passing models are slower.** 21.5 seconds versus the comparator's 4.8. For batch verification
that's irrelevant. For anything interactive it isn't, and I haven't tested that. Note which way this
cuts: the fast sovereign model was the unsafe one.

**Zero false support is doing a lot of load-bearing work here — and it's fragile.** In an earlier
round, on an earlier set, Kimi-K3 produced one false support. Exactly one, and the only one any
*passing* model has produced in this project. That's why the production design doesn't trust a
single judge: one model judges by default, a second from a different family re-judges the
load-bearing and low-confidence cases, and when they disagree **the verdict degrades to the more
conservative of the two — never upward toward `supports`**. Escalation must never be a path to
"yes".

**Catalog churn is real.** My first attempt to measure Mistral returned 55 straight
`404 model_not_found` — its id had changed under me between runs, and re-running on the live id is
what produced everything above. Ids move on EU inference, and the second judge I settled on is
flagged `lifecycle: eval` by its provider. Treat model choice as configuration, not architecture.

**Part one: passed, with a named exception.** Verification on European open models is real and
cheap — and not a property of European open models in general. Three of four cleared the bar; the
fourth failed in the exact direction that would poison a research product, and it was the one you'd
have picked on cost and speed. The capability is available. It is not the default.

## Part two: can a curated crawl beat the whole web?

The second premise is the one every "sovereign AI" story leans on without testing: that you don't
need the whole index, because a well-chosen corpus in a defined domain beats general search inside
that domain.

Setup: a hand-curated registry of 57 Nordic and EU sources — tech press, public broadcasters,
government and agency newsrooms, all four Nordic data protection authorities, EU institutions,
policy media, think tanks. Crawled to twelve months' depth, robots.txt obeyed unconditionally, one
request per host at a time. **24,352 documents, roughly 87,000 chunks**, into Postgres with pgvector
on Scaleway. Retrieval is chunked multilingual embeddings plus Swedish and English full-text search,
fused by reciprocal rank fusion, then reranked, surfacing ten documents.

40 research questions, 24 Swedish and 16 English, spread across Nordic tech, Nordic business,
Nordic government and EU policy, every one anchored to a settled event inside the crawl window.
Each question has one to three *essential* key sources, defined as fact-level equivalence classes —
a canonical URL plus any document reporting the same underlying fact, any outlet, any date. That
definition is deliberately generous to my system: it doesn't have to find *the* article, only *an*
article carrying the fact.

A question passes only if every essential key source is surfaced in the top ten. The baseline —
Perplexity Pro Search, default model, fresh thread per question, verbatim question text — is scored
on exactly the same scale against its full citation list, uncapped.

The bar, again set in advance: **≥ 80% coverage, and at least as good as the baseline.**

| System | Coverage |
|---|---|
| Sovereign pipeline | **22/40 = 55.0%** |
| Perplexity Pro Search | **40/40 = 100.0%** |

**Failed. On both prongs, and not narrowly.**

That result stands as measured. No reruns, no adjustments to the answer keys, escape hatch unused.

### Where it failed, which is the interesting part

Aggregate 55% would be a boring failure. Split by domain, it stops being boring:

| Category | Coverage | Read |
|---|---|---|
| Nordic tech | **8/10 = 80%** | at the bar — the premise holds here |
| Nordic government | 4/7 = 57% | primary sources crawl well; announcement-day news gaps |
| EU policy | 6/12 = 50% | institutional bot-walls bite hard |
| Nordic business | 4/11 = 36% | the paywalled-dailies beat; open substitutes too thin |

And when the pipeline did find a document, it ranked it well: of 22 hits, **16 at rank 1**, three at
rank 2, two at rank 3, one at rank 9. This is not a system with a ranking problem. It's a system
with a *possession* problem — it either had the document and put it first, or never had it at all.

Which the miss-cause autopsy confirms. Every failed key source got a cause, adjudicated against the
crawl manifest, so "the crawler never saw it" and "the crawler saw it and retrieval buried it" are
distinguishable rather than a matter of opinion. Across 19 failed essential sources:

- **Crawl gap — 11.** Dominated by structural walls, not crawler bugs: public broadcasters with no
  crawlable archive (sitemaps covering ~48 hours, or JavaScript shells), bot-walled outlets, and
  robots-blocked pagination on an EU institutional site. I obey robots.txt unconditionally, so a
  disallowed archive is simply a document I do not have.
- **Retrieval gap — 5.** Two reproducible engineering failures, both mine. One known-indexed
  document — three variants of it in the index — never surfaced for the query it directly answers.
  And cross-lingual retrieval underdelivers: English queries never returned Nordic-language
  documents, even where the carrying document was verified present. The multilingual-embedding bet
  needs more than parsing queries in two languages.
- **Registry gap — 3.** Sources I simply didn't include: no primary case-law source, no coverage of
  two agency newsrooms. Named fixes, not mysteries.

So of the 19 misses, **11 are access, 5 are my engineering, 3 are my curation**. The single largest
cause is documents I am structurally not permitted or not able to fetch.

### What the baseline's 100% actually measures

Perplexity's panels averaged around ten sources with unrestricted outlet choice, and in 16 of 56
cases it returned the literal canonical URL from my answer key. That is the whole-web advantage,
measured honestly, and it is a real advantage — not an artifact of the metric.

But notice what it *isn't* evidence of. It is not evidence of better retrieval engineering. It's
evidence of a larger, less restricted index: crawling infrastructure at a scale one person cannot
replicate, plus commercial arrangements and reach into content my crawler is turned away from. The
gap between 55% and 100% is mostly a gap in **access**, and only partly a gap in **algorithm**.

## Part three: is the verified brief actually usable?

Retrieval passed the bar in exactly one vertical, so the synthesis test runs there: the eight
Nordic-tech questions the pipeline retrieved well, on the frozen corpus. For each one, the system
drafts a cited brief in which every claim cites exactly one retrieved document — enforced by
schema, not by prompt, because a claim citing two documents has no single source to be judged
against. The draft is decomposed into atomic claims, and every claim is judged against the full
text of the document it cites, by the same judge that passed Part One — GLM-5.2 by default, with
the second-family escalation and the never-upgrade rule from the production design. A claim the
judge calls partial gets rewritten down to what the source supports and re-judged, once. Anything
the judge won't back is dropped. Nothing unverified reaches the page.

The verification machinery did what it was built to do. Across eight briefs, **145 claims were
judged: 121 kept, 13 narrowed and re-judged, 11 dropped as unsupported** — and one dropped
because its judgment failed outright, which is the correct failure mode: a claim that cannot be
verified does not ship. The load-bearing escalation fired on 51 judgments and the second judge
agreed with the first on 50 of them, overturning none. The whole slice, including every failed
and repeated run, metered **≈ €2.5**.

Whether the resulting briefs are *usable* is being answered blind as I write this: external
evaluators, each seeing both systems' answers unlabeled with A/B balanced exactly 50/50, every
question judged by three people. The bar, set in advance: usable-or-better on at least 6 of 8
questions.

The verdict: **one of eight.** Against a bar of six, the sovereign brief was judged usable or
better on a single question — and that one was also its only strict win; there was not a single
tie in all 24 judgments. Pooled across judgments rather than questions, the sovereign brief was
preferred 33% of the time. Failed, and not narrowly.

The one win deserves its own sentence: it was the **shortest sovereign brief of the eight** — 99
words, the one where a claim had been dropped because its judgment failed. The only brief that
won is the one that came closest to the format the readers wanted.

### The confound I have to declare before the number arrives

Whatever the verdict says, one thing about this comparison was discovered too late to fix and
must be said first: **the two systems were not answering in the same format, and the questions
favour one of them.**

The eight questions come from the retrieval benchmark, where they were built to test whether a
crawl surfaces key sources — single-fact questions with crisp answers, "for how much did Workday
agree to acquire Sana?" My spec then requires a 200–400 word cited brief as the output. The
baseline, free to answer however it likes, answers six of the eight in **22–38 words** — one or
two sentences, because that is what the question actually asks for. The sovereign briefs average
146 words against the baseline's 69. A question whose true answer is "$1.1bn" does not want a
brief, and the eval never holds format constant.

The panel made the mechanism unusually legible. Three of four evaluators swept their entire
packet in one direction — and their blind comments named coherent, opposite preferences: two
wanted a short answer to what felt like a googled question and took the baseline twelve times out
of twelve; one wanted prose with sources to fall back on and took the sovereign brief six out of
six. The fourth judged question by question, noted that *both* systems answered everything
correctly, and still broke four-to-two for the baseline — the strongest single data point that
the gap is reading experience, not accuracy. One of the sweepers added, about the long answers,
that the ones that worked had flow and the rest were *stilted* — and the stiltedness is mine, and
structural: claims are written to be judged atomically, one verifiable assertion per sentence,
and it reads exactly like that. Verification-shaped prose is real prose someone has to want to
read, and this pipeline does not yet produce it.

Neither of those is a finding about whether EU-hosted models can verify claims — Part One
answered that, and the 145-claim run backs it. But both belong in any honest account of whether a
*verified brief* beats a *fluent answer* in front of a real reader, and the fair test — same
question types, same format constraints, prose written for reading and then verified — has not
been run yet. This benchmark's contribution is the machinery and the confound, named.

## What the three results say together

Line them up.

**Verification** — the part I expected to be the hard sovereignty problem, where a European stack
should lose to American frontier models — passed at parity, at a fifth of the cost, on an
adversarial set. Not uniformly: one of the four candidates failed it dangerously, and cheaply
detectably. But the capability is there for the taking.

**Retrieval** — the part I treated as plumbing — failed, and failed hardest exactly where the open
web is least open: paywalled business dailies, bot-walled institutions, broadcasters without
archives.

**Synthesis** — the part I thought was a rendering detail — failed too, and failed on how the
answer reads rather than on whether it was true. Readers who agreed both systems were correct
still chose the other one.

**Not one of the three outcomes turned on model quality.** The model was the easy part. What
decided the other two was the world around it: which documents you are permitted to hold, and what
a particular reader wants an answer to look like.

That reframes the whole build. If the binding constraint were model quality, the answer would be to
wait — open weights improve monthly and the gap closes on its own. It isn't, so waiting buys
nothing. The documents live behind robots.txt directives, paywalls, JavaScript shells and 48-hour
archives; those are commercial and legal boundaries, and they move only through licensing,
partnerships and negotiation. The reading experience moves through product design. Both are slower
and much less glamorous than model selection, and both are where the actual leverage sits.

Two secondary findings are nearly as useful. **The corpus premise survives per domain, not in
general** — the one category where the curated corpus was deep, open and archive-accessible hit the
bar exactly. "A curated crawl can surface the key sources" is not true as stated and not false as
stated; it's true *for a vertical you have chosen carefully and can actually reach*. A vertical
strategy is not a smaller version of a general strategy — it's the only version that works. And
**verified claims are a substrate, not a format.** The same judged sentences can render as a
one-liner, a brief, or a report section, each inheriting its verification. A fluent one-blob answer
cannot be re-rendered with any guarantee at all, because nobody knows which sentence is backed by
what.

## What it cost

Every model call is metered. Measured, not estimated:

| Layer | Cost |
|---|---|
| Crawl — 24,352 documents across 43 sources | €0 metered (HTTP only) |
| Embedding — 87k chunks, backfill + refresh | €0.61 |
| Retrieval — 40 benchmark questions | €0.027 (≈ €0.0007 per question marginal) |
| Verification — the four runs behind every judge number above | €2.82 |
| Verification — seven earlier prompt-iteration runs | ≈ €2.80 |
| Amortized worst case per question (whole corpus ÷ 40) | ≈ €0.016 |
| Synthesis + per-claim judging — 8 briefs incl. smoke runs and failures | ≈ €2.5 |
| **Total metered API spend to date** | **≈ €8.8** |

Flat costs on top: a managed Postgres instance at roughly €12/month (covered by free credit), and
an existing Perplexity Pro subscription for running the baseline.

Splitting the verification row is deliberate. €2.82 covers the four runs on the frozen prompt that
produce every judge number in this post; those reconcile to the cent against two independent
accounting methods. The rest is what it cost to get the prompt and the verdict taxonomy right —
roughly as much again, spent on runs whose results I then discarded. That ratio is the honest
picture of this kind of work, and it is the line most benchmark write-ups quietly omit.

Every spend cap I set went unbound. The interesting line is the third one: at this corpus size,
retrieval is **effectively free** — a twentieth of a cent per question marginal, under two cents
fully amortized. Verification, at €2.64 per thousand judgments, is the cost centre, and it's still
small. Whatever kills a project like this, it isn't inference cost.

## What I didn't test

The pipeline never ran unattended. The original plan had two weeks of scheduled runs on a hosted
box, to see what breaks when nobody is watching — feeds that rot, sites that restructure, tokens
that expire, the ordinary decay that turns a working demo into a broken one. That didn't happen,
and I'm not going to imply it did. Everything above is measured on a laptop driving European
infrastructure, over a frozen corpus snapshot.

Also untested: any language beyond Swedish and English, any domain beyond Nordic tech/business and
EU policy, and interactive latency.

## Verdict

One goal of three passed. Here is the decision that evidence actually supports, which is narrower
and more specific than "sovereign AI works" — and more useful.

**The verification core is a go.** EU-hosted open models judged claim-against-source at frontier
parity on an adversarial set, and then did it 145 times in production shape without letting one
unverified sentence through. Every claim that shipped carries the document, fetch time, and
content hash behind it. That capability — every claim carries its evidence — was the thing in
doubt, and it is buildable on sovereign infrastructure for a fifth of frontier cost.

**Retrieval is a go per vertical, and the roadmap item is a contract, not a crawler.** The
curated-corpus premise held exactly where sources were dense, open, and archive-accessible, and
failed where they were walled. The work that changes that is licensing and partnership — slower
and less glamorous than model selection, and where the leverage actually sits. A vertical whose
key sources cannot be reached should not ship at all.

**The fixed-format brief is a no-go — and the substrate it was rendered from is the asset.** What
failed the blind eval was a reading experience, judged so by readers who found the content itself
accurate. The same verified claims can render as a one-line answer, a brief, or a report section,
each inheriting its verification — a fluent one-blob answer cannot be re-rendered with any
guarantee at all, because nobody knows which sentence is backed by what. Format is a downstream
choice. Verification is format-agnostic. That inversion is what this benchmark leaves behind.

## What's published

The 40 research questions, the 55 judgment pairs, and the full scoring protocol — pre-registered
thresholds, gold-lock dates, the equivalence-class definition, the adjudication conventions and the
escape-hatch record — are published so the benchmark can be argued with, at
[github.com/stevewerme/sovereign-research-benchmark](https://github.com/stevewerme/sovereign-research-benchmark).

The judgment pairs carry the source URL and a SHA-256 of the extracted text rather than the text
itself: several of these outlets explicitly refuse AI training, and republishing 55 of their
articles to prove a point about verification would be a poor way to make it. Re-fetch, hash, and
you can confirm you are judging the same bytes I did.

The source registry stays private. Everything here — essay and data — is CC BY 4.0.

## What comes next

The retrieval failures I can fix are the small half: a reproducible ranking bug, cross-lingual
recall that needs more than parsing queries in two languages, three named sources missing from the
registry. The half that decides whether any of this becomes a product is the one no amount of
engineering touches — the documents behind paywalls, bot-walls and 48-hour archives move through
licensing and partnerships, not code.

If you're working on sovereign retrieval, or on licensed access to Nordic sources, I'd like to
compare notes: **steve@werme.io**.

---

*Corrections are made in place and logged below with dates. Disputes about the numbers are best
raised as issues on the benchmark repo, where the data is.*

---
