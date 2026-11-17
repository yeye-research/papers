# The Attribution Ceiling on a Single Machine

### How Far Silent-Failure Causation Can Be Pushed From Traces Alone

**Abstract only** · the full report (Chinese, ~24,000 characters, nine sections) is **not released**.
What is public here is the abstract. Every number below is taken from the frozen data snapshot
described at the end of this page.

> 本页只放摘要。中文全稿（约 2.4 万字符、九节）**未公开**。

---

## Abstract

**Problem.** A single personal-assistant agent runtime ran from 2026-10-15 to 2026-11-16 and produced 148 empty generations: calls that completed normally but returned no content. For a system observed only from the outside, the practical question is not which component failed but how far attribution can go. This report asks two things. Does the temporal shape of the 148 events favour a few bad periods over one failure triggering the next? And given that shape, how many candidate causes can be raised above mere co-occurrence with the traces at hand?

**Method.** Every generation recorded in the window was extracted from the execution traces. The trace archive is a growing store, so the count is taken as one snapshot of the window rather than as a running total: **117,562 generations, of which 0.1259% returned an empty choice set**. Numerator and denominator come from the same snapshot. Events were placed on a local-time axis and described at three resolutions: by calendar day, by hour inside the peak days, and by conditional intensity, defined as the probability of a further event within a given number of minutes divided by the Poisson expectation over the same interval. Candidate causes were then graded by whether an identification strategy exists. Grade A means a contrast or a counterfactual was available. Grade B means co-occurrence only. Grade C means no usable data.

**Results.** The shape is concentrated rather than self-exciting. Three days account for **127 of the 148 events, or 85.8%**. Their daily rate is **0.8366%** against **0.0205%** over the other 33 days, a factor of **40.8**. Inside the peak day, a single hour reached **31 of 136 generations, 22.8%**.

**How much of that concentration is the slicing (revision).** The three days are 2026-11-12, 11-09 and 11-10 — **not contiguous**: 11-11 sits between them with only 2 events. Under the three ways of cutting, enrichment stays large but the headline percentage does not:

| slicing | days taken | events | share | enrichment over days |
|---|---|---:|---:|---:|
| top three days by event count | 11-12 / 11-09 / 11-10 | 127 | **85.8%** | **10.3×** |
| best **contiguous** three-day window | 11-10 … 11-12 | 82 | 55.4% | 6.6× |
| best **equal-width** three-day bin | 11-09 … 11-11 | 80 | 54.1% | 6.5× |

**The concentration is robust (6.5–10.3× under all three); the 85.8% is not, and is quoted only with its slicing.**

The 148 events are **not 148 retries of identical input**: their prompt-token counts take at least **142 distinct values**, and of **73** adjacent pairs less than two minutes apart only one shares an input. That excludes byte-identical resends only; a same-session rewording would change the token count and **was not checked**. Workload does not explain the pattern: two days differing by **10.4%** in volume differ by a factor of **42.6** in rate. The rank correlation between a day's generation count and its failure rate is **−0.165** over the 14 days that had events (**not significant at n = 14**: t = −0.58, 95% CI [−0.64, +0.40]) and **+0.051** over all 36 days. Context length, the most plausible candidate, survives only weakly under same-hour self-matching: **35.1%** higher when the two medians are compared directly, and a median of **1.054** across within-hour ratios after pairing. **The two figures are different statistics and cannot be subtracted from one another**; both point the same way. The direction is real, with **70.9%** of events above their own hour's median against **49.9%** for controls, but the magnitude is small. Graded by identification strategy, **0 of 5 candidates reach grade A**.

**Boundary.** The study covers one machine, one operator and one window of 33 days; it generalises to no other system. No intervention, no replay and no interview were available, so the design is observational throughout. Conditional intensity decays across several hours, which is compatible both with a long-memory excitation kernel and with slow heterogeneity; the two are not separated here. A single candidate change point exists in the record, and concurrent task drift cannot be ruled out around it. Nothing in this report identifies a root cause, and the grading is not a ranking of plausibility.

**Implication.** Two things transfer even though the numbers do not. First, a conditional rate can replace a raw count once the denominator is measurable; on this machine the raw count and the rate told different stories about whether busy days are worse. Second, the ceiling of attribution under non-intervention can itself be measured, by grading every candidate against the identification strategy it would require. A report that stops at co-occurrence is more useful to an outside investigator than one that names a cause it cannot support.

**Keywords:** silent failure; empty generation; temporal clustering; conditional intensity; attribution ceiling; case-crossover; single-case design

---

## Data snapshot

Frozen **2026-11-17 19:08** (gate 2, against a content-frozen copy of the archive), trace fingerprint `ebc7aae7…`:

| item | value |
|---|---|
| trace files (frozen copy) | 7,325 |
| generations, 36 days with records (2026-10-12 – 11-17) | 117,562 |
| empty-`choices` events | 148 |
| observation window used for rates (calendar) | 2026-10-15 – 2026-11-16 (33 days) |
| event span, first → last | 2026-10-16 15:29 → 2026-11-16 14:56 (743 h / 31.0 days) |

Three different counts of "days" appear above and they are not interchangeable: **36** = calendar days
that carry trace records; **33** = the observation window used for daily rates; **31.0** = the span between
the first and the last event, which is the denominator behind the conditional-intensity rate. The 148 events
all fall inside the 33-day window; the three days outside it carry zero events.

Two archive totals also exist and differ by construction: **117,562** (all generations in the 36 days with
records) and **118,902** (all generations in the frozen copy, which also holds rows outside those 36 days).
Figures in this abstract use 117,562 only.

## Relation to paper 05

Paper 05 asked the same class of question on a **four-day window** (2026-11-09 – 11-12) and reported
**129** events out of 17,569 generations, with a base rate of 0.735%. That paper and this one share the
archive but not the unit of analysis: 05 conditions on a short burst and asks whether a root cause is
identifiable **inside** it; this report takes the **whole 36-day record** and asks how far attribution can
go **at all**, including whether the burst shape itself is an artefact of slicing. The archive has also
grown between the two freezes (110,013 → 118,902 generations), so the totals are not comparable either.
Read 05 for the fault-signature matrix on the burst; read this for the ceiling.

## Reproducibility

Numbers come from scripts that re-scan the raw traces. Four of them carry built-in self-tests; **four do not**,
and that gap is stated rather than hidden:

| script | self-test | what it does |
|---|---|---|
| `_独立复算_第3篇.py` | yes | re-scans traces from scratch, checks 14 claims against the text |
| `_token去哪了.py` | yes | token accounting: completion vs reasoning, prompt medians |
| `_日粒度率.py` | yes | per-day generation and empty counts; the 36-day denominator |
| `_台账与trace对账.py` | yes | reconciles the hand ledger against the traces |
| `_等宽稳健性.py` | **no** | the three slicings in the table above |
| `_上下文暴露.py` | **no** | same-hour pairing; the paired ratio distribution |
| `_聚集判别.py` | **no** | conditional intensity; Poisson baseline over the event span |
| `_重试链.py` | **no** | retry-chain check; the 73 adjacent pairs |

The frozen copy of the archive is content-stable, so a re-run reproduces the same numbers; a re-run against
the live archive does not, and the report never quotes live-archive totals without their timestamp.

---

*2026-11-17. Paper 7 of the series. Companion to **Unidentifiability of Failure Root Causes Under Single-Machine Conditions** (paper 5).*
