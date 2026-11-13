# Unidentifiability of Failure Root Causes Under Single-Machine Conditions

### A Deep Investigation of 129 Silent Stops in a Locally Deployed LLM Service

**Abstract only** · the full report (Chinese, ~17,000 characters, eleven sections) is **not released**.
What is public here is the abstract. Every number below is taken from the frozen data snapshot
described at the end of this page.

> 本页只放摘要。中文全稿（约 1.7 万字符、十一节）**未公开**。

---

## Abstract

**Problem.** Over three and a half days (9–12 November 2026), a locally deployed LLM service produced 17,569 generations. In 129 of them the model returned nothing to the user: the response was HTTP 200 with `status: ok`, but `choices` was an empty array and the body contained zero tokens. No error was raised and no retry occurred. We ask a single question: under single-machine conditions — no control group, no fault injection, server internals invisible — **can the root cause of these 129 silent stops be uniquely determined from the observable data?**

**Method.** Every generation was flattened into 15 observable columns (tokens, cache hits, reasoning share, latency, prior tool span, `finish_reason`, status). We then stratified by two candidate conditions — whether a reasoning channel was used, and context length — re-clustered the 129 events at four temporal thresholds, and built a fault-signature matrix over 17 candidate causes following structural diagnosability. All reported figures were recomputed by an independent script that re-scans the raw traces without reading any intermediate table.

**Results.** Silent stops are not uniformly distributed. They require both conditions: a reasoning channel *and* a context of at least 300k tokens.

| | reasoning used | no reasoning |
|---|---:|---:|
| **context ≥ 300k** | **128 / 7,223 (1.77%)** | **0 / 3,414 (0%)** |
| **context < 300k** | 1 / 4,672 (0.02%) | 0 / 2,234 (0%) |

Length alone is **not** causal. The rate is non-monotonic (3.41% at 325–350k, 3.56% at 400–425k, 0.97% at 475–500k) and **falls to 0 / 449 above 500k tokens**. Seven competing explanations were eliminated by data, including budget exhaustion (completion median **324** against a demonstrated ceiling of **13,236**), streaming truncation, usage-only tail chunks, proxy corruption, cache misses, timeouts, and "body tokens counted as reasoning" (normal responses: `completion − reasoning` minimum **2**, exactly zero in **0** cases).

**The two halves of silence.** A silent stop needs both an empty response *and* a host that does not act on it. The second half has direct evidence: **129 / 129** stops are the last generation in their trace, with **no subsequent retry of any kind**.

**Boundary — what remains unidentifiable.** Why no body token was ever generated after reasoning ended. Three mechanisms — internal abort, post-hoc filtering, and an independent reasoning cap — produce identical observable fingerprints in `usage` (body = 0, all tokens in `reasoning`, no `finish_reason`). Separating them requires a `finish_reason` that does not exist when `choices` is empty, or server-side logs the client cannot see.

**Why we stopped.** Replay experiments cannot help: at a base rate of **0.734%**, twenty replays yield an all-negative outcome with probability **0.863**. And "0 of 5,648" supports only an upper bound — a true rate at most **0.053%** under 95% one-sided confidence, i.e. at least 20× lower than the reasoning group, not "never happens".

**Implication.** The investigation yields three things that can be handed over: the conditions under which the service returns empty `choices`; a repairable half of the problem on the client side (retry and alerting); and an explicit statement of what additional observation would be required to close the gap — the single cheapest being a top-level `finish_reason` on empty-`choices` responses.

**Keywords:** unidentifiability; silent failure; LLM service reliability; single-machine observation; structural diagnosability

---

## Data snapshot

Frozen 2026-11-13 17:56 (gate 2), fingerprint `60f89a50bbef…`:

| item | value |
|---|---|
| trace files | 6,982 |
| generations (whole archive) | 110,013 |
| empty-`choices` events (whole archive) | 148 |
| window used in this report | 2026-11-09 – 2026-11-12 |
| generations in window | 17,569 |
| empty events in window | 129 |

The difference (148 − 129 = 19) lies outside the four-day window. Both figures are correct;
what differs is the window, not the criterion — verified earlier against five windows
(59 / 78 / 83 / 84 / 100), where **all** variation came from window endpoints.

## Reproducibility

Three scripts, each with a built-in self-test, plus an independent recomputation that
re-scans raw traces without reading any intermediate table (22 / 22 assertions matched):

| script | self-test | what it does |
|---|---|---|
| `_停机组特征.py` | 4 / 4 | flattens every generation into 15 columns |
| `_静默双侧.py` | 4 / 4 | reasoning × length stratification; retry check; 25k-bin scan |
| `_独立复算3_第3篇.py` | 4 / 4 | re-scans traces from scratch, compares 22 claims |
| `_开题三检.py` | 5 / 5 | length bins, budget-ceiling test, day × bin |

---

*2026-11-13. Paper 5 of the series. Previous: **How Many Times Does One Incident Count?** (paper 4).*
