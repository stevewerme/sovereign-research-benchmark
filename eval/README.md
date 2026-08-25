# Blind evaluation artifacts

Published **after** the write-up went live, which is the point: while the evaluation was running,
the assignment map was the blinding secret. Releasing it afterwards is what makes the blinding
claim checkable rather than merely asserted.

| File | What it is |
|---|---|
| `packets/packet-ev-NN.md` | Exactly what each evaluator received — six questions, two unlabeled answers each, in the eval rendering |
| `assignment-map.json` | Which system was shown as "A" for every evaluator × question, plus the seed |
| `scoring.md` | The scored result: per-question majorities, pooled rate, strict wins |

## What you can check with these

- **That the blinding held.** No packet names either system, carries citation markers, or renders
  the two answers differently. Both were normalised through the same function.
- **That A/B was balanced, not left to chance.** The map should show an exact 12–12 split of
  which system held slot "A" — independent coin flips would have let position bias favour one
  system systematically.
- **That every question got the same number of judgments.** Each of the eight appears exactly
  three times across the four packets.
- **That the assignment is reproducible.** It was generated from the recorded seed; the same seed
  and roster reproduce it exactly.

## What is not here

The evaluators' reply files. Those contain four people's own written comments, and permission to
publish their words was never asked for — the two comments quoted in the write-up were cleared
individually. `scoring.md` carries the result their replies produced.

Evaluator identities are pseudonymous throughout (`ev-01`…`ev-04`) and the mapping to real people
exists nowhere in this repository.
