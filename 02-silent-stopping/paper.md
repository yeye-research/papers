# The Runtime Host Is Not a Transparent Pipe

### A single-case study of attribution misplaced in a self-built governance layer

**Ye Ye** · research account · November 2026

> **Working paper — not peer reviewed.**
>
> **Translation status (read this first).** This file is the English edition of a
> 95,696-character Chinese manuscript (v16, 2026-11-12). What is translated here in full: the **Abstract** and
> **§1 Introduction** (§1.1–§1.6). Not yet translated: §2 Related work, §3 Method, §4 Results,
> §5 Discussion, §6 Limitations, and Appendices A–C. The Chinese source is authoritative for every
> number; where the two differ, the Chinese wins and this file is wrong.
>
> **The one deviation this file used to carry has now been closed in the source (2026-11-12 18:3x).**
> The Chinese Abstract used to place the 79 stops in "the three days *after* 11-09", which cannot
> close the arithmetic — inside a 10-12 → 11-11 window only two days lie after 11-09, yet the same
> paragraph assumes 28 + 3 = 31. This file had rendered it as "the three days from 11-09 through
> 11-11". **The Chinese source (v16) has now been corrected the same way and states the arithmetic
> outright: "79 次集中在 11-09 起的这三天（47 + 31 + 1）".** No number changed; no deviation remains
> on this point. (Two adjacent sentences were corrected in the same pass: the conditioning-rate table's
> denominator 750 vs. the marginal rate's 775, and the high-risk window 3–8 → 1–8 turns. Both are in
> sections not yet translated here.)
>
> Nothing in this file is asserted that the released scripts cannot re-derive.


---

## Cross-reference map (read before §1)

Every `§x.y` and `Appendix x` cited in this file points into the **Chinese manuscript**
(`_论文_全文_初稿v15_分章终裁后_20261111.md`, 2,304 lines). Those targets are **not yet translated**,
so the identifiers are defined here rather than left dangling. Line numbers are the definition site
in the Chinese source.

| Cited as | Where it is defined | Translated here? |
|---|---|---|
| `§2.9` | Chinese line 265 — the "does not claim to have discovered it" statement on the mechanism | No |
| `§3.1.1` | Chinese line 283 — window arithmetic (nominal 46.5 h / 46.4 h by trace → 46) | No |
| `§3.4.1` | Chinese line 435 — attribution admission threshold ("≥2 counterexamples under the same conditions") | No |
| `§4.1.2` | Chinese line 754 — context length as a 15–18× correlate | No |
| `§4.1.3` | Chinese lines 797, 816 — the 35-rejection ledger | No |
| `§4.1.3.1` | Chinese line 816 — the gate's verdict, and the ownership statement for "classification precedes response" | No |
| `§4.2.6` | Chinese lines 219, 1272 — the 6 observationally equivalent pairs and the `prompt_tokens` restoration | No |
| `§4.2.8` | Chinese line 1582 — deviance explained (10.49% / 18.78%) | No |
| `§5.1` | Chinese line 1719 — the operational definition of attribution misplaced (three necessary conditions) | No |
| `§6.3` | Chinese line 2010 — absolute magnitudes small and highly concentrated; why the "only two days" limitation can be dropped | No |
| `Appendix A` | Chinese line 2178 — four occasions this study got its own numbers wrong | No |
| `Appendix B` | Chinese line 2207 — prospective observation after the window closed (19 more stops in 53 minutes) | No |
| `Appendix C` | Chinese line 2249 — the 31-day daily table underlying §6.3 | No |

**So: no reference in this file is a broken link — it is an untranslated target with a stated
location.** When a section is translated, its row is deleted and the citation resolves inside this
file.

---

## Abstract

AI agent systems routinely load governance — rules, checks, constraints — to control their own
behaviour. The **cost** of governance is easy to observe. The **causal assumption** behind it is
rarely tested: on what grounds do we believe that the indicator being governed stands in a causal
relation to the outcome we actually want?

This paper reports one n=1 critical case. A self-built governance system on a personal computer
loaded a per-turn automatic check (hereafter *the gate*) in order to "cure" an agent that stopped
part-way through its work. Over a 46-hour observation window (nominal 46.5; 46.4 by trace;
rounded to 46 throughout, see §3.1.1) the system recorded **775 turns and 11,086 model
generations**, of which **59 were *silent stops*** (window frozen at 2026-11-10 22:30) — the model
returned a well-formed empty response (`choices: []`, content tokens exactly 0, no
`finish_reason`), and the host runtime treated it as a success.

**This fault is not the normal state, and that has to be said first, or the case will be read as
"this system is broken all the time."** Extending the same predicate to **31 days**
(2026-10-12 → 11-11) gives **6,409 turns and 98 stops** — of which **79 fall in the three days from
11-09 through 11-11 (28 + 3 = 31)**, and **18 of the 31 days contain none at all**; the per-turn
stop rate over the preceding 28 days is **0.34%**. **The two days of the observation window
(11-09, 11-10) sit entirely inside the outbreak.** Miss that, and a single episode is read as a
standing property.

**The first group of findings is negative: all three targets of the governance missed** ("missed"
means *it cannot explain the stops*, **not** that it is unrelated to them — context length remains a
strong correlate at 15–18×, §4.1.2). Context size (114 turns with prompt median 432k, max 523k →
zero stops); governance overhead (a 90.2% cut, still 12 stops that same day); execution format (all
35 rejections rest on format predicates; across **22 real faults during the gate's active window it
rejected on the correct grounds 0 times**). The denominators of the three counterexamples are 1,500
generations, 12 stops and 22 real faults respectively — **not large, but the direction is settled,
and every one of them can be re-run.**

**Two positive quantities are given alongside.** Stops are not independent events: under an i.i.d.
null the expected longest run across 775 trials is **2.28 turns**; the observed run is **7** — 3.1×
the expectation, rejecting i.i.d. at **1.05 × 10⁻⁵**. And putting every known non-circular variable
into a model explains **10.49% of deviance** (18.78% if one variable suspected of being a
consequence is admitted, §4.2.8); nine tenths of it lies outside the measured variables.

**The second group must be stated in tiers, because it is not all this paper's.** The evidence
converges on a decoupling between the observation layer and the data layer, and that layer has three
components of quite different weight:

- **One that has an owner.** The same fault is visible to one party, invisible to another, and
  neither raises an error — this is *gray failure* (differential observability) as characterised in
  distributed systems, and 58 of the 59 are a textbook instance. **This paper cannot claim that
  component as a finding.**
- **One that is this paper's.** **Misclassification by the governor amplifies an isolated fault into
  a cluster.** The gate's predicates are format-only; on an empty response it can read no text at
  all, so it classifies "nothing was generated" as "you forgot to write the checklist", rejects,
  triggers a new turn, and that turn is empty too. **The evidence, in the order it bears weight**: the
  gate's own verdict wording — it asserts "you forgot to write the checklist" about a turn that was
  empty, and that assertion fails upstream 27 times out of 35 (§4.1.3); then, more weakly, timing —
  of 11 verifiable rejections, **9 follow an empty response within a median of 3 seconds**; the
  denominator is hard too — **22 real faults during the gate's active window, 0 rejections on the
  correct grounds**. **It is neither gray failure (that is "cannot see") nor "the gate was too
  strict" (that is sensitivity), but "it saw, it classified wrongly, and it acted in a way that
  worsened the fault".**
- **One left open.** Interface labels were refreshed after the failure (6 of 22 "cancelled" became
  "completed"). **This paper does not claim the mechanism** — a terminal-state difference has two
  possible sources (the old value was overwritten / the old value never reached the store), the data
  cannot separate them, so it is reported as an observation only.

**Compounding consequence:** the true stop rate can only be given as an interval, not a point
estimate (0.532%–2.63%, the upper bound being a scenario bound rather than a confidence interval) —
**the first failure was invisible, and the later ones were manufactured.**

On that basis the paper gives one strong, re-runnable statement: on the **deployed observation
schema** (span count, name sequence, per-span duration ±100 ms, `trace_end`, `totalTokens`) there
exist observationally equivalent pairs with opposite outcomes, so **the outcome is not identifiable
on that schema**; and among the fields already present in the raw response but never written into
the trace, **restoring one (`prompt_tokens`) separates every equivalence class**. This is not "I did
not find it" but "it is not identifiable under this observation" — constructively, as one concrete
equivalence-class construction, not as an epistemological excuse — and it is *priced*: the price is
one field.

The same error occurs once more in each of three other places: the **governor** checked only format;
the **governed** took its own action as the cause; the **researcher** took context length as the
cause. The common shape of all four is *explaining an invisible distant cause by a visible nearby
variable*, and its structural source is that **the shape of a hypothesis is fixed by the boundary of
observation**.

Contributions: falsification of three prevailing explanations; one operational definition (three
necessary conditions for *attribution misplaced*, the third being "the system cannot discover its
own error"); **a quantification of one mechanism** (amplification by misclassification — the
mechanism is not new; what is new is three numbers); and one judgement the case supports — **the
runtime host is not a transparent pipe.**

**Keywords:** governance systems · attribution misplaced · gray failure (differential
observability) · amplification by misclassification · observability · n=1 case study

---

## §1 Introduction

### 1.1 The causal assumption behind governance is rarely tested

Once an AI agent system is deployed into real work, it will almost certainly acquire governance:
constraints in the system prompt, a per-turn check script, retry rules after failure, thresholds for
human review. All of these share one causal assumption — **that some observable indicator stands in
a causal relation to the outcome actually wanted.**

The assumption is rarely tested, for a practical reason: the **cost** of governance is immediate and
measurable (one more script, one more format requirement, a few more seconds), while its **benefit**
is counterfactual and not directly observable ("would it have been worse without this rule?"). When
the cost appears daily and the benefit can never be falsified, a governance system drifts naturally
towards **adding**, not towards verifying.

This is the skipped question the study asks: **what happens if that causal assumption does not
hold?**

### 1.2 One concrete case

On 9 November 2026, an agent working system on a personal computer developed a user-visible fault:
a conversation would end mid-flight — "it just went away without saying anything" — and
progressively more often, from ten-odd seconds at first to five or six seconds later.

User and agent diagnosed it together: the agent was "stopping half-way" (delivering a section and
handing in the paper without continuing). The remedy was **to add a gate**: before each turn ends,
check that the agent has declared, in the required format, what it did this turn; if not, reject and
require continuation. Between 18:15 and 19:40 on 9 November the gate was installed, revised four
times and finally switched off. The logs after it was switched off show the gate did do something —
**Two sources disagree on this count, and both numbers are given.** The case's internal record says
it stopped **4** cases of "declaring incomplete and handing in"; the 35-rejection ledger of §4.1.3
verifies only **3** of the same type (class C, writing "done: 2/3" oneself). **The difference of one
is neither merged nor guessed**; each place reports its own ledger, and the gap is left standing.

The fault did not disappear. From 10 November to 22:30 (**inside the window**) the same fault
occurred 12 more times; for that whole calendar day it was 31 (Appendix C — the two denominators are
never mixed).

### 1.3 Three diagnoses, three times wrong

What is worth reporting is not the fault but that **the diagnostic process itself failed three
times**, each failure with the same shape.

| Order | Diagnosis | Proposed by | Refuted by |
|---|---|---|---|
| 1 | Context is too long; the model cannot cope | Researcher | 114 turns (prompt median 432k, max 523k) with zero stops |
| 2 | It hands in as soon as the checklist is written (the format has become the habit) | Governed | 57.6% (34/59) of stops occur *after* a tool had already been called |
| 3 | A stop token (EOS) was generated | Governed | The 59 stops have **no** `finish_reason`; all 679 EOS (`stop`) cases carry content, minimum 21 content tokens |

All three have one thing in common: **each points at a visible, nearby variable** — context length
is the number I can obtain; checklist and EOS are the actions she can feel.

**All three above are "proposed diagnoses", and they come from only two positions** (researcher
once, governed twice). The other two positions **propose no diagnosis; they issue verdicts
directly**: the gate judges "violation" (§4.1.3.1), the host judges "completed normally" (§4.2.6).
So the accurate statement is — **four positions, five judgements: three diagnoses plus two verdicts,
and not one of them holds.**

The real failure lies elsewhere: **the model's return layer produced a well-formed empty response,
and the host treated it as a normal ending.**

### 1.4 What this paper claims

This paper does **not** claim to have found the cause of the fault. On the contrary, **the claim is a
negative proposition, and it is held to be more valuable than a positive one**:

> On this n=1 system, actors at four different positions — governor, governed, researcher, runtime
> host — each read the same failure through their own observation boundary (**the governed and the
> researcher propose diagnoses; the governor and the host issue verdicts**), and **not one reading
> holds**; and the failure of the fourth layer (observation) removes, at the root, the possibility of
> correction in the first three. *(The second clause follows from the definition two paragraphs down —
> if correction requires discovering that the judgement was wrong, then a system that cannot discover
> it cannot correct. **It is the definition's content, not a further observation**, and it is placed
> here only because the claim is stated whole.)*

The phenomenon is named **attribution misplaced**, with an operational definition (§5.1). It is not a
synonym for "a judgement error": it has three necessary conditions, of which the third — **the system
cannot discover that its own judgement is wrong** — is the one that matters.

**But the claim must be read in tiers; read as one block it overstates the paper.**

| Tier | Content | Belongs to |
|---|---|---|
| **Strong** | The three targets of governance (context size / governance overhead / format compliance) **miss simultaneously** on this system; the denominators of the three counterexamples are **1,500 generations / 12 stops / 22 real faults** | This paper's own falsification; each target has its own literature, **falsifying them together is what this case contributes** |
| **Strong** | On the deployed schema there are **6 pairs that are observationally equivalent with opposite outcomes** (minimum difference 19 ms); restoring **one field** (`prompt_tokens`) separates all six ⇒ **the blind spot is priced**; and known non-circular variables explain only **10.49%** of deviance | Method borrowed from identifiability; **the enumeration, the tolerance sweep, the augmentation test and this 10.49% are all supplied by this case** |
| **Strong** | **Misclassification by the governor amplifies an isolated fault into a cluster**: 9 of 11 verifiable rejections follow an empty response (median 3 s); 22 real faults during the gate's active window, **0** rejections on the correct grounds (§4.1.3.1) | **The mechanism has an owner** (in engineering practice, "classification precedes response" — see the ownership statement in §4.1.3.1); **what this paper supplies is quantification**: 3 s, 9/11, 0/22, plus one counterexample (after the gate was deleted a run of 7 still occurred, so the gate is an amplifier, not the sole cause) |
| Weak | The observation layer records failure as success (differential observability broken) | **Covered by gray failure (Huang et al., 2017)**; this paper contributes one instance in a single-machine, self-built governance setting |
| **Open** | Interface labels were refreshed after the failure (6 of 22 "cancelled" became "completed") | **No mechanism is claimed**: "the data layer was rewritten" and "the UI transient never reached the store" cannot be distinguished, so it is reported as phenomenon, not claim |

**Why the third row is new and the fifth is not.** Both concern what happens after a failure, but in
opposite directions:

| | Amplification by misclassification (row 3) | Interface refresh (row 5) |
|---|---|---|
| Who acts | **The governor** — it makes a judgement and acts on it | The interface — no judgement, only rendering |
| Verifiable evidence | **The verdict's own wording**: the gate's feedback reads "you ran a tool call but did not write the checklist" (quoted from the transcript, §4.1.3), and the fact it asserts does not hold upstream — 27 of 35 times. Timing (2–6 s, case by case) is the **weaker** of the two | Terminal state only; no intermediate state to align |
| Consequence | One failure becomes a string of failures (countable: 22 faults, 0 correct rejections) | One failure is displayed as another (not countable; interval only) |
| Can it be refuted | Yes — by showing those rejections coincided with the stops, or share another common cause | Yes — but once refuted nothing remains |

**Two kinds of evidence, and why the timing is the weaker one.** The 2–6 s interval shows only that
the rejection was a *reaction* rather than a retrospective audit — a rejection following an empty
turn by minutes could be a later review. **On its own it cannot rule out a clock-bound confound**:
a read timeout of a few seconds, firing whenever a turn ended empty, would reproduce the same
intervals while involving no misclassification at all.

What rules that out is a different record, and this is the one the third row rests on: **the gate's
own verdict wording**, quoted from the transcript — "you ran a tool call but did not write the
checklist". That sentence **asserts a fact about the upstream turn**, and the assertion fails 27 times
out of 35 when checked against that turn's text (§4.1.3). A timeout label makes no such assertion and
**cannot be wrong in this way**. So the misclassification is established by the wording, not by the
interval; the interval only tells us when it happened.

**That is why the fifth row can only be "open"**: its only support is a terminal-state difference,
and such a difference has two sources (the old value was overwritten vs the old value never existed),
which this data cannot separate (external review r01 rejected it on exactly this point).
**What cannot be separated does not enter the claim; it enters the observations.**

**The third row is different — it cannot be withdrawn on that ground, because it is a temporal
alignment, not a difference that could be explained two ways.** And it links the first three layers: the gate's
predicates are format-only → on an empty response it reads no text → it judges "you forgot to write"
→ rejects → triggers a new turn → empty again. **One misreading becomes a string of failures.**

### 1.5 Contributions

1. **Falsification of three prevailing explanations** — context length, governance overhead,
   execution format. Each has a direct counterexample (114 turns with zero stops / still 12 stops
   after a −90.2% cut / 22 real faults with zero correct rejections), with denominators of 1,500
   generations, 12 stops and 22 real faults respectively — **not large, but the direction is settled,
   and each can be re-run.**
2. **One operational definition** — the three necessary conditions for attribution misplaced, and how
   it differs from "a judgement error", together with an **attribution admission threshold**
   (§3.4.1): any explanation entering the body must survive "≥2 counterexamples under the same
   conditions". Every explanation this paper overturns (the three most common being context length /
   governance overhead / execution format; §3.4.1 separately records **six** the researcher
   articulated that day — the two sets overlap but are not equal, and are not merged into one number
   here) died on exactly this clause.
3. **A re-runnable, priced observation blind spot** — on the deployed schema, **6 pairs** that are
   observationally equivalent with opposite outcomes (minimum difference 19 ms) are enumerated;
   restoring **one field**, `prompt_tokens`, separates all six (§4.2.6). Meanwhile known non-circular
   variables explain only **10.49%** of deviance (§4.2.8). **This is stronger than "I cannot explain
   it", because it puts a price on the blind spot.**
4. **A quantification of one mechanism: amplification by misclassification** — the gate's predicates
   are format-only; on an empty response it therefore reads no text — **that step is what
   "format-only" means, not an observation** — so it classifies "nothing was generated"
   as "you forgot to write the checklist", rejects, triggers a new turn, and that turn is empty too.
   **The judgement itself is not new** (engineering practice has long had "classification precedes
   response"; the verification is recorded in §4.1.3.1); **what this paper gives is three numbers**:
   9 of 11 verifiable rejections follow an empty response (median 3 s); 22 real faults during the
   gate's active window, **0** rejections on correct grounds; and one counterexample — **after the
   gate was deleted a run of 7 still occurred** (produced by deliberate back-to-back sending, see
   Appendix B), so the gate is an amplifier, not the sole cause.
5. **A complete, verifiable dataset** — turn-by-turn records of 46 hours, 775 turns, 11,086
   generations; raw API returns for all 59 stops, checkable character by character; six predicate
   scripts with self-tests (15/15, 6/6, 21/21, 10/10, 6/6, 5/5), every count frozen at 22:30 and
   re-runnable.

**Items 2 and 3 are new in the full sense**; **item 4 is a quantification of an existing mechanism**
(the mechanism is not new — §2.9 already declines to claim its discovery); item 1's novelty is
"falsified simultaneously on one system"; item 5 is a methodological supply.

**One item was demoted**: interface label refresh (6 of 22) **has been withdrawn from the claim** and
kept as an observation only. The reason is the two-row table in §1.4 — its only support is a
terminal-state difference, and such a difference has two sources which this data cannot separate.

### 1.6 Organisation

§2 places the study in four lineages and states which line it does *not* overlap, marking the
verification status of every citation. §3 sets out method: three data sources, operational
definitions, the attribution admission threshold, five discriminating counterexamples, six known
contaminations. §4 gives eleven findings F1–F11 along the exclusion axis, ending with re-run
instructions (including two frozen truncations). §5 discusses the four-layer misplacement, the
failure of the common pattern, and four transferable lessons. §6 states limitations, separating the
tiers of the claim, the not-claimed list, and compensatory controls. Appendix A records four
occasions on which this study itself got its own numbers wrong. Appendix B records one prospective
observation made after the window closed — **it does not enter any denominator, but it rules out one
alternative explanation that the body could not.**

---

## Translation status

| Section | Chinese source | Status |
|---|---|---|
| Abstract | `_论文_全文_初稿v15_分章终裁后_20261111.md` 摘要 (2,259 chars) | **Done** |
| §1 Introduction (1.1–1.6) | same file, §1 (4,479 chars) | **Done** |
| §2 Related work | 7,232 chars | Pending |
| §3 Method | 14,035 chars | Pending |
| §4 Results (F1–F11) | 42,083 chars | Pending |
| §5 Discussion | 9,199 chars | Pending |
| §6 Limitations | 6,903 chars | Pending |
| Appendices A–C | 6,121 chars | Pending |
| References | 1,819 chars | Pending |

**Why it is published in this state rather than withheld.** The abstract and introduction carry the
claim, the tiers of the claim, and the not-claimed list — that is, everything a reader needs in order
not to over-read the paper. Withholding them until §4 is translated would leave the repository
claiming an English edition that does not exist, which is the defect this file replaces.

**Replication.** Every number above names the script that produces it; the scripts ship with the
Chinese manuscript under `_论文_分章/_投稿/`. The observation window is frozen at 2026-11-10 22:30;
counts are not updated after freezing.
