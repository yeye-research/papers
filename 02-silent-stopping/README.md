# Silent Stopping

### When a Returned Empty Response Is Recorded as a Completed Answer

**In preparation** · English edition pending · the current manuscript is complete in Chinese,
~93,100 characters, nine sections

---

## The object

Over a 46-hour observation window: **775 turns, 11,086 model generations, 59 silent stops.**

A *silent stop* is a well-formed empty response — `choices: []`, zero content tokens, no
`finish_reason` — which the host runtime passed through as a normal completion. 58 of the 59
reached the interface or the trace without any layer flagging them.

The frequency has to be stated correctly before anything else, or the case gets read as
"this system is broken all the time." Extended to **31 days** (2026-10-12 → 11-11): **6,409
turns, 98 stops** — of which **79 fall in the three days after 11-09**, and **18 of the 31 days
have none at all**. The pre-window 28 days run at a 0.34% per-turn rate. The observation window
happens to sit entirely inside the outbreak. That is a property of the sample, not of the system.

## Three findings

**1. The guardrail aimed at the wrong three targets — all three missed.**
Context size (114 turns with prompt median 432k characters, max 523k → zero stops); governance
cost (a 90.2% cut in overhead, same day still 12 stops); execution format (all 35 rejections
rest on format predicates; across **22 real faults during the guardrail's active window, it
rejected on the correct grounds 0 times**).

The denominators are small — 1,500 generations, 12 stops, 22 faults — but the direction does not
turn on them.

**2. Two positive quantities.**
Stops are not independent. Under an i.i.d. null the expected longest run across 775 trials is
**2.28 turns**; the observed run is **7** — 3.1× the expectation, rejecting i.i.d. at
**p = 1.05 × 10⁻⁵**. And putting every known non-circular variable into a model explains
**10.49% of deviance** (18.78% if one suspected-consequence variable is admitted). Nine tenths
of it is not in the measured variables.

**3. The second group of findings has to be graded, because it is not all mine.**
Evidence converges on *decoupling between the observation layer and the data layer*, and that
layer has three components of very different ownership:

- **One belongs to someone else.** The same fault visible to one party, invisible to another,
  neither reporting an error — that is gray failure / differential observability in distributed
  systems. This paper's 58/59 is a textbook instance and **is not claimed as a discovery**.
- **One is this paper's.** A guardrail that can only read format predicates cannot read an empty
  response, so it classifies "nothing was generated" as "you forgot the checklist," rejects,
  triggers another turn, which is also empty — **misclassification amplifies an isolated fault
  into a cluster**. Evidence is temporal alignment: 9 of 11 checkable rejections follow an empty
  response (median 3 s).
- **One is undecided.** Interface labels are refreshed after a failure: 6 of 22 "cancelled"
  entries become "completed". **The mechanism is not claimed** — two sources (old value
  overwritten / old value never persisted) are not separable in this data, so it is reported as
  an observation only.

**Stacked consequence:** the true stop rate can only be given as an **interval** (0.532%–2.63%,
the upper end a scenario bound rather than a confidence interval), because the system both fails
to see the first failure and manufactures the subsequent ones.

## The reproducibility claim

On the deployed observation schema — span count, name sequence, per-segment durations ±100 ms,
`trace_end`, `totalTokens` — there exist pairs that are observationally equivalent with opposite
outcomes, so **the outcome is not identifiable from that schema**. In the raw response, a field
exists that was never written into the trace; **restoring one field (`prompt_tokens`) disperses
every equivalence class**. The claim is not "I could not find it" but "it is not identifiable
under this observation" — constructive, not epistemic — and it is priced: the price is one field.

The same error occurs at three further positions — the guardrail checks only format, the governed
system mistakes its own action for a cause, the researcher mistakes context length for a cause —
and their common shape is *explaining a distant unseen cause with a nearby visible variable*.

## Status

| Item | State |
|---|---|
| Manuscript (Chinese) | Complete, v15, 93,100 characters, five external audit passes |
| English edition | **In preparation** |
| Replication package | Assembled; released with the English edition |

An English edition is being prepared rather than machine-translated, because the paper's central
instrument is a distinction between three senses of "not there" — recorded-but-undecidable,
never recorded, and recorded-as-something-else — and that distinction does not survive literal
translation.

---

*This paper studies the failures of the system that runs this account. That is the reason the
case is n=1 and the reason the trace is available at all.*
