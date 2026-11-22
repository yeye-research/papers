# The Shape That Cannot Be Decided

### Whether Cross-Session Misses Are a Load Problem or a Stage Problem — and Why Five Records Cannot Tell

**Abstract only** · the full report (Chinese, ~8,900 characters, seven sections plus an appendix) is
**not released**. What is public here is the abstract. Every number below is taken from the frozen
data snapshot described at the end of this page.

> 本页只放摘要。中文全稿（约 8,900 字符、七节加一个附录）**未公开**。

---

## Abstract

**Problem.** A single-operator agent working across sessions cannot carry forward what it learned in the previous session. Four concrete symptoms recur: a tool that worked smoothly in one task feels unfamiliar in the next; long procedures lose steps near the end; an existing tool is overlooked and a new one is built first; and commands already written into a file are received but not executed. Handover notes, closing checklists, and daily memory logs have been added, and each helped, yet the failures continue. A prior judgement frames the question: errors introduced by remediation outnumber the errors of the original framework, and each fix tends to manufacture a new error. This report therefore does not ask how to make the agent remember better — that would be a proposal, and this report proposes nothing. It asks a prior question: are these failures a systematic break at one stage, or an exceeded total load?

Three established lines each address one stage only. Cross-session memory degrades: commercial assistants and long-context models show a 30% accuracy drop on a 500-question long-dialogue benchmark. Context compression retains on average only 17% of side constraints, and 30 of 100 sessions showed unapproved message deletion. Standardized handover reduced medical errors by 23% and preventable adverse events by 30% across 10,740 cases; yet one RCT reported 4.8% versus 4.1% with no significant difference, a systematic review found 8 of 26 outcomes significantly improved, 11 described as improved without statistical testing and 6 unchanged (8+11+6=25, one short of the stated 26), and none of four studies showed improvement in mortality. Each of these measures a single stage; none maps where failures fall along a whole workflow.

**Method.** Five failures recorded during one real work session (from 22:5x on 22 November to 01:5x on 23 November 2026; both endpoints are read to the nearest ten minutes, a span of roughly two hours and fifty minutes), each traceable to a timestamp and to what was missed, were classified twice using two independent schemes. The material supplied to the second classifier contained no terms from the first scheme; it constructed its own dimension of "information-processing stages".

**Results.** Under one scheme the five failures fell one per locus across five distinct loci; under the other, two fell in one of three classes. The two schemes thus give different pointers for the same five cases; but with five and three classes respectively, the report's own rule (equal class counts are required before any side-by-side comparison) forbids reading these two pointers as a comparison. This disagreement neither supports the two-type distinction nor refutes it. A further caveat: the load-type signature itself (failures scattered at random) may be internally inconsistent, since if stages differ greatly in exposure the failures under overload should concentrate in the high-frequency stages rather than spread evenly. Under independent uniform assignment, the probability that five items occupy five distinct loci is 3.84%, and the probability that the largest class holds exactly two is 37.04%; that some class holds at least two follows from the pigeonhole principle and is certain. Crucially, both figures require every stage to be equally likely to receive a failure — a premise with no support: the boundary section records that per-stage exposures were not counted. The two figures are therefore arithmetic performed under an unsupported premise, not properties of these data. The data cannot test the distinction.

**Boundary.** n = 5, one session spanning roughly two hours and fifty minutes, one operator, two classification schemes only, no blinded second coder, and no positive control. No empirical rate is reported, because the number of exposures per stage is unknown and the denominator cannot be fixed. One earlier claim — that output was the weak link — was withdrawn for the same reason.

**Implication.** The next step is not further classification. It is, in order: define what counts as one exposure at a stage; record per-stage exposure counts; accumulate at least 30 cases, pooled across sessions only where those denominators exist; agree on a granularity whose class count is identical in both schemes; and add a positive control set drawn from outside these data, since whether a set is concentrated is the very thing at issue.

**Keywords:** cross-session memory; cognitive load; failure taxonomy; classification granularity; negative result

---

## What this abstract rests on

| Item | Value |
|---|---|
| Failures recorded | **5**, from one continuous work session |
| Session span | 22:5x on 22 Nov 2026 to 01:5x on 23 Nov 2026, roughly **2 h 50 min** (both endpoints read to the nearest ten minutes) |
| Classification schemes | 3 independent ones — mine (5 loci), one external model (3 classes), one blind classifier (4 classes) |
| Retired claims | **22** — every one written into an immutable list before delivery |
| Frozen snapshot | 80 files, aggregate fingerprint `23fe9b46…` |
| Arithmetic | every ratio independently recomputed; the 3.84% figure also checked by exhaustive enumeration of 5⁵ = 3,125 assignments |

### What is *not* established

This paper reports a negative result, and the negative result is narrower than it first looks.

1. **No positive control.** The distinction has not been tested on data known to concentrate in one stage.
2. **No per-stage exposure counts.** Every probability quoted here is conditional on an assumption — equal likelihood across stages — that was not measured and has no support.
3. **No blinded second coder.** The five loci are my own division; nobody reviewed my assignments.
4. **The failures are self-reported.** Whether all five count as misses was not checked by a second person.
5. **One session.** No cross-session replication.
6. **The classifier that gave the load-type signature may be internally inconsistent.** If stages differ in how often they are exercised, failures under overload should concentrate in the high-frequency stages rather than spread evenly — which contradicts the signature itself.

### The methodological point

Adding a checklist is not free. If the fault is an exceeded total load, another check adds load. That is why this report asks which fault it is before proposing any repair — and why it ends by saying the data at hand cannot answer that.
