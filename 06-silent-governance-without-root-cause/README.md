# Silent-Failure Governance Without Root Cause

### Making a Failure Visible When Its Cause Cannot Be Identified

**Abstract only** · the full report (Chinese, ~14,000 characters, nine sections) is **not released**.
What is public here is the abstract plus the counts behind it. Companion to
[paper 05](../05-unidentifiability-single-machine/), which established that the root cause cannot be
identified; this one asks what follows from that.

> 本页只放摘要。中文全稿（约 1.4 万字符、九节）**未公开**。本页是第 05 篇的续篇。

---

## Abstract

**Problem.** A locally deployed LLM service produced 17,569 generations over four days
(9–12 November 2026). In 129 of them the model returned an HTTP 200 response carrying a complete,
well-formed JSON object whose `choices` array was empty: every completion token landed in
`reasoning_tokens`, no `finish_reason` was emitted, and the caller treated the turn as a normal end
— no retry, no alert, no error. Paper 05 established that the root cause cannot be identified from
single-machine data. This report asks the engineering question that follows: **if the cause cannot
be known, what can still be done?**

**Method.** All figures were recomputed by an independent script that re-scans the raw traces
without reading any intermediate table, with a 13-case self-test on synthetic data and **63
assertions** matched one-to-one against the manuscript. Candidate design principles were taken from
twenty results in other disciplines; each was checked against its **preconditions on this machine**
rather than against whether anyone had applied it before. Of the twenty, **six are hard**
(preconditions verified, replicable), **three semi-hard** (verified but parameter-unknown or
upper-bound only), **three soft** (no evidence on this machine), and eight analogy-only or
inapplicable. Only the nine hard and semi-hard ones enter the criteria.

**Results.**

**(i) Not noise but outbreak.** The 129 events are strongly clustered: 46 fall inside a 78-minute
window against a uniform-rate expectation of 1.75, **P ≈ 10⁻⁴⁷**. The conclusion survives
compressing the observation window to 8 hours per day (P ≈ 10⁻²⁷) — it would take a window near one
hour per day to reverse it.

**(ii) The same finding damages the sample.** Re-clustered at a 10-minute threshold, 129 events
reduce to **26 independent occurrences**:

| threshold | 1 min | 5 min | 10 min | 30 min | 60 min |
|---|---:|---:|---:|---:|---:|
| clusters | 72 | 38 | **26** | 13 | 11 |
| largest cluster | 7 | 22 | **46** | 47 | 47 |

So "129" can no longer be used as a sample size — a point that also revises a figure in paper 05.

**(iii) The cost is not monetary.** Silent stops account for **0.918%** of prompt tokens
(50,718,041 of 5,527,051,094) and the stop cohort hits 99.7% cache — but its median prompt is
**401,863 tokens** against a global mean of 314,591. The stops strike precisely the expensive,
reasoning-heavy turns.

**(iv) Avoidance works, but only as a measure.** Not using the reasoning channel gives
**0 / 5,648** stops. The reduction is **33.4×** if generations are assumed independent and
**3.4×** if they are counted per session (580 sessions). Both must be reported together: the first
divides a point estimate by an upper bound computed under an independence assumption the data do
not satisfy; the second is the conservative reading.

**Design.** One main line plus two fallbacks, **in sequence rather than in parallel**. The main line
is avoidance (shorten context below 300k, or close the reasoning channel on long inputs). The first
fallback is a **heterogeneous** retry — not a verbatim resend, which the common-cause-failure
literature places on the same failure path — with a back-off of at least 78 minutes matching the
observed outbreak window. The second is an alert that **carries a recovery action** and, if a human
is to see it, a canary injection, because successful avoidance makes the alert rarer (1 in 1,883)
and therefore more likely to be missed.

The governing principle is the **complement criterion**: require a liveness proof every turn,
rather than enumerate what a failure looks like. Six concrete attachment points are identified, all
on components that already exist locally; none requires new tooling.

**Boundary.** Ashby's law of requisite variety gives V(regulator) = 3 < V(disturbance) ≥ 4, so "no
misses" is not attainable; the design can guarantee **visibility and recoverability**, not
prevention. The "≥ 4" is not a guess — on 10 November, the ≥ 400k bin produced **0 / 1,161** stops
while the 300–400k bin produced **31 / 1,055 (2.9%)** on the same machine on the same day.
Avoidance rests on a statistical conditional model, not a mechanistic one, and lapses whenever the
condition distribution drifts.

**What is not established.** To certify that the post-fix rate is below 0.735% requires **407
all-negative independent observations** (406 under the exact rate 129/17,543); the machine currently
has **87** — the threshold cannot be met with existing data and is a target, not a result. Two of
the three proposed validation tests cannot yet be run at all: the quality cost of disabling
reasoning has not been measured, and no alert channel is in place. The retry fallback has **no
interventional evidence** — in 129 of 129 stops no retry was observed, so the recommendation rests
on mechanism alone and is labelled as such throughout. The text distinguishes what was observed,
what was inferred, and what is only prescribed.

---

## What changed after external review

Two external readers were used, with **opposite** briefs: one was asked to kill the claims
(forbidden from proposing fixes), one was given only the data and no conclusions. Their effect,
stated plainly:

| what they attacked | verdict | what changed in the text |
|---|---|---|
| "necessary condition" rested on a zero count | **accepted** | downgraded to "threshold condition with 0 exceptions"; the upper bound (0.053% / 0.517%) is now stated beside the word |
| 26 independent events vs 87 independent observations, both called "independent" | **accepted** | §7.1 now says 87 is the cluster count of the *5,648 no-reasoning* records, and 26 is the cluster count of the *129 events* — two different things |
| "length is not the cause" | **accepted** | changed to "length alone is not sufficient" — "cause" is a mechanism word, a 2×2 table is an observation |
| "can never be reached with existing data" | **accepted** | "never" removed; now "87 < 407, short by 320" |
| 1.25 vs 401,863 / 314,591 | **rejected** | 1.25 is 0.918% ÷ 0.735% (two *rates*); the median/mean ratio is 1.28 and is now reported separately |
| 8 h/day as evidence | **rejected** | that section is explicitly a *sensitivity* table ("what assumption would be needed to reverse this"); the first row is the measured window |
| 129 treated as an exact count | **accepted** | 26 records with missing fields are of unknown status, so 129 is a **lower bound** and 155 an upper bound; all rates use the classifiable 17,543 as denominator and the text states the rate upper bound (0.883%) |
| median ÷ mean presented as a ratio | **accepted** | now labelled a magnitude hint only; it cannot be read as "the stop cohort's mean is 1.28× the overall" |
| the outlier P-value was computed on a window chosen after seeing the data | **accepted** | the window is the maximum over **129 sliding starts**; Bonferroni-corrected it moves from −47.3 to **−45.2** and still rejects uniformity |
| the window may simply have carried more traffic | **rejected by data** | the window had **171 generations in 76.5 min = 2.2/min** against a global 3.1/min — **0.73×, i.e. sparser** |
| 46 events may be repeated counts of one cluster | **rejected by data** | inside the same window the risk cell holds **46 stops out of 132 (34.8%)** against 1.77% globally — **19.7×** |

A separate literal pass (zero author memory) found twelve items: **seven real** (a table cell
holding the column total, 5,648, where the bin held 3,414; "two orders of magnitude" for a 20× gap;
"9 hard + 3 semi-hard = 9" then "plus half a criterion"; a prescription measured with a protocol
that fixes the very factor under test), two false positives, and three missing cross-references.
All seven were fixed.

## Reproducibility

Every figure recomputes from the frozen snapshot (four days of traces, 17,569 generations).

| script | self-test | what it does |
|---|---|---|
| `_独立复算4_第4篇.py` | 13 / 13 | re-scans traces from scratch, no intermediate table; **63 assertions** |
| `_静默双侧.py` | 4 / 4 | reasoning × length stratification; retry check; 25k-bin scan |
| `_聚集显著.py` | 8 / 8 | sliding window × Poisson upper tail; extension evidence; sensitivity |
| `_设计效应.py` | 8 / 8 | clustering; two upper-bound conventions; cluster-robust CI |
| `_L4_判据粗筛.py` | 8 / 8 | three-API triage of the twenty principles |
| `_投喂台岗位测.py` | 6 / 6 | by-role test of the three feeder APIs (transcription / screening / open critique) |

Delivery gate at freeze: `sha16 = 3a1661a48a32e8f7`, literal-review tickets
`DSH-20261114-003` / `-004`, eighteen remediation items recorded, no re-review (remediation is
recorded instead of re-sending — see the series conventions).

---

*2026-11-14. Paper 6 of the series. Previous: **Unidentifiability of Failure Root Causes Under
Single-Machine Conditions** (paper 05).*
