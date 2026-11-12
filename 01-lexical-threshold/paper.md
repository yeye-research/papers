# What a Lexical Threshold Commits You To: Auditing Set-Overlap Metrics in Citation Verification

**Anonymous authors** · Paper under double-blind review

---

**Abstract.** Text generation and citation-checking pipelines routinely decide whether a sentence is supported by a source using a raw lexical-overlap threshold: if the longest common substring reaches $t$, the sentence is accepted as sourced. The threshold appears in system configuration as a bare integer — for instance 7 — with no statement of what it commits its operator to.

We present a complete audit of one real citation-checking pipeline: 308 sentences under review, 1,803 constructed negative pairs, 46 source documents, and 100 blind human labels drawn by stratified sampling. **The paper has one main result, and it is negative; the other two numbers are consequences it forces.**

**Main result: substituting a set-overlap metric does not work.** On nested decision sets, an overlap index such as Jaccard similarity $J$ reduces to a ratio of two empirical distributions — the denominator is cancelled — so it carries no information about the quantity of interest, which *is* that denominator. The distinction matters: this is **non-identifiability, not weak correlation**. Weak correlation can be mitigated with more data; non-identifiability cannot, because the index simply does not contain the information at that value. In our human-labelled sample, **127 sentence pairs share an identical overlap value yet received opposite labels**. **Scope:** what is ruled out is *this family* (set-overlap indices); non-overlap approaches (semantic embeddings, entailment judges) are **not tested and are outside the scope of this claim**.

**Consequence one: with this family of metrics closed off, the threshold is the only knob the present system can turn — and scanning it turns the bare integer into a statement of cost.** The deployed $t=7$ is equivalent to asserting that missing a fabricated claim is **1.16–1.41 times** more costly than wrongly flagging a genuine one; under equal weighting the optimum is $t=5$, differing from $t=7$ by **0.92 percentage points**. Across nine combinations (three false-negative regimes × three false-positive accountings), $t=7$ remains optimal in six.

**Consequence two: the scan then hits a harder constraint.** $\mathrm{EPR}(t)$ equals $\alpha$ under the null ($t=7 \Leftrightarrow \alpha=3.00\%$). But the task involves **308 comparisons**: family-wise error control at 5\% under a **Bonferroni correction** requires $\alpha\le1.62\times10^{-4}$, while 1,803 negative samples resolve only $5.5\times10^{-4}$ — not certifiable with this data; false-discovery-rate control at 5\% requires $t\ge9$. Cost justification and error control are separate standards, and we achieve only the former.

We also report three constraints on our own results: the 3.00\% false-positive rate is a **non-adversarial lower bound**; the 23-point spread of our interval comes entirely from how "unclear" labels are treated; and category proportions shift between 30\% and 80\% across four labelling passes. **An additional fully independent rater, re-labelling all 100 sentences, moves the central estimate (unclear-dropped accounting) from 26.57\% to 31.39\% (a 4.82-point gap), while agreement on A versus non-A stays at 74\%** — so our central figure is a magnitude, not a precise value.

**Keywords**: citation verification; threshold audit; non-identifiability; cost-sensitive decision making; multiple comparisons; annotation reliability

---



---

# 1 Introduction

## 1.1 A judgement that happens every day and is rarely measured

Citing a source while writing, and checking afterwards whether a sentence really came from
that source, are two sides of the same act. Whether performed by a person or by a machine,
the check ends in one concrete move:

> Given a sentence under review and a source document, decide whether the sentence is
> **sourced**, **unsourced**, or **unclear — a human should look**.

The machinery that reaches this decision can be elaborate — retrieval, reranking, a
cross-encoder — but the final step is usually plain: compute a **lexical overlap** score and
compare it against a **threshold**. This is not because the plain step is theoretically
optimal, but because it is operationally necessary. It is interpretable, recomputable, and
auditable. If an auditor asks why a sentence was judged fabricated, the answer "its longest
common substring with the source is five characters and the threshold is seven" is a
defensible answer. "Because the activation at layer 13 of a 700-million-parameter model was
low" is not.

**The object of this paper is that final step**: one threshold sweep, and the constant
sitting on top of it.

## 1.2 The question nobody asked

How was that constant chosen? In practice it arrives from one of three places: **experience**
("we tried it, seven looked reasonable"), **convention** ("the previous version used seven"),
or **tuning** ("seven gave the best F1 on the validation set"). All three are reasonable, and
none of them answers this question:

> **Placing the threshold at seven commits you to trading some amount of wrongful rejection
> for some amount of wrongful acceptance. How much of each?**

The question is not rhetorical. It has a precise form: given a cost ratio $w$ between letting
a fabricated claim through and wrongly rejecting a genuine one, the optimal threshold is
determined; conversely, **a chosen threshold implies a cost ratio**. This is textbook material
in cost-sensitive learning and signal detection theory (Elkan 2001; Provost & Fawcett 2001;
Drummond & Holte 2006; upstream, Neyman and Pearson 1933). **We do not claim to have
discovered it.**

**What we ask is something else.** In the applied literature and engineering documentation of
such checkers, this theorem is almost never used. The threshold is written as a constant, and
the cost premise is never written down. What results is a peculiar state of affairs:

> **The system holds a definite position on costs, but that position appears in no document.
> It is hidden inside an integer.**

The consequence is not that the system is wrong. It is that **the position cannot be
discussed**. An auditor cannot say "I disagree with your cost assumption" because the
assumption is invisible; a maintainer cannot say "the deployment changed, the threshold should
move" because nobody recorded what was originally assumed.

## 1.3 What this paper does: one main result, two consequences

We present a complete accounting of one real citation-checking pipeline: **308 sentences under
review, 1,803 constructed negative pairs, and 46 source documents totalling roughly 1.28
million characters**.

**The paper has one main result, and it is negative. The other two numbers are consequences
that result forces.** The ordering is not rhetorical; it determines how the other two should
be read. Only after knowing that the shortcut is closed can one see why the plodding
alternative is the only one available, and why its numbers are worth computing.

### Main result: substituting a set-overlap index does not work

Faced with the tedium of sweeping twelve thresholds, a natural substitute is to compress the
whole curve into a single number, such as the Jaccard similarity $J$. **We show that this
loses information, and that the amount lost can be quantified:**

$$J\big(S(t_1),S(t_2)\big)=\frac{\hat F(t_1-1)}{\hat F(t_2-1)}$$

On nested decision sets, $J$ is nothing but a ratio of two empirical distributions. **The
denominator cancels, and the denominator is precisely what the false-negative rate is about.**

Constructively: whenever $J$ is identically $0.5$, the false-negative rate may take **any
value in $(0,1]$**; on our stratified data, $J=0.5$ corresponds to two situations with
false-negative rates of 18% and 80% — **a gap of 62 percentage points**.

> **The precise word matters here: this is non-identifiability, not weak correlation.**
> Weak correlation is a statistical property: on average the two disagree, and **more data can
> mitigate it**. Non-identifiability is a structural property: there exist cases with identical
> index values and opposite labels. **More data would estimate those cases more precisely; it
> would not make them disappear.**
> On our data: **127 sentence pairs share an identical overlap value yet received opposite
> human labels**, and **all ten** LCS buckets contain both labels simultaneously.

**The boundary of this claim must be stated in the same breath, or it will be read as
claiming someone else's result:**

- **The mechanism has prior owners.** On nested sets, overlap indices degenerate; Baselga
  (2010, *GEB*) decomposes Jaccard exactly into turnover and nestedness components and notes
  that under the nestedness component the reading reflects only the ratio of set sizes. Webber,
  Moffat and Zobel (2010, *TOIS*) state this explicitly in motivating rank-biased overlap;
  Fagin et al. (2003) and Koleff et al. (2003) point the same way.
- **The phenomenon has prior owners too.** That sentence-level overlap metrics correlate
  weakly with human judgements has been measured for over a decade in NLP evaluation (Reiter
  and Belz 2009; Reiter 2018; Novikova 2017; Kryscinski 2019; Bhandari 2020; Fabbri 2021), and
  on this specific task of sentence-level attribution (Rashkin 2023, AIS; Gao 2023, ALCE;
  Yue 2023, AttrScore).
- **Our increment is three things, and none of them is the mechanism**: why it happens (a
  structural non-identifiability), how much it costs in this setting (how many pairs, how many
  points), and **where it therefore forces the choice to go** — the two consequences below.

> One further narrowing: we prove **information loss**; we do **not** prove that $J$ is biased
> in any particular direction. So we may say "do not use $J$ to skip the sweep"; we may **not**
> say "$J$ is a bad index". It is not wrong; it answers a different question.

### Consequence one: this family's reading cannot serve as evidence, leaving only the threshold — and the sweep translates the integer into a cost

**First, this is not our finding.** That the optimal threshold is determined once the ratio of
the two error costs is given is a **standard result**, with a lineage running from
Neyman–Pearson (1933) in statistics, through the burden-of-proof analysis of law and economics
(Posner 1973; Kaplow 2012), to cost-sensitive learning (Elkan 2001). Law and economics
articulated it earlier still — the optimal standard of proof shifts with the ratio of the two
error costs — and used it to explain why civil proceedings employ a preponderance standard
while criminal proceedings require proof beyond reasonable doubt.

**What we do is instantiate and quantify it** on the LCS threshold of a citation checker:

> The deployed $t=7$ is equivalent to asserting that **letting a fabricated claim through** is
> **1.16–1.41 times** more costly than **wrongly rejecting a genuine one** (point-estimate
> accounting). Under equal weighting ($w=1$) the optimum is $t=5$, **0.92 percentage points**
> away from $t=7$. Across the nine combinations of three false-negative regimes and three
> false-positive accountings, $t=7$ remains optimal in **six**.

That multiplier **can be argued with**: an auditor who holds that the first error is five times
worse gets $t\ge 9$; one who holds that the two are equal gets $t=5$. **The disagreement now
has coordinates.**

> **A counter-intuitive support.** One might object that the value of $w$ was settled long ago
> in law. **The opposite is true.** Blackstone's "better that ten guilty persons escape than
> that one innocent suffer" is **not an operable $w=10$**; it is rhetoric. The ratios actually
> recorded in history contradict one another: Hale gives 5, Fortescue 20, some sources upward
> of 1,000. **That the most mature adjudicative system in existence failed, over centuries, to
> settle on a single number is evidence that $w$ has no default value. It can only be written
> down case by case.** We do not ask that $w$ be set correctly. We ask only that it be
> **written down**.

> **Methodological position.** We do not assume a $w$. We **invert** the system's own choice of
> $t=7$ and ask which interval $w$ must lie in for $t=7$ to be optimal. **This is a conditional
> statement, not an estimate**: what can be recovered from behaviour is a *constraint*, not the
> number $w$ itself. The latter is a normative judgement, and data cannot supply it. The move
> is one of **revealed preference**: rather than asking an auditor what they consider costly,
> we read it off the threshold they actually chose.

### Consequence two: the sweep runs into a harder constraint

$\mathrm{EPR}(t)$ is the fraction of unrelated pairs judged sourced. It **is** the per-comparison
significance level $\alpha$ under the null. Once that identity is written out, a constraint
appears that is more fundamental than whether the threshold has a justification:

| Error-control standard | Requirement | Our situation |
|---|---|---|
| **FWER 5%** (at least one error with probability $\le 5\%$) | $\alpha\le1.62\times10^{-4}$ | 1,803 negative samples resolve only $5.5\times10^{-4}$ → **not certifiable with this data** |
| **FDR 5%** (a controllable error proportion) | $t\ge 9$ | deployed $t=7$ → **too permissive** |

**This is not a proposal; it is the result of the arithmetic.** Our negative sample
(1,803) **cannot certify any non-trivial threshold**; $t=7$ was set on the scale of a
per-comparison $\alpha=3.00\%$.

> **The relation of this to the two items above must be stated.** It is **not** a consequence
> of the main result, and **not** a product of the sweep — **it is intrinsic to the shape of the
> task.** Deployment requires deciding **308 sentences one at a time**, which is **308
> comparisons**; because there are 308 comparisons, a per-comparison $\alpha$ is insufficient.
> **308 is the number of sentences under review; 12 is the number of thresholds — two different
> dimensions, not to be conflated.** Sweeping twelve thresholds was an exploratory act of
> research; **the multiple-comparison burden comes from the former and has nothing to do with
> the sweep.**
> **This also makes it harder than the two items above**: it depends on none of our claims. To
> avoid it one would have to change the one-at-a-time deployment itself — **swapping the index
> changes nothing.**
> (A separate note: in saying the shortcut is closed we also **do not** say the sweep is the
> only road. Non-overlap approaches are untested here; see §8.4.)

## 1.4 An independent empirical check: LCS is not a monotone proxy for a likelihood ratio

The three items above form a chain. This one **is not part of that chain**; it is a problem of
the three-way structure itself, and is recorded separately.

The three-way output (sourced / to-human / unsourced) is not our invention. It is the
match / clerical review / non-match structure of record linkage (Fellegi and Sunter 1969).
That framework carries a theorem: **given upper bounds on the two error rates, ordering by
likelihood ratio with two cut-offs minimises the proportion sent to human review.**

**Our score is LCS, not a likelihood ratio.** Measured: $\Lambda(5)=4.71$ while
$\Lambda(6)=1.80$ — **the evidence at LCS six is weaker than at LCS five**, an inversion
produced by a pile-up of negative samples at $k=6$.

> So our two cut-offs **do not inherit that optimality guarantee**.
> **This is as far as the inference goes.** Not inheriting a guarantee means **there is no
> proof of optimality**; it does **not** mean that too much has already been sent for review.
> Whether that happens, and **how many items** are involved, we have **not measured**. To write "a batch is being
> sent to human review for nothing" would be to report an expired guarantee as an incurred
> loss.
> This is the one measurement in this chapter that is entirely our own: not a citation, but a
> counterexample we produced.

**A word on placement is required here**, or a reader will wonder why the main result sits in
the last analytical chapter. **The full proof of the main result — the non-identifiability of
$J$ — is in §6; this chapter gives only the statement and its boundary.** The reason is
dependence of material, not narrative preference: the proof requires $\hat F(t)$ and the
decision sets $S(t)$ that §5's sweep produces. Only after §5 does every symbol in §6 have
numbers to point at. **The physical order of this paper is therefore "do the plodding thing
first, then explain why only the plodding thing was available", while the logical order is
"show the shortcut is closed, then show where the plodding thing goes". The abstract and
introduction follow the logical order; the body follows the order of the material.**

## 1.5 Five things we do not claim

| We do not claim | Because |
|---|---|
| To be first to observe that a threshold implies a cost ratio | **Law and economics states it at textbook level** (Posner 1973; Kaplow 2012); cost-sensitive learning (Elkan 2001; Provost and Fawcett 2001; Drummond and Holte 2006) is another branch of the same result; upstream, Neyman–Pearson 1933 |
| To be first to apply a formalised standard of proof outside law | **Burden-of-proof analysis in law and economics already covers non-legal settings** (internal organisational decisions, contractual acceptance criteria, medical triage). The sentence would be killed by a single citation |
| To be first to reveal that $J$ degenerates on nested sets | **Has prior owners**: Baselga 2010; Webber 2010; Fagin 2003; Koleff et al. 2003. Our increment is the dose and the consequences |
| To be first to note that sentence-level overlap disagrees with human labels | **Has prior owners**: **over a decade** of measurement in NLP evaluation (see **the boundary of the main result** in §1.3). What we add is a sharper word — non-identifiability |
| To be first to decide provenance by lexical overlap plus a threshold | Decades of routine practice in plagiarism detection and code cloning (MOSS, JPlag, winnowing) |

**Our position is this**: each of these tools and conclusions already exists, but **nobody has
strung them together into a single recomputable accounting on a real checking pipeline** —
from "why this family of indices cannot be substituted" through "the threshold is the only knob
left" to "the cost ratio and significance level the sweep produces", with a concrete number at
every step.

## 1.6 Something that must be said up front

The central number in this paper — a false-negative rate of **26.57%** — **depends on a set of
human labels**, so we audited those labels five times over. What the audit found is more
specific, and more tractable, than "not accurate enough".

**Terminology.** Numbering follows the **raters**, not the rating passes — the paper involves
**three raters and five rating passes**, and a numbering by pass would not add up to three people.
**Rater 1** is an author, who labelled in three passes (passes one, two and three).
**Rater 3** is an independent rater who re-labelled 50 sentences blind, under the same written
criteria.
**Rater 2** is a further independent rater who re-labelled all 100.

**① "Is it A (findable)?" is more stable than "B or C?" — but both layers show disagreement, and
neither can be called clean.**

| Pairing | What changed | Three-way agreement | **A vs non-A** |
|---|---|---:|---:|
| Rater 1: pass 1 vs pass 2 | same person, **no criteria** | 54% | **64%** |
| Rater 1: pass 2 vs pass 3 | same person, **criteria added** | 68% | **70%** |
| Rater 1 vs Rater 3 | **different person**, same criteria | 60% | **74%** |

**A-versus-non-A agreement is 74% across raters, 70% for the same person with criteria added,
and 64% on pure retest. All three fall between 64% and 74% — but each pair changed a
different thing, and no pair isolates "a different person" as a separate contribution.**
Moreover the A-sets of Rater 1 and Rater 3 stand in a relation of **proper containment**
(Rater 1: 40 sentences; Rater 3: 27; **zero in the reverse direction**).

**Random error runs in both directions; a difference in strictness runs in one.** We observe
one direction only — so this looks more like a difference in leniency than like random error.

> **But this cannot be pushed to "so nobody mislabelled anything".**
> ① In a finite sample, random error can perfectly well appear in one direction only;
> ② proper containment is **compatible** with a difference in strictness, and equally
> compatible with one rater having mislabelled a batch — the data cannot separate these;
> ③ and **74% is agreement, not accuracy** — two raters can agree and both be wrong.
> What can be said: **the agreement at this layer is not random noise**; and **we cannot say
> who is right.**

Intersecting the two raters' A-sets yields **27 doubly-confirmed A**.

**When Rater 2 re-labelled all 100 sentences, the 74% appeared again** (§8.1.4): three-way
agreement was only **49%**, disagreement **51%** — higher than the same-person retest — **yet
A-versus-non-A was again 74%**.

→ **Two independent cross-rater pairings (Rater 1 vs Rater 3, Rater 1 vs Rater 2), one over 50
sentences and one over all 100, both give 74% on A versus non-A.**

> **But this can only be called a repetition, not a stable core.** **Two observations cannot
> support a claim about stability** — stability itself was never measured (that would require
> repeated pairings of the same two raters). What can be said is that the number **reproduced
> under two different conditions**.

→ At the same time: **the 51 disagreements are not "almost all B versus C"**. Broken down by
the figures in this paper:
> **26 are A-versus-non-A disagreements and 25 are B-versus-C — almost exactly half each.**
> (Of the 74% A-versus-non-A agreement, 47 sentences are jointly non-A; within those 47, B and
> C are judged alike in 22, i.e. 46.8%; three-way agreement is 27 + 22 = 49 sentences,
> consistent with 49%.)
> **So the accurate statement is that neither layer is perfect** — not that "A is stable while
> B/C is not".

**② Once a sentence is not A, whether it counts as B or as C is largely personal habit.**

Of the 10 sentences that three passes of Rater 1 called B, **Rater 3 called 7 of them C and
none A**; under the same written criteria, **Rater 1 used C zero times while Rater 3 used it 16
times (32%)**.

**③ The error components must therefore be listed separately, each with its base, and never
combined into a single $\pm$:**

| Component | Value | Base |
|---|---:|---|
| Sampling error | ±6.65 points | 308 sentences, stratified extrapolation |
| Accounting spread (how C is counted) | 23 points | **308 sentences** |
| *(same-base comparison)* | *36 points* | *50 sentences* |
| **Rater drift (category proportions)** | **≤50 points** | **50 sentences** |
| **Rater difference (central estimate, point accounting)** | **4.82 points** | **100 sentences, across raters** |

> **The 50-point and 23-point figures rest on different bases (50 vs 308 sentences) and cannot
> be subtracted.** Drift on the 308 sentences was **not measured**, so **the 50 points cannot be
> added to 26.57**. What can be said is that **the uncertainty in this number is much larger
> than its confidence interval.**
> **The last two rows both belong to the rater dimension but measure different things** (one
> measures how the A proportion drifts, the other how far the central estimate moves); their
> bases differ, and they can be neither added nor converted.

**This is not a footnote. It determines how the conclusions should be read:**

- **The central figure uses the accounting that drops unclear labels (point accounting).** This
  now has a hard justification, not merely a preference for the middle: both the upper- and
  lower-bound accountings let the least stable category, C, enter directly, whereas **only the
  numerator of the point accounting depends on A alone, the most stable category**;
- **The two ends of the interval are not two defensible accountings; they are how far the
  number can drift when the criteria are unclear**;
- This appears first in §8.1, §8.1.1 and §8.1.2, and **is never separated from any conclusion**.

> **An honest cost analysis must count its own measurement error as a cost.** We do this
> imperfectly: we measured how large the error is, but **could not eliminate it**. This is the
> minimum that honesty demands of this paper, and our honest report of where we fall short.

## 1.7 Structure

| Section | What it does | Logical position |
|---|---|---|
| §2 | Related work in four blocks: text-reuse detection; significance of sequence alignment; cost-sensitive decisions and cut-off setting; objections from the retrieval community. Each block's **contribution is delimited against ours** | — |
| §3 | Problem formalisation: similarity plus threshold as a membership function and decision sets; definitions of $\mathrm{ENR}$ and $\mathrm{EPR}$; the three-way structure | — |
| §4 | Data and scorer: 308 sentences under review, 1,803 negatives in three families, the secondary-scorer comparison, and the 100 human labels | — |
| §5 | The threshold sweep: point results for twelve thresholds, locating the cost ratio, the three-families × three-accountings **nine-cell** table, and **multiple comparisons from a significance standpoint** | **The two consequences live here** |
| §6 | Information loss: the identity for $J$ and the constructive counterexample | **The full proof of the main result lives here** |
| §7 | Discussion | — |
| §8 | Limitations: four foundations, four holes, ordered by how easily a single sentence could break them | — |
| §9 | Conclusion | — |

> **The last column exists for the reviewer.** The physical order of this paper is the order of
> the material (§5 then §6), not the logical order (§6 then §5). The abstract, introduction and
> conclusion follow the logical order, leading with the main result; the body follows the order
> of the material. **The inconsistency is deliberate, and the reason is one sentence: §6's proof
> depends on Table 5.1 from §5, so the body must supply the table first; whereas the abstract,
> introduction and conclusion must let a reader grasp the main line first.**

---

# 2 Related Work

> The task of this chapter is **not to write a survey** but to **draw the boundary of this paper's
> increment** — every section must end on "how much of this overlaps with us, and what we add".

---

## 2.0 The three questions this chapter answers

1. Has **anyone done this before**? (If so, this paper can only count as an instantiation.)
2. When they did it, **did they state the cost premise explicitly**?
3. What is the **smallest increment** this paper can claim?

> Three known threats must each be met head-on; **pretending not to know them is not allowed**:
> ① "a chosen threshold implies a cost ratio" is a **known theorem** (Elkan / Provost–Fawcett /
> Drummond–Holte);
> ② "compressing a curve into a single scalar loses information" is partly pre-empted by Hand 2009;
> ③ "setting a threshold on lexical overlap, and the fact that paraphrase escapes it" is **common
> knowledge** in plagiarism detection.

---

## 2.1 Text reuse and provenance (plagiarism detection / code clones)

**What it does**: for decades this field has been doing "given a piece of text, decide whether it came
from some source", and it **has long known that a lexical threshold can be escaped by paraphrase**. The
engineering tradition of MOSS/JPlag, winnowing fingerprints and greedy string tiling coverage scoring all
live here.

**Relation to this paper**: the shape of the decision rule (substring overlap plus a threshold).

**What this paper adds**: they report "similarity and hits" and almost never **the two error rates**;
this paper turns the whole threshold curve into $\mathrm{ENR}/\mathrm{EPR}$ and computes the cost.

**Further references**
- Schleimer, Wilkerson & Aiken 2003, *Winnowing: Local Algorithms for Document Fingerprinting*, SIGMOD,
  76–85, `10.1145/872757.872770`
- Wise 1996, *YAP3*, SIGCSE, 130–134, `10.1145/236452.236525`
- Prechelt, Malpohl & Philippsen 2002, *Finding Plagiarisms among a Set of Programs with JPlag*, JUCS
  8(11), 1016–1038, `10.3217/jucs-008-11-1016`

> **Writing discipline**: do not write "we are the first to use lexical overlap to judge provenance".
> That is the everyday practice of this field.
> **The mainstream implementation in this field is MinHash + LSH, and the mechanism matters:** **Jaccard is a quantity "between two
> sets"; a single document does not have "a Jaccard number".**
> MinHash compresses **a set** into a signature with a family of hash functions, and the signature
> **estimates the Jaccard of the two sets without bias**; LSH then uses signatures for nearest-neighbour
> retrieval. **It is an estimator of pairwise similarity, not a per-sentence decision rule.**
> **Jaccard is a pairwise quantity (between two texts), not a property of one document** — a single
> document has no "Jaccard of its own". This matters beyond terminology: getting it wrong makes people
> read the criticism in §6 as attacking a mechanism that does not exist. Stated accurately, §6's target
> is clearer — it attacks "using a pairwise similarity to certify a single sentence", not "the MinHash
> algorithm is bad". **When §6 criticises "compressing a decision into a single scalar", it must attack
> the LSH version, not a straw man.**

### 2.1.1 What must be conceded: this paper's decision rule is in the same family as ROUGE-L

**If this is not stated in the paper, a reviewer will state it for us in the review.**

In automatic summarisation and machine translation, evaluation relies on a standard metric,
**ROUGE-L** (Lin 2004),
which computes $R_{\mathrm{LCS}}$, $P_{\mathrm{LCS}}$ and $F_\beta$ from the **longest common
subsequence**. The decision rule here uses **the longest common contiguous substring, an absolute length,
and a fixed threshold**.

| | This paper | ROUGE-L |
|---|---|---|
| What is taken | contiguous substring | subsequence (may skip) |
| Normalisation | none | length-normalised |
| Can an empirical null be computed and read as $\alpha$ | **yes** | **yes, also** |

> **A common misjudgement first**: some write "ROUGE-L **cannot** be read as $\alpha$".
> **Both families can be read as $\alpha$**; the convenience here is not "it can be read" but **what you
> do with the number you read** (below).

> **This paper does not claim the shape is new**, only that it does something **more convenient** with
> it: **the threshold is an integer, recomputable, and requires no prior decision on "what to normalise
> by"**, so $\alpha$ can be inverted directly from $t=7$ (§5.8).
> **Correction, so that the claim is not overstated**: an empirical null $P_0(S\ge t)$ can be computed
> for any normalised statistic $S$ and read as $\alpha$ as well — **an absolute length is not necessary**.
> **Nor may it be said that "an absolute length is a sufficient condition for comparability across
> lengths" — that is backwards**: the same absolute $t$ is *easier* to reach in **long** texts, so
> **absolute length is precisely what is not comparable across lengths**. §7.5 says as much: "in long
> source documents, and in terminology-dense ones, even an unrelated sentence can easily produce a long common
> substring".
> The accurate statement is: **not normalising by length buys "a simple convention" (no need to decide
> what to normalise by), at the cost of "no comparability across lengths"** — a trade-off, not an
> advantage. The reasons for choosing it are the three in §3.1.0 (conservative direction; contiguity is
> itself evidence; direct inversion of $\alpha$).
>
> References: Lin, C.-Y. 2004, *ROUGE: A Package for Automatic Evaluation of Summaries*, **WAS 2004**;
> the subsequence version used in MT evaluation, Lin & Och 2004, ACL, `10.3115/1218955.1219032`.

---

## 2.2 The significance basis for a threshold (sequence alignment and extremes)

**What it does**: bioinformatics solved "is this similarity score high" thirty years ago — **it reports
not only the raw score but its significance** (BLAST gives Score / bit score together with an E-value).
> **BLAST reports both.** Saying "it does not report the raw score, only the significance" would
> suggest significance **replaces** the similarity score, whereas in fact **two numbers are given and the
> E-value decides whether to trust them**. That distinction is exactly the idea borrowed here: **give the
> score, compute the confidence separately.**

The distribution of the longest common substring between random strings has closed-form results, so the
threshold can be inverted from "how large a chance exceedance probability do we allow" instead of being
guessed.

**Relation to this paper**: **the idea only**. No analytic null is used here; the null is **empirical**
(three negative-sample families).

**What this paper adds**: reading $\mathrm{EPR}(t)$ explicitly as a significance level $\alpha$, and doing
**multiple comparisons** in $\alpha$ coordinates (§5.8).

**Further references**
- Altschul et al. 1990, *Basic local alignment search tool*, J. Mol. Biol. 215, 403–410,
  `10.1016/S0022-2836(05)80360-2`
- Karlin & Altschul 1990, PNAS 87, 2264–2268, `10.1073/pnas.87.6.2264`

> One boundary must be stated: sequence alignment has an **analytic null** **under specific
> conditions** — Karlin–Altschul give Gumbel parameters for **a particular scoring matrix, particular
> background frequencies and local alignment**, **and ungapped versus gapped must be handled separately**
> (the latter needs an effective-length correction). Here there is only an **empirical null** (three
> negative families). **This is not "weaker"; it is "no closed form is available"** — a "match" in
> citation checking has neither a **scoring matrix** (it counts the length of the longest common
> substring, not a weighted score) nor stable **background frequencies** (source length, language and
> domain all differ).

---

### 2.2.1 The other line working on the same task: NLG attribution evaluation

**This is the line closest to the present paper, and the one most often left out of such
comparisons.**

§2.1 discusses "judging whether text is reused"; §2.2 discusses "is the score significant". There is a
third line closer to this paper — **it does exactly "decide whether a sentence is supported by a
source"**, with the same task framing, except that its methods live at the semantic level.

**The task and label structure of the three works (read in full, not from metadata)**

| Work | How it judges | Label structure | Basis |
|---|---|---|---|
| **Rashkin et al. 2023**, *Measuring Attribution in NLG Models*, *CL* 49(4):777–840, `10.1162/coli_a_00486` | **pure human annotation**, two stages | **binary + flag**: ① interpretable (yes/no) ② AIS-supported (yes/no) ③ **flag = the item is removed from the scoring pool** | AIS framework |
| **Gao et al. 2023** (ALCE), **EMNLP 2023**, `10.18653/v1/2023.emnlp-main.398` | an **NLI entailment model** (TRUE T5-11B) decides whether the cited passage entails the statement | citation recall / precision (sentence level, 0/1) | AIS |
| **Yue et al. 2023** (AttrScore), Findings of EMNLP 2023, `10.18653/v1/2023.findings-emnlp.307` | prompted LLM or a fine-tuned small model | **three classes**: Attributable / **Extrapolatory** / Contradictory | own scheme |

> **AIS is not, however, a three-way classification.**
> In the original, **"underspecified source"** is **one reason for a flag**, not a label value — flagged
> items **do not enter the denominator** ("thus excluded from the interpretability and AIS scores").
> **The genuine three-way scheme is AttrScore's, not AIS's.**

### One point in this paper's favour: the AIS flag mechanism is isomorphic to the "point accounting" here

When AIS faces "the source itself does not say", it **removes the item from the denominator rather than
assigning it a value**.

**That is exactly what the point accounting (C dropped from the denominator) does in §5.**

> So the sentence in §4.6.8 — "the point accounting is chosen because **it needs no value assigned to
> C**" — now has external support: **it also matches how AIS, currently the most authoritative
> attribution-evaluation framework, handles unclear sources.** This is not "taking the middle"; it is
> **following a mature framework**.

### How this paper relates to them, in three sentences

1. **Same task**: given a sentence and a source, decide whether it is supported.
2. **Different level**: they work at the **semantic** level (NLI / QA / LLM), this paper at the
   **lexical** level.
3. **The difference is not "better" but "more computable"** — the lexical level has three properties, but
   **only two constitute a difference**:
   > · **Model-free** — **this does not count as a difference**: AIS is **pure human annotation**, so it is
   >   model-free too. What actually depends on a model is ALCE (an NLI model) and AttrScore (LLM prompt /
   >   fine-tuning).
   > · **$w^*$ can be inverted** (§5.5) and **$\alpha$ can be read** (§5.8) — **they have neither**, and
   >   these are the two interfaces that connect "is the judgement right" to **cost** and **significance**.
   > Conversely, **this paper cannot give what they are good at**: whether a paraphrase counts as sourced
   > — the lexical level cannot judge it.
   > **The difference is two things, not three**: "model-free" is not one of them, since AIS is
   > model-free as well.

> **No claim of greater accuracy is made here**: no controlled experiment supports one. Nor is there
> a priority claim — AIS and ALCE are 2023 work, while judging provenance by lexical overlap is
> decades older (§2.1). **Neither side is new.**

### Three measurements: they have already quantified that overlap metrics are unreliable

This line of work **does not only build semantic methods; it has also measured how unreliable the lexical
metrics are** (detail in §6.0.1 and `S1`):

| Source | Measurement |
|---|---|
| **ALCE Appendix A, Table 10** | the degenerate baseline returning only the best retrieved passage: **ROUGE-L 19.1, claim recall 3.0**; ChatGPT is 20.6 / 12.0 → **the metric differs by 7%, true quality by a factor of four** |
| **ALCE Appendix D, Table 11** | the **copying baseline** (return the retrieved passage as the answer, self-cited): **citation recall 99.4 / precision 99.4**, while fluency is 20.8 and accuracy 35.1 |
| **Krishna et al. 2021** (NAACL, `10.18653/v1/2021.naacl-main.393`) | replacing retrieved documents with **random documents** leaves **ROUGE-L almost unchanged** (Figure 1c); the authors write "ROUGE-L is not an informative metric… and can be easily gamed" |

**So "overlap metrics are unreliable" cannot be treated as this paper's own observation.** The positioning
of §6 gives up one more notch accordingly, see §6.0.1.

### One thing they concede, which endorses the C band here

ALCE states explicitly that its NLI metric **cannot handle "partial support"**, and that trying ChatGPT
for it "yields poor results". AttrScore's error analysis reports that the model misjudges
**Extrapolatory as Attributable about 70%** of the time, of which "insensitivity to fine-grained
information" accounts for **46.7%** (numbers, dates).

> **"Partial support" and the C band here (cannot tell) are the same thing**; "insensitive to numbers
> and dates" and the §5.7 question "does verbatim terminology overlap count as letting something
> through" are the same class of problem.
> But these two points **only endorse "this distinction is hard"; they do not endorse "keep the C band"** —
> **the step in between has to be supplied here**: "hard" admits at least three treatments: ① abolish C
> and force a dichotomy; ② merge C into B; ③ keep C and mark it as needing a human.
> **The reason for choosing ③ is not "it is hard" but "collapsing a 23-point span into one number throws
> away the discriminative information the reader needs"**: a forced dichotomy would collapse the
> **23-point** span into a single number, and that span **is enough to change the direction of the
> conclusion** (upper bound 40.14% versus lower bound 17.30%). **Keeping C makes that uncertainty
> visible; it is not shelving a problem because it cannot be solved.**
> What the external literature establishes is "this is not a shortcoming of this paper's rater"; **it
> cannot choose the treatment on this paper's behalf**.

**Further references**: the four works in the table above.

---

## 2.3 Cost sensitivity and cut-off setting (Neyman–Pearson → law and economics → cost-sensitive learning → standard setting)

**What it does**: this is the section where this paper should **bow lowest**, because the main conclusion
of §5 is **a known theorem** here: "given a cost ratio $w$, the optimal threshold is where the equal-cost
line touches the convex hull of the ROC curve; conversely, a chosen operating point implies a cost
ratio."

> **The lineage must be written in order, not only the machine-learning branch**:
> **Neyman–Pearson 1933 (statistics) → burden of proof in law and economics (Kaplow 2012) →
> cost-sensitive learning (Elkan 2001 / Provost–Fawcett 2001 / Drummond–Holte 2006)**.
> **Citing only Elkan reads as not having done the homework** to a law-and-economics reviewer — that
> branch is thirty years older than the machine-learning one.

### 2.3.1 Law and economics: the burden of proof is an explicit cost ratio

**What it does**: law and economics writes "how high to set the standard of proof" as **minimising the
expected social cost of two error types** — **the optimal posterior threshold moves with the ratio of the
two error costs**. This is a **textbook-level conclusion**, not a fringe view. Civil law uses
"preponderance of the evidence" (about 50%) and criminal law "beyond reasonable doubt": **two different
cost ratios**.

**Relation to this paper**: **almost all of it, at the level of principle**.

**What this paper adds (stated narrowly)**: **applying that principle to the LCS threshold in citation
checking, and measuring the concrete interval **1.16–1.41×** — the contribution is **instantiation and
quantification**, not the principle. See the demoted wording of contribution 1 in §1.3.

> **A counter-intuitive use of this, stronger than "finding a precedent"**:
> Blackstone's "better that ten guilty persons escape than that one innocent suffer" **is not an operable
> $w=10$**; it is rhetoric.
> The evidence: the ratios written down historically **contradict each other** — Hale records 5,
> Fortescue 20, some more than 1000 (systematically documented by Volokh 1997).
>
> **So the argument must not be "there is a precedent, we follow it", but**:
> **"even the most mature adjudicative systems, over centuries, failed to settle on a ratio (5 / 20 /
> 1000+ contradict each other) — which shows that $w$ cannot have a default value, and can only be
> written down case by case."**
>
> The latter depends on no particular number, so **it cannot be knocked down by "10 is rhetoric"**, and it
> is stronger than the original: **what this paper asks for is not "get $w$ right" but "write it down".**

**Further references**
- Kaplow 2012, *Burden of Proof*, **Yale Law Journal 121**
- Kaplow & Shavell 1994, *Accuracy in the Determination of Liability*, J. Law & Econ., `10.1086/467304`
- Volokh 1997, *n Guilty Men*, U. Pa. L. Rev.

> **Writing discipline**: do not write "we find that a threshold implies a cost assumption", and do not
> write "the first application of this theorem outside economics" — only "this standard conclusion **has
> not been written down** in citation-checking practice; this paper reads it out and quantifies it".

### 2.3.2 Cost-sensitive learning and cost curves (the machine-learning side)

**What it does**: the machine-learning statement of the same thing — given a cost ratio, the optimal
threshold follows; cost curves draw a whole family of cost ratios on one plot.

**Relation to this paper**: **almost all of it**.

**What this paper adds**: turning $w$ from "a given parameter" into "**a constraint interval inverted
from the chosen threshold**" (see 2.3.3).

### 2.3.3 A methodological position: this paper takes the "revealed cost" route

This paper does not assume a $w$; it **inverts the $w$ interval implied by the system's chosen $t=7$**
(**1.16–1.41×**). This is a **revealed-preference** move: rather than asking an auditor "which do you consider
more costly", it **reads it out of the threshold they actually chose**.

> This sentence blocks "where does your $w$ come from" — the answer is "**out of your system, not set by
> me**".
> **But it has a premise, the same as §8.6**: the inversion relies on "the threshold has not changed the
> distribution of the text being checked". Once the threshold is public and the audited party rewrites
> accordingly, **the inversion breaks**.

**Further references**
- Elkan 2001, IJCAI (proceedings of that year have no DOI; cite as a conference paper)
- Provost & Fawcett 2001, Machine Learning 42, 203–231, `10.1023/A:1007601015854`
- Drummond & Holte 2006, *Cost curves: An improved method for visualizing classifier performance*,
  Machine Learning 65, 95–130, `10.1007/s10994-006-8199-5`
- Hand 2009, Machine Learning 77, 103–123, `10.1007/s10994-009-5119-5`
- Angoff 1971, *Scales, norms, and equivalent scores*, in Thorndike (ed.), *Educational Measurement*
  (2nd ed.)
- Further upstream: Neyman & Pearson 1933; Green & Swets 1966; Youden 1950.

### 2.3.4 The three things this paper adds (stated narrowly)

1. Applying the theorem **to the concrete setting of citation checking and measuring the number**
   ($w^*=1.16$–$1.41$, §5.5);
2. Pointing out that **in the practice literature, that $t=7$ never had its cost premise written down** —
   this is an **absence**, not a discovery;
3. On the standard-setting methodology (Angoff / Bookmark) of "let people judge where the cut-off goes",
   this paper is **one sub-standard instance** (single rater, §8.1).

> **The claim is narrower than "we find that a threshold implies a cost assumption".** What holds is
> that **the theorem has not been written down** in citation-checking practice; this paper reads it
> out and quantifies it.
> **Nor is this the first application of this theorem outside economics**: burden-of-proof analysis in
> law and economics is one.

### 2.3.5 One further line that must at least be pointed at here: degeneracy of set-overlap metrics

The **main result** of this paper (§1.3) is that set-overlap metrics (Jaccard $J$, say) are
**non-identifiable** on nested decision sets. Its main literature discussion **does not live in this
section** but in §6.0 — yet related work cannot be silent about it, or a reader will think this paper does
not know:

- **Mechanism**: Baselga 2010 (*GEB* 19(1):134–143) decomposes Jaccard exactly into turnover and
  nestedness, and shows that under nestedness the reading reflects only the ratio of set sizes; Webber–
  Moffat–Zobel 2010 (*TOIS* 28(4) Art. 20) state it in motivating RBO; Fagin et al. 2003 and Koleff et al.
  2003 agree.
- **Phenomenon**: sentence/segment-level overlap metrics correlate weakly with human judgement — Reiter &
  Belz 2009 / Reiter 2018 / Novikova 2017 / Kryscinski 2019 / Bhandari 2020 / Fabbri 2021.
- **Increment here**: changing the word from "weak correlation" to "**non-identifiability**", measuring
  the dose in this setting, and stating where it forces the choice. **Full discussion in §6.0, §6.0.1,
  §6.0.2.**

---

## 2.4 The objection from information retrieval (which must be met first)

**What it does**: mainstream IR **deliberately avoids absolute similarity thresholds** — it ranks and
reports precision@k, **without cutting once on the score axis**.
> **But it cannot be said that "there is no truncation"**: precision@k itself takes the top $k$ **by
> rank**, which is of course a truncation. The only defensible sentence is: **what IR avoids is "an
> absolute similarity threshold", not "all truncation"**. So the claim "information retrieval teaches us
> how to set thresholds" **is simply false**, and writing it would be a hard error.

**The role of this section is defence**: a reviewer is very likely to say "why agonise over this
threshold? we never set one".
The answer here: **because the setting of this paper is adjudication, not ranking** — citation checking
must give, item by item, "has a source / needs a human / no source", not merely a rank.

**Further references**
- Robertson & Walker 1994, *Some Simple Effective Approximations to the 2-Poisson Model for Probabilistic
  Weighted Retrieval*, SIGIR '94, 232–241, `10.1007/978-1-4471-2099-5_24`
- Robertson & Zaragoza 2009, *The Probabilistic Relevance Framework: BM25 and Beyond*, Foundations and
  Trends in IR 4(1), 1–174, `10.1561/1500000019`
- Manning, Raghavan & Schütze 2008, *Introduction to Information Retrieval*, Cambridge,
  `10.1017/CBO9780511809071`

> This section produces no method, only one sentence: why this paper must set a threshold.
> **But it must be guarded against backfiring**: IR avoids absolute thresholds precisely because
> **absolute scores are not comparable** (score scales differ across queries). The $t$ here is comparable
> because **the score is defined on the same ruler** (the LCS length of **the same sentence**; sources differ but
> the unit is the same) —
> **but that guarantees only a common unit, not a common null distribution**: $P_0(\mathrm{LCS}\ge t)$
> still changes with sentence length, source length, character set and text distribution.
> So what the next sentence supports is only "**within the same corpus**, the same $t$ reads the same
> $\alpha$", **not** "the $t$ transfers across datasets". This must be written, or one reviewer sentence
> — "your threshold does not transfer across datasets" — pierces §5.

---

## 2.5 Three-band output and operating point (Fellegi–Sunter / target-decoy)

**What it does**: the three bands here — "has a source / needs a human / no source" — are not invented;
they are match / clerical review / non-match from record linkage, and that framework also supplies a
theorem: **given upper bounds on the two error rates, ordering by likelihood ratio and setting two
cut-offs minimises the share sent to clerical review**.

**Relation to this paper**: the three-band structure (§3.1.1 concedes it).

**What this paper adds**: **the score here is not a likelihood ratio; $\mathrm{LCS}$ and $\Lambda$ are not
equivalent** — §5.6.1 measures the inversion $\Lambda(5)=4.71>\Lambda(6)=1.80$, showing that ordering by
$\mathrm{LCS}$ **does not inherit that optimality guarantee** — **it may waste review capacity that could
have been saved, but how much was not measured**. **This is the only "structural" increment this chapter
can claim, and its form is negative (a missing guarantee), not "a known N extra items sent".**

**Another line to cite**: the target-decoy tradition from mass spectrometry — "construct negatives to
estimate FDR" is not an improvisation of this paper but a standard practice of over a decade.

### 2.5.1 Why this paper does **not** use target-decoy to estimate $\mathrm{FDR}$

Target-decoy is the standard way to estimate $\mathrm{FDR}$ in proteomics: spike the library with
"decoys" (shuffled or reversed sequences); **the number of decoy hits** then gives a direct estimate of
false positives, so $\mathrm{FDR}\approx\#\text{decoy}/\#\text{target}$ (times two when searching
separately).

**It was not adopted, for three reasons:**

1. **There is no decoy library here.** Decoys must be **drawn from the same distribution as the material
   under test** and **can only be false positives**. The 1,803 negatives here are **constructed** (by
   pairing other documents), not shuffled out of the material under test — they do not satisfy "same
   distribution".
2. **The factor of two depends on "two libraries of equal size, half the false positives each"; neither
   holds here.** 1,803 against 308 is a size ratio of 5.85; and the false-positive rate ($\mathrm{EPR}$)
   and the error rate on positives ($\mathrm{ENR}$) **have different denominators to begin with**.
3. **The $\mathrm{FDR}$ here has a recomputable definition** (the Bayes conversion in §2.5.2), so a second
   estimator is not needed.

> **A pitfall that has to be named explicitly**: if that formula were copied over,
> $$\mathrm{FDR}\overset{?}{\approx}2\times\frac{\text{decoy hit rate}}{\text{target hit rate}}$$
> it would differ from the $\mathrm{FDR}=\dfrac{\pi_0\mathrm{EPR}}{\pi_0\mathrm{EPR}+(1-\pi_0)(1-\mathrm{ENR})}$
> of §2.5.2 **by a factor of $2/\pi_0$** (**6.67 times** at $\pi_0=0.3$; substituting $\mathrm{EPR}=2.995\%$
> and $\mathrm{ENR}=26.57\%$ gives 11.45% and 1.72% respectively).
> **Where the difference lies**: the former estimates "the false-positive fraction that decoys imply", the
> latter "given a prior, how many of those judged sourced are truly unsourced" — **two different
> quantities**.
> **Only the latter is used here** (every FDR statement in §5.8 rests on it, recomputably).
>
> Two quantities with different names and different formulas must not be listed side by side merely
> because both are called FDR.

### 2.5.2 $\mathrm{EPR}\ne\mathrm{FDR}$ (the two words most easily confused in this paper)

| Quantity | Denominator | Meaning | Section here |
|---|---|---|---|
| $\mathrm{EPR}(t)$ | the negative pool, 1,803 | the share of **unrelated pairs** judged sourced | §5.4 / 5.7 |
| $\mathrm{ENR}(t)$ | sentences that **are** sourced (denominator at most $n_1=308$) | the share of genuinely sourced items judged unsourced (wrongly-rejected rate, a conditional probability) | §5.2 |
| $\mathrm{FDR}(t)$ | items judged **sourced** | the share of those that are **truly unsourced** | this section |

> **The literal value $207/308=67.21\%$ is not $\mathrm{ENR}$.**
> **Nor is it an upper bound on $\mathrm{ENR}$**: the number wrongly rejected is $\le207$ and the number
> truly sourced is $n_1\le308$ — **those two bounds do not imply $\mathrm{ENR}\le207/308$** (if $n_1=207$,
> $\mathrm{ENR}$ can reach 100%).
> Its precise definition is: **the value under the specific accounting "treat all 308 as genuinely
> sourced"** — an accounting, not a bound. **Calling it an upper bound mistakes an accounting for a bound.**
> $\mathrm{ENR}$ is a **conditional** probability: $P(\text{judged unsourced}\mid\text{genuinely sourced})$.
> That is exactly what the $(1-\pi_0)(1-\mathrm{ENR})$ in the conversion below needs; substituting an
> unconditional proportion for a conditional probability biases the whole $\pi_0$ sensitivity analysis.

The conversion ($\pi_0$ = the prior that a candidate is in fact unsourced):

$$\mathrm{FDR}=\frac{\pi_0\,\mathrm{EPR}}{\pi_0\,\mathrm{EPR}+(1-\pi_0)(1-\mathrm{ENR})}$$

**$\pi_0$ is unknown** (determining it would require human checking of all 308 items, which is exactly
what §4 does), so only a **sensitivity analysis** is given, never a single point. At $t=7$, with
$\mathrm{ENR}$ as a point estimate:

| $\pi_0$ | 0.1 | 0.2 | 0.3 | 0.5 | 0.7 |
|---|---:|---:|---:|---:|---:|
| $\mathrm{FDR}(7)$ | 0.45% | 1.01% | 1.72% | 3.92% | 8.69% |

> The table is computed at full precision, $\mathrm{EPR}=54/1803=2.995\%$, while the text displays it as
> 3.00% — recomputing with 3.00% differs in the second decimal ($\pi_0=0.5$ gives 3.93 rather than 3.92,
> $\pi_0=0.7$ gives 8.70 rather than 8.69). **The difference is between a displayed and a computed value,
> not an arithmetic error.**

> The table also works in reverse, and that is the more useful direction: **given an auditor's target
> $\mathrm{FDR}$, invert it into an upper bound on $\mathrm{EPR}$**.
> With a target $\mathrm{FDR}\le5\%$ and $\pi_0=0.3$, $\mathrm{EPR}$ need only be $\le9.02\%$ — measured
> here at 3.00%, **there is headroom**. The conversion replaces "is my false-positive rate low" (an
> absolute quantity with no scale) with "is it low enough given $\pi_0$" (which has a scale).

#### 2.5.2a What must be conceded: this expression is **identical** to the PPV formula from intrusion detection

The conversion above is not this paper's. With $p=1-\pi_0$, $P_D=1-\mathrm{ENR}$, $P_{FA}=\mathrm{EPR}$, it
is the positive predictive value that intrusion detection has used for over twenty years:

$$\mathrm{PPV}=\frac{P_D\,p}{(P_D-P_{FA})\,p+P_{FA}}\qquad\text{(Axelsson 2000, Eq. 4)}$$

**Measured identical** (`S1·_基率_EQ_FDR.py`, 11 self-test assertions): over 2000 random parameter sets the
maximum absolute deviation is **$<10^{-12}$** (numerically zero, floating-point error only).

> **So this paper cannot claim any "conversion contribution"** — it is a direct consequence of Bayes'
> theorem, written in PPV form in the security literature by 2000.
> What can be claimed here is only "**applying it to citation checking, and giving a feasible region with
> numbers**". This is the second self-demotion in this section (the first being the cost lineage in 2.3).

**Upgrade: from a sensitivity table to a feasible region** (Axelsson's inversion)

If the auditor requires that **at least a proportion $1-p$ of the items judged "sourced" be genuinely
correct**, then from $\mathrm{PPV}\ge 1-p$:

$$\mathrm{EPR}\ \le\ \frac{p\,(1-\pi_0)\,(1-\mathrm{ENR})}{(1-p)\,\pi_0}$$

> **This is easy to get wrong**: swapping $\pi_0$ with $1-\pi_0$ and $p$ with $1-p$ and substituting
> $p=0.05,\pi_0=0.3,\mathrm{ENR}=26.57\%$ gives $5.98$ ($>1$, i.e. no constraint at all).
> Re-derivation: $\mathrm{PPV}=\dfrac{(1-\pi_0)(1-\mathrm{ENR})}{(1-\pi_0)(1-\mathrm{ENR})+\pi_0\mathrm{EPR}}\ge 1-p$
> $\Rightarrow p(1-\pi_0)(1-\mathrm{ENR})\ge(1-p)\pi_0\mathrm{EPR}$ $\Rightarrow$ the expression above.
> Substituting gives $0.0902$, consistent with the **9.02%** quoted at the start of this section.
> **This kind of error raises no error at all; it surfaces only when the numbers are substituted.**

→ **It can be read in reverse**: given a target $\mathrm{FDR}$ and a $\pi_0$ assumption, the expression
gives an upper bound on $\mathrm{EPR}$ — compare it with the measured 3.00% and the headroom is
immediate. For example, with a target $\mathrm{FDR}\le5\%$ and $\pi_0=0.3$, the bound is
$\mathrm{EPR}\le9.02\%$, against a measured 3.00%, **headroom**.
> **Only a single substitution is made here, not a full table**: $\pi_0$ is an unknown in this paper,
> and a full table would suggest it had been estimated.

**It also explains an old puzzle in §5.8**: the Axelsson expression has a direct consequence —
**as $P_{FA}\to0$ while $P_D$ falls more slowly, $\mathrm{PPV}\to1$.**
> **It must not be written as "even if $P_D\to0$"**: when both go to zero the limit depends on their
> relative rates. With $P_D=P_{FA}=x\to0$, substitution gives $\mathrm{PPV}=p=1-\pi_0$ (not 1); with
> $P_D=x^2,P_{FA}=x$, $\mathrm{PPV}\to0$. **"Each tends to zero" is not "tends to zero along an arbitrary
> path".**

Translated: **push the threshold all the way up, judge almost everything "unsourced", and the few items
judged "sourced" become very reliable** — because almost nothing is judged sourced.
> **But this sentence must carry the caveat above, or it contradicts its own counterexample**: "almost
> nothing is judged sourced" **only makes $P_{FA}$ small**; it does not guarantee $\mathrm{PPV}\to1$ — the
> path $P_D=x^2,P_{FA}=x$ reaches 0. The accurate statement is: **when $P_{FA}$ falls faster than $P_D$,
> $\mathrm{PPV}\to1$**; at comparable rates it need not. **"Rarely" is not "accurately".**
> **This is exactly the origin of the "trivial solution" at $t=13$ in §5.8.** It is a special case of an
> identity, not a discovery.

**Reference**: Axelsson, S. 2000, *The Base-Rate Fallacy and the Difficulty of Intrusion Detection*, ACM
TISSEC **3**(3), 186–205, `10.1145/357830.357849` (a conference version appeared at CCS **1999**,
`10.1145/319709.319710`, with "its implications for" in the title; the journal version is cited).

### 2.5.3 A free sanity check

Solving $\mathrm{FDR}=\pi_0$ gives $\mathrm{EPR}+\mathrm{ENR}=1$.

**Only when the decision rule is completely uninformative at that operating point (equivalent to random
guessing) does $\mathrm{FDR}$ equal the prior $\pi_0$.**

> **Two points must be narrowed**:
> ① **$\mathrm{EPR}+\mathrm{ENR}=1$ is a property of one operating point, not of the whole rule** — a
>    scorer with overall discriminative power can perfectly well **land on the chance line at some
>    threshold** (Youden $J=1-\mathrm{EPR}-\mathrm{ENR}=0$). **Which kind of "uninformative" must be
>    stated**: it is **a single operating point** that lands on the chance line, not the entire curve.
> ② **Boundary cases were missed**: whenever $\mathrm{FDR}$ is defined, $\pi_0=0$ or $\pi_0=1$ both give
>    $\mathrm{FDR}=\pi_0$, **without requiring the two rates to sum to 1**.
> → The accurate statement: **in the interior $0<\pi_0<1$, $\mathrm{EPR}+\mathrm{ENR}=1$ is equivalent to
> "$\mathrm{FDR}=\pi_0$ at that operating point"**, i.e. that point performs like random guessing.
> (**Not** when $\mathrm{EPR}=\mathrm{ENR}$ — equality of those two quantities is not a criterion at all.)

And from $\mathrm{FDR}<\pi_0\iff\mathrm{EPR}+\mathrm{ENR}<1$ it follows that **this criterion is
independent of $\pi_0$** — whether the screening has any gain depends only on whether the two error rates
sum to less than 1. Measured at $t=7$: $3.00\%+26.57\%=29.57\%$, **far below 1**.

> Conversely it draws a line: **$\mathrm{FDR}$ equals $\pi_0$ exactly only when $\mathrm{EPR}+\mathrm{ENR}$
> equals exactly 1.**
> **It must not be written as "close to 1 and we are back at $\pi_0$"**: counterexample —
> $\pi_0=0.3$, $\mathrm{EPR}=0.01\%$, $\mathrm{ENR}=98.99\%$ (sum 99%), substituted into the PPV
> expression gives $\mathrm{FDR}\approx0.42\%$, still far from 30%. **What holds at "equals" cannot be
> extrapolated to "close to".** This paper is far from that line (29.57% versus 100%).

> Generated with `S1·_FDR_decoy.py` (14 self-test assertions).

**Further references**
- Fellegi & Sunter 1969, *A Theory for Record Linkage*, **JASA 64(328), 1183–1210**,
  `10.1080/01621459.1969.10501049`
- Elias & Gygi 2007, *Target-decoy search strategy for increased confidence in large-scale protein
  identifications by mass spectrometry*, Nature Methods 4, 207–214, `10.1038/nmeth1019`

---

## 2.6 The four things this paper can claim (fixed at the end of the chapter)

> **The order must not change**: putting "threshold = cost" first would read as "the most important
> thing here is instantiating a known theorem" — **it is not.**

| # | Increment | How minimal it is |
|---|---|---|
| **1** (main result) | On nested decision sets, set-overlap metrics are **non-identifiable** for the false-negative rate; this paper applies it to citation checking and **measures the dose**: $J=0.5$ can correspond to 18% and 80% (a 62-point gap), the real data contain **127 counter-examples at zero distance**, and all ten LCS buckets contain both judgements | † **The mechanism and the phenomenon both have prior sources.** Nestedness degeneracy: Baselga 2010 *GEB* / Webber–Moffat–Zobel 2010 *TOIS* / Fagin 2003 / Koleff et al. 2003; "overlap metrics disagree with human judgement": over a decade of measurement in NLP evaluation (§6.0.1). **Three increments here: the word (non-identifiability, not weak correlation), the dose, and where it forces the choice.** **"First to prove $J$ degenerates" is forbidden**; full discussion and references in §6.0 |
| **2** | **First applying the standard conclusion "threshold = cost assumption" to citation checking, and measuring the concrete interval** ($w^*=1.16$–$1.41$) | † **Demoted wording**: the principle is standard — **Neyman–Pearson 1933 → law and economics (Kaplow 2012) → cost-sensitive learning (Elkan 2001)** — and **the contribution is instantiation and quantification, not the principle**. Neither "first to discover" nor "first application outside economics" |
| **3** | Reading $\mathrm{EPR}(t)$ as $\alpha$, doing **multiple comparisons** in $\alpha$ coordinates, and concluding "under FWER the available data cannot certify it; under FDR, $t\ge9$" | A **concrete arithmetic result**, not a framing |
| **4** | Showing that $\mathrm{LCS}$ is **not** a monotone proxy for a likelihood ratio ($\Lambda$ inverts at $k=5\to6$), so the three cut-offs do not inherit the Fellegi–Sunter optimality | The measurement most specific to this paper |

> **Three claims not permitted**: ① "first to find that a threshold implies a cost"; ② "first to point
> out that a single scalar loses information"; ③ "first to judge provenance by lexical overlap".
> **A fourth not permitted**: **"first to apply the formalisation of the standard of proof outside
> law"** — burden-of-proof analysis in law and economics is itself about non-legal settings (internal
> organisational decisions, contract acceptance criteria, medical treatment), and that sentence is knocked
> down by a single citation.

---

## 2.7 How this chapter relates to the rest

- Every **cost number** in §5 appears once here (the "known theorem" of 2.3) and **is not reported twice**;
- **§8** draws on two items from here: 2.3 (the single-rater design is sub-standard) and 2.5
  (LCS ≠ likelihood ratio);
- The contribution list in **§1** copies the four items of 2.6 directly, **with no extra item and no change
  of order** (the main result must come first).

---

# 3 Problem Formalisation

## 3.0 Three explicit assumptions

**A1 (labels available).** The dataset carries labels $y_i\in\{0,1\}$. In the main dataset
**every item is positive** ($n_1=308$, $n_0=0$); negatives come from a separately collected
sample (§5.3). The two denominators are never merged.

**A2 (monotone criterion).** A higher score makes "sourced" more likely: increasing
$\mathrm{LCS}$ never flips a decision from 1 back to 0. This guarantees that decision sets are
**monotonically nested in the threshold**.

**A3 (treatment of the to-human category).** How the to-human category (written C) enters the
error statistics **is not a mathematical question but three different accountings**: upper bound
(C counts as found) / point (C removed from the denominator) / lower bound (C counts as not
found). We compute and report all three (§5.2.2). This section only explains **why the category
exists** (§3.1.1 and below).

**Which proposition needs which assumption.** This table is the skeleton of the paper.

| Proposition | Requires | Form |
|---|---|---|
| **2a** (size ratio) | **not A1**, only A2 | $J=\|S(t_1)\|/\|S(t_2)\|$ — pure set theory |
| **2a′** ($\hat F_1$ form) | **A1 + A2** | $J=\hat F_1(t_1-1)/\hat F_1(t_2-1)$ — uses the positive-subsample CDF |
| 1 | A1 only | $\mathrm{ENR}=\hat F_1$ shifted |
| 2b | A1 + A2 | without A1, $\mathrm{ENR}$ is undefined and the proposition cannot be stated |

**The literature has A2 and lacks A1, so it can only report stability. This paper supplies A1,
which is what allows stability and decision quality to be placed side by side — and what reveals
that the former carries no information about the latter.**

---

## 3.1 The decision rule, thresholds, and two different senses of "merging"

The object under audit is a **decision rule**:

$$D(x;\theta) \rightarrow \{1,\,0,\,?\}$$

where $x$ is an item under review and $\theta$ an ordered scale parameter. The three outputs read
"sourced", "unsourced", "to-human". The symbol $?$ is not a missing value: it is the rule
**declaring its own uncertainty**.

**Why this category exists must be stated precisely.** It is **not a mathematical necessity** — a
forced two-way decision computes $\Pr(D(x)\ne y)$ perfectly well, and the error rate does not
"lose its meaning". It is a **design choice**: in citation verification, forcing a verdict on a
borderline case costs more than marking it for a human. **It is a trade-off, not a theorem.**

The concrete instance is a text-provenance rule: with $\mathrm{LCS}(x,s)$ the longest common
**substring** length and cut-offs $\theta=(lo,hi)$,

$$\mathrm{LCS}\ge hi \Rightarrow 1,\qquad lo<\mathrm{LCS}<hi \Rightarrow ?,\qquad \mathrm{LCS}\le lo \Rightarrow 0$$

### 3.1.0 Substring is not subsequence, and not ROUGE-L

Our $\mathrm{LCS}$ is the longest common **contiguous substring**. The well-known ROUGE-L
(Lin 2004) in NLP evaluation uses the longest common **subsequence** (not necessarily contiguous)
with length normalisation. **These are not the same quantity**, and we state the difference
explicitly to avoid being read as reinventing a standard metric.

| | This paper (substring, absolute length) | ROUGE-L (subsequence, normalised) |
|---|---|---|
| What is matched | a contiguous block | a possibly non-contiguous sequence |
| Inserting one word | **breaks** it (11 characters before and 8 after become two blocks) | does not break (still embeds in order) |
| Reordering words | **breaks** it | **also drops sharply** — a subsequence preserves order, so the LCS of `ABCD` and `DCBA` falls from 4 to 1 |
| Normalisation | none (absolute length) | $R_{\mathrm{LCS}}$ / $P_{\mathrm{LCS}}$ / $F_\beta$ |
| Readable as a significance level $\alpha$ | **yes, within one corpus** (fixed threshold ↔ fixed $\alpha$, §5.8; the null distribution moves with the text distribution, so this is not a cross-corpus constant) | also yes, once a normalisation convention is fixed — **not "no"** |

**How far apart the two definitions are, measured** (`S1·_子串vs子序列.py`, 10 self-tests):

| Source | Sentence under review | Change | Substring | Subsequence | Our band |
|---|---|---|---:|---:|---|
| The study reduced latency by reordering candidate passages | …passages did **not** reduce latency | one character inserted mid-string | **11** | 15 | to-human (7–11) ← **blocked** |
| The method achieved the best result on all three datasets | …the **worst** result | antonym swapped at the end | **15** | 18 | **sourced (≥12) ← missed** |
| The system judged every test sample as sourced | …judged as **un**sourced | antonym swapped at the end | **14** | 16 | **sourced (≥12) ← missed** |

**Row 1 shows the substring version is more conservative** (a single insertion breaks the run and
the item falls into "to-human"). **Rows 2 and 3 show it is not foolproof** — with a long identical
prefix and a change near the end, it still judges "sourced" while the meaning is inverted. Both
must be stated: **the substring is a stricter choice, not a safe one.** (This failure mode appears
again in §8.)

**Three reasons for substring rather than subsequence:**

1. **Citation verification must be conservative**: better to send more to a human than to accept
   too readily. The substring version pushes borderline cases **toward human review** — the
   direction is "block", not "reject".
2. **Contiguity is itself evidence.** Twelve consecutive identical characters are almost certainly
   copied from one place; twelve characters scattered across a source of tens of thousands of
   characters may be ordinary vocabulary on a shared topic.
3. **Absolute length with a fixed threshold makes the inversion for $\alpha$ direct** (§5.8).
   After length normalisation, the same score means different things on sentences of different
   lengths, and reading it as $\alpha$ first requires fixing a normalisation convention — doable,
   but one more choice, and one that must stay within a single corpus.

All experiments use $hi=12$, $lo=6$. To phrase threshold sensitivity as a single-parameter
problem we write the **cutoff threshold** $t$:

$$S(t) = \{x : \mathrm{LCS}(x) < t\}\quad\text{(judged unsourced)},\qquad t = lo+1 = 7$$

### 3.1.1 The three-way structure is Fellegi–Sunter 1969, not our invention

The classical record-linkage framework partitions a pairwise comparison into three regions —
**match / clerical review / non-match** — and orders by the **likelihood ratio**
$\Lambda(x)=\Pr(x\mid\text{same entity})/\Pr(x\mid\text{different entity})$. Its central result:
**given upper bounds on the two error rates, ordering by $\Lambda$ with two cut-offs minimises the
proportion sent to clerical review.**

| Fellegi–Sunter | This paper |
|---|---|
| match | sourced ($\mathrm{LCS}\ge12$) |
| clerical review | to-human ($7\le\mathrm{LCS}\le11$) |
| non-match | unsourced ($\mathrm{LCS}\le6$) |

**A second source, whose middle category is semantically closer to ours.** The three-way structure
has an independent counterpart in attribution evaluation — **AttrScore** (Yue et al. 2023, Findings
of EMNLP):

| AttrScore | Meaning | Here |
|---|---|---|
| **Attributable** | the reference fully supports the claim | sourced |
| **Extrapolatory** | the reference **lacks enough information** to decide | to-human |
| **Contradictory** | the reference **contradicts** the claim | unsourced |

> **Two different C's must be distinguished here, and they do not overlap.**
> · **the to-human the rule itself assigns** — $\mathrm{LCS}\in[7,11]$, **62 items**, produced by a
> **rule**;
> · **the C a human labeler calls "unclear"** — produced by a **person** on the 100-item stratified
> sample.
> Extrapolatory's semantics ("the source does not contain this information") are closer to the
> **latter**; the three-way **structure** corresponds to the **former**. **Same name, different
> object — a citation must say which.**

**The two sources are not the same thing, and must be kept apart.** Fellegi–Sunter's middle
category is "**not enough evidence, hand it to a human**" (procedural); AttrScore's Extrapolatory
is "**the source does not contain this information**" (semantic) — closer to what our C actually
means. So the three-way structure here is neither arbitrary nor copied from a single source:
**the procedural structure comes from record linkage, the semantic content from attribution
evaluation.** Nor is this paper equivalent to AttrScore: it classifies three ways with a model,
we classify three ways with a lexical threshold — **same structure, different criterion**.

**The deepest difference between the two frameworks lies in the score itself**: Fellegi–Sunter
scores with the likelihood ratio $\Lambda$; we score with $\mathrm{LCS}$.

**Two points must be stated, or this will be read as borrowed.** First, **the structure has prior
sources; the cost does not**: Fellegi–Sunter gives a theorem that two cut-offs are optimal *for a
given pair of error rates*; it does not say *what those error rates should be*. The latter is a
cost judgement, and that is the subject of §5. The two are complementary, not duplicative.
Second, **our score is not a likelihood ratio**: $\mathrm{LCS}$ is a coarse proxy for $\Lambda$,
equivalent only where $\Lambda$ is monotone in $\mathrm{LCS}$. §5.6 measures this and finds the
monotonicity **inverted at $k=5\to6$** — **a contribution this paper can claim**, and **the reason the
three-way structure here does not inherit Fellegi–Sunter's optimality guarantee**.

> **We may say "does not inherit the guarantee"; we may not say "has been shown to be
> suboptimal".** A single inversion does not settle optimality, and moreover $\mathrm{LCS}=5$ and
> $6$ fall in the **same band** ($\le6$ is uniformly unsourced), so **this inversion does not even
> change the banding under the two operative cut-offs**.

> **The population of $S(t)$.** The definition above is over **all items**. Every later use of
> $|S(t)|=n_1\hat F_1(t-1)$ requires $S(t)$ to lie **inside the positive subsample**. The main
> dataset has $n_0=0$ (all items positive), so the two coincide automatically; but as a general
> proposition this must be said, or the derivation in §3.4 is disconnected from the definition here.

**Two senses of "merging", which are not the same thing.**

| | Merged? | Why |
|---|---|---|
| **Reporting level**: false negatives and false positives | **not merged** | weighting requires a cost ratio, which the deployer must supply. Collapsing them into one "total error rate" performs their value judgement for them |
| **Analysis level**: the three-way output | **collapsed to binary** (only at a single $t$) | otherwise "error rate" has no unique denominator. This is an **analytical simplification**, not a reporting one |

**In one sentence: what is not merged is the two error types; what is collapsed is the three-way
output. They sit at different levels and coexist.**

### A3: how the to-human category enters the denominator

Computing $\mathrm{ENR}$ at $t=7$ leaves three ways to treat the to-human band
($7\le\mathrm{LCS}\le11$), **differing by 20 percentage points**:

| Accounting | Numerator | Denominator | ENR(7) | Effect on the rule |
|---|---|---:|---:|---|
| **① not counted as an error (adopted)** | 207 | 308 | **67.21%** | **most favourable** (to-human counts as "not wrong") |
| ② folded into "unsourced" | 269 | 308 | 87.34% | severest |
| ③ removed from the denominator | 207 | 246 | 84.15% | exempts the rule |

**We adopt ①.** The to-human category means "the rule reached no conclusion"; counting it as an
error **deducts marks on its behalf**, and removing it from the denominator **exempts it on its
behalf**. ① gives it the widest latitude.

> **All three rows rest on the same assumption: that all 207 items judged unsourced are wrongful
> rejections** (② additionally folds in the 62 to-human items, which is why its numerator is 269
> rather than 207). **Human labels falsified this assumption**: of the 207, only **53–124 are
> actually findable in their sources (point estimate 82)**.
>
> **"We adopt ①" binds the *rule* of ①, not the *numbers* of ①.** The values printed in that row
> — 67.21 / 87.34 / 84.15 — **are void and must not be cited as true values**; **the rule of ①
> remains in force.** This is not a table beyond repair: on the contrary, **the three accountings
> are recomputed from the 53–124 range along the same route** (upper 40.14 / point 26.57 /
> lower 17.30; §5.2.2). What is void is **the numbers in this table**, not **the route "how the
> to-human band is counted"**.

> **Two axes; their landing points must be stated separately, or a reader will multiply them.**
> · **This table's axis**: does the to-human band (call it L4) enter the numerator? We say no — L4
> enters neither the numerator nor, by removal, the denominator. In the arithmetic: **the numerator
> counts only the A labels within L1–L3; the denominator remains all 308, including those 62**.
> · **The three-accounting axis**: within L1–L3, how is a human "unclear" (C) counted? Upper bound
> counts it found / point removes it / lower bound counts it not found.
> · **The two axes have disjoint item sets**: L4 is **62 to-human items**; L1–L3 is **207 items
> judged unsourced**; no item belongs to both. The axes therefore combine orthogonally, and **the
> combination is exactly the three figures reported in §5.2.2 — those are the numbers this paper
> finally cites.**

> **① is the *lowest* of the three error rates — the one most favourable to the rule.** It is not
> "conservative"; in statistical usage "conservative" means not overstating an effect, and the
> direction here is the opposite.
>
> **This is not a weakness but the strongest sentence in the paper**: the argument is that the
> error rate is high, and we use the *lowest* of three available numbers — **under the old
> accounting, even under the accounting most favourable to the rule, the false-negative rate still
> reaches 67.21%.**
>
> **After the human labels, that sentence must be recomputed**: the true value (point estimate) is
> **26.57%**, not 67.21%. 67.21% is an upper bound reproduced by no accounting. So we no longer
> use "even the most favourable gives 67%" as the main weapon. The strongest sentence now
> available is: **"even under the accounting least favourable to the rule (upper bound), the
> false-negative rate reaches 40.14%; under the point accounting it is 26.57%."** The point
> estimate is about six tenths lower than the original figure (67.21 → 26.57, a drop of 60.47%),
> but it is backed by the labels.

**One further observation (§5 uses it):** accounting ② gives $(207+62)/308=269/308=87.34\%$,
**exactly the value at $t=12$.** This is not a coincidence: $t=12$ is
$\{\mathrm{LCS}<12\}=\{\mathrm{LCS}\le11\}$, i.e. **the to-human band abolished and folded entirely
into unsourced**. So $t=12$ can be read as the cost of the policy "no human review".

**Notation discipline.** $S(t)$ is **monotone increasing** in $t$:
$t_1<t_2\Rightarrow S(t_1)\subseteq S(t_2)$.

---

## 3.2 Two error rates and their populations

Positive subsample $\{(x_i,y_i=1)\}_{i=1}^{n_1}$, negative subsample
$\{(x_j,y_j=0)\}_{j=1}^{n_0}$ (collected independently, §5.3).

$$\mathrm{ENR}(t) = \frac{\#\{i:y_i=1,\ \mathrm{LCS}(x_i)<t\}}{n_1}\qquad\text{(false-negative rate)}$$

$$\mathrm{EPR}(t) = \frac{\#\{j:y_j=0,\ \mathrm{LCS}(x_j)\ge t\}}{n_0}\qquad\text{(false-positive rate)}$$

**Terminology discipline.** We write "false-negative rate" throughout, and **never "misjudgement
rate"**. The main dataset has $n_0=0$, so a quantity including false positives cannot be formed on
its denominator; calling it a "misjudgement rate" would **pass a conditional error rate off as a
total one**.

---

## 3.3 Proposition 1: the error curve is the score distribution of the positive subsample

> **Proposition 1.** Let
> $$\hat F_1(k) = \frac{\#\{i:y_i=1,\ \mathrm{LCS}(x_i)\le k\}}{n_1}$$
> be the empirical cumulative distribution function **on the positive subsample**. Then
> $\mathrm{ENR}(t) = \hat F_1(t-1)$.

**Proof.**
$\mathrm{ENR}(t)=\#\{\mathrm{LCS}<t\}/n_1=\#\{\mathrm{LCS}\le t-1\}/n_1=\hat F_1(t-1)$. ∎

> The premise is that LCS takes integer values (always true here); for non-integer thresholds one
> writes $\hat F_1(\lfloor t\rfloor)$. The population **must** be the positive subsample — read
> literally as the full sample, the CDF would not match $\mathrm{ENR}$'s positivity condition and
> any recomputation would fail. The subscript 1 is the reminder.

**This proposition is a threat to the paper, not an ornament, and we state it first.** It means the
main figure in §5 contains **mathematically nothing beyond the score distribution**. A reviewer may
say:

> *"The proposed ENR curve is, by construction, the empirical CDF of the score; the figure contains
> no information beyond the score distribution itself."*

**That sentence is correct, and we do not dispute it. We only point out that it refutes the wrong
target.**

1. **We do not claim to have discovered a new curve.** We claim to translate the language of audit
   from "ranking stability" into "decision error rate". The same $\hat F_1$, read as a
   distribution, is a descriptive plot; read as $\mathrm{ENR}$, it is an executable judgement —
   "a threshold of seven means **26.57%** of genuine material will be judged fabricated" (point
   estimate; the **three-accounting spread** is 17.30–40.14%, §4.6.5 — that is not a confidence
   interval).

2. **Shape has decision meaning, but not global meaning.** The per-unit slope differs by **a factor
   of 8.38** across different segments of the distribution. Source (§5.2.1; accounting §2.2):

   | Interval | Increase per step (percentage points) |
   |---|---:|
   | $t=2\to3$ (largest) | **21.75** |
   | $t=11\to12$ and $t=12\to13$ (smallest, tied) | **2.60** |

   $67/8 = 8.375 \to$ **8.38×** (computed from single-step **count differences**). So "can the threshold be moved"
   is not a global property but a property of **which segment you stand in**. All eleven adjacent
   intervals are recomputed step by step in §5.2.1.

3. **Proposition 1, read the other way, supports the paper.** Precisely because $\mathrm{ENR}$ and
   $\hat F_1$ are the same object, **reporting an error rate costs nothing once labels exist** — no
   new model, no retraining, only a reading of the distribution already in hand in decision terms.
   The barrier is low enough that there is no excuse for not doing it.

**We explicitly do not claim** that this identity is new. It follows directly from the definitions.

---

## 3.4 Proposition 2: set overlap carries no information about the error rate

$$J\big(S(t_1),S(t_2)\big) = \frac{|S(t_1)\cap S(t_2)|}{|S(t_1)\cup S(t_2)|}$$

> **Proposition 2a (needs only A2, not A1).** If decision sets nest monotonically in the threshold
> ($t_1\le t_2\Rightarrow S(t_1)\subseteq S(t_2)$, **including the equal-size case**) **and
> $S(t_2)\ne\varnothing$**, then
> $$J\big(S(t_1),S(t_2)\big) = \frac{|S(t_1)|}{|S(t_2)|}$$
> — pure set theory, independent of both sample and labels.
>
> **$S(t_2)\ne\varnothing$ must be stated**: if $S(t_1)=S(t_2)=\varnothing$ the right-hand side is
> $0/0$, the ratio is undefined, and the proposition fails at that point. This occurs at the low end
> of the threshold scale and **cannot be waved away with "usually non-empty"**.
>
> **Proof.** Under nesting $S(t_1)\cap S(t_2)=S(t_1)$ and $S(t_1)\cup S(t_2)=S(t_2)$; substitute. ∎

> **Corollary 2a′ (needs A1 + A2).** If $S(t)$ lies inside the positive subsample, then
> $|S(t)|=n_1\hat F_1(t-1)$, hence
> $$J\big(S(t_1),S(t_2)\big) = \frac{\hat F_1(t_1-1)}{\hat F_1(t_2-1)}$$
> i.e. $J$ **reads $\hat F_1$ at exactly two points**.

**Numerical check** (the three readings used in §6, matching digit for digit):

| Comparison | Arithmetic | $J$ |
|---|---|---:|
| $S(4)$ vs $S(7)$ | $120/207$ | 0.5797 |
| $S(6)$ vs $S(7)$ | $187/207$ | 0.9034 |
| $S(7)$ vs $S(8)$ | $207/223$ | 0.9283 |

**The reference point cannot be dropped**: all three $J$ values are computed against **the decision
set at $t=7$**, so **the denominator follows the reference point** (the first two share
$|S(7)|=207$; the third uses $|S(8)|=223$). **Misusing** $J(S(4),S(8))=120/223=0.5381$ would give 0.538,
not 0.580.

> **Proposition 2b (a single reading carries zero information; needs A1 + A2).** Under A1 and A2,
> **given a single $J$ reading**, $\mathrm{ENR}(t_2)$ can take any value in $(0,1]$ (at the level of
> the population CDF). The proposition is stated for **one** reading; **the direction qualifier on
> the sweep, stated below, is part of the claim** and is not an afterthought.
>
> **Proof.** By 2a′, $J=\hat F_1(t_1-1)/\hat F_1(t_2-1)$ and
> $\mathrm{ENR}(t_2)=\hat F_1(t_2-1)$. Given $J\in(0,1]$ and any target $e\in(0,1]$, set
> $$\hat F_1(t_2-1)=e,\qquad \hat F_1(t_1-1)=J\,e$$
> and place the remaining mass $1-e$ entirely above $t_2-1$ (e.g. $\hat F_1(k)=1$ for all
> $k\ge t_2$). Since $Je\le e\le1$, the resulting function is non-decreasing, right-continuous and
> ends at 1: a valid CDF. ∎
>
> **Two qualifiers belong inside the statement of the proposition**: ① **under A1 and A2** (without
> A1, $\mathrm{ENR}$ is undefined); ② **a single $J$ reading**.
> **Finite-sample granularity**: with $n_1$ finite, $\mathrm{ENR}$ can only take multiples of
> $1/n_1$, so "any value" is strictly at the level of the population CDF. The grid spacing **shrinks
> toward zero** as $n_1$ grows (about 0.32 percentage points at $n_1=308$), **which does not change the
> zero-information conclusion** — **however dense the grid, it never fills $(0,1]$**.
> When $J=1$, $Je=e$, the two values coincide and $S(t_1)=S(t_2)$, the equal-size nested case
> already covered.

**Counterexample (recomputable, `S1·_反例_JvsENR.py`)**

| Pair | Rule | $J=\hat F_1(2)/\hat F_1(6)$ | $\mathrm{ENR}(7)$ | Minimum $n_1$ |
|---|---|---:|---:|---:|
| α | A | **0.500000** | 18.00% | 100 |
| α | B | **0.500000** | **80.00%** | 5 |
| β | C | **0.222222** | 90.00% | 10 |
| β | D | **0.222222** | 45.00% | 20 |

Same $J$, error rates **62.00 points apart** (pair α) and **45.00 points apart** (pair β).

Stronger construction: fix $J=0.5$ and **invert** to ask for the **minimum** number of positive
samples needed to realise a given $\mathrm{ENR}$:

| Target $\mathrm{ENR}$ | $\hat F_1(t_1-1)$ | $\hat F_1(t_2-1)$ | Actual $J$ | Minimum $n_1$ |
|---:|---:|---:|---:|---:|
| 1% | 0.0050 | 0.0100 | 0.500000 | 200 |
| 50% | 0.2500 | 0.5000 | 0.500000 | 4 |
| 99% | 0.4950 | 0.9900 | 0.500000 | 200 |

**→ Given $J$, every error rate from 1% to 99% is feasible.**

### A qualifier we must add ourselves

**Proposition 2b is about "a single $J$ reading", not about "the quantity $J$".**

Treated as a whole curve in $t$, **the sweep direction must be stated correctly**. Fix the
reference $t_2$ and sweep $t_1 > t_2$; then

$$J\big(S(t_2),S(t_1)\big) = \frac{\hat F_1(t_2-1)}{\hat F_1(t_1-1)} \xrightarrow[t_1\to\infty]{} \hat F_1(t_2-1)$$

(since $\hat F_1(\infty)=1$ and the reference set $S(t_2)\ne\varnothing$). The constant is thereby
determined and, point by point, $\hat F_1(t_1-1)=\hat F_1(t_2-1)/J(t_1)$.

> **The condition $S(t_2)\ne\varnothing$ cannot be dropped**: if the reference set is empty then
> $\hat F_1(t_2-1)=0$ and the recovery formula becomes $0/0$, so point-wise recovery fails. We have
> already noted that empty sets occur at low thresholds; they cannot be assumed away here.
> **Two cases must also be distinguished**: when $S(t_1)\ne\varnothing$ but $S(t_2)=\varnothing$,
> $J(t_1)=0$ (a number); **when $S(t_1)$ is also empty, $J$ is $0/0$ — undefined, not zero**.
> Writing an undefined point as $0$ supplies a reading that does not exist.

- **If only $t_1<t_2$ is swept**: $\hat F_1$ is determined on $(-\infty,t_2-1]$ only up to a
  multiplicative constant, $\hat F_1(\infty)=1$ **cannot be used**, and recovery fails.
- **Premise**: only **finite sample size** is required — an empirical CDF equals 1 beyond the
  sample maximum ($\hat F_1(\max\mathrm{LCS})=1$).
  > **Boundedness is not a necessary condition**: the empirical CDF of a finite sample is already 1
  > beyond the maximum, independent of any theoretical bound. $\mathrm{LCS}\le\min(|x|,|s|)$ only
  > guarantees that 1 is reached at a **finite** $t$ (i.e. no limit need be taken); it does not
  > affect the limit itself.

So the accurate conclusion of Proposition 2 is:

> **Reporting "robustness" through a single set-overlap reading (or a few) carries zero information
> about decision error rate in this setting; what carries information is the whole curve — and what
> the literature reports is precisely a summary reading.**
> **"Zero information" must be read with its direction**: the two arguments above cover only the
> **$t_1<t_2$** sweep direction. If a reading includes points with $t_1>t_2$, the constant can in
> principle be approached (that is the $t_1\to\infty$ limit above) — **but the approach requires
> reading far out at large $t_1$, where a reported summary number never goes.** The accurate claim
> is therefore: **within the range of readings that reports actually supply (the low-threshold side,
> or an undirected summary), it carries zero information**; it is **not** "any finite set of
> readings carries zero information", which would turn a conclusion conditional on reporting
> practice into a purely mathematical one.

**"A few" does not weaken the conclusion**: fixing $t_2$, any finite number of readings with
$t_1<t_2$ fails to determine that constant. **Note the direction built into this sentence** — what
it supports is "readings on the low-threshold side are useless", not "all readings are useless".

**§6's target of criticism is therefore restricted to "a single summary stability number of the
kind reports print", not "the Jaccard index itself".**

---

## 3.5 Relation to existing frameworks: no new method, only an absence

Threshold selection (the ROC tradition) is fifty years old, and **we claim nothing new in that
step**.

### A chain of substitutions, not a critique

Literature reports position shift — but that presupposes $N$ objects that can be ranked. A decision
rule has no cross-item ranking, only per-item correctness: position is undefined here.

```
The literature's default output is "position shift"
        ↓  only defined when there are N rankable objects
A decision rule has no cross-item ranking, only per-item correctness:
position is undefined here
        ↓
Only "set overlap" remains as a substitute
        ↓  and its single reading carries zero information about the error rate (2b)
The chain breaks at the last link — and what this paper measures is exactly that break
```

**We are not arguing that "position shift is a poor tool"; we are pointing out that position does
not exist in this setting, that the literature can only fall back to set overlap, and that the
quantity it falls back to does not carry **information about decision quality** either.**

| Already exists | We do not claim | What we do |
|---|---|---|
| Threshold selection with labels (ROC / cost-sensitive learning) | not claimed | point out that **it is absent from the robustness-audit framework for composite indicators** |
| Threshold sensitivity can reverse rankings (Kelemen et al. 2024) | not claimed | replace "ranking reversal" with "**decision error**", because a decision rule has no ranking |
| Reporting band/set stability without labels (Saisana et al. 2005; the band-switch proportions of MDPI 2025) | not claimed | prove that **a single reading of that proxy carries zero information** (2b) |

### Where the conclusion stops

**Actual evidential reach of this paper**: on a **decision rule with a monotone criterion**, a
robustness report given as **a single set-overlap reading** carries zero information about decision
error rate.

**Not claimed**: that set overlap fails in all decision settings; that the index itself is
worthless (the whole curve does carry information). **This is a question of applicability, not of
the index being good or bad.**

---

## 3.6 Chapter summary (cited by §1 and §6)

1. **Three assumptions** A1 / A2 / A3; the proposition–assumption correspondence is in §3.0.
2. **Two senses of "merging" kept apart**: what is not merged is the two error types; what is
   collapsed to binary is the three-way output.
3. $\mathrm{ENR}(t)=\hat F_1(t-1)$, with $\hat F_1$ **defined on the positive subsample**
   (Proposition 1). Not claimed as new; claimed to make the cost of reporting an error rate zero.
4. **Proposition 2a needs only A2** (size ratio, pure set theory); **Corollary 2a′ needs A1** (the
   $\hat F_1$ form).
5. Proposition 2b: **a single reading** carries zero information about the error rate, with
   counterexamples (gaps of 62 and 45 points). The claim is **qualified by item 6**: it holds for a
   single reading, in the sweep direction stated there.
6. **Qualifier**: what carries information is the whole $J$ curve (and it must be swept in the
   $t_1>t_2$ direction, with a non-empty reference set); the object of criticism can only be a
   summary single number of the kind reports print.
7. **The chain of substitutions**: position is undefined here → fall back to set overlap → **that
   fallback also fails to deliver what you wanted** (note the qualifier: **a *single reading* fails,
   not "the index carries no information"** — the whole $J$ curve does, see §6.1).
8. **Accounting ① is the one most favourable to the rule**, and it is the one we use — under the old
   accounting, "even under the accounting most favourable to the rule, the false-negative rate still
   reaches 67.21%". **Once the human labels are in, that figure becomes 26.57% (three-accounting
   spread 17.30–40.14%, not a confidence interval); the direction of this section's conclusion is
   unchanged, its force reduced — see the closing note to A3.**

> **26.57% is not "some count ÷ 308"**: it is **stratified-weighted** —
> $\widehat{\mathrm{ENR}}(7)=\frac{1}{308}\sum_{h} N_h\,p_h$, where $p_h=A_h/(A_h+B_h)$ is the
> conditional proportion **within stratum $h$** and $N_h$ the stratum's pool size. So **no integer
> count divides out to 26.57%** ($81/308=26.30\%$, $82/308=26.62\%$ — neither matches). This is not
> an arithmetic error; **it is not a count ratio**. Per-stratum counts (pools 18/66/123; draws
> 18/30/25) are in §4.6.2; the construction of the three-accounting interval is in §5.2.2.

---

# 4 Data and Scorers

> **Two claims this chapter must establish.**
> ① **The choice of scorer does not change the order of magnitude of the conclusion** — swapping in a
> mechanistically different scorer leaves the same operating point in place (**qualifier: on family
> D-pure negatives; it does not hold on family B-main, see §4.4, item 2**).
> ② **But the ground truth itself has one unverified soft spot, and it bears directly on the central
> number of §5** — this chapter must put it on the table rather than route around it.
> Data follow `S2`; scripts `S1·_第二把尺子.py` (14 self-tests) and `S1·_人判抽样.py` (12 self-tests).

---

## 4.1 Where the data comes from, and why the ground truth holds

### 4.1.1 Two pipelines

The main dataset is **308 pairs** of "sentence inside a harvested note ↔ the document it came from".
The pipeline has three steps:

```
video / paper / transcript ──① extract──▶ harvested note (our annotations + excerpts)
                                                    │
                                              ② split into sentences ──▶ sentence pool
                                                    │
                                              ③ filter out 22 internal-workflow markers
                                                    ▼
                                              308 "conclusion-grade" sentences
```

- **① Extraction** leaves 408 items.
- **② Sentence splitting**: split on full stops, question marks and line breaks; keep 12–100
  characters; drop lines beginning with "date / source / compiled by".
- **③ Filtering**: removes 100 sentences containing any of **22 internal-workflow markers** — markers
  of the form `_工具/` (tooling), `_流程/` (process), `台账` (ledger), `收工条` (shift-end note), and
  similar project-internal vocabulary. **The complete marker list is in
  `S1·_分歧实验_真实改写.py`**, so the filter is recomputable; we do not reproduce the list here.

### 4.1.2 What the ground truth is — this must be nailed down first

The original script records its own premise:

> Every harvested note was extracted from a source document or transcript, so **its substantive
> content necessarily originates in the source**. But the notes also carry our own annotations
> (pointers, checklists, "original filed"), which were never in the source; judging those
> unsourced is correct.

**So the *stipulation* for the 308 is: all sourced, $y_i\equiv 1$, $n_0=0$.**

> **It is a stipulation, not a verified ground truth**, and the difference is not a matter of
> wording. Writing "the ground truth is $y\equiv1$" would forbid us from later saying "some of these
> are unsourced by definition" — the two statements would contradict. Writing "the stipulation is
> $y\equiv1$" allows us to say: the stipulation holds as stated, but §4.1.4 finds up to 84 items that
> may violate it. **Every conclusion in this paper rests on the uncertainty of this stipulation, and
> that is where the interval comes from.**

This must be fixed, because it determines how every number in the paper is read:

| Term | Means | Must not be read as |
|---|---|---|
| the rule's "unsourced" band ($\mathrm{LCS}\le6$) | the **name of an output band** | ❌ "unsourced" in the ground truth |
| **the 207 judged unsourced** | **the rule put them in the unsourced band** | ❌ "these 207 truly have no source" |
| false-negative rate 26.57% (point estimate; **three-accounting spread** 17.30–40.14%, not a confidence interval) | **among sentences stipulated sourced, the proportion judged unsourced** | ❌ a total error rate |

**One wording error must be corrected here**: calling the 207 "truly unsourced" invites the reader to
conclude that the ground truth is "unsourced", whereas the stipulation is $y=1$. **We therefore write
"judged unsourced" throughout** and never use "truly unsourced" as a band name.

### 4.1.3 The soft spot: the filter rests on 22 markers

The stipulation $y\equiv1$ holds only if **what remains after filtering out sentences with internal
markers is in fact sourced.**

That premise is weak. Internal prose that happens to contain none of the 22 markers still slips
through:

- structural lines: `2. Formalisation of the C-A-F loop (see §3.2, lines 158–189)`
- extraction headings: `Harvest · EvoSci (one-pager)`
- transition sentences: `So the right reading is not "all memory plugins are junk", but:`
- summary lines: `① single agent — one brain, a pile of tools`

**None of these can be found in any source document.** If they are counted among "genuinely sourced
sentences", then the rule judging them unsourced **is correct under the literal-reuse criterion**, and
$\mathrm{ENR}$ is overestimated. (**This human check was later carried out**: all 18 L1 items were
drawn and judged, and **5 were found in their sources (27.8%)** — see §4.1.4. So the correct statement
is "correct under the literal-reuse criterion"; of the 18, five turn on the criterion rather than
agreeing empirically.)

### 4.1.4 Morphological audit: how large is the soft spot (the most important table in this chapter)

Three morphological criteria sort the 308 into three bands (criteria in `S1·_人判抽样.py::形态`; these
are **machine criteria and may be wrong**):

| Band assigned by the rule | definite meta-information | suspicious form | clean | total |
|---|---:|---:|---:|---:|
| **judged unsourced (≤6)** | **18** | **66** | **123** | **207** |
| to-human (7–11) | 8 | 24 | 30 | 62 |
| sourced (≥12) | 5 | 10 | 24 | 39 |

**Total with suspicious form = 18 + 66 = 84 items** (40.6% of the 207). This 84 is **not** the 73
sampled in §4.6.

**Criteria for "definite meta-information"**: begins with "harvest" (heading line) / contains
`(see §…)` or `line N` (internal pointer) / contains "one-pager" (filename-style heading) /
"1–10 + ★" (sub-heading) / "date / source / original / transcript:" (field line).

**None of these 18 can be found under the literal-reuse criterion** (internal pointers such as
`(see §…)` and `line N` cannot occur in the cited source), so the rule judging them unsourced **ought
to be right**.

**And the words "by definition" do not hold for part of the 18:**

| Sub-class | n | Can one say "necessarily unsourced"? |
|---|---:|---|
| **pure pointers** (containing `(see §…` / `line N`) | **5** | ✅ yes — such pointers can never occur in the cited source |
| **heading lines** ("harvest · X (one-pager)" / "1–10 + ★") | **13** | ❌ **no** — the **proper names come from the source** (`EvoSci` is in the source); we added the wrapper |

**Take `Harvest · EvoSci (one-pager)`: `EvoSci` is sourced; `Harvest · … (one-pager)` is our shell.**
Under the §4.1.2 criterion ("substantive content originates in the source") it counts as sourced;
under "the whole sentence must be verbatim" it does not. **The classification of these 13 is itself a
question of criterion, not something a rule can settle — only a human can.** So all 18 L1 items were
drawn and judged, with no answer presupposed.

**$y_i\equiv1$ is a stipulation made when the data were produced, not a verified fact.** The 18 above
(plus the 66 awaiting judgement) are the **candidate exceptions** to that stipulation. The correct
reading of §4.1.2 is therefore:

> **"The stipulation is $y\equiv1$; §4.1.4 finds up to 84 items that may violate it, so $y\equiv1$ is
> a hypothesis awaiting test, and the intervals in this paper are built on that uncertainty."**

**Before the human labels**, only an interval could be given. Those values are superseded and are kept
here only to illustrate the method — the current ground truth is the three accountings below:

| Case | Arithmetic | Value (superseded) |
|---|---|---:|
| upper bound (all 207 counted as wrongful rejection — §3's accounting ①, a rule not a ground truth) | $207/308$ | 67.21% |
| middle A: excluding the 5 pure pointers only — the most conservative account, deducting only what is certainly correct | $202/308$ | 65.58% |
| middle B: excluding all 18 definite meta-information items (assuming the 13 heading lines are also unsourced) | $189/308$ | 61.36% |
| lower bound (all 84 morphologically suspicious items counted as correctly judged) | $123/308$ | 39.94% |

**That interval was 27.27 points wide — which is why it had to be resolved before §5 rather than
carried forward.**

**The human labels are in, and that interval is void.** The 100-item stratified sample (pool 207,
drawn 73) gave **A 22 / B 24 / C 27**. Extrapolated:

| Accounting | How C is counted | $\widehat{\mathrm{ENR}}(7)$ | 95% interval | Implied wrongful rejections (pool 207) |
|---|---|---:|---|---:|
| upper bound | C counts as found | **40.14%** | [32.61%, 47.67%] | 124 |
| point estimate | C removed from the denominator | **26.57%** | [19.93%, 33.22%] | 82 |
| lower bound | C counts as not found | **17.30%** | [11.34%, 23.26%] | 53 |

**The three-accounting spread is $[17.30\%,40.14\%]$ — this is not a confidence interval.** It is the
range of three point estimates produced by three different conventions; each point estimate carries
its own 95% interval (fourth column of the table in §5.2.2). **The two kinds of "range" must not be
conflated.** The upper bound 40.14% is only **0.20 points above** the old lower bound 39.94% — **the
old and new intervals overlap, they do not abut**: the new $[17.30\%,40.14\%]$ and the old
$[39.94\%,67.21\%]$ overlap on $[39.94\%,40.14\%]$. **The old interval was shifted up by a whole
segment; the human labels brought it down.**

Why the old upper bound does not stand: 67.21% presupposes that all 207 are wrongful rejections, and
the labels say only **53–124** (point estimate 82) can actually be found in their sources.

**One old derivation was falsified on the spot**: the "lower bound" row above assumed the 18 definite
meta-information items could never occur in the source. **All 18 L1 items were drawn and judged, and 5
were found (27.8%)** — the proper names inside the headings (`EvoSci`, `DeepSeek Super-agent`) are
indeed in the source; we had merely added a shell. **That criterion was a surface-form criterion, not a
content criterion.** The four values above (67.21 / 65.58 / 61.36 / 39.94) are therefore **retained
only to illustrate the method and must not be cited as ground truth**; the ground truth is the three
accountings above. Full definitions are in `S2` §2.7.3.

**What this subsection is worth is not its numbers but the fact that it quantifies what we do not
know.** Treating the soft spot as an abstract premise ("provided the filtering rule is complete") is
how one waves a problem away. What one should do is count, then give an interval and a test plan.

### 4.1.5 The negatives' ground truth is constructed too — but firmer

The negative sets used in §4.4.4 and §4.5 have so far appeared only in parentheses ("vs 1,072
independent negatives"). Here is **why they are negative**:

| Family | How it was built | Why the ground truth holds | n |
|---|---|---|---:|
| **D-pure** | sentences from our own rules / process / experience → paired with the topic-nearest source document | the rule documents describe this workspace's process, which external sources do not contain → **absent under the literal-reuse criterion** | 721 |
| **B-main** | a sentence from one source document → paired with a **different** source document | cross-document → not in **that** document (not necessarily absent from all) | 351 |

**Two anti-contamination measures**: LCS isolation against the 10 items (group C) used to choose the
cut-offs — maximum LCS < 8; and high-Jaccard pairs inside family B (possibly sibling videos of the
same paper, sharing proper names) are **split off into their own family, excluded from the main
register**, used only as a comparison.

**351 and 64 are two parallel families, neither contained in the other**: B-main has 351 items
(measured Jaccard range **0.088–0.186**) and B-twin 64 items (**0.356–0.923**). **The 351 in the AUC
table does not include those 64.**

**The 0.30 threshold has no literature basis, but it falls inside a gap.** Between the B-main maximum
of 0.1856 and the B-twin minimum of 0.3565 there is a **gap 0.17 wide**; 0.30 lies inside it, so **any
threshold in (0.186, 0.356] produces exactly the same grouping.** This is stronger than "chosen by
experience": not "I picked 0.30, hence this grouping" but "**any** threshold in that gap gives this
grouping" — the conclusion is provably insensitive to the parameter.

**These two ground truths are firmer than the positive one, and firm in a different way:**

| | positives (308) | negatives (1,072) |
|---|---|---|
| proposition to establish | "this sentence **is** sourced" | "this sentence is **not** in **this** document" |
| exhaustible? | ❌ proving "is" requires locating it | ✅ proving "not in this one" needs one full text search |
| direction of error | mixed-in and extracted sentences counted as sourced (**84 candidates found**) | can only miss sentences that happen to restate the source |
| measured risk | high (§4.1.4 interval) | **low but not zero** — see below |

**So the evidential strength of this paper is not uniform**: the interval in §4.1.4 (positive ground
truth) is weak; the order-of-magnitude comparison in §4.5 (negative ground truth) is comparatively
firm. **The abstract and conclusion must allocate their confidence accordingly, and must not treat the
two as equally reliable.**

#### 4.1.5.1 "The maximum LCS of family D-pure is only 5" holds only for purely natural sentences

**The claim "at $t\ge6$ the false-positive rate of family D is structurally zero" has a premise**: it
rests on "maximum LCS = 5", and **5 is the maximum of the *purely natural* band** — for family D-full
it is not zero (this is where the 3.00% in §5.7 comes from).

Family D in full has **1,803 items** (including structured debris such as dates, paths and identifiers).
**The two numbers must be connected**: the **721 above is a subset of that 1,803** (D-pure = the full
set minus 1,082 date/path/identifier sentences), **not a parallel set**. Its maximum is **12**, with
**54 hits** at $t=7$ (3.00%). **The portion where $t\ge7$ can be measured is precisely the portion that
was removed when "purely natural" was split off.**

The 54 are divided by **the substantive composition of the matched string** (script
`S1·_假阳分层.py`, 15 self-tests):

> **The labels on these 54 come from a script, not from item-by-item human review** — the self-test
> covers 15. **So the table must be read as "three classes produced by three rules"**; the 6 in
> "content · Chinese" are explicitly marked as awaiting the P6 human check, and the other two classes
> rest on mechanical rules (character counts), which are recomputable.

| Composition of the matched string | Criterion | n | Share of 1,803 | Counts as an error? |
|---|---|---:|---:|---|
| **content · terminology** | alphabetic LCS ≥ 3 | **37** | 2.05% | **no** — English proper names are verbatim in the source; "sourced" is right under §4.1.2 |
| **format · digits and symbols** | both Chinese and alphabetic LCS < 3 | **11** | 0.61% | **yes** — what matched is a date/path/identifier; the scorer cannot tell format from content |
| **content · Chinese** | Chinese LCS ≥ 3 | **6** | 0.33% | **awaiting judgement** — genuine false-positive candidates, need the P6 human check |

> **A trap worth recording.** The first pass split the 54 by "is the Chinese character count zero" and
> obtained "48/54 have zero Chinese". **The label was wrong** — English letters also have a Chinese
> character count of zero, and English letters are content (proper names, terminology), not format.
> **Putting that label into the paper would invite a single reviewer sentence — "you counted your own
> terminology overlap as a format defect" — and the claim collapses.**
> The **52/54** figure must not be cross-checked against the 37/11/6 split: $37+11=48$, and 52 is not
> reproducible — it belongs to the earlier two-way convention, not this three-way one. The correct
> procedure is to compute three LCS values separately (Chinese only / alphabetic only / as-is) and
> split three ways.

**Hence what can accurately be said about the $t\ge7$ end is this:**

> On the 1,803 family-D negatives, of the **3.00%** false-positive rate at $t=7$,
> **2.05 points are verbatim overlap of English terminology** (not an error under our criterion),
> **0.61 points** are the scorer mistaking dates/paths/identifiers for content (**a demonstrable defect
> of the scorer**),
> and **0.33 points** are substantive Chinese overlap (genuine false-positive candidates, awaiting
> judgement).
>
> That is: **the false positives at $t=7$ are neither "unmeasurable" nor "trivially zero"**, but
> **0.61 (established) + 0.33 (awaiting judgement) = an upper bound of 0.94 points**.
> **The whole 0.94 cannot be reported as "the real cost"**: **only 0.61 may enter a conclusion**; if
> the 6 Chinese items are judged findable (not errors), the cost stops at 0.61, **and the claim that
> "scorer defects are two thirds of it" would become 100%.** One part of the 0.94 hangs in the air.

**How much support this number has**: 1,803 items, five times the 351 of family B-main in
§4.5. **So P5 (adding negatives at the $t\ge6$ end) drops from "must be newly constructed" to "existing
data already answers part of it"** — but two things must be written into §8: ① the 0.61 rests on the
criterion choice "terminology overlap is not an error", and a different criterion would move it; ②
**only $t=7$ was measured** — the false-positive rates at $t=6$ and $t\ge8$ **were not computed**, and
closing one point does not close the high-threshold end.

---

## 4.2 The first scorer: LCS

### 4.2.1 Definition

$$\mathrm{LCS}(x,s)=\max\{\,|u| : u \text{ is a common substring of } x \text{ and } s\,\}$$

the length of the **longest contiguous run** of shared text. Why this scorer: it is the most basic
instrument in text-reuse detection and the easiest for an auditor to reproduce independently — given a
document and a suspected source, it needs no model, only a few lines of code.

### 4.2.2 Three known defects, each with evidence

**Defect one: taking only the longest run makes it sensitive to insertion.**

Controlled measurement:

| | Sentence | LCS | C3 |
|---|---|---:|---:|
| verbatim copy | This study proposes a new retrieval-augmented method | 15 | 1.000 |
| one word inserted | This study proposes a new **improved** retrieval-augmented method | **9** | 0.688 |
| relative drop | | **−40.0%** | −31.2% |

**Note that the two scorers barely differ here (40% vs 31%), so this is not evidence that the second
scorer is better** — only that they move in the same direction.

**Defect two: a long digit string, path or identifier can inflate it in one stroke.**

When the source happens to contain a long identifier, a sentence whose content is entirely its own is
still lifted once that identifier is copied (controlled measurement: appending an identifier that also
appears in the source raises LCS by more than a factor of two).

**This is a defect shared with C3** — C3 is inflated too. So it cannot be solved by changing scorers,
only by denoising. **And the simplest denoising variant was tried, with worse results**: removing
digits and punctuation and recomputing dropped AUC from 0.812/0.705 to 0.778/0.677. **This goes into
§8.**

> **We may not write "the denoising route has been tried and closed".** Only **one** denoising method
> was tested (global removal of digits and punctuation). **Selective denoising** (removing only the
> identifier that also appears in the cited source), **weighting** (down-weighting digit positions
> rather than deleting them), and **whitelisting** (ignoring known identifier lists) — **none of these
> three was tested**, so no claim that "the whole route is dead". What can be said: **the most naive
> denoising degrades discriminative power**, which is itself evidence that "copying an identifier" is a
> discriminative signal.

**Defect three: it answers "whether", not "how much".**

This is not a defect but a **scope of applicability**. It leads directly to the **three** kinds of
disagreement in §4.4 (classes A / B / C, table in §4.4.2).

---

## 4.3 The second scorer: C3 (character trigram coverage)

### 4.3.1 Definition

Let $P_3(s)$ be the set of all contiguous length-3 segments of $s$ after removing punctuation and
whitespace:

$$\mathrm{C3}(x)=\frac{|P_3(x)\cap P_3(R(x))|}{|P_3(x)|}$$

Read as: **what fraction of this sentence's trigrams also occur in the source.** Range $[0,1]$; higher
means more likely sourced.

### 4.3.2 Three reasons for choosing it, and two routes not taken

1. **Recomputability is a hard constraint**: no overseas model weights are reachable from this machine,
   so embedding cosines are unavailable. Character n-grams have zero dependencies, run locally, and an
   auditor can reproduce them with one regular expression.
2. **Its statistical character differs from LCS**: LCS is an **extreme-value** statistic (only the
   longest run), C3 is a **proportion** statistic (overall coverage). Extremes are very sensitive to a
   single anomaly; proportions are insensitive to single points but sensitive to overall dilution.
3. **It is comparable to the same family the opponents use**: Kelemen et al. use literal-overlap
   indices (rank and band flips), so C3 needs no separate theoretical defence.

**Why not Jaccard**: sentences are short (median 34 characters) and sources are long (about 40,000
characters), so Jaccard's denominator would be dominated by source length. Containment divides only by
the sentence's own trigram count and does not include source length.

**Why not embedding cosine**: beyond the unavailability of weights, there is a methodological reason —
semantic similarity scores "another way of saying the same thing" as highly similar, whereas what this
paper must judge is **literal reuse**. Judging "was this sentence copied" by semantic similarity
**conflates paraphrase with quotation, and the scorer's readings then fall out of alignment with this
paper's ground-truth criterion.**

> **But the correct statement is "out of alignment", not "the ground truth stops being binary"**: the
> ground truth is fixed by **this paper's criterion of "is it literal reuse"**, and **no change of
> scorer moves it**. A scorer can disagree with the ground truth, but **a scorer cannot change the
> ground truth's range.**
> This is **a deliberate exclusion, not an impossibility** (see the boundary table in §4.8).

### 4.3.3 Distribution

| Quantity | Value |
|---|---:|
| median | 0.200 |
| quartiles | 0.026 / 0.429 |
| maximum | 1.000 |
| **sentences with C3 = 0** | **76** (24.7%) |
| sentences with C3 ≥ 0.999 | **2** (0.6%) |

Read together: **76 sentences have not a single trigram in their source, while only 2 have all of
them.** In this dataset copying is not all-or-nothing; the overwhelming majority is partial coverage —
which explains why the middle of the C3 distribution is thick while the LCS distribution is pressed
low.

---

## 4.4 Two scorers: agreement, disagreement, and what disagreement means

### 4.4.1 Rank correlation

$$\rho_{\text{Spearman}}=0.9122,\qquad \tau_{b}=0.7475$$

Strongly aligned in direction, but **not the same scorer**: $\tau_b=0.7475$.

> Under a no-ties assumption, $\tau_b=P(\text{concordant})-P(\text{discordant})$ with
> $P(\text{concordant})+P(\text{discordant})=1$ gives
> $P(\text{discordant})=(1-0.7475)/2=12.62\%$ — about one pair in eight (precisely 7.9 pairs).
> **But that assumption does not hold on these data, and this must be stated**: the scores are integer
> LCS values with many ties, and $\tau_b$'s denominator is itself adjusted for ties
> ($\tau_b=(C-D)/\sqrt{n_1n_2}$, where $n_1,n_2$ are the comparable pair counts on each side). With
> many ties, $P(\text{concordant})+P(\text{discordant})=1$ fails, and **12.62% can only be read as an
> order of magnitude.**
> **We therefore do not perform that translation.** The text reports $\tau_b=0.7475$ and $\rho=0.9122$
> and says "strongly aligned, but not the same scorer"; **the statement "one pair in eight" requires a
> no-ties assumption these data do not satisfy, and has been withdrawn.**
> **This is the most insidious kind of error**: the number is right (0.7475), the arithmetic is right
> (12.62%), and only the step that translates it into plain language is wrong. **That is exactly the
> step reviewers like to catch.**

### 4.4.2 Band cross-tabulation

C3's band boundaries are taken at its own 67.2% / 87.3% quantiles on the 308 items (< 0.365 unsourced /
0.365–0.611 to-human / ≥ 0.611 sourced).

| | C3 unsourced | C3 to-human | C3 sourced | total |
|---|---:|---:|---:|---:|
| **LCS unsourced** | **188** | 16 | 3 | 207 |
| **LCS to-human** | 15 | **35** | 12 | 62 |
| **LCS sourced** | 3 | 11 | **25** | 39 |
| total | 206 | 62 | 40 | 308 |

Diagonal 248/308 = **80.5%**; disagreement 60/308 = **19.5%**.

**The 60 disagreements must be broken down by cell, not just by the two ends** (reporting only classes A
and B, 33 items, and leaving 27 unaccounted for, is not enough):

| Class | Cell | n | Meaning |
|---|---|---:|---|
| **A** | LCS sourced → C3 to-human / unsourced | **14** | LCS says sourced, C3 says not |
| **B** | LCS unsourced → C3 to-human / sourced | **19** | LCS says not, C3 says sourced or to-human |
| **C** | LCS to-human → C3 unsourced / sourced | **27** | **both scorers "cross a whole band", in opposite directions** |
| | | **60** | = 14 + 19 + 27 ✓ |

> **Class C (27 items) was previously unaccounted for, an omission.** Its character differs from the
> other two: A and B are "one says yes, the other says no"; C is one scorer saying **to-human** while
> the other has already **committed to a verdict**.
> For our purposes, C shows that **the to-human band itself is not portable between scorers** — when §5
> asks whether $t$ should move down, this band is the main battlefield, and 27 items cannot be left out.
> **We do not attribute causes for class C** (we did not read the matched strings item by item); we
> report the count only. **Reporting a count without explaining it beats inventing an explanation.**
> **The 80.5% must not be read as "agreement".** C3's band boundaries were **forced to align** with
> LCS's band proportions — **proportional alignment raises the diagonal**, so this number is largely
> constructed rather than discovered.
> **But "alignment guarantees identical marginals and hence a high diagonal" would be overstating it**:
> the actual marginals are $207/62/39$ versus $206/62/40$ — **not identical**; and **identical marginals
> would not entail a high diagonal anyway.**
> What can be said: **approximate alignment systematically raises the diagonal**, so 80.5% is only
> corroboration that "the two scorers point broadly the same way", **not an agreement statistic**.
> The only genuine independence evidence is **rank correlation (which depends on no alignment)** and
> **AUC (§4.4.4)**.
> **Any number of the form "because it is set up this way, it must come out that way" cannot be used as
> evidence.** Two other cases in this paper have the same shape: the C-group's maximum LCS happening to
> be 3, from which false positives at $t=4$ "must" be zero; and Jaccard on nested sets being
> identically the size ratio.

### 4.4.3 The 60 disagreements, three directions; the causes of A and B lie in the longest matched string

**Class A: LCS says sourced, C3 says to-human or unsourced — 14 items**

For each item we printed the **longest common substring** (this is a verdict, not an inference):

| LCS | C3 | **Chinese characters in it** | Matched string |
|---:|---:|---:|---|
| 22 | 0.605 | **0** | `The Last AI Built by Humans` |
| 22 | 0.562 | **0** | `Qwen3-4B (non-thinking)` |
| 20 | 0.594 | **0** | `Reliable verification` |
| 18 | 0.560 | **0** | `A finger can't speak.` |
| 18 | 0.533 | **0** | `specs are the new code` |
| 16 | 0.580 | **0** | `reasoning focus` |
| 15 | 0.355 | **0** | `selective recomp…` |
| 14 | 0.524 | 2 | `多块，30` |
| 13 | 0.564 | 12 | `查合作伙伴，线索越来越完整` |
| 13 | 0.538 | **0** | `Claude Opus 4.8` |
| 12 | 0.387 | **0** | `oop detection` |
| 12 | 0.353 | **0** | `Agent Harness` |
| 12 | 0.386 | **0** | `2610.14224v1` |
| 12 | 0.254 | 12 | `构成以知识为中心的记忆图` |

> **How to read the table**: `LCS` is the algorithm's output (length **after removing whitespace**);
> `Matched string` is an **illustrative fragment**, given to show what kind of thing matched (English
> phrase / identifier / Chinese), **not a verbatim reproduction**.

**In 11 of the 14, the matched string contains zero Chinese characters — it is an English phrase or an
identifier.**

**So what can be said is: the longest matched string of these sentences is an English phrase (or an
identifier).**

> **One may not infer "the whole sentence is Chinese-English mixed"**: zero Chinese characters in the
> matched string **proves only that the common substring is English**, not that the sentence contains
> Chinese — **the sentence could be entirely English**. The table has no full-sentence text, so this
> step is an inference, not a measurement.
> The discussion of "the Chinese half" below must therefore be read **conditionally**: **if** the
> sentence contains Chinese, whether that part was rewritten from the source is something this paper has
> no evidence about.
> **Note that the claim "rewritten from the Chinese" is not made here**: **"rewritten" is a causal claim, and this paper has no
> evidence for it.** What was measured is only "the English half is verbatim in the source"; the Chinese
> half **can only be called "the other part that co-occurs"** — whether it was rewritten from the source
> or written by the author **cannot be distinguished** (which is what the strength declaration below
> says).

> **The evidential strength must be held back**: what was measured is only **that the English half is in
> the source**. Whether the Chinese half was rewritten from the source **has no independent evidence in
> this paper** — the basis is "it contains no internal markers, and it co-occurs with the English
> phrase". So this is as far as any statement can go: **"in a mixed sentence, one stretch of text is
> verbatim in the source, and this is confirmed by the longest matched string; whether the sentence
> counts as sourced depends on the criterion choice in §4.1.2, and is not decided by measurement."**
> **One must not write "the sourced status of the whole sentence is supported by measurement".**

- **LCS saying "sourced" is consistent with the §4.1.2 criterion**: that English phrase really is
  verbatim in the source; 12–27 consecutive identical characters cannot be coincidence in a
  40,000-character source.
- **C3 saying "to-human" is not wrong either**: the Chinese half of the sentence accounts for more than
  half its trigrams, so coverage naturally falls to about 0.5.
- **The two scorers are answering two different questions here**:
  - LCS asks "**is there a stretch of verbatim text in this sentence?**" → yes
  - C3 asks "**how much of this sentence is verbatim?**" → half

**This distinction is not "who is wrong"; it exposes a question the ground-truth criterion must answer:
does a mixed sentence count as sourced?** The §4.1.2 criterion ("substantive content necessarily
originates in the source") counts it **sourced**, so LCS agrees with the criterion and C3 does not.
**This supports continuing to use LCS, and shows C3 cannot simply replace it.**

**Class B: LCS says unsourced, C3 says to-human or sourced — 19 items** ($16+3$)

15 of the 19 are ≤ 40 characters (median length 30, shorter than the overall 34). Short sentences have
a small trigram base, so a few accidental matches take a large share — **C3 is unstable on short
sentences**, a defect in the opposite direction to LCS's.

**The two scorers' biases are complementary**: LCS is loose on mixed sentences (one verbatim stretch is
enough), C3 is loose on short sentences (a few trigrams are enough). **They fail on different sets of
sentences** — which is exactly why the two scorers deliver consistent overall conclusions (§4.5) rather
than identical ones.

### 4.4.4 Discriminative power: AUC is almost the same

Using the independent negatives of §5.3 (1,072, disjoint from the cut-off-selection sample):

| Scorer | AUC (308 vs D-pure 721) | AUC (308 vs B-main 351) |
|---|---:|---:|
| LCS | 0.812 | 0.705 |
| **C3** | **0.824** | **0.720** |

C3 is marginally higher, by 0.01–0.02. **At sample sizes of 308 / 721 / 351 this gap is insufficient to
claim "which scorer is better"**; the only statement available is that their discriminative power is of
the **same order**.

> **This paper does not advocate changing scorers.** Changing scorers **does not alter this chapter's
> conclusion**, and does not improve any **decision** conclusion. C3's entire role is **cross-validation**:
> showing that §5's conclusion is not an artefact of the LCS implementation.
> **But "does not improve any number" would be overstating it**: in the table above C3 is higher on
> **both** families — D-pure **0.824 vs 0.812** (+0.012), B-main **0.720 vs 0.705** (+0.015).
> **One may not say "each is higher on one family"**: **C3 is higher on both**; there is no "each".
> The accurate statement: **C3 is marginally higher on both families (0.01–0.02), but at these sample
> sizes the gap is insufficient to claim "which scorer is better"** — same order of discriminative
> power, and that is all that can be said.

---

## 4.5 Would a different scorer give a different optimal cut-off?

If the optimum moved elsewhere under a different scorer, the whole threshold discussion of §5 would be
an artefact of LCS. Youden optimisation on the independent negatives (positives = 308, negatives = each
family):

| Scorer | Negative family | Out-of-sample optimum | False-negative rate | False-positive rate there |
|---|---|---|---:|---:|
| LCS | D-pure | $t=4$ | 38.96% | 2.50% |
| LCS | B-main | $t=4$ | 38.96% | 24.79% |
| **C3** | D-pure | $c=0.10$ | 36.69% | 5.27% |
| **C3** | B-main | $c=0.25$ | 53.90% | 4.27% |

**Three observations, each more important than the last:**

1. **Both scorers' optimal operating points fall in the same error-rate band**: false negatives 37–39%
   (D-pure), i.e. "about four in ten". **This order of magnitude is scorer-independent.**

2. **But they are not fully consistent, and this must be said plainly**: LCS points to $t=4$ on **both**
   families (same direction); C3 gives two different points (0.10 and 0.25), and on B-main its optimum
   sits higher with false negatives rising to 53.90%. **"The two scorers converge on the same point"
   holds only on D-pure, not on B-main.**

3. **C3 on false positives: worse on one family, better on the other — not summarisable in one phrase**:
   · on D-pure, LCS **2.50%** / C3 **5.27%** — C3 more than double, **worse**;
   · on B-main, LCS **24.79%** / C3 **4.27%** — C3 lower by **20.52 points**, **better**.
   > **This cell must be read off the table**: C3 on B-main is **4.27%**, far better than LCS's
   > 24.79%. **The direction of superiority is completely reversed** relative to D-pure. This also shows
   > that one phrase like "slightly worse / slightly better" cannot hold a result that runs in opposite
   > directions on two families.

**The sentence this chapter needs (qualified version)**: on the **D-pure** negatives, both scorers'
optimal operating points fall in the "false negatives around four in ten" band, but **they should not be
swapped** — LCS has lower false positives on **D-pure** (the reverse on B-main), and its criterion
matches the ground-truth criterion (mixed sentences count as sourced); so the main results continue to
use LCS, and C3's value is corroboration.

> **The qualifier must be written inside the sentence**: "order-of-magnitude independence" in that
> sentence **holds only on D-pure**. On B-main, C3's optimum is 53.90%, already outside the 37–39% band.
> **One may not write "order-of-magnitude independence" as holding for all negative samples.**

> **A harder qualifier.** The "false-negative rate" column of that table is **entirely on the
> upper-bound accounting** (all judged-unsourced counted as wrongful rejections, i.e. the $207/308$
> convention).
> - **Comparable within the table**: LCS and C3 use the same convention, so "the two scorers' chosen
>   points are close" holds within the table;
> - **Not comparable outside it**: after the human labels, LCS's point estimate at $t=4$ is **15.40%**,
>   not 38.96%; and **C3 has no human labels at all and cannot be converted to a point estimate**.
> - **So "both scorers fall in the same error-rate band (37–39%)" may be read only as "they are close
>   under the same upper-bound convention", not as "both have true error rates of about four in ten".**
>   A cross-convention comparison is beyond this paper, and is not attempted.

> **This sentence must be read together with the interval in §4.1.4**: both 38.96% and 36.69% are
> numbers "under the stipulation $y\equiv1$".
>
> **The step to "the relationship does not change" was challenged, and rightly so**: the previous
> subsection says the two scorers fail on different sentences; this subsection would then say the
> cross-sectional relationship is invariant to the criterion — **that step had no derivation.**
>
> **Honest answer: it cannot be derived.** If the ground-truth correction affects LCS and C3 differently
> (they fail on different sentences), the relative positions of the two scorers could indeed move.
> **So that sentence is demoted to**:
>
> > "Under the same stipulation, the two scorers point to the same order of magnitude" — a **measured
> > conclusion under the current stipulation**, not a "criterion-invariant theorem".
> > Whether they remain aligned after the ground-truth correction **depends on the P6 results**: if the
> > L2/L3 human judgements affect the two scorers very differently, this conclusion must be rewritten.
>
> **One may not write "this relationship will not change".**

---

---

## 4.6 Human validation of the ground truth (P6)

### 4.6.1 Design: what is judged is the ground truth, not the algorithm

The blind package asks one question only:

> **Can this sentence, in the original source document in front of you, actually be found?**

The algorithm's verdict (LCS / C3 / band / stratum) **appears nowhere** in the blind package. Once a
rater sees how the machine decided, what they judge is "do I agree with the machine", not "what do I
see".

### 4.6.2 Sampling: 73% of the judging power sits on the 207 items, the other 27% serves two other questions

Stratified by the morphological audit of §4.1.4. **An earlier sampling scheme (stratified by
disagreement between the two scorers) was discarded** — it spent 38% of the judging power on the
to-human band, which does not enter any agreement denominator, whereas the real crux is how many of the
207 were judged correctly.

| Stratum | Pool | Drawn | Purpose |
|---|---:|---:|---|
| **L1** judged unsourced · definite meta-information | 18 | **18 (all)** | test whether the morphological criteria are right; expected all "not found" |
| **L2** judged unsourced · suspicious form | 66 | **30** | how many of these 66 were judged correctly |
| **L3** judged unsourced · clean form | 123 | **25** | the scale of genuine wrongful rejection |
| L4 to-human (7–11) | 62 | 15 | which way this band should move |
| L5 sourced (≥12) | 39 | 12 | the reverse direction: are genuinely sourced items ever mislabelled |

L1+L2+L3 = **73 items, 73% of the sample**. (Note: this 73 is the **number drawn**; the pool of
suspicious-form items is 84 — the two are not the same number, see the correction note in §4.1.4.)

### 4.6.3 Scoring: extrapolating to a corrected false-negative rate

$$\widehat{\mathrm{ENR}}=\frac{18\,p_1+66\,p_2+123\,p_3}{308}$$

$p_h$ is the proportion judged "A, findable" within stratum $h$. L1 is drawn in full, so it is counted
directly.

**A self-consistency check was performed**: if all three strata were judged A (i.e. "all 207 are
wrongful rejections"), the formula gives $207/308=67.21\%$ — exactly the figure of accounting ① in §3.
**The formula reproduces the original number; it is not a separate quantity.** (Note: 67.21% here is
the **count share under accounting ① of §3, the old convention**; after the human labels it is no
longer the true false-negative rate, see §4.6.5.)

**Precision**: with $n_h\approx25$–$30$ the standard error of a stratum proportion is about 0.09, and
after weighting to 308 about **±9 percentage points (95%)**. That suffices — what must be
distinguished is 40% from 67%, a gap of 27.3 points.

> **The arithmetic, spelled out.** In the worst case $p=0.5$ without a finite-population correction,
> $\mathrm{Var}=((66/308)^2\cdot 0.25/30)+((123/308)^2\cdot 0.25/25)=1.98\times10^{-3}$,
> $\mathrm{SE}=0.0445$, $1.96\,\mathrm{SE}=\pm0.087\approx\pm9$ points. (L1 is drawn in full, has no
> sampling error, and does not enter the variance.) **Without the arithmetic a reader is left guessing
> where the 9 comes from.**

> **Why the formula contains only the three judged-unsourced strata.** The 62 L4 to-human items **do
> not enter the numerator** — under accounting A3 of §3, "to-human" was not judged "unsourced" and does
> not count as a wrongful rejection; but they **do enter the denominator of 308** (they could have been
> right). So $\widehat{\mathrm{ENR}}$ is already a full-population estimate, not a partial one covering
> only 207. **L4's role is not to correct ENR but to decide how the threshold recommendation in §5 is
> worded** — the two must not be conflated.

### 4.6.4 Three numbers must be reported together, none omissible

| What is reported | What it is for |
|---|---|
| **Corrected false-negative rate** (extrapolated) | the central number of §5; **supersedes** 67.21% and 39.94% |
| **$p_1$ (share of the definite-meta-information stratum judged A)** | **self-check of the morphological criteria**: if $p_1>0.05$, real content sentences were misclassified as meta-information and the criteria must change |
| **Where the L4 to-human band went** | directly determines how "the threshold should move down" is worded in §5 — mostly "not found" supports moving down; mostly "found" means moving down would create new wrongful rejections |

### 4.6.5 Status

**The blind judging is done, and this chapter can now give point values.**

- Given to the rater: `S1·_人判_盲判包_100条.md` (containing no algorithm verdict and no stratum)
- Answer key: `S1·_人判_答案键.json` (**not** given to the rater)
- Returns: `S1·_人判_结果_20261108.json` (all 100 judged, A 34 / B 34 / C 32)
- Scoring: `S1·_人判_核分.py` (9 self-tests) → **upper 40.14% / point 26.57% / lower 17.30%**

**What this chapter can now write, replacing "corrected value unknown":**

1. the false-negative rate at $(t=7)$ is **26.57% (point estimate; three-accounting spread
   17.30–40.14%)** — **that spread is the range of three point estimates, not its confidence interval**
   (the 95% interval is [19.93%, 33.22%]);
2. $p_1=5/18=0.278$ — **the 0.05 self-check line for the morphological criteria is breached**, so the
   criterion "definite meta-information can never occur in the source" does not hold and has been
   changed;
3. L4 to-human band (15 drawn): A 5 / B 7 / C 3 — **B is the largest (7), but $7/15=46.7\%$ is not a
   majority, so "mostly not found" does not hold.**

> **Both of the tempting summaries overshoot.** ① $7/15=46.7\%$ is "the largest", not "the majority";
> the non-B total of 8 is actually larger. ② Even if it were a majority, these 15 are only a
> **sample** (pool 62), and "the threshold should move down" does not follow.
> → The accurate statement: **B is somewhat more frequent in L4, so the direction does not support
> raising the threshold; but the strength is only enough to "not object to moving down", not to
> "support moving down".**
> §5 therefore does not use this as a reason to move down; it uses it only to show that "where the 62
> to-human items belong is itself a criterion choice" (§5.6).

> **One sentence that still cannot be written**: **the value of $\mathrm{ENR}(t)$ at $t\neq7$.** The
> human labels sample only the judged-unsourced region; other $t$ can only be extrapolated
> proportionally (see `S2` §2.7.4).

### 4.6.6 Intra-rater reliability: how firm are the three bands? (50 blind re-ratings)

**All three numbers above (40.14 / 26.57 / 17.30) rest on one person judging once.** To learn what they
are worth, 50 blind re-ratings were carried out.

**Design** (all three details matter, or the numbers below mean nothing): 50 drawn at random from the
first 100; the page **contains no previous answer**, identifiers changed to R001–R050, and **order
shuffled**; the source is presented in **exactly the same way** as in the first round (same automatic
locating and highlighting, not the full text).

**Three-way cross-check passed**: R→S→original answer against the main answer key, **0 mismatches**;
the package's sentences against the main package's, **50/50 character-identical** — **the numbers are
real; this is not a mapping error or a broken package.**

**Results** (`S1·_重判包.py --核分`, 8 self-tests):

| Metric | Value |
|---|---|
| overall disagreement rate $q$ | **46.00%** (27/50 agree) |
| observed / chance agreement | 0.540 / 0.321 |
| **Cohen $\kappa$** | **0.322 — fair, not substantial** |
| A/B binary (C excluded) agreement | 67.7% (21/31) |

**Band-level agreement (rows = first pass, columns = second):**

| pass 1 ＼ pass 2 | A | B | C | agreement |
|---|---:|---:|---:|---:|
| **A** (findable) | **12** | 3 | 0 | **80%** |
| **B** (not findable) | 7 | **9** | 1 | **53%** |
| **C** (unclear) | 8 | 4 | **6** | **33%** |

> **This table says something far more important than "46%": the flow of C is essentially one-way**
> (**not** strictly — one item flowed in from B).
> Of the 18 C items in pass 1, only 6 were still C the second time (8 moved to A, 4 to B); and of the 7
> C items in pass 2, **none came from pass-1 A and only one from B**.
> **The only thing that can be said: the boundary of C moved between the two passes, and the movement
> was one-way (18 → 7).**
> **But one may not say "this is not memory noise, it is that the criteria were never fixed"**:
> ① what 50 re-ratings establish is "**C drained out markedly and band agreement is low**", **not the
> cause** — memory, fatigue, unclear criteria and sampling fluctuation all produce the same pattern;
> ② "the rater drew the line by feel each time" describes a **psychological process**, and this paper
> has no process data at all.
> **What can be said is that "the criteria were never fixed" is a reasonable explanation, not an
> established cause.**

> **So the 23-point accounting spread (17.30–40.14) must be reinterpreted — it is not a confidence
> interval.** Upper (C counts as found) and lower (C counts as not found) **both assign a value to C**,
> and C's intra-rater agreement is only 33%. The two ends of the interval are therefore **not "two
> equally defensible conventions" but "how much the same rater, on the same sentences, varies with the
> convention for C"** — **a criterion difference, not a drift magnitude**.
> **Do not read "the difference between three assignment conventions" as "re-rating drift"**: the 22.84
> points (40.14 − 17.30) are **the difference between three conventions on the same 100 items**; the 50
> re-ratings **did not recompute those two endpoints**. **The two quantities cannot stand in for each
> other.**
> The measured re-rating drift appears in §4.6.7 (A share 30% → 54% → 80%); **that is a separate
> account.**
> Conversely, **point accounting (C removed from the denominator) is the only convention that assigns no
> value to C** — which is the **hard reason** for taking 26.57% as the central number, not merely
> "taking the middle".

> **Two statements that cannot be supported.**
> ① **"The rater's single-pass error rate is 35.9%" cannot be stated directly.** That conversion
> $p=(1-\sqrt{1-2q})/2$ assumes the two passes err **independently and at random**, whereas the measured
> errors are **systematically one-way** (C leaking out); the premise fails.
> ② **Nor does it follow that "human labelling is unreliable and the ENR conclusions are void"** —
> that over-infers. Band A is 80% stable, and the central number (point accounting) **does not directly
> depend on the assignment of C**.
> **But "does not directly depend" is not "is unaffected"**: $A/(A+B)$ does contain no $C$, yet **once a
> C is re-judged as A or B, numerator and denominator change** (which is exactly what §4.6.8 reports).
> The accurate statement: **point accounting needs no value for C, but cannot escape C's drift** — the
> latter is already counted in the "rater difference of 4.82 points" below.

**Two things not done (they go into §8)**: write C's criteria as hard rules and re-rate the 50 (about
20 minutes, **the only way to genuinely repair this**); and recruit another independent rater for
inter-rater work (the only way to answer the "single rater" criticism raised by Angoff, 1971).

### 4.6.7 Third pass: with hard criteria, the A share rises from 30% to 80%

The diagnosis of §4.6.6 was "C lacks criteria". So a written criterion set was produced
(`S1·_判据_C档.md`), whose core is to narrow C's entry to **"the passage in the source is itself
unintelligible"** (not "I am unsure", not "not verbatim", not "roughly related"), and the **same 50
items** were re-rated once more. **The result is poor — and more informative than "poor".**

**Three passes over the same 50 items:**

| Pass | A | B | C | A share | upper (C found) | point (C removed) | lower (C not found) |
|---|---:|---:|---:|---:|---:|---:|---:|
| 1 | 15 | 17 | 18 | **30%** | 66.0% | 46.9% | 30.0% |
| 2 | 27 | 16 | 7 | **54%** | 68.0% | 62.8% | 54.0% |
| 3 (criteria) | 40 | 10 | **0** | **80%** | 80.0% | 80.0% | 80.0% |

**① The criteria did not meet their own standard (two of three pre-committed thresholds failed):**

| Standard | Required | Measured | Verdict |
|---|---|---|---|
| C agreement | ≥60% | **no C this pass** | ❌ |
| C count | ≤10/50 | 0/50 | ✅ |
| A/B binary agreement | ≥80% | **79.1%** | ❌ by 0.9 points |

**② And it committed the very error it forbade**: of the 16 B items in pass 2, **8 (50%) were absorbed
into A**. The clause "a synonymous rewrite counts as found" was **over-applied** — precisely the "roughly
related also counts as A" that the criteria explicitly forbid.

**③ The most important point: the A share rises monotonically across the three passes
(30% → 54% → 80%), while C falls monotonically (36% → 14% → 0%).**

This pattern admits **two opposite explanations, and the data cannot separate them**:

| Explanation | What it says | Where the central number would go |
|---|---|---|
| **Correcting over-strictness** | many former C items were really A (rewrites taken for unclear), and the criteria put them back | ENR is **underestimated** |
| **Criterion drift** | the criteria licensed "rewrite counts as found", so the rater widened pass by pass and even absorbed B into A | ENR is **overestimated** |

> **And a design flaw must be admitted here**: the three passes are a **pre-post series, not a
> controlled intervention.**
> The evidence: **pass 1 → pass 2 added no criteria at all, yet A rose from 30% to 54% (+24 points)** —
> that is, **merely judging the same sentences a second time moved the standard.**
> Of the further 26 points in pass 3, how much is the criteria and how much is repetition **cannot be
> separated** (the "pass 3 without criteria" arm was never run).
> **One may not say "only one variable changed"** — this is a **pre-post series**, not a controlled
> experiment.

**④ Treatment of the central number: it does not change, but the error structure must be reported
separately.**

The central 26.57% still comes from **the Rater 1 series** (the 100-item stratified extrapolation,
§4.6.5) — **not from "pass 2" as used in this subsection.**

> **Two numbering systems coexist here, and conflating them makes the provenance of the central number
> unexplainable**:
> · **the 50-item series**: pass 1 (Rater 1) / pass 2 (Rater 1, blind re-rating) / pass 3 (Rater 1,
> with criteria) — this measures **drift**;
> · **the 100-item series**: Rater 1 (first 100) / Rater 2 (full re-judging) — this measures
> **cross-person** variation.
> **26.57% comes only from "Rater 1" in the second series**; the 50-item series' point accounting is
> 62.8%, and the two have different denominators and different samples — **they cannot be substituted
> for each other**.

Why the Rater 1 series is retained: because **it carried no prior disposition**, whereas the pass-3
criteria are **introduced by this paper and on the loose side** (8 B items absorbed), so **the pass-3
standard is the one that should not be trusted**.

But the error components below **must be listed separately and never combined into a single ±** (three
dimensions, four rows):

$$\underbrace{\pm6.65}_{\text{sampling}}\ \Big|\ \underbrace{23}_{\text{accounting}}\ \Big|\ \underbrace{\le50}_{\text{drift}}\ \Big|\ \underbrace{4.82}_{\text{cross-rater}}\qquad\text{four parallel terms, \textbf{not additive}}$$

> **The four must be written side by side and must not be added**: the form
> `±6.5 + 23 + ≤50 + 4.82` **itself teaches the reader to add them** (which would read as about 84
> points).

| Component | Value | Base | Source |
|---|---:|---|---|
| Sampling error | ±6.65 points | 308 items, stratified extrapolation | half-width of the 95% interval |
| Accounting spread (how C is counted) | 23 points | **308 items** | upper 40.14 − lower 17.30 |
| *(same-base comparison)* | *36 points* | *50 items* | three-accounting spread within one pass |
| **Rater drift (category share)** | **≤50 points, direction unknown** | **50 items** | lower accounting across three passes, 30% → 80% |
| **Rater difference (central number, point accounting)** | **4.82 points** | **100 items, different raters** | §8.1.4 (26.57 ↔ 31.39) |

> **The four are not additive** (different sources, different bases), but **must be reported
> separately**.
> **In particular: "26.57% ±6.65 points" suggests a total uncertainty only 13.3 points wide, when in
> fact it is far wider.**
> **The last two rows both belong to the rater dimension but are two different measurements**: one
> measures how the A share drifts (50 items), the other how far the central number moves (100 items).
> **Different bases; neither addable nor convertible.**
> **What can be said**: on the same 50 items, drift (≤50) and accounting spread (≤36) are of the **same
> order** — the axis "even whether something counts as A moves" is at least as large as "how C is
> counted", while the interval in the text covers only the latter.
> **What cannot be said**: adding the 50 points to 26.57 — **drift on the 308 items was not measured**,
> only on 50; nor "drift 50 > accounting 23 in the text" (**different bases**).
> **So: the central figures of this paper can be read as an order of magnitude, not as precise values.**
> This is the hardest limitation in the paper and appears in §8.1.1.

### 4.6.8 After a second rater: the ambiguity is not "A or not" but "B or C"

§4.6.7 said "the direction of drift cannot be determined" — but that came from **one person** rating
three times, which cannot separate "this person is moving" from "the task itself is unjudgeable". So a
**second rater** was recruited, and rated the **same 50 items** on the **same criteria page**. That
separated them.

**Four-way distribution (same 50 items):**

| Pass | A | B | C | A share | C share |
|---|---:|---:|---:|---:|---:|
| pass 1 (Rater 1) | 15 | 17 | 18 | 30% | 36% |
| pass 2 (Rater 1, blind re-rating) | 27 | 16 | 7 | 54% | 14% |
| pass 3 (Rater 1, criteria) | 40 | 10 | **0** | 80% | **0%** |
| **Rater 3 (same criteria)** | 27 | 7 | **16** | 54% | **32%** |

> **Under the same criteria, Rater 1 produced 0 C items and Rater 3 produced 16 (32%)** — **so "the
> criteria closed off C's entry" does not hold**; it held only in that one pass, by the rater who wrote them.

**Three pairings: the two ends are single-factor contrasts; the middle one moves two factors at once.**

> **One may not say "each differs by one factor"**: "pass 2 vs pass 3" changes **both the number of
> repetitions and the presence of criteria**, so that row's difference **cannot be attributed to the
> criteria alone**.
> The two ends are clean: pass 1 vs pass 2 differ only in **repetition**; pass 3 vs Rater 3 differ only
> in **person**. **To isolate the criteria one would need a fourth arm — same person, same pass count,
> criteria only — which this paper does not have.**

| Pairing | What differs | Three-way agreement | κ | **A vs non-A agreement** |
|---|---|---|---:|---:|
| pass 1 vs pass 2 | **pure retest** (same person, no criteria) | 54% | 0.322 | **64%** |
| pass 2 vs pass 3 | retest + criteria | 68% | 0.365 | 70% |
| pass 3 vs Rater 3 | **pure change of rater** (different person, same criteria) | 60% | 0.259 | **74%** |

**① On A versus non-A the three pairings give 64% / 70% / 74% (pure retest / same person with criteria
/ different person with criteria); all three fall between 64% and 74%, and no single change drives
agreement down.** But **there is no "same person, same criteria, different occasion" arm**, so how much
"changing the rater" contributes on its own cannot be separated (§8.1.2 ②).
→ **Agreement does not fall when the rater changes (74%), slightly above the pure retest of 64% — but
those 10 points are mixed with "criteria present or not".**
> **But one may not say "two people are steadier than one person twice"**: ① those 4 points are within
> the pass-to-pass noise measured in §8.1.5; ② the 70% arm is itself "retest + criteria", **not a clean
> same-person baseline**. **What can be said is that "changing the rater did not drive agreement down";
> not that "changing the rater is steadier".**

**② Self-consistency is higher with criteria (68% > 54%), but that difference is mixed with drift from
repetition and cannot be credited to the criteria wholesale.**
> **This pairing changes both "retest" and "criteria"**: §4.6.7 itself measured that pass 1 → pass 2,
> with nothing but a second reading of the same sentences, moved A from 30% to 54% (+24 points).
> And changing the person gives only 60%, **with κ falling from 0.322 to 0.259** — **this finding is not
> affected by that confound, and "the criteria were internalised individually rather than executed
> jointly" still stands.**

**③ The most valuable structure: the A verdict is in a relation of proper containment, not random
disagreement.**

| pass 3 ＼ Rater 3 | A | non-A | total |
|---|---:|---:|---:|
| **A** | **27** | 13 | 40 |
| **non-A** | **0** | 10 | 10 |

**Zero in the reverse direction — Rater 3's A-set is a proper subset of Rater 1's pass-3 A-set.**

> **Why this matters more than an agreement rate**: **random error produces mistakes in both directions;
> a difference in strictness only in one.** We observe one direction only → **this looks more like a
> difference in leniency than like random error.**
> **But it cannot be pushed to "so nobody mislabelled anything"**: in a finite sample random error can
> perfectly well appear in one direction only; proper containment is **equally compatible** with "one
> rater mislabelled a batch"; and **74% is agreement, not accuracy**.
> And "a difference in leniency" is **actionable**: taking the intersection yields **27 doubly-confirmed
> A items** (54% of the 50).

**④ How unreliable the B/C boundary is, in one number:**
of the 10 items pass 3 called B, **7 were called C by Rater 3 and none A**.
→ **"Not A" is agreed on; "B or C" is almost personal habit.**

**⑤ This is the third and firmest reinforcement of the central number (point accounting):**

| Accounting | Definition | Dependence on A | Dependence on C |
|---|---|---|---|
| upper | $(A+C)/n$ | first order | **C enters the numerator directly** |
| **point** | $A/(A+B)$ | **first order (numerator is A alone)** | **C appears nowhere in the ratio** (neither numerator A nor denominator A+B contains it; C acts only through being re-judged as A or B) |
| lower | $A/n$ | first order | **C enters the denominator directly** |

Measured: **agreement within the A band (74%) exceeds that within B/C (46.8%)**, and the A verdict shows
one-way containment; **C is the least stable** (0 vs 16 under the same criteria).
> **But one may not say "A is the most stable"**: §1.6 already computed that of the 51 disagreements
> **26 are A-versus-non-A and 25 are B-versus-C — almost exactly half each**. **The accurate statement is
> "agreement is higher in the A band", not "A is stable while B/C is not".**
→ **Neither the numerator nor the denominator of point accounting contains $C$**, so **no value need be
assigned to C**.
> **But "unaffected by C" does not hold**: a re-judgement $C\to A$ or $C\to B$ changes numerator or
> denominator. The real reason for choosing point accounting is that **its denominator moves with A and
> thereby offsets A's systematic offset**; the mechanism is in §8.1.2 ⑥.
**The central number uses point accounting, and the reason has changed from "taking the middle" to
"resting on the only reliable binary".**

> **A fourth reason, and an external one: this is what the AIS framework does.**
> In the attribution evaluation framework of Rashkin et al. 2023 (*CL* 49(4), `10.1162/coli_a_00486`),
> the rating labels have only two values (interpretable yes/no, AIS-supported yes/no);
> **"underspecified source" is not a third label value but a flag — flagged items are removed from the
> scoring queue entirely and do not enter the denominator.**
> The authors' own words: "thus **excluded** from the interpretability and AIS scores".
>
> **So this paper's point accounting (C removed from the denominator) matches how that framework treats
> an underspecified source.** The central number uses point accounting not merely because it "avoids
> assigning a value to C", nor merely because it "rests on the only reliable binary" — **it is also an
> established convention.** (Full comparison in §2.2.1.)

**⑥ One thing that could be done and was not (it goes into §8.5, item 5):**
**replacing a single rater with the intersection of two is the only way to shrink the rater
component.** This paper demonstrates it on 50 items only; shrinking it across all 308 would require a
second rater to judge the whole pool — not realistic.

> See `S2` §2.7.11; §8.1.2 of this paper; `S1·_四方判断_20261108.md`,
> `S1·_四方对照.py` (7 self-tests) and `S1·_四方包含.py`.

---

---

## 4.7 Other measurements of data quality

| Check | Result | Treatment |
|---|---|---|
| sentences containing any of the 22 internal-workflow markers | **0** (filter effective) | — |
| meta-information lines (containing links / frame IDs / "line N") | **1** | reported, not removed |
| median share of half-width characters | 24% | — |
| median sentence length | 34 characters | — |

That one meta-information line reads `38 pairs × 3 seeds (Table 2, line 1151)` (LCS = 6).

> **It lies inside the 18 "definite meta-information" items of §4.1.4**: it matched the criterion on
> "line N", the second item of the definite-meta-information rule.
> **That is, the "1" here is a subset of the "18" there, not a separate class.** The two are neither
> double-counted nor contradictory.

Removing it gives **206/307 = 67.10%** (against 207/308 = 67.21%), a difference of **0.11 percentage
points**, **which does not change the main conclusion.**

> **But one may not say "does not change any conclusion"**: **the median moves** — with $n=308$ it is
> the mean of the 154th and 155th values, $(4+5)/2=4.5$; after removing one item with $\mathrm{LCS}=6$
> we have $n=307$ and the median becomes the 154th value, **4**.
> So the accurate statement is "**this removal does not move the three-accounting central numbers, but
> it does move the distributional description**" — and this paper **did not** remove it, so the 4.5 in
> the table needs no correction. (Both figures are **count shares**, not false-negative rates; the
> latter, after the human labels, are the three accountings 40.14 / 26.57 / 17.30.) **It was not
> removed because under the stated criterion it belongs to this set; removing it afterwards on
> "does it look like a content sentence" would be changing the data to suit the result.**

### 4.7.1 Distribution

| Quantity | Value |
|---|---:|
| $n$ | 308 |
| median LCS | **4.5** |
| quartiles ($s_{n/4}$ / $s_{3n/4}$) | 3 / 8 |
| 10% / 90% quantiles | 2 / 13 |
| items with LCS = 2 | 67 |
| judged unsourced (≤6) | 207 (67.21% of 308 — a **count share**, not a false-negative rate; and under the **old** convention, see §5.2.2) |
| to-human (7–11) | 62 (20.13%) |
| sourced (≥12) | 39 (12.66%) |

> **The median is 4.5, not 4.** With $n=308$ even, the median is the mean of the 154th and 155th
> values, $(4+5)/2$. Earlier internal drafts wrote 4, copied forward without recomputation. **It enters
> no formula and appears only in descriptive sentences — which is exactly the kind of number most
> likely to enter a manuscript wrong.**

The shape of the distribution deserves a remark: **67 items are piled at LCS = 2**, 21.8% of the total.
**One possible explanation**: sentences were cut to 12–100 characters, so a short sentence's longest
common substring with a source of tens of thousands of characters will not be long anyway. **But this is
an explanation only; this paper did not test it** — varying source length, varying corpus, and
comparison against a null model were all left undone.
> **"This is not a coincidence" may not be said**: that would **treat an untested causal story as an
> established mechanism**, and the sentence also says it will be used when §5 discusses the shape of the
> threshold curve — an untested explanation would then be inherited downstream.
> **What the quantiles support is the observation "the dynamic range is narrow"** (median 4.5, and the
> threshold of 7 already sits at the 67th percentile); **the "why is it narrow" half is conjecture.**

---

## 4.8 The honest boundaries of this chapter (summary, carried into §8)

| # | Boundary | Impact |
|---|---|---|
| 1 | **The constructed ground truth has been validated by human labels; the earlier "84 morphologically suspicious items" assumption was overturned** | ground truth moves from the old interval $[39.94\%,67.21\%]$ to **$[17.30\%,40.14\%]$** (point estimate 26.57%); first item of §8 |
| 2 | One meta-information line is left in the dataset | 67.21% vs 67.10% (**both count shares**), a difference of 0.11 points |
| 3 | Both scorers are inflated by long digit strings and paths | changing scorers does not fix it; denoising was tried and failed (AUC fell) |
| 4 | C3's optimal cut-off on B-main does not point the same way as LCS's | "the two scorers converge" holds only on D-pure |
| 5 | C3 is unstable on short sentences (15 of the 19 disagreements are ≤40 characters) | a short-sentence corpus should not use C3 alone |
| 6 | The 80.5% diagonal is a construction of quantile alignment | may not be cited as agreement evidence |
| 7 | Semantic similarity was not used | it would conflate paraphrase with quotation — **a deliberate exclusion** |
| 8 | The morphological criteria are machine criteria (regular expressions) and may themselves be wrong | self-checked by drawing all of L1 ($p_1$) |
| 9 | The data cannot be released | recomputability rests on scripts and a criterion table, not on data sharing |
| 10 | **For mixed sentences only the English half was measured to be in the source**; whether the Chinese half is a rewrite has no independent evidence | §4.4.3 may claim "one stretch of verbatim text", not "the whole sentence is sourced" |
| 11 | The negatives' ground truth is constructed too, but **in the opposite direction** (proving "not present" is exhaustively checkable) | firmer than the positive ground truth; but anyone reusing them to compute false positives must be careful |
| 12 | **"The two scorers are of the same order" is a measured result under the current stipulation, on D-pure only — not a criterion-invariant theorem** | whether they remain aligned after the ground-truth correction is decided by P6; **it does not hold on B-main** (C3's optimum 53.90% is outside the 37–39% band), so the family qualifier must appear with the sentence |
| 13 | **The 0.94 points of false positives at the $t\ge7$ end rest on the criterion choice "verbatim overlap of English terminology is not an error"** | under the other criterion (counting terminology overlap as an error) the figure is 3.00%; both must be reported, and the smaller one may not be reported alone (§4.1.5.1) |
| 14 | **The 11 "format · digits and symbols" items (11/1,803 = 0.61%) are a scorer defect, not a data defect** | it shows LCS is inflated on corpora with dates/paths/identifiers; if §5 deploys LCS, this 0.61 is a floor cost |
| 15 | **The 27 class-C disagreements are counted but not attributed** | not an omission but a decision: without reading the matched strings item by item, no explanation may be invented (§4.4.2) |

> **On item 7, one may say only "deliberately not used", not "not feasible"**: "not feasible" is a
> **capability judgement**, and the paper records no attempt and no evidence of feasibility. The row
> therefore claims only "deliberately not used"; on whether it could be done, **this paper neither tests
> nor takes a position.**

---

## 4.9 Cross-language recomputation: the statistical chain of this chapter, recomputed in a second language

> **The title may not say "every number".** What was recomputed is the **statistical chain** — the
> three ENR accountings, the nine endpoints, the intervals, the two-rater intersection, and the
> orthogonal-contrast sums of squares.
> **These items were not recomputed**: descriptive statistics such as "median share of half-width
> characters 24%" and "median sentence length 34 characters", and the quantiles of §4.7.1 — they lie
> outside this chain. **So what this section shows is that "the arithmetic of the central numbers is
> right", not that "every number in the chapter is right".**

Every statistic in this chapter (and the central number of §5) is a hand-written formula, and the easiest
error to make is one that is wrong without raising an error. Two rounds of **cross-language
recomputation** were therefore run, with **raw data** as input (item-level LCS, item-level human labels)
rather than pre-computed intermediates — handing over intermediates would let the original
implementation mark its own homework.

| Round | What was verified | Environment | Result |
|---|---|---|---|
| 1 | item-level LCS for 2,462 items, the full FN/FP table, the old upper bound 67.21% (superseded, see §5.2.2), the extreme-value ratio 8.375 | MATLAB Online R2026b | **7/7 passed**, 2,462 items **0 mismatches** |
| 2 | the three stratified extrapolations, analytic variance and intervals, the two-rater intersection, the orthogonal-contrast sums of squares | same | **19/19 passed**, maximum deviation 0.0047 |

Round 2, item by item (expected values taken from **the numbers printed in this paper**, not from
recomputation):

| Item | MATLAB | This paper | Difference |
|---|---:|---:|---:|
| ENR upper / point / lower (Rater 1) | 40.1429 / 26.5718 / 17.2987 | 40.14 / 26.57 / 17.30 | ≤0.003 |
| six endpoints (three accountings × upper and lower bounds) | see table | see table | ≤0.005 |
| ENR upper / point / lower (Rater 2) | 36.5065 / 31.3885 / 27.3247 | 36.51 / 31.39 / 27.32 | ≤0.005 |
| intersection accounting | 13.7922 [7.9354, 19.6490] | 13.79 [7.94, 19.65] | ≤0.005 |
| orthogonal contrast **SSW / SSB** | **6.2533 / 0.2907** | 6.2533 / 0.2907 | **0.0000** |

> **All deviations are ≤0.005 and their signs are random** — that is the effect of rounding to two
> decimals in the text, not an algorithmic difference.
> **The two sums of squares agree to four decimals** — because their inputs are integers 0/1 and carry
> no rounding error.
> **But the phrase "exactly identical" must be withdrawn**: the table shows four decimals, and
> "difference 0.0000" establishes only **agreement at that display precision**; integer inputs cannot
> rule out floating-point rounding in intermediate steps such as means and divisions either.
> What can be said is that **the two items show no distinguishable difference at display precision**,
> which is enough to exclude implementation error — but it must not be written as a mathematical
> identity.

**This round did catch a real bug** (in the §4 implementation, not in §5's numbers): MATLAB's
`textscan` returns **column vectors**, and `./` against a row vector **silently broadcasts into a
matrix**, turning ENR from a scalar into a vector. Only after fixing it did everything pass.
**It raises no error; only checking it against a known answer exposes it** — which is exactly why this
paper insists on recomputing **to the numbers printed in the paper** rather than "to roughly the same
thing".

> **What this section does and does not establish must be kept apart.**
> **It establishes "the arithmetic is right"** — two languages counted, extrapolated and computed
> variance independently, and 19 items agree item by item.
> **It does not establish "the conclusion is right"** — whether 26.57% is real, and whether C should be
> removed, depend on the ground-truth labelling itself, and **that is not something computation
> settles**. Cross-language agreement can exclude implementation error; **it cannot substitute for
> checking the ground truth.**

> To reproduce: run `S1·_matlab_stats/verify_stats.m` in MATLAB Online (with the five input `.tsv`
> files in the same directory). Login and upload are browser automation and are not provided with the
> manuscript. Conclusions and per-item output: `S1·_matlab_stats/MATLAB验证结论_20261109.md` and
> `S1·_matlab_stats/MATLAB输出_20261109_0027.txt`.

---

## 4.10 Supplementary material: every number can be re-run

Every number in this paper comes from an executable script. Three groups of supplementary material
accompany the manuscript. **Throughout the text, `S1·`, `S2 §x.y` and `S3 图n` refer to these.**

| ID | Content | Size |
|---|---|---|
| **S1 recomputation package** | 30 scripts (**all with `--自测`**), 11 label data files, 7 per-item records, the MATLAB package (`verify_stats.m` plus 5 input `.tsv`), the SVG sources of both figures | — |
| **S2 data conventions** | glossary, three definitions, **the provenance of every number**, and a blacklist (statements already falsified or downgraded) | 1 file |
| **S3 figures** | vector sources of Figures 1 and 2 | 2 files |

**How to run** (inside `S1_复算包/论文_分章/_实验/`):

```
python _斜率全扫.py --自测        # §5.2.1 extreme-value ratio 8.375
python _§5_人判新数.py            # Table 5.2 and the nine-cell table
python _核Lambda表.py             # the Λ table of §5.6.1 (the inversion)
python _核DOI.py                  # §2.7 DOI verification, item by item
```

**S1 keeps its original relative directory structure**, because the scripts locate data in three
different ways (bare filename / two levels up from `__file__` / a single working-root constant);
flattening the directory makes most of them fail. **All 30 scripts in the package pass `--自测`.**

> **Two boundaries must be stated.**
> ① **Anonymisation touched only identity and paths** (internal role names became "Rater 1 / second
> rater"; absolute paths became relative), **with no number or line of logic changed**; the only logical
> change replaced a hard-coded working root with a derivation from `__file__`, after which all
> self-tests pass.
> ② **The five scoring pages (`.html`, about 1.6 MB each) were not included** — they embed the source
> transcripts of the 100 sentences, and both their size and the redistribution of third-party text make
> inclusion inappropriate. The **structure and criteria** of the scoring pages are in
> `S1·_判据_C档.md`, and the item-level labels are in `S1·_人判_结果_20261108.json` and
> `S1·_人判_结果_第二名判者_20261108.json` (containing identifiers and labels only).

---

# 5 Threshold Sweep: Main Results

> **Three claims this chapter must establish.**
> ① **Under the point and upper-bound accountings, no point in $t=4\dots8$ holds both ends down.**
> Requiring a false-positive rate ≤ 5% forces false negatives of at least **26.57%** (point estimate;
> **three-accounting spread** 17.30–40.14%, not a confidence interval); requiring false negatives ≤ 25%
> pushes false positives to **6.60%** (point estimate; 16.42% under the least favourable accounting).
> **The one exception is the lower-bound accounting**, where $t=7$ and $t=8$ both satisfy the two
> constraints — see §5.3.
> ② **$t=7$ is not a well-tuned number; it is equivalent to an assumption about cost** — that letting a
> fabricated claim through is **1.16–1.41 times** more costly than wrongly rejecting a genuine one
> (point estimate; 1.76–2.14 under the least favourable accounting). Change the cost ratio and the
> optimal $t$ jumps to another cell immediately.
> ③ **And that assumption depends on which negative family you measure it against** — against purely
> natural negatives, **$t=7$ does not even have an optimal interval**, because $t=6$ is better on false
> negatives in that family (24.00% against 26.57%), at the price of higher false positives on family B
> (5.98% against 2.56%). **So "strict dominance" cannot be claimed** (dominance requires being no worse
> on both), only that "on this family $t=7$ has no defensible optimality".
> **A qualifier on the two numbers**: **26.57% is pinned by human labels**; **24.00% is a scaled
> extrapolation** ($\mathrm{ENR}(6)=f\cdot\mathrm{ENR}_{\text{machine upper}}(6)$; the labels cover only
> $t=7$). So "t=6 is better" **is a directional judgement in this family, not a measured result.**
> **A boundary, stated first**: the human labels pin down $t=7$ only; other $t$ are obtained by
> proportional scaling $\mathrm{ENR}(t)=f\cdot\mathrm{ENR}_{\text{machine upper}}(t)$, which **is an
> assumption, not a measurement** ($f$ = 0.5973 / 0.3954 / 0.2574 for upper / point / lower). Cells at
> $t\ge9$ (high end) and $t\le3$ (low end) are marked "scaled extrapolation" and are not cited as
> conclusions — **this paper is answerable only for $t=4\dots8$** (reasons at the end of §5.5.4).

---

## 5.0 Pinning down the accounting

The same $t$ appears below with three different denominators, and mixing any one of them up
invalidates everything:

| Quantity | Denominator | Meaning |
|---|---|---|
| false-negative rate $\mathrm{ENR}(t)=\hat F(t-1)$ | **$n_1=308$** | among sentences stipulated sourced, the share judged unsourced — **wrongful rejection** |
| false-positive rate $\mathrm{EPR}(t)$ | **1,803** (family D-full) | among fabricated sentences, the share judged sourced — **letting through** |
| the same quantity, for comparison | **351** (family B-main) | letting-through rate at the same $t$ against "another paper from the same domain, mismatched" |

**Two English names serve only as reminders; the formal criterion is the symbol.** This paper treats
"sourced" as the **positive** class, which is **the opposite** of the positive class in medical testing
("diseased"), so any naming that follows "who is harmed" attaches to the wrong error type:

> **wrongful rejection = $\mathrm{ENR}$ = false negative = a genuinely sourced sentence judged
> unsourced** (the sentence is what suffers);
> **letting through = $\mathrm{EPR}$ = false positive = a fabricated claim judged sourced** (the reader
> is what suffers).
> **Write the two definitions down; do not reason by feel.** Wherever the prose says "how many wrongly
> rejected, how many let through", **convert to ENR / EPR before reading on.**

**Three disciplines; violating any one voids this chapter's numbers:**

1. **Never write "accuracy".** All 308 are positive ($n_0=0$), so accuracy is undefined (`S2` B1).
2. **Never write "total error rate".** The two errors live on different denominators and cannot be added.
3. **$\mathrm{ENR}(7)$ has three accountings and must be reported together.** Human labels give
   **40.14% (upper, C counts as found) / 26.57% (point, C removed) / 17.30% (lower, C counts as not
   found)**, interval $[17.30\%,40.14\%]$; see §5.2.2. **Any one alone is a naked number.**

---

## 5.1 How the sweep is done

For each candidate $t\in\{2,\dots,13\}$, compute two numbers. That is the whole procedure:

```
FN(t) = #{positives with LCS < t} / 308
FP(t) = #{family D-full with LCS >= t} / 1803
```

The sweep stops at 13, because at $t=13$ $\mathrm{FP}=0$ (the maximum LCS in family D-full is exactly
12); raising $t$ further only increases FN without changing FP.

> **These two lines compute the machine convention, not the ground-truth convention this paper
> reports.** The $\mathrm{FN}$ above has denominator 308 and numerator "the machine's count judged
> unsourced" — it corresponds to the **old upper bound in which all 207 count as wrongful rejections**,
> giving **67.21%** at $t=7$. **The 26.57% this paper finally reports is not produced by this formula**;
> it is recomputed after the human labels under the three accountings (§5.2.2).
> Why keep the two lines then: **they give the shape of the curve** (which segment is steep, which
> flat), and shape is invariant to the labelling correction (the three accountings differ only by a
> positive multiplier; §5.5.3). **To recompute this paper's conclusions, go to §5.2.2; to understand the
> shape of the curve, read these two lines.**

> **Why not "take the top $k$ items" and reorder the sample**: this paper's decision rule is a
> **one-sided cutoff** — it looks only at this sentence and its longest common substring with the
> source, and depends on no other sample. The threshold sweep therefore re-cuts a fixed reading rather
> than re-ranking. The distinction reappears in §6 (decision sets are monotone **increasing** in $t$).

---

## 5.2 Table 5.1 and Figure 1: the two curves move in opposite directions

**Table 5.1 · the complete threshold sweep** ($n_1=308$; family D-full $n=1{,}803$; family B-main
$n=351$)

| $t$ | judged unsourced | false-negative rate (point) | D-full FP count | D-full FP rate | B FP rate |
|---:|---:|---:|---:|---:|---:|
| 2 | 9 | 1.16% | 1,632 | 90.52% | 96.30% |
| 3 | 76 | 9.76% | 829 | 45.98% | 58.97% |
| 4 | 120 | 15.40% | 296 | 16.42% | 24.79% |
| 5 | 154 | 19.77% | 160 | 8.87% | 15.38% |
| 6 | 187 | 24.00% | 119 | 6.60% | 5.98% |
| **7** | **207** | **26.57%** | **54** | **3.00%** | **2.56%** |
| 8 | 223 | 28.63% | 28 | 1.55% | 0.85% |
| 9 | 236 | 30.29% | 15 | 0.83% | 0.57% |
| 10 | 248 | 31.83% | 6 | 0.33% | 0.28% |
| 11 | 261 | 33.50% | 2 | 0.11% | 0.00% |
| 12 | 269 | 34.53% | 1 | 0.06% | 0.00% |
| 13 | 277 | 35.56% | 0 | 0.00% | 0.00% |

> Generation: `S1·_出图.py`; self-test: `--自测` (10 assertions, including that $t=7$ must reproduce
> 207/308 and 54/1,803).

> **The "false-negative rate" column is the stratified-extrapolated point estimate (point accounting:
> unclear labels removed from the denominator).** It is **not** "judged unsourced ÷ 308" — that would be
> the **earlier upper-bound convention**, which assumes all 207 are wrongful rejections (207/308 =
> 67.21%); **it is not what is used here**. The full three-accounting curve is in §5.2.2.
> This is more than a footnote: **recomputing the cost under a different accounting moves the optimal
> $t$** (comparison table in §5.5.4). Every false-negative rate in this chapter must therefore be
> answerable as to **which accounting it is.**

**Figure 1 (`S3 图1`)** — two curves (red = false-negative rate, blue = family D-full false-positive
rate) sliding in opposite directions, **crossing exactly once**. The crossing point moves with the
accounting, which is itself a useful observation:

| Accounting | at $t=4$: FN vs FP | at $t=5$: FN vs FP | crossing falls in |
|---|---|---|---|
| upper | 23.27% **>** 16.42% | — | $(3,4)$; at $t=3$, 14.74% < 45.98% |
| point | 15.40% **<** 16.42% | 19.77% **>** 8.87% | $(4,5)$ |
| lower | 10.03% < 16.42% | 12.87% > 8.87% | $(4,5)$ |

**The less favourable the accounting, the further left the crossing moves** — because false negatives
are raised and catch up with false positives sooner. Under all three accountings both sides of the
crossing are "good at one end, bad at the other", and the crossing itself is not a usable point. What
this chapter argues is not "there is no crossing" but that **the frontier contains no simultaneously
acceptable operating point** (§5.3).

### 5.2.1 The cost of one step is wildly uneven: 8.38×

The threshold steps are equidistant on the number line, but what each step buys is far from equivalent.
**All 11 adjacent intervals in $t\in\{2,\dots,13\}$:**

| Interval | Increase per step (percentage points) | Interval | Increase per step (percentage points) |
|---|---:|---|---:|
| $t=2\to3$ | **21.75** ← largest | $t=8\to9$ | 4.22 |
| $t=3\to4$ | 14.29 | $t=9\to10$ | 3.90 |
| $t=4\to5$ | 11.04 | $t=10\to11$ | 4.22 |
| $t=5\to6$ | 10.71 | $t=11\to12$ | **2.60** ← smallest (tied) |
| $t=6\to7$ | **6.49** | $t=12\to13$ | **2.60** ← smallest (tied) |
| $t=7\to8$ | 5.19 | | |

**Extreme-value ratio $=67/8=8.375\to 8.38×$** (computed from raw counts; dividing the rounded 21.75 and
2.60 from the table gives 8.37 — the difference is rounding, and both are stated). The minimum is
**tied** between $t=11\to12$ and $t=12\to13$, not a single interval.

> **The quantity in this column is "difference in judged-unsourced count ÷ 308", independent of the
> false-negative accounting** (however many of the 207 count as wrongful rejections, they are all
> "judged unsourced"). This section is therefore orthogonal to the three-accounting dispute — **it
> measures how fast the scorer cuts, not how correctly**.

> **And the ratio is a lower bound, not a point value.** What can be guaranteed is **8.375** ($67/8$);
> 8.38 is only its two-decimal display — **it must not be written as "$\ge 8.38$"**. The reason: the
> sweep stops at $t=13$. Extending it **cannot shrink** the ratio: at $t=13$, 277 are already judged
> unsourced, leaving only $308-277=31$ positives, so any single-step slope at $t\ge14$ is bounded by
> $31/308=10.06$ points, **far below 21.75**; the maximum is locked at $t=2\to3$, and the minimum can
> only fall further. So the worry that "extending the sweep would widen the ratio" has **the direction
> reversed** — extending it only makes the ratio larger.

> **Why this matters**: any argument of the form "we moved the threshold by 1" must first say **which
> segment's 1**. Moving 1 near $t=2$ equals moving 8.38 times near $t=11$ in terms of what is bought.

> **Why all 11 adjacent intervals must be swept**: an extreme-value ratio computed from three selected
> segments cannot support the word "any"; and treating a **two-step average** (4.05 / 3.40) as "the
> per-unit slope" and dividing it into the single-step 21.75 is **an inconsistent numerator and
> denominator**. Recomputed step by step, the minimum is 2.60 at $t=11\to12$ and the extreme-value
> ratio is **8.375** (exactly $67/8$). Recompute with `S1·_斜率全扫.py` (8 self-tests).

### 5.2.2 The false-negative rate has three accountings; reporting one alone is a naked number

The column in Table 5.1 is the **point estimate**. This section explains where it comes from and what
the other two are — because **which accounting is used changes every multiplier in the second half of
this chapter.**

**Source**: the P6 human validation of 2026-11-08. The 308 were stratified into 5 layers by the rule's
output and sentence form, and 100 were drawn for a human to judge "can the content of this sentence be
found in the original source document" (A found / B not found / C unclear). The three relevant strata
together (pool 207, drawn 73): **A 22 / B 24 / C 27**. Stratified extrapolation
$\widehat{\mathrm{ENR}}(7)=\sum_i N_i p_i/308$, with $p_i$ the within-stratum share answering A.

**Three accountings and their intervals** (analytic form with finite-population correction, cross-checked
by 4,000 bootstrap replicates):

| Accounting | How C is counted | $\widehat{\mathrm{ENR}}(7)$ | 95% interval | Implied wrongful rejections (**base 308**) |
|---|---|---:|---|---:|
| **upper** | C counts as found | **40.14%** | [32.61%, 47.67%] | 124 |
| **point** | C removed from the denominator | **26.57%** | [19.93%, 33.22%] | 82 |
| **lower** | C counts as not found | **17.30%** | [11.34%, 23.26%] | 53 |
| **intersection** (a different definition of A) | A only if **both raters** say A; everything else non-A | **13.79%** | [7.94%, 19.65%] | 42 |

> **The base of that column is 308, not 207**: $124=308\times40.14\%$, $82=308\times26.57\%$,
> $53=308\times17.30\%$, $42=308\times13.79\%$.
> **The last row is not the same kind of thing as the first three**: the first three dispute "what C
> counts as", this one changes **"how A is defined"** (one rater saying A does not suffice; both must).
> It comes from the full two-rater judging of 2026-11-08 (§8.1.4); **it is not a fourth accounting of C
> but a fourth algorithm**.
> **Being lower than the lower bound does not make it closer to the truth** — it merely raises the bar
> for A. **The two cannot be compared in magnitude; each must be reported on its own.**
> If the central number were actually switched, the consequences are concrete: the cost ratio would move
> from **1.16–1.41** to **0.60–0.73**, and the equal-weight optimum from **5** to **8**
> (`S1·_交集口径.py`). **This paper does not switch**, for the reasons in §8.1.4.

**The full curve** ($t=2..13$; **only the $t=7$ row is pinned by human labels, the rest are scaled**):

| $t$ | upper | point | lower | | $t$ | upper | point | lower |
|---:|---:|---:|---:|---|---:|---:|---:|---:|
| 2 | 1.75% | 1.16% | 0.75% | | 8 | 43.25% | 28.63% | 18.64% |
| 3 | 14.74% | 9.76% | 6.35% | | 9 | 45.77% | 30.29% | 19.72% |
| 4 | 23.27% | 15.40% | 10.03% | | 10 | 48.09% | 31.83% | 20.73% |
| 5 | 29.86% | 19.77% | 12.87% | | 11 | 50.61% | 33.50% | 21.81% |
| 6 | 36.26% | 24.00% | 15.63% | | 12 | 52.17% | 34.53% | 22.48% |
| **7** | **40.14%** | **26.57%** | **17.30%** | | 13 | 53.72% | 35.56% | 23.15% |

> Generation: `S1·_§5_人判新数.py` (12 self-tests); scoring: `S1·_人判_核分.py` (9 self-tests).

**Three caveats that must be read with this table:**

1. **The old interval $[39.94\%,67.21\%]$ is void.** 67.21% presupposes that all 207 are wrongful
   rejections; the labels say only **53–124** (point estimate 82) can actually be found in their
   sources. **Even the upper accounting 40.14%, the one least favourable to the rule, only just reaches
   the old lower bound of 39.94%** — no accounting reproduces the old upper bound.

2. **C accounts for 32%, which is why the interval is 23 points wide.** This is not imprecision; it is
   that **a human genuinely cannot decide** — half of these are "found only half of it" or "true if
   reworded". **Reporting 26.57% without the interval would pretend that 32% ambiguity does not
   exist.**

3. **Half of the old lower bound's derivation was falsified on the spot.** The old lower bound assumed
   that the 18 definite meta-information items (starting with "harvest" / containing "see §…" / "line N"
   / "one-pager") could never occur in the source. **All 18 L1 items were drawn and judged, and 5 were
   found (27.8%)** — the proper names in the headings (`EvoSci`, `DeepSeek Super-agent`) are indeed in
   the source. That criterion **was a surface-form criterion, not a content criterion**, and it has been
   corrected everywhere it is cited.

> **Fourth: how firm these three bands are has been measured.** The same rater re-judged the same 50
> items after one round: **overall agreement was only 54% ($\kappa=0.32$, fair)**, but **the bands
> differ enormously** — **A (found) 80% agreement, B (not found) 53%, C (unclear) only 33%** — and
> **C's flow is essentially one-way**: of the 18 C items in the first pass only 6 remained C (8 moved to
> A, 4 to B), with **only one flowing in from B and none from A**.
> **What can be said is that C's usage still differs by 16 items under the same criteria, showing that
> the criteria themselves were never fixed.**
> **But one may not say "so it is not the rater's ability"**: the 50 re-ratings **measured agreement
> only** and did not independently measure ability — they cannot establish "ability is fine", only that
> "unfixed criteria are a reasonable explanation".
>
> **This does not overturn the central number; it supplies an additional reason for it**: upper (C
> counts as found) and lower (C counts as not found) **both assign a value to C**, whereas **point
> accounting is the only convention that assigns none** — **the most robust to C's instability**. Taking
> 26.57% is no longer merely "taking the middle"; **it has a reliability argument behind it.**
>
> **But the interpretation of the interval's two ends must change**: 17.30 and 40.14 are **not "two
> equally defensible conventions"** but a measure of **"how far the number can drift when the criteria
> are unclear"**. Details in §4.6.6 (confusion matrix, three-way cross-check) and `S2` §2.7.9.
> **The statement "the rater's single-pass error rate is 35.9%" is forbidden** — that conversion
> requires the two passes to err independently and at random, and the measured errors are systematically
> one-way, so the premise fails.

> **Fifth: the error has three dimensions and four rows, which must not be combined into a single ±, and
> each base must be stated.**
>
> | Component | Value | Base | Source |
> |---|---:|---|---|
> | Sampling error | ±6.65 points | 308 items, stratified extrapolation | half-width of the 95% interval |
> | Accounting spread (how C is counted) | 23 points | **308 items** | upper 40.14 − lower 17.30 |
> | *(same-base comparison)* | *36 points* | *50 items* | three-accounting spread within one pass |
> | **Rater drift (category share)** | **≤50 points, direction unknown** | **50 items** | lower accounting across three passes, 30% → 80% |
> | **Rater difference (central number, point accounting)** | **4.82 points** (26.57 ↔ 31.39) | **100 items, different raters** | §8.1.4 |
>
> **The last two rows belong to the same dimension but measure different things and rest on different
> bases**: one measures how the A share drifts (50 items), the other how far the central number moves
> (100 items). **Neither addable, nor convertible, nor substitutable.**
> **This row carries a theoretical burden**: when the two raters differ by 20 C items, the lower
> accounting drifts by **10.03** points while point accounting drifts only **4.82** — **taking point
> accounting is no longer merely "taking the middle".** But the comparison must be stated precisely:
> **point accounting is steadier than lower, and upper is steadier still (3.64 points)**; what point
> accounting really buys is **"no value need be chosen for C"**, not "minimum drift" (see the
> cross-pass table in §8.1: upper 14.0 / point 33.1).
>
> A third pass added hard criteria and re-rated the same 50: the A share went **30% → 54% → 80%** and C
> went **36% → 14% → 0%**.
> **On the same 50 items, drift (≤50) and accounting spread (≤36) are of the same order** — that is,
> the axis "even whether something counts as A moves" is **at least as large as "how C is counted",
> while the interval in this section covers only the latter**.
> The direction cannot be determined: did the criteria correct over-strictness, or cause criterion
> drift? **This paper has no external gold standard and no control arm, and cannot separate them.**
> (Evidence: pass 1 → pass 2 **added no criteria** and A still rose 24 points — merely re-judging once
> moves the standard.)
>
> **The form "26.57% ±6.65 points" suggests a total uncertainty only 13.3 points wide — it is far wider.**
> **But the 50 points must not be added to 26.57**: **drift on the 308 items was not measured** (only on
> 50), still less may one write "drift 50 > accounting 23 in this section" (**different bases**).
> **The central figures of this paper can be read as an order of magnitude, not as precise values.**
> Details in §4.6.7, §8.1.1, `S2` §2.7.10.

> **Sixth: agreement at the "is it A" layer is higher than at "B or C" — but both layers disagree.**
> **One may not say "the ambiguity is not in A"**: 74% A-versus-non-A agreement still means **13 of the
> 50 crossed the A/non-A boundary**. The material supports "A is comparatively steadier", **not "there
> is no ambiguity in A".**
>
> A second rater was recruited, judging the **same 50 items** on the **same criteria page**. **Of the
> three pairings the two ends are single-factor contrasts; the middle one moves two factors at once**
> (retest plus criteria).
>
> | Pairing | What differs | three-way | κ | **A vs non-A** |
> |---|---|---:|---:|---:|
> | pass 1 vs pass 2 | **pure retest** | 54% | 0.322 | **64%** |
> | pass 2 vs pass 3 | retest + criteria | 68% | 0.365 | 70% |
> | pass 3 vs Rater 3 | **pure change of rater** | 60% | 0.259 | **74%** |
>
> **① On A versus non-A the three pairings give 64% / 70% / 74% — all within 64–74%, and no single
> change drives agreement down.** "Changing the rater is steadier" cannot be said, because **the
> "change of rater" term cannot be isolated** (this paper has no "same person, same criteria, different
> occasion" arm) (§8.1.2 ②).
> **② Criteria raise self-consistency (54 → 68%) but not cross-rater consistency (60%, with κ falling
> to 0.259)** — the criteria were internalised individually, not executed jointly.
> **③ And the A verdict stands in proper containment**: pass 3 has 40 A items, Rater 3 has 27, **zero
> in the reverse direction** (Rater 3's A ⊂ Rater 1's A).
> **Random error runs both ways; a difference in strictness runs one way — we observe one way, so this
> is a difference in leniency, not misjudgement.** The intersection gives **27 doubly-confirmed A**.
> **④ The B/C boundary is personal habit**: of the 10 items pass 3 called B, **7 were called C by
> Rater 3 and none A**.
>
> **⑤ This is decisive for the three accountings:**
>
> | Accounting | Definition | Dependence on A | Dependence on C |
> |---|---|---|---|
> | upper | $(A+C)/n$ | first order | **C enters the numerator** |
> | **point** | $A/(A+B)$ | **first order (numerator is A alone)** | **C removed, only second-order through the denominator** |
> | lower | $A/n$ | first order | **C enters the denominator** |
>
> Measured: **agreement within the A band (74%) exceeds that within B/C (46.8%)**, the A verdict shows
> one-way containment, and **C is the least stable** (0 vs 16 under the same criteria).
> **But one may not say "A is stable, B/C is not"**: of the 51 disagreements, 26 are A-versus-non-A and
> 25 are B-versus-C — **almost exactly half each** (§1.6).
> → **Neither the numerator nor the denominator of point accounting contains $C$**, so **no value need be
> assigned to C**; but "unaffected by C" does not hold ($C\to A$ / $C\to B$ changes one or the other).
> The real mechanism is in §8.1.2 ⑥.
> **This paper takes point accounting (26.57%), its reason having moved from "taking the middle" to
> "resting on the only reliable binary"** — the third and firmest reinforcement.
>
> **Two statements are also withdrawn**: ① "the human labels are unreliable" → only the **B/C boundary**
> is unreliable; A is usable; ② "the criteria closed off C" → it held only in that one pass, by the rater who wrote
> them; under the same criteria others still used 32% C.
> Details in §4.6.8, §8.1.2, `S2` §2.7.11.

> **This is not a disclaimer; it changes how the conclusions are read**:
> - **When saying "26.57%"**: that is the point estimate; the **three-accounting spread** is 17.30–40.14%
>   (**not a confidence interval** — its 95% interval is [19.93%, 33.22%]);
> - **When saying "to keep FP ≤ 5% we must accept at least X wrong rejections"**: **X must take the
>   smallest of the three, 17.30%**, so that all three hold; writing 26.57% holds only for upper and
>   point and **overstates by 9.27 points** under the lower accounting. **The minimum is the only form
>   that can enter a conclusion** — and it is markedly weaker than the old lower bound's "at least
>   39.94%".

---

---

## 5.3 No good point on the frontier: four business floors, four bills

An auditor usually does not state a cost ratio directly; they state a floor ("letting through no more than X"). Substituting
the floor gives the following returns (**false negatives always in three accountings**, because this
section's conclusion is sensitive to the accounting):

| Auditor's floor | Smallest $t$ meeting it | False-negative rate there (upper / point / lower) | Wrongful rejections (upper / point / lower) |
|---|---:|---|---|
| FP ≤ 1% | 9 † | 45.77% / **30.29%** / 19.72% | 141 / 93 / 61 |
| FP ≤ 3% | 7 | 40.14% / **26.57%** / 17.30% | 124 / 82 / 53 |
| FP ≤ 5% | 7 | 40.14% / **26.57%** / 17.30% | 124 / 82 / 53 |
| FP ≤ 17% | 4 | 23.27% / **15.40%** / 10.03% | 72 / 47 / 31 |

> The first row is marked with †: $t=9$ lies outside the range covered by human labels, so that row can only
> be read as a **scaled extrapolation** and is not used as a conclusion (see the last item of `S2`
> `S2` §2.7.4).

Asking the reverse — "I do not want to reject too much" (**solutions sought only in $t\le8$**, same
reason):

| Auditor's floor | Largest $t$ meeting it (upper / point / lower) | FP rate there, family D-full (upper / point / lower) | Family B-main (same $t$) |
|---|---|---|---|
| FN ≤ 3% | 2 / 2 / 2 | 90.52% / 90.52% / 90.52% | 96.30% |
| FN ≤ 25% | **4 / 6 / 8** | 16.42% / 6.60% / 1.55% | 24.79% / 5.98% / 0.85% |
| FN ≤ 40% | **6 / 8 / 8** | 6.60% / 1.55% / 1.55% | 5.98% / 0.85% / 0.85% |

**Two readings of these tables:**

1. **Whether "both ends look respectable" is achievable depends entirely on which accounting is used —
   this is the most honest admission this chapter has to make.**
   - **upper**: the former needs $t\ge7$, the latter $t\le4$ → **the two conditions do not intersect**
     (empty feasible region);
   - **point**: the former needs $t\ge7$, the latter $t\le6$ → **do not intersect, but are one step
     apart**;
   - **lower**: the former needs $t\ge7$, the latter allows $t\le8$ → **a solution exists, namely
     $\{7,8\}$**. **The statement "the solution set is $\{7,8\}$" is contingent on this paper's citation policy**
   (only $t\le8$ is cited). Considering the numbers alone, under the lower accounting $t=9\dots13$ also
   have false negatives of 19.72–23.15% (≤25%) and false positives ≤5%, so the solution set would extend
   to $\{7,\dots,13\}$. **But that range lies outside the human-label coverage** — so we report only
   $\{7,8\}$, while stating this contingency explicitly rather than presenting it as the mathematically
   unique solution set. **That is: taking the lower accounting, the one most favourable to the rule, "FP ≤ 5% and FN ≤ 25%"
   can be satisfied simultaneously, and the deployed $t=7$ is inside the solution.** This is not a
   comfortable sentence, but it is what the table computes. **One may not say "the frontier never
   contains a feasible point"; only "under the point and upper accountings it contains none".**

2. **$t=4$ is no longer the most "balanced" point.** Under the point estimate the equal-weight optimum
   moves to $t=5$; at $t=4$, wrongful rejections are 15.40% and letting-through 16.42%–24.79% — both ends in the teens to low twenties. **It is far more respectable than before, but it is still "the least bad point", not
   a good one.**

> This is the section that must **not** be read as "so the threshold should be adjusted".
> The shape of the frontier (hugging the top-left corner) is determined by the **discriminative power of
> the scorer** — $\mathrm{AUC}$ 0.812 / 0.705 (§4.4.4). Adjusting $t$ can only move along that curve;
> **it cannot move off the curve**.
> **But "cannot move off the curve" rules out only "adjusting $t$" as a fix; it does not entail that
> "there are only two routes to a fix".** Changing features, changing data, changing the decision
> structure (from a cutoff to an interval rule) — **none of these was tested**, and no exclusionary
> conclusion is drawn. **What this chapter establishes is "adjusting the threshold on the existing
> scorer cannot buy a respectable operating point" — subject to the one exception: under the lower
> accounting, $t=7$ and $t=8$ do satisfy both constraints.**
> **Stated accurately**: not under the point and upper accountings; **marginally yes under the lower
> accounting** — and that "yes" itself depends on the assignment most favourable to the rule.
> The effect of changing scorers is in §4.6 (holding only on D-pure); reducing task difficulty is a
> direction for §9, not a measured result of this paper.

---

## 5.4 Change the negative family and the false-positive rate at the same $t$ differs by nearly 10×

The false-positive rate is not a number but **a family of numbers**, depending on what counts as
"fabricated":

| Negative family | $n$ | FP at $t=4$ | FP at $t=7$ | How it was built |
|---|---:|---:|---:|---|
| **D-pure** (natural sentences → external source) | 721 | **2.50%** | 0.00% (trivial) | sentences from one harvested note paired with the source of a **different** document |
| **D-full** (including dates / paths / identifiers) | 1,803 | **16.42%** | **3.00%** | as above, without removing structured debris |
| **B-main** (another paper in the same domain → wrong source) | 351 | **24.79%** | **2.56%** | **another paper** in the same field, mismatched |

**At the same $t=4$: D-pure 2.50% against B-main 24.79%, a factor of 9.9.**

> **Not every difference in a table is a difference.**
> D-full's 3.00% and B-main's 2.56% **differ by only 0.44 points, far below the ±3.8-point simultaneous
> confidence band** — **the two cannot be read side by side as "different"; they are statistically
> indistinguishable.**
> At $t=4$ the pairwise differences among the three rates are **8.37 points (24.79−16.42) / 13.92 points
> (16.42−2.50) / 22.29 points (24.79−2.50)**, and **the smallest pair, 8.37, still exceeds the
> 3.83-point band**, so the conclusion that "**these three are in fact statistically distinguishable**"
> holds.
> **But that does not substitute for the causal account of "nearly 10×"**: **a DKW band can only rule
> out "the difference is noise"; it has no power to establish "the difference comes from construction".**
> **The division of labour must be stated**: the band handles "is it noise", the mechanism handles "why
> so large". The attribution rests on the mechanism below (the high end pairs **a different source**, the
> low end pairs a **topic neighbour**); **a statistical test cannot underwrite a causal explanation.**
> **One may not say "differs by 13.9 points or more"**: only two pairs reach 13.9, and the smallest is
> 8.37 — the conclusion is unchanged, but "or more" overstates it.

> The 9.9× is not noise but a **construction effect** — **both ends must be attributed, not just the
> high one**:
> - **High end (B-main 24.79%)**: the negatives are sentences from **another source document** (§4.5
>   definition: a sentence from one source → paired with **another** source). The two documents share a
>   domain, hence much terminology and syntax, and any "literal overlap" scorer is inflated.
> **It must not be written as "another passage of the same paper"** — that is the construction of
> B-twin (64 items, Jaccard 0.356–0.923), **which is not in the main register**; the 351 in the main
> register have Jaccard only 0.088–0.186. Conflating the two constructions leaves the reader unable to
> tell whether the 24.79% includes within-document mismatches.
> - **Low end (D-pure 2.50%)**: sentences are paired with **other documents** and structured debris has
>   been removed, so the longest common substring is short by construction (the family's maximum LCS is
>   **only 5**). **2.50% does not mean "the rule is accurate"; it means "this family is easy".**
> Both ends are produced by **our chosen constructions**; neither is a bound on a "true false-positive
> rate".
> Such samples are called **hard negatives** in the contrastive-learning literature (Robinson et al.,
> 2021, ICLR) and **annotation artifacts** in the NLI / fact-checking literature (Gururangan et al.,
> 2018, NAACL; Schuster et al., 2019, EMNLP).
> **The two terms are not synonyms, and this paper uses only the former.** *Hard negatives* are defined by
> **learning difficulty** (negatives that are hard to separate); *annotation artifacts* are **spurious
> cues left by data construction that a model can exploit**. They can overlap but **neither entails the
> other** — "hard to separate" is not "contains annotation artefacts".
> **Family B here belongs to the former**: it is hard because of **terminology overlap within a domain**,
> not because of a shortcut left by the labelling process. So one may say "in the same direction as the
> annotation-artifact literature", **not "this paper uses annotation artifacts". Neither statement is a
> new finding** — that does not change.

**So this chapter's conclusion must carry a qualifier:**

> "The false-positive rate at $t=7$ is 3.00%" — **this holds only on family D-full.**
> On B-main it is 2.56%; on D-pure it is 0. **A false-positive rate divorced from its negative family
> means nothing.**

> This qualifier also explains why this paper does not dare to give a "recommended threshold":
> recommending a threshold requires first recommending a negative family, and the choice of family
> depends on **the auditor's own deployment distribution**.
> The established method is conformal calibration (Angelopoulos & Bates, 2021): **reselect the quantile on
> the auditor's local calibration set rather than copying someone else's $t$**. This paper can give
> "how to compute", not "which value to use".

---

## 5.5 Change the question: not "which $t$ is best" but "what is your cost ratio"

The first four sections all ask "given $t$, how much goes wrong". What an auditor actually cares about
is "given the cost, which $t$". This is the cost-loss formulation (Thompson & Brier, 1955; Murphy, 1977;
Richardson, 2000; Elkan, 2001; Drummond & Holte, 2006).

Define expected error:

$$E(t;w)=w\cdot \mathrm{ENR}(t)+\mathrm{EPR}(t),\qquad w=\frac{C_{\text{wrongful rejection}}}{C_{\text{letting through}}}$$

> **From here on, "optimal $t$" has one fixed and narrow meaning, stated up front**: it always means
> "**among the LCS threshold rules $t=2,\dots,13$, the one minimising $E(t;w)$**" — **not "the optimum
> over all possible decision strategies"**. The latter's optimum requires ordering by **likelihood ratio**
> (§5.6.1 shows that this paper's LCS ordering does not satisfy that premise).
> So the sentence "under equal weighting the optimum is $t=5$ under the **upper and point** accountings
> (**$t=8$** under the lower)" reads accurately as "**within the family of LCS threshold rules, $t=5$ is
> best**"; under a rule ordered by $\Lambda$ the optimum might fall elsewhere — **but that comparison was
> not computed here** (end of §5.6.1).

> **Two words were fixed in §5.0; one reminder: the numerator of $w$ is wrongful rejection
> ($\mathrm{ENR}$) and the denominator is letting through ($\mathrm{EPR}$).**
> So in $w\cdot\mathrm{ENR}+\mathrm{EPR}$, **it is wrongful rejection that is multiplied by $w$**, not
> letting through — which is exactly why "when wrongful rejection is more expensive, reject less, i.e.
> lower $t$".
> **Do not reason from the surface wording.** This paper's positive class (sourced) is the opposite of
> the medical-testing positive class (diseased), and swapping the positive class swaps the meaning of a
> false negative. **The only criterion: look at whether the subscript is ENR or EPR.**

**But the three statements below hold only when the two error rates are monotone in $t$**:
$\mathrm{ENR}(t)$ **non-decreasing**, $\mathrm{EPR}(t)$ **non-increasing** — i.e. "raising the threshold
increases wrongful rejections and decreases letting-through". This paper's measurements satisfy it (the
FN and FP columns of Table 5.1 are monotone row by row; recomputable with `S1·_§5_人判新数.py`). **But
with a non-monotone rule, all three directional conclusions below fail** — this is not a verbal caveat
but the foundation of the reasoning.

- $w>1$: **wrongfully rejecting** a genuine sentence costs more than **letting through** a fabrication →
  **lower $t$** (judge fewer as unsourced, reject less)
- $w<1$: letting through costs more → **raise $t$**
- $w=1$: the two are equally serious

**Table 5.2 · the optimal $t$ under four cost ratios** (negatives = family D-full; false-negative rates
in three accountings, because **the optimum moves with the accounting**)

> **This table has two premises, and either one missing voids it**: the **negative family** (changing it
> shifts the whole table, §5.5.2) and the **false-negative accounting** (three columns, §5.5.4).
> So every row can only say "under family D-full and a given accounting, this cost ratio corresponds to
> this $t$", never "cost ratio $w$ corresponds to $t$".

| Assumption | Optimal $t$ (upper / point / lower) | Expected error $E$ there (upper / point / lower) | FN rate there (upper / point / lower) | FP rate there (D-full) |
|---|---|---|---|---|
| $w=0.1$ (letting through 10× worse) | 10 / **11** / 11 † | 5.14% / **3.46%** / 2.29% | 48.09% / 33.50% / 21.81% | 0.33% / 0.11% / 0.11% |
| $w=0.3$ (letting through 3.3× worse) | 8 / **10** / 10 † | 14.53% / **9.88%** / 6.55% | 43.25% / 31.83% / 20.73% | 1.55% / 0.33% / 0.33% |
| $w=1$ (equal) | **5 / 5** / 8 | 38.74% / **28.64%** / 20.19% | 29.86% / 19.77% / 18.64% | 8.87% / 8.87% / 1.55% |
| $w=3.5$ (wrongful rejection 3.5× worse) | 2 / **4** / 4 | 96.65% / **70.32%** / 51.53% | 1.75% / 15.40% / 10.03% | 90.52% / 16.42% / 16.42% |
| **deployed $t=7$** | 7 (optimal under none of the three, but the gap narrows) | 43.14% / **29.57%** / 20.29% | 40.14% / 26.57% / 17.30% | 3.00% |

> **The two †-marked rows must be split by accounting, not assessed by extrapolation as a whole**:
> · $w=0.1$: 10/11/11 — **all three at $t\ge10$, so the row is entirely extrapolation**, directional
> reference only;
> · $w=0.3$: **the upper column's $t=8$ lies inside the $t=4\dots8$ this paper is answerable for — a
> measured result**; $t=10$ in the point and lower columns is extrapolation.
> **The $w=3.5$ columns must be read separately, not merged into one sentence**:
> · **upper's optimum $t=2$ lies in the low-end extrapolation region** ($t\le3$), directional only;
> · **the point and lower optima at $t=4$ are inside $t=4\dots8$ and are measured results** — they say
> "when wrongful rejection is 3.5× dearer, the optimal threshold drops to 4", at which point wrongful
> rejections are 15.40% (point) and **10.03%** (lower), with letting-through **16.42% in both**, i.e.
> **both ends in the mid-teens**.
> **Rounding convention**: the $E$ column is computed from **unrounded** values, while the two columns to
> its right (FN rate, FP rate) are **rounded**; so recomputing $E$ by hand from the table may differ by
> 0.01–0.02.
> **Principle**: every column is printed rounded from its own base, $E$ from the unrounded one — a
> 0.01–0.02 difference when adding two columns by hand is normal. **Faced with a 0.01-level discrepancy,
> check this first; do not take it for an arithmetic error.** Likewise the multiplier columns use the
> reciprocals of **unrounded** interval endpoints.
> Example: for the "upper / format + Chinese" row the $w$ interval is exactly
> $[0.1258925,\ 0.1496236]$, displayed as $[0.126,\ 0.150]$; the multipliers are
> $1/0.1496236=6.6834\to\mathbf{6.68}$ and $1/0.1258925=7.9433\to\mathbf{7.94}$.
> **Using the displayed $0.150$ gives 6.67; that 0.01 is exactly here.**
> Generation: `S1·_§5_人判新数.py` (Table 5.2 section).

> **The row of this table most worth remembering is "how far $t=7$ is from optimal under equal
> weighting"**: upper by **4.40** points, point by **0.92**, lower by **0.10**.
> **This difference used to be 14.8 points** (under the old upper-bound convention, $E$ at $t=7$ was
> 70.20% against an optimum of 55.38%). After the labels, **only the point and lower accountings can be
> called "essentially a tie"** (0.92 / 0.10); **the upper differs by 4.40, which is not a tie** — it is
> the accounting least favourable to $t=7$, and that 4.40 is a **measured** gap (both $t=5$ and $t=7$ lie
> inside $t=4\dots8$), not extrapolation.
> **The three accountings must be stated separately**: upper's 4.40 is not a tie; only point (0.92) and
> lower (0.10) can be called near-ties. **The old value of 14.8 must not be used again.**

### 5.5.1 Figure 2: the optimal $t$ is a step function of $w$

> **The methodological position, stated first**: this section **does not assume a $w$**. It
> **inverts** the system's own choice of $t=7$ to ask **which interval $w$ must lie in for $t=7$ to be
> optimal**.
> **This is a conditional statement, not an estimate**: this paper **does not, and cannot, "recover the
> value of $w$"** — $w$ is a normative judgement, and data cannot recover it.
> **The wording must be consistent**: what can be recovered is "the $w$ interval required for $t=7$ to be
> optimal", not "what $w$ is" — the latter would contradict §5.5.2's statement that "$w$ can only be
> supplied by the auditor's own cost accounting".
> This is the **revealed-cost** route, as distinct from "imposing an exogenous cost ratio".
> The advantage: **rather than asking the auditor "which do you consider dearer", it reads the answer off
> the threshold they actually chose** — so 1.16–1.41 is not a number this paper set.
> **The price**: it depends on the premise that "the threshold does not in turn change the distribution
> of the text being checked". **Once $t$ is public and the checked party rewrites adversarially, this
> conditional interval also fails.** See §8.6.

**Figure 2 (`S3 图2`)** — thin lines are the cost curves for each fixed $t$, the heavy black line is the
lower envelope (the minimum expected error for a given $w$), and each segment of the envelope is labelled
with the $t$ that is optimal there.

Three readings:

1. **The envelope is a chain of straight segments, not one slanted line** — because for fixed $t$,
   $E(t;w)=w\cdot\mathrm{ENR}(t)+\mathrm{EPR}(t)$ **is a straight line in $w$** (slope
   $\mathrm{ENR}(t)$, intercept $\mathrm{EPR}(t)$), and the lower envelope takes the lower edge of these
   lines, hence **piecewise linear**.
> **The envelope is not a staircase**: **what is a staircase is the optimal $t^*(w)$**, not the envelope.
> Conflating the two makes a reader take the vertical axis (minimum expected error) for a series of
> horizontal plateaus and misread even the meaning of the slope.
> Touchstone: **each segment of the envelope has a slope (= that segment's $\mathrm{ENR}$), while
> $t^*(w)$ is constant on each segment** — one is a line segment, the other a step.
2. **The deployed $t=7$ is a segment of the envelope under all three accountings**: upper
   $w\in[0.468,0.569]$ (letting through **1.76–2.14×** dearer) / point $w\in[0.708,0.861]$
   (**1.16–1.41×**) / lower $w\in[1.084,1.318]$ (**0.76–0.92×**).
3. **At $w=1$ the envelope carries $t=5$, not 7, but the gap is small.** Under the point estimate
   $E$ at $t=7$ is 29.57% against 28.64% at the optimum $t=5$, **a difference of only 0.92 points**
   (upper 4.40, lower just 0.10).

> **"The existing system did not choose wrongly" — but this sentence has two premises, and omitting
> them puts it in conflict with §5.5.2.**
> **Premise one**: negatives from **family D-full** (counting verbatim English terminology overlap as an
> error); **premise two**: $w$ lies in 1.16–1.41.
> Under both, switching to $t=5$ would pay a false-positive rate rising from 3.00% to 8.87%, i.e.
> letting-through rising from 54 to 160 items — **that is not "correcting an error", it is "buying a
> different cost".**
> **Change a premise and the conclusion changes**: under **D-pure** (not counting terminology overlap as
> an error), $t=6$ strictly dominates $t=7$, and then the phrase "20 items wasted" does hold (the fork in
> §5.5.2).
> **Whether $t=7$ is reasonable depends entirely on what $w$ the auditor assigns**: under the point
> estimate, **$t=7$ falls inside the optimal interval only when letting through is 1.16–1.41 times
> dearer than wrongful rejection** —
> **and that 1.16–1.41 is the interval for the point accounting, not shared across all three**: upper is
> **1.76–2.14×**, lower **0.76–0.92×** (table in §5.5.2). The three differ because their
> $\mathrm{ENR}$ differ by a factor — **citing this number requires naming the accounting.**
> **It must not be written "1.16× or more"**: that would include 2× and 10×, where the optimum has long
> since moved ($t=11$ at $w=0.1$).
> **This paper does not take a position on $w>1$ or $w<1$.** "Letting through a fabrication is dearer than
> reading a few more true sentences" is a **business judgement; this paper neither measures it nor cites
> cost data that would support it** — what this paper argues is something else: **that judgement has
> never been written down; it belongs in the system documentation, not hidden inside the constant `7`.**
> The substantive change brought by the human labels: **the bar for this judgement drops from "3× dearer"
> to "1.2× dearer" — that is, $t=7$ requires a far milder business assumption than before.** The direction
> of the conclusion is unchanged ($t=7$ is still not the default optimum), but **the strength of the
> reason has changed** — if a reviewer asks "is this not just a number picked off the shelf", the answer
> now is: "no; it is equivalent to a 1.2× assumption, and nobody wrote that assumption down."

### 5.5.2 Change the negative family and the "optimality condition" for $t=7$ changes

This is the hardest passage in the chapter, because without it §5.5 is an empty frame —
$\mathrm{EPR}(t)$ in $E(t;w)$ is determined entirely by the negative family, and changing it shifts every
number.

| Negative family | **point**: $w$ interval for $t=7$ → multiplier | upper | lower | Can it be optimal? |
|---|---|---|---|---|
| **D-full** ($n$=1,803) | $w\in[0.708,\ 0.861]$ → letting through **1.16–1.41×** | $[0.468,0.569]$ → 1.76–2.14× | $[1.084,1.318]$ → **0.76–0.92×** | **yes, under all three** |
| **B-main** ($n$=351) | $w\in[0.841,\ 1.318]$ → **0.76–1.19×** | $[0.556,0.881]$ → 1.14–1.80× | $[1.288,2.042]$ → 0.49–0.78× | **yes, under all three** |
| **D-pure** ($n$=721) | **none** | **none** | **none** | **no, under none — $t=7$ is never optimal in this family** (mechanism below) |

> **This comparison must be made within a family**: using family B's false positives as a reason in a
> D-pure comparison is switching families.
> Measured (`S1·_§5_人判新数.py`): **under D-pure, $\mathrm{EPR}$ is 0 at both $t=6$ and $t=7$**, while
> $\mathrm{ENR}(6)=24.00\%<\mathrm{ENR}(7)=26.57\%$ — **same false positives, lower wrongful rejection:
> $t=6$ is strictly better.**
> So the conclusion for this family is not "each wins somewhere" but "**$t=7$ wins no cell in this
> family**".

> Under the machine convention the multipliers for these two rows were 2.95–3.59 (D-full) and 1.91–3.02
> (B-main), **both void**; but "**$t=7$ is never optimal on D-pure**" **is unaffected by the labels** and
> holds under all three accountings.

**Why $t=7$ is a pure loss on D-pure** ($\mathrm{FP}$ does not move with the accounting; $\mathrm{FN}$ in
three accountings):

| Family | $t=6$ (FN / FP) | $t=7$ (FN / FP) | $\Delta$ |
|---|---|---|---|
| D-full | 24.00% / 6.60% (upper 36.26% / lower 15.63%) | 26.57% / 3.00% (upper 40.14% / lower 17.30%) | $\Delta$FN **+2.57** (upper +3.88 / lower +1.67) / $\Delta$FP −3.61 |
| **D-pure** | same / **0.00%** | same / **0.00%** | $\Delta$FN **+2.57** (upper +3.88 / lower +1.67) / $\Delta$FP **±0.00** |
| B-main | same / 5.98% | same / 2.56% | $\Delta$FN same / $\Delta$FP −3.42 |

The maximum LCS in D-pure is 5, so false positives at $t\ge6$ are **identically 0**. $t=6$ and $t=7$ are
**identical** on false positives; the entire difference is in wrongful rejections.

> **"20 more judged unsourced" is not "20 more wrongly rejected"** — keep them apart:
> · **machine verdict**: $t=6$ judges 187 unsourced, $t=7$ judges 207, **20 more**;
> · **actual wrongful rejections (proportional scaling, base 308)**: upper $36.26\%\to40.14\%$
> (**+3.88 points, equivalent to 11.95 items**), point $24.00\%\to26.57\%$ (**+2.57 points, 7.92 items**),
> lower $15.63\%\to17.30\%$ (**+1.67 points, 5.14 items**).
> Most of those 20 are **genuinely unsourced sentences** (judged correctly); only a small part falls on
> wrongful rejection.

> **So "is $t=7$ good" is not a question a formula can answer; it is a semantic judgement:**
> **do you accept that "a match on dates / identifiers / paths" counts as letting through?**
> - Yes → use D-full, that cell gives $t$ its 3.61 points of false positives, and $t=7$ has its reasons.
> - No → use D-pure, where $t=6$ is **strictly better** with **identical false positives (both 0)** and
>   lower wrongful rejection ($t=6$'s 24.00% is a **scaled extrapolation**, $t=7$'s 26.57% is measured);
>   the deployed threshold then judges **20 more items unsourced**, of which the true wrongful
>   rejections under point accounting amount to **7.92 items** (not 20 — see "more judged ≠ more wrongly
>   rejected" above).
> **But "semantic judgement" is not "independent of data"**: it has a second layer of dependence — whether
> those 84 suspicious sentences can actually be found in their sources requires human judging (§5.5.4).
> Both layers must be stated for the answer to be complete.

> This reversal must appear in the same chapter as §5.4. Read alone, §5.5 suggests "the cost framework
> gives a definite answer"; read together, the framework gives **a set of conditional statements**.
> And this is precisely what a reviewer will ask: "where does your $w$ come from?" This paper's answer:
> **$w$ cannot be recovered; it can only be supplied by the auditor's own cost accounting. What this
> paper gives is the map from a given $w$ to the $t$ that should be chosen, not $w$ itself.**

**So how does one actually compute $w$?** This paper does not estimate $w$ (that needs the auditor's
internal data), but for the chain to be walkable by others the steps must at least be given. **The three
steps below are an operational template, not an empirical result of this paper**; they are written so
that §5.5 is not half a chain:

1. **Convert each error type into a common unit.** Letting through a fabrication → expected loss from
   "cited / retracted / rework"; wrongfully rejecting a genuine sentence → labour cost of "one human
   review". **Both must be in the same unit** (money or hours), or they cannot be divided.
2. **Convert to a per-item basis using this paper's rates.** Both $\mathrm{ENR}$ and $\mathrm{EPR}$ here
   are per-item rates, so the costs must be per item too; a per-audit total cannot simply be divided.
3. **Write $w=C_{\text{wrongful rejection}}/C_{\text{letting through}}$, then look up the lower envelope
   in Figure 2.** Whichever segment it lands on, that segment's $t$ is the auditor's optimal threshold —
   this step is the map this paper provides.

> **The landing in step 3 must report three numbers together**: $w$, the segment's $t$, and **that
> segment's negative family and $\mathrm{ENR}$ bounds**. Omit one and the answer is not unique (§5.5.2
> and §5.5.4 each change it once).

---

---

### 5.5.3 Three thresholds that are almost never optimal (family D-full only)

With $t\in\{2,\dots,13\}$ and **family D-full as negatives**, $t=3$ and $t=6$ are **never strictly
optimal** under any cost ratio; $t=12$ is **tied** with $t=11$ and $t=13$ at the single point $w=0.054$
(**a tie is still optimal, only not unique**).

> **The accurate classification**: $t=3$ and $t=6$ lie **above** the convex hull (never optimal);
> $t=12$ lies **on the hull boundary** (tied at one point). **The three cannot be merged into one
> sentence, "never optimal under any cost ratio".**

> **And the reasons differ**:
> $t=3$ and $t=6$ fall **above** the lower convex hull, with positive gaps: 0.20 points and 1.39 points;
> **$t=12$ falls on the boundary** — it is **collinear** with $t=11$ and $t=13$, tying only at $w=0.054$.

> **Wording: one must not write "dominated".** Pareto dominance requires a point that is no worse on
> **both** criteria; here $\mathrm{FN}(t)$ and $\mathrm{FP}(t)$ are each monotone (larger $t$ raises
> wrongful rejection and lowers letting-through), so **every adjacent pair trades one against the other
> and no point dominates another**.
> The correct statement is that these three $t$ **lie above the lower convex hull in the
> $(\mathrm{FN},\mathrm{FP})$ plane**, hence are never strictly optimal for the linear objective
> $w\cdot\mathrm{FN}+\mathrm{FP}$. The gaps are as follows (identical across the three accountings,
> measured):

| $t$ | smallest gap (relative to the optimum at that $w$) | occurs at | why |
|---:|---:|---|---|
| 3 | **0.20 points** | $w\approx5.2$ | almost on the hull: the line joining $t=2$ and $t=4$ differs from the measurement by **0.196 points** (unrounded data) |
| 6 | **1.39 points** | $w\approx0.86$ | the line joining $t=5$ and $t=7$ differs by **1.388 points** (unrounded) |
| 12 | **0 (tied at one point only)** | $w=0.054$ | $t=11/12/13$ are **collinear**: letting-through halves each step (0.1109→0.0555→0), wrongful rejection rises 1.03 points each step |

> **The three must be read differently, not merged into one sentence**:
> · $t=3$ and $t=6$ **are optimal under no $w$**, but $t=3$ misses by only **0.20 points** —
> **"not optimal" does not mean "far off"**;
> · $t=12$ is of a different kind: it is **collinear** with $t=11$ and $t=13$ and ties only at
> $w=0.054$, **so mathematically there is no difference between choosing 11, 12 or 13** — the difference
> lies entirely in the citation policy (§5.6).

> **The conclusion of this section is independent of the false-negative accounting** (it still holds
> once the human labels are in), **but which conclusion must be named precisely**: the three accountings merely
> **multiply $\mathrm{FN}$ by a positive number** (0.597 / 0.395 / 0.257), **so the set "which $t$ never
> appear in the optimal set" is unchanged** — **and this section is about exactly that set** ($\{3,6,12\}$),
> which is indeed accounting-invariant.
> **One may not slide into "the optimal solution of the linear objective is unchanged"** — that is a
> different sentence, **and it is false**: multiplying $\mathrm{FN}$ by $f$ is equivalent to replacing $w$
> with $wf$, so at the same $w$ the **optimal point moves across accountings** (the $w=1$ row of Table 5.2:
> upper $t=5$ / point $t=5$ / lower $t=8$).
> **One may not write "the optimal solution is unchanged"** (that would conflict with §5.5.4's "the lower
> the accounting, the larger the optimal $t$"): **what is invariant is hull membership; what changes is
> the optimum at a given $w$.**
> And §5.5.2's "on D-pure $t=6$ becomes optimal" is not a contradiction — **change the family and the
> shape of the hull changes**.

> **Change the family and it differs** (point accounting, same across the three): **the optimal set on
> family B-main is $\{2,4,6,7,8,10,11\}$** — **$t=6$ appears in this family** (it does not on D-full);
> what is never optimal is $\{3,5,9,12,13\}$ (against $\{3,6,12\}$ on D-full).
> **The two lists differ, and $t=6$ is a dead cell in one family and a member of the optimal set in the
> other** — the hardest evidence that "change the family and the answer changes".
> So **any "this threshold is a dead cell" judgement must name the negative family**.

> The use of this: if someone proposes "a small adjustment to the threshold", a cell **above that
> family's hull** (never optimal) is a pure loss. The test is the `分段()` routine in `_代价三族.py`.
> **Note: "never optimal", not "dominated"** — the latter requires a single point no worse on both, which
> does not exist here.

---

### 5.5.4 With the human labels in, $t=7$ **has** an optimal interval under all three accountings

§5.5.1–§5.5.3 now use the **point** accounting throughout. This section asks: **do the conclusions of
§5.5 survive under the other two?**

**Optimal $t$ under the three accountings of $\mathrm{ENR}$** (family D-full as negatives;
`S1·_§5_人判新数.py`, 12 self-tests):

| $w$ | upper | point | lower | consistent across three? |
|---|---:|---:|---:|---|
| 0.1 (letting through 10× worse) | $t=10$ † | $t=11$ † | $t=11$ † | no (all in the extrapolation region) |
| 0.3 | $t=8$ | $t=10$ † | $t=10$ † | no |
| **1.0 (equal)** | $t=5$, $E$=38.74% | **$t=5$**, $E$=28.64% | $t=8$, $E$=20.19% | **no** |
| 3.5 (wrongful rejection 3.5× worse) | $t=2$, $E$=96.65% | $t=4$, $E$=70.32% | $t=4$, $E$=51.53% | no |

| Accounting | optimal $w$ interval for $t=7$ | multiplier |
|---|---|---|
| upper (40.14%) | $w\in[0.468,\ 0.569]$ | letting through **1.76–2.14×** |
| **point (26.57%)** | $w\in[0.708,\ 0.861]$ | letting through **1.16–1.41×** |
| lower (17.30%) | $w\in[1.084,\ 1.318]$ | letting through **0.76–0.92×** |

> **The earlier claim — "take the lower bound for the false-negative rate and $t=7$ loses its optimal
> interval outright" — does not survive the labels.** Under the lower accounting $t=7$ has the interval
> $[1.084,1.318]$; it merely requires that letting something through be no more costly than wrongly rejecting it.
> **The old lower bound of 39.94% was derived by counting all 84 suspicious items as correctly judged,
> and the labels show most of those 84 can indeed be found in their sources — but the same labels also
> overturned 5 of the 18 L1 items**, and the two effects offset, leaving the ground truth near 26.57%,
> not 39.94%.
> **So the old "upper versus lower" comparison table ($t=4$ vs $t=5$, interval or not) is void in its
> entirety**; anyone encountering 39.94% or 67.21% should replace them with the three accountings here.

**Why do the optima differ so much across accountings?** Because when $w$ is small $E$ is dominated by
$\mathrm{EPR}$ and when $w$ is large by $\mathrm{ENR}$, while the difference between accountings is a
**proportional rescaling of the whole curve** ($f$ = 0.597 / 0.395 / 0.257) — rescaling does not change
the shape, only "how heavy the $\mathrm{ENR}$ term is", equivalent to multiplying $w$ by $f$. So **the
lower the accounting, the larger the optimal $t$ overall**.
> **But one may not say "consistent with a rightward shift down the table"**: the $w=1.0$ row is
> **upper 5 / point 5 / lower 8** — **upper and point tie; that is not a shift.** The accurate statement
> is "**the overall trend shifts right, but at $w=1$ upper and point coincide**".

> **Two boundaries must be stated together**:
> ① **High end**: cells at $t\ge9$ are scaled extrapolation (all three cells of the $w=0.1$ row, with
> optima 10 / 11 / 11, and the point and lower cells of the $w=0.3$ row, both 10). The labels cover only
> $t=7$; **whether the sentence composition at high $t$ changes was not measured**.
> ② **Low end**: $t\le3$ likewise has no human data — at $t=2$ the machine judges only 9 items unsourced,
> and all 100 labelled items fall inside the 207 pool at $t=7$, **giving no information about which layer
> flips first**. So in the $w=3.5$ row: **upper's $t=2$ is extrapolation only**; point and lower's $t=4$
> lie inside the $t=4\dots8$ this paper is answerable for.
> **This paper is answerable only for the values in $t=4\dots8$; both ends are directional reference.**
> This is a statement of scope, not a disclaimer.
> **But §5.5.6 does not cover those two ends**: it tests **one** assumption ("global scaling → per-form
> scaling") and shows only that "with a weaker assumption the direction is unchanged"; **it says nothing
> about the cells at $t\ge9$ or $t\le3$ that have no human data.**

### 5.5.5 The nine-cell summary: $t=7$ survives in 6 of 9 cells

§5.5.4 varied only the false-negative rate ($\mathrm{ENR}$, three accountings); §5.7 varies only the
false-positive rate ($\mathrm{EPR}$, three accountings). The two quantities together give **3×3 = 9
cells**, and reporting only some of them would read as cherry-picking. All computed
(`S1·_§5_人判新数.py`, 12 self-tests):

| $\mathrm{ENR}$ | $\mathrm{EPR}$ | optimal $w$ interval for $t=7$ | multiplier | equal-weight optimal $t$ |
|---|---|---|---|---|
| **upper** 40.14% | all 3.00% | $[0.468,\ 0.569]$ | letting through **1.76–2.14×** | 5 |
| **upper** 40.14% | format + Chinese 0.94% | $[0.126,\ 0.150]$ | letting through **6.68–7.94×** | 4 |
| **upper** 40.14% | format only 0.61% | **none (never optimal)** | — | 4 |
| **point** 26.57% | all 3.00% | $[0.708,\ 0.861]$ | letting through **1.16–1.41×** | 5 |
| **point** 26.57% | format + Chinese 0.94% | $[0.191,\ 0.226]$ | letting through **4.42–5.25×** | 5 |
| **point** 26.57% | format only 0.61% | **none (never optimal)** | — | 4 |
| **lower** 17.30% | all 3.00% | $[1.084,\ 1.318]$ | letting through **0.76–0.92×** | 8 |
| **lower** 17.30% | format + Chinese 0.94% | $[0.292,\ 0.347]$ | letting through **2.88–3.43×** | 5 |
| **lower** 17.30% | format only 0.61% | **none (never optimal)** | — | 4 |

**$t=7$ survives in 6 of the 9 cells** (2 of 6 under the machine convention). The location of the failures
is very orderly: **all three "format only" cells drop out; the other six all remain.**

> **This is not "the conclusion is unstable"; it is "the shape of the conclusion"**:
> the reasonableness of $t=7$ depends **mainly** on **one** condition — **whether the false-positive
> accounting counts "Chinese terminology overlap" as letting through**.
> - Counted (all, or format + Chinese) → $t=7$ is in, requiring letting through **0.76 to 7.94×** dearer;
> - Not counted (format only) → $t=7$ disappears outright, and the equal-weight optimum drops to $t=4$.
> The false-negative accounting no longer decides whether $t=7$ lives, only **how dear it requires
> letting-through to be** (0.76× under lower → 2.14× under upper).
>
> **But "depends on only one condition" says too much**: two other conditions remain —
> ① **the cost ratio $w$ must fall inside that row's interval** (outside it, $t=7$ is simply not optimal);
> ② **the negatives must be family D-full** (switching to D-pure removes the interval, §5.5.2).
> The accurate statement: **given family D-full and $w$ inside that cell's interval, whether $t=7$ stands
> depends only on the EPR accounting** — all three conditions are indispensable.
>
> **Does this amount to circular reasoning?** No. Once the nine cells are converted into "how many times
> dearer letting-through must be", **the upper bound of that multiplier rises monotonically along the EPR
> direction** (upper: 2.14 → 7.94 → disappears; point: 1.41 → 5.25 → disappears; lower: 0.92 → 3.43 →
> disappears), **each chain holding on its own and in the same direction**.
> **Do not reverse the direction**: these are upper bounds on $1/w$ ($2.14=1/0.468$, $7.94=1/0.126$), not
> interval lower ends. Seen as $w$ intervals, it is the **lower end that falls** ($0.468\to0.126$).
> **Each statement is correct in one direction; mixing them leaves the reader unable to match the table.**
> This is a testable monotonicity, not a cell picked after the fact.
>
> **It also explains why no "recommended threshold" is given**: recommending requires first choosing a
> cell. Choose "format only" and $t=7$ is gone; choose "all" and the cost ratio must fall between
> **0.76–2.14×** (1.16–1.41× under the point accounting). **Which cell to choose is the auditor's
> business, not this paper's.**

### 5.5.6 Robustness: replace "global scaling" with "per-form scaling" and the main conclusion holds (the lower accounting's optimum moves one step)

The full curves of §5.5.4 use a **strong** assumption:
$\mathrm{ENR}(t)=f\cdot\mathrm{ENR}_{\text{machine upper}}(t)$ — equivalent to "the wrongful-rejection
share is identical at every $t$". A **strictly weaker** assumption is: require only that the
within-stratum share $p_i$ does not change with $t$, while the strata's item shares change naturally:

$$\mathrm{ENR}(t)=\frac{1}{308}\sum_{i\in\{\text{definite},\text{suspicious},\text{clean}\}} C_i(t)\cdot p_i$$

$C_i(t)$ is the count with $\mathrm{LCS}<t$ in stratum $i$ (machine-computable), and $p_i$ is the
stratum-level result of the human labels on the $t=7$ pool. At $t=7$ the two are **necessarily exactly
equal** (same calibration); they differ only at other $t$ (`S1·_§5_人判新数.py` §7).

| Accounting | global-scaling optimal $t$ ($w=1$) | **per-stratum optimal $t$** | conclusion changed? |
|---|---:|---:|---|
| upper | 5 | **5** | no |
| point | 5 | **5** | no |
| lower | 8 | **7** | **yes: $t=7$ becomes optimal instead** |

The curves themselves are also close: at $t=8$, upper is 43.25% (global) against 43.57% (per-stratum), a
difference of 0.32 points; point is 28.63% against 29.11%, a difference of 0.48.

> **The value of this comparison is not that "the numbers agree" but that the strongest assumption was
> replaced by a weaker one and the conclusion did not move.**
> And under the lower accounting the optimum moves from 8 to 7 — a result of **directional validation**,
> **not evidence that "$t=7$ is more defensible"** (see the next line).
> **What still cannot be done**: the $p_i$ of per-stratum scaling rest on only 18 / 30 / 25 samples
> (particularly: 18 drawn in full for the middle stratum, and **30 and 25** for the other two), **so the
> error in $p_i$ itself already exceeds the difference between the two curves**. The table is therefore
> directional validation only, not "another point estimate".

---

---

## 5.6 $t=12$ can be read as the cost of abolishing human review

The decision rule is three-way:

```
LCS >= 12   sourced
LCS 7-11    to-human
LCS <= 6    unsourced     <- the deployed cutoff t = 7
```

At $t=12$ the decision set is $\{x:\mathrm{LCS}<12\}=\{\mathrm{LCS}\le11\}$ — exactly **the to-human band
abolished and folded entirely into unsourced**.

At $t=12$ the number judged unsourced is $269 = 207+62$. **The same count yields three different
false-negative rates under the three accountings:**

| Accounting | $t=7$ | $t=12$ (human review abolished)† | Difference |
|---|---:|---:|---:|
| upper (scaled extrapolation) | 40.14% | 52.17% | 12.03 points |
| **point** | **26.57%** | **34.53%** | **7.96 points** |
| lower | 17.30% | 22.48% | 5.18 points |

> † The $t=12$ false-negative rate is a **scaled extrapolation** (the labels pin only $t=7$) and, under
> this paper's citation policy, is not used as a conclusion — only to indicate the **order of magnitude**
> of "what abolishing human review costs". **Its count basis ($269=207+62$) is hard; the rate is soft.**

> **So "abolishing human review" is not an abstract threshold point; it reads as the cost of a concrete
> decision**: under the point accounting, wrongful rejections rise from 26.57% to 34.53% — **7.96 points
> more, equivalent to 24.5 items** ($0.0796\times308$).
> **Not 62 items**: 62 is the **pool size** of the L4 to-human band, and $62/308=20.13$ points is a
> different account (and belongs to a superseded convention).
> Treating it as this sentence's increment would make a reader think all 62 are sourced sentences
> rescued by human review — **the converted figure is only 24.5** ($0.0796\times308$ is a **proportional
> conversion**, not an item-by-item count).

> This paper does not recommend abolishing human review. The value of the sentence is that **it puts a
> price on the to-human band**.
> The current 26.57% is the result *with* 62 items handed to a human; 34.53% is the result with *none*
> handed over. **Those 7.96 points (24.5 items) are what human review buys back.**
> **The difference moves with the accounting**: upper 40.14%→52.17% (12.03), lower 17.30%→22.48%
> (5.18). **All three must be shown together.**

---

### 5.6.1 An incidental measurement: LCS is not a monotone proxy for a likelihood ratio

The Fellegi–Sunter structure of §3.1.1 requires ordering **by likelihood ratio**, whereas this paper
orders by $\mathrm{LCS}$. The two are equivalent only if $\Lambda(k)$ is monotone in $k$ — and it is not:

$$\Lambda(k)=\frac{\Pr(\mathrm{LCS}=k\mid\text{genuinely sourced})}{\Pr(\mathrm{LCS}=k\mid\text{unrelated})}$$

| $k$ | 0 | 1 | 2 | 3 | 4 | **5** | **6** | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| positives | 3 | 6 | 67 | 44 | 34 | 33 | **20** | 16 | 13 | 12 | 13 | 8 | 8 | 7 | 2 | 1 |
| negatives (D-full) | 63 | 108 | 803 | 533 | 136 | **41** | **65** | 26 | 13 | 9 | 4 | 1 | 1 | 0 | 0 | 0 |
| $\Lambda$ | 0.28 | 0.33 | 0.49 | 0.48 | 1.46 | **4.71** | **1.80** | 3.60 | 5.85 | 7.81 | 19.0 | 46.8 | 46.8 | ∞ | ∞ | ∞ |

**$\Lambda(5)=4.71$ but $\Lambda(6)=1.80$ — the evidence at $\mathrm{LCS}=6$ is only $1/2.62$ of that at
$\mathrm{LCS}=5$.**

> **The numerator and denominator of this table must be stated**:
> $\Pr(\mathrm{LCS}=k\mid\text{sourced})$ has **denominator all 308 positives**, **not the 287 that the 16
> cells of the table sum to** — **21** items have LCS $\ge16$ and fall outside those cells. On the
> negative side the 16 cells sum to exactly 1,803, with no gap.
> **Using 287 as the denominator yields $\Lambda(5)=5.06$, $\Lambda(6)=1.93$** — a wrong denominator, not
> a wrong conclusion. Every $\Lambda$ here is generated by `S1·_核Lambda表.py` and is recomputable cell by
> cell.

The cause is a **pile-up of negatives at $k=6$** (65 items, against only 41 at $k=5$): the mass of
"date / path / identifier" structural debris in family D-full happens to sit at length 6 and washes out
the discriminative power at $k=6$. **At $k\ge13$ the negatives are 0, so $\Lambda$ divides by zero (shown
as $\infty$); that is neither an inversion nor a value** —
> those three cells **have no reproducible value at all**; $\infty$ denotes division by zero only.
> At $k=10\dots12$ the negatives number 4 / 1 / 1, **so a single item can double $\Lambda$**, and no
> conclusion is drawn there either.

**Three layers of significance, ordered by firmness:**

1. **Firm**: $\mathrm{LCS}$ and $\Lambda$ are not equivalent — so this paper's three-way cut-offs
   ($lo=6$, $hi=12$) are **not** the cut-offs that minimise clerical review volume for a given error rate
   in the Fellegi–Sunter sense. This is the increment this paper can claim: **ordering by $\mathrm{LCS}$
   does not inherit Fellegi–Sunter's optimality guarantee.**
   **One step further does not hold**: not inheriting a guarantee is not the same as having already sent
   too much. **By how much, this paper has not measured** — measuring it would require also running
   the "ordered by $\Lambda$" rule, which was not done.
2. **Partly firm**: the inversion at $k=5\to6$ **holds at the point estimate**, but on 20 positives and 65
   negatives the error is large. **It cannot be asserted as real.**
3. **Not firm (must not be written)**: nothing follows about moving the cut-off from 6 to 5 or 7. The
   $\Lambda$ at $k=6$ is depressed by the **construction** of the negatives; change the family (e.g. to
   D-pure) and $\Lambda(6)$ becomes 95.8 instead — **the inversion itself is family-dependent, not a
   universal property.**

> The use of this section: it **cannot** be used to change the threshold, only to answer "why use LCS
> rather than a likelihood ratio" — the answer being **that LCS is interpretable, recomputable, and does
> not depend on an estimate of the negative family**; the price is that it loses some discriminative power
> in the middle range.
> **§8 must record this limitation**: this paper's three-way structure has Fellegi–Sunter's skeleton but
> **does not inherit its optimality guarantee**.

---

## 5.7 Of the 3.00% false-positive rate at the $t\ge7$ end, only 0.61 points are definite errors (upper bound 0.94)

§5.2–§5.5 use the 3.00% of family D-full throughout, which is the reading **least favourable to the
rule** — it counts "verbatim English terminology overlap" as letting through. Broken down item by item:

| Type | Criterion | n | Share of 1,803 | Counts as an error? |
|---|---|---:|---:|---|
| content · terminology | alphabetic LCS ≥ 3 | **37** | 2.05% | **no** — proper names verbatim in the source |
| format · digits and symbols | Chinese and alphabetic LCS both < 3 | **11** | 0.61% | **yes** — the scorer cannot tell format from content |
| content · Chinese | Chinese LCS ≥ 3 | **6** | 0.33% | **awaiting judgement** (P6) |

> **Why the format criterion is "both Chinese and alphabetic LCS < 3", and not "zero Chinese
> characters".** A split on "is the Chinese character count zero" would count English letters as format,
> because they also contain no Chinese characters — and **English letters are content** (proper names,
> terminology), not format. **Writing the paper on that label would invite one reviewer sentence — "you
> counted your own terminology overlap as a format defect" — and the claim collapses.** The three LCS
> values must therefore be computed separately (Chinese only / alphabetic only / as-is) and the split made
> three ways; the three counts then sum to 54 ($37+11+6$).

> **External corroboration**: this is not a defect peculiar to this paper.
> The error analysis of AttrScore (Yue et al. 2023, Findings of EMNLP) reports that among cases where the
> model mistook "insufficient information" for "attributable", **"insensitivity to fine-grained
> information" accounts for 46.7% (numbers, dates, times)**.
> — **the numbers were right, the wording changed, and neither the model nor literal overlap can see it.**
> **One may not say "this band is the same failure type as theirs"**: 46.7% establishes only that
> **AttrScore has this class of error too**; "same type" would require a **mechanism analogy** (both
> criteria rely on literal overlap, both fail on format changes) — **that step was neither written nor
> verified here.**
> What can be said: **the failure mode "insensitive to fine-grained information" has been separately
> listed in someone else's error analysis**, and this band belongs to the same **phenomenon class**;
> whether the mechanism is the same **is not asserted**.

> **A second explanation for the three-way difference, more concrete than "business judgement"**: it is
> not only a choice about "does terminology overlap count as letting through", but also that **the LCS
> signal-to-noise ratio differs across sources and languages** — English sources have high verbatim
> terminology overlap, Chinese sources naturally low.
> **So applying a uniform $t=7$ is equivalent to "being systematically stricter on low-precision
> sources"** — **which suggests the threshold might need to be stratified by source.**
> **Only "suggests", not "ought to"**: "different signal-to-noise" supports only "a systematic bias may
> exist", **not "stratification is the right thing to do"** — the latter needs a verified gain from
> stratification, and **no stratification experiment was run** (as the next sentence concedes).
> (This comes from the "signal precision" branch of the statistical-discrimination literature; footnote
> level only — no stratification was done, see §8.5.)

**The real cost lies between 0.61 and 0.94 percentage points, not at a point value** — the heading says
so, and **0.94 must not be written into a heading as a point value**.

- **0.61 is the definite part** — the format items (11): the scorer genuinely cannot tell format from
  content; that is a hard error.
- **0.33 awaits judgement** — the Chinese items (6): whether they can be found in their sources requires
  P6. **If all 6 still count as false positives, the figure is 0.94%; if all are judged "indeed findable"
  (not false positives), it returns to 0.61%.**

> The two numbers must appear together: **3.00% (least favourable) and 0.94% (upper bound)**. Reporting
> only 0.94% invites the charge of picking the flattering number; reporting only 3.00% turns terminology
> overlap into a defect, and one sentence — "you counted English proper names as fabrication" — dismantles
> the whole section.
> **0.94 must not be written as a point value**: the three levels must be kept apart — 3.00 (least
> favourable) / 0.94 (upper bound) / 0.61 (definite). **Adding a pending item to a definite one is a hard
> error.**

**Sensitivity: change the EPR accounting and the optimal $t$ moves — but the three accountings differ and
must be reported separately.** Measured (equal weight $w=1$):
**upper** 5 → 4 → 4 (moves in both); **point** 5 → 4 → **5** (unchanged under "format + Chinese");
**lower** 8 → 4 → 5 (moves most).
> **One may not say "it moves, and by a lot"**: **across the three accountings five cells move and one does
> not** (point, under "format + Chinese"), and the size of the move ranges from 1 to 4 steps.

That table decomposes only the $t=7$ cell. To judge whether the 3.00% used in the main analysis biases the
conclusions, all three accountings must be computed **along the whole $t$ axis**
(`S1·_EPR三档.py`, 10 self-tests, all 1,803 items in 12 seconds):

| $t$ | as-is (all) | format only | format + Chinese | difference (points) |
|---:|---:|---:|---:|---:|
| 2 | 90.52% | 55.46% | 76.32% | −35.05 |
| 3 | 45.98% | 12.87% | 33.00% | −33.11 |
| 4 | 16.42% | 3.49% | 7.49% | −12.92 |
| 5 | 8.87% | 0.94% | 2.50% | −7.93 |
| 6 | 6.60% | 0.83% | 2.00% | −5.77 |
| **7** | **3.00%** | **0.61%** | **0.94%** | **−2.38** |
| 8 | 1.55% | 0.44% | 0.55% | −1.11 |
| 10 | 0.33% | 0.11% | 0.17% | −0.22 |
| 12 | 0.06% | 0.00% | 0.00% | −0.06 |

**The difference is not a constant**: 35.05 points at $t=2$ but only 2.38 at $t=7$. So "a uniform downward
shift" is wrong; **the low-$t$ end is revised down far more**.

**Consequences — the real conclusion of this section:**

| EPR accounting | optimal $w$ interval for $t=7$ (point-accounting ENR) | equal-weight optimal $t$ |
|---|---|---|
| as-is (3.00%) | $w\in[0.708,\ 0.861]$ (letting through **1.16–1.41×**) | $t=5$ |
| **format + Chinese** (0.94%) | $w\in[0.191,\ 0.226]$ (**4.42–5.25×**) | $t=5$ |
| **format only** (0.61%) | **none — $t=7$ never optimal** | $t=4$ |

Across 8 cost ratios (0.1 / 0.2 / 0.3 / 0.5 / 0.7 / 1 / 2 / 3.5), **the number of times the three EPR
accountings give the same optimal $t$ is 0** — not once do all three agree. (For the upper and lower
accountings of ENR, only 1 cost ratio each achieves agreement.)
> **These two groups must not be mixed**: the first half counts the three accountings of **how
> false positives are computed**; the upper and lower in the second half are accountings of **how false
> negatives are computed** — **two dimensions, two denominators; neither addable nor comparable.**

> **So "t=7 implies letting-through is 1.2× dearer" depends on the accounting far more deeply than it
> appears**:
> remove terminology overlap from false positives (which is defensible — a proper name verbatim in the
> source is not a fabrication) and $t=7$ no longer means "1.2× dearer" but requires letting through to be
> **4.42–5.25×** dearer; judge the Chinese overlap as "not an error" as well and $t=7$ **occupies no cell
> at all**.
>
> **The more the accounting is corrected in the direction favourable to the rule, the less $t=7$ stands
> up** — the same coin as the reversal of the A3 argument in §3.1: a threshold that cannot be saved even
> under the most permissive accounting is genuinely without foundation.
>
> **An easy misreading**: supposing "EPR shifts down overall → the optimal $t$ shifts left".
> What actually happens is not "the optimal $t$ shifts left" but "$t=7$'s optimal interval moves sharply
> toward 'letting through much dearer', narrows, and disappears". The difference: the former says "a
> smaller $t$ should be chosen"; the latter says "$t=7$ requires a more extreme cost assumption".
> **Neither can be inferred; both must be computed.** Recompute with `S1·_EPR三档.py`.

**Why the main analysis uses 3.00% rather than the seemingly more "accurate" 0.61% — a choice with a
substantive reason, not a conservative posture.**

If the 37 terminology-overlap items are judged "not false positives", **there are two ways to compute,
differing by 0.015 points (format-only column) and 0.023 points (format + Chinese column), and which one
is used must be stated**:
> ($11/1760-11/1803=0.0149$ points; $17/1760-17/1803=0.0230$ points — **one difference per column; 0.015
> alone must not be reported**.)

| Method | Numerator | Denominator | $\mathrm{EPR}(7)$ (format only / format + Chinese) |
|---|---|---|---:|
| **(a) denominator unchanged, count one class within it** ← used above | 11 / 17 | **1,803** | **0.61% / 0.94%** |
| (b) actually remove the 37 from the pool | 11 / 17 | **1,760** | 0.625% / 0.966% |

> **The two denominators must not be mixed**: the main analysis uses **(a)**, keeping the full denominator
> and counting only the items that belong to the format class — which neither changes the pool nor lets
> the scorer under test take part in selecting its own test set.

**But that removal cannot be performed, because it would be self-serving:**

> The criterion for removing these items is "has a 3-gram overlap with the source".
> The scorer under test ($\mathrm{LCS}$) also judges "has an overlap with the source".
> **Using the same family of criteria to clear the errors of the scorer under test is letting the
> defendant be the judge** — the false-positive rate after removal would necessarily improve, and the part
> that improves **is precisely the point in dispute**.

> **But the same family of criteria was used earlier in this section**: the composition table above splits
> the 54 with "Chinese / alphabetic $\mathrm{LCS}\ge3$". **A reader will ask: why is the same operation
> legitimate in one place and not the other?**
> The difference is **what the classification is used for**:
> · **there it is description** — it merely decomposes 3.00% into three parts, **leaving the denominator
> untouched and the main numbers unchanged**; the reader sees "how much of it is format items";
> · **here it would be adjudication** — it removes items from the denominator and **directly changes the
> main result**, and that change lands exactly on the disputed point.
> **The same criterion, used to describe, is transparent; used to decide, it is self-serving.** This
> section rejects the latter.

So this chapter's choice is: **the main analysis removes nothing** ($\mathrm{EPR}=3.00\%$, full
convention, all 1,803 items in), and the narrow "denominator unchanged, count format items only"
convention (0.61%) is offered as a **sensitivity upper bound** in the table, so the reader can see where
that operation would push the conclusion. The price of this choice must be stated: **3.00% does overstate the false-positive rate** (the
terminology is verbatim in the source, so those 37 items probably are sourced). This paper accepts the
overstatement, because **the direction of the overstatement is against the rule being audited**, and the
conclusion is that the rule is inadequate — **reaching that conclusion from a number unfavourable to the
rule is what makes it independent of the accounting choice.** The same logic as the reversal of the A3
argument in §3.1.

---

## 5.8 A change of coordinates: $t$ is a significance level $\alpha$

> This section changes no existing result; it only gives $t$ a different coordinate system.

### 5.8.1 Identity: $\mathrm{EPR}(t)$ is $\alpha$

The false-positive rate here is defined as

$$\mathrm{EPR}(t)=\Pr\big(\mathrm{LCS}(x,s)\ge t\ \big|\ \text{the two sentences are unrelated}\big)$$

In the language of statistical testing the right-hand side is the **type-I error rate under the null**:
taking "$\mathrm{LCS}\ge t$" as the criterion for rejecting "unrelated", $\mathrm{EPR}(t)$ is the
probability of a type-I error.

> **But in the sentence "equals $\alpha$", three numbers must be kept apart and not treated as one**:
> · **target level 3.00%** — the standard referenced when $t=7$ was chosen;
> · **measured value 2.995% (54/1,803)** — computed from the data, differing from 3.00% only by rounding;
> · **finite-sample upper bound 3.05%** — given by **conformal** reasoning ($2.995\%+1/1804$, §5.8.3).
>   **3.05% is not a DKW bound**: DKW gives a **band on the difference between two proportions**
>   ($\pm3.83$ points, for comparison), **not an upper bound on a single proportion**. **Two tools, two
>   jobs, not interchangeable**: DKW → are two numbers separable; conformal → an upper bound on one.
> **One may not call it "the nominal significance level"**: "nominal" usually means a target fixed in
> advance that a discrete score may fail to attain, whereas 2.995% here is a **measured rejection rate**;
> calling it nominal suggests it was set rather than computed.

**So "choose a threshold $t$" and "choose a significance level $\alpha$" are linked on this rule by a
monotone (non-increasing) correspondence $\alpha=\mathrm{EPR}(t)$.**
> **But one may not write "one-to-one"**: $t\mapsto\mathrm{EPR}(t)$ is **monotone non-increasing, not
> injective** — in this section's own D-pure table, $t=6$ and $t=7$ both give $\alpha=0$, a ready
> counterexample. **A monotone correspondence suffices (a given $t$ determines $\alpha$ uniquely); the
> converse does not.**

> **"Correspondence" is not "the same thing"**: $\alpha$ is the level you **want** (set in advance, not
> necessarily attainable with a discrete score); $\mathrm{EPR}(t)$ is what is **actually attained**
> (computed). They coincide numerically **only at the chosen $t$** — calling them "the same thing"
> conflates target with measurement.

| $t$ | 5 | 6 | **7** | 8 | 9 | 10 | 12 |
|---|---:|---:|---:|---:|---:|---:|---:|
| $\alpha$ (D-full, $n$=1,803) | 8.87% | 6.60% | **3.00%** | 1.55% | 0.83% | 0.33% | 0.06% |
| $\alpha$ (D-pure, $n$=721) | 0.42% | **0** | **0** | 0 | 0 | 0 | 0 |

- The deployed $t=7$ corresponds to **$\alpha=3.00\%$** (D-full) — the comparable scale for "how permissive
  is this threshold".
- Against a clean natural-sentence null (D-pure), **$\alpha$ at $t=7$ is 0 outright**: among 721 items the
  **longest common substring is at most 5**, so none crosses 7.

### 5.8.2 A new problem: this paper makes 308 comparisons, not 1

$\alpha=3\%$ is the per-comparison probability of letting something through. But the actual task is
**judging 308 sentences one at a time**, and what matters is how many of those 308 are let through.

**Two standards give two entirely different answers — and must be reported separately:**

| Standard | What it controls | Requirement | Does the deployed $t=7$ pass? |
|---|---|---|---|
| **FWER** (Bonferroni) | probability of **at least one error** $\le5\%$ ($\Pr(V\ge1)\le0.05$) | each $\alpha\le0.05/308=1.62\times10^{-4}$ | **not certifiable with this data** — resolution only $5.5\times10^{-4}$ |
| **FDR** (Benjamini–Hochberg style, $q=5\%$) | "among those declared sourced, the **expected proportion in error** ≤ 5%" ($\mathbb E[V/R]$, with the convention for $R=0$) | $\alpha(t)\le q\cdot k(t)/308$, $k(t)$ = items declared sourced | **no** — requires $t\ge9$ |

Item-by-item comparison ($k(t)=308-$ judged unsourced):

| $t$ | 6 | **7** | 8 | **9** | 10 |
|---|---:|---:|---:|---:|---:|
| $\alpha=\mathrm{EPR}(t)$ | 6.60% | **3.00%** | 1.55% | **0.83%** | 0.33% |
| FDR 5% allowed ceiling $q\,k(t)/308$ | 1.96% | **1.64%** | 1.38% | **1.17%** | 0.97% |
| passes? | ✗ | **✗** | ✗ | **✓** | ✓ |

> **This is the most honest cell in the whole chapter**:
> - controlling FWER at 5%, **the data are simply insufficient** — to certify a magnitude of
>   $1.62\times10^{-4}$, 1,803 negatives resolve only $5.5\times10^{-4}$;
> - under "a controllable error proportion" (FDR 5%), **the answer is $t\ge9$, and the deployed $t=7$ does
>   not suffice.**
>
> **But this step verifies only one threshold inequality; it does not mean "FDR is controlled at 5%"**:
> the table computes $\alpha(t)\le q\,k(t)/308$ — a **screening condition under a common threshold**. The
> BH procedure's FDR guarantee also requires **valid $p$-values** (LCS was not converted to $p$-values
> here; only $\alpha(t)$ is used as an approximation) and assumptions about the dependence structure.
> So what can be written is "**under the approximation $\alpha(t)\le q\,k(t)/308$, $t=7$ does not
> suffice**", **not "$t=7$'s FDR exceeds 5%"** — the latter would require establishing $p$-values first.
> **$t=7$ fails under both standards, but fails differently**: one is "not enough data", the other
> "threshold too permissive".
>
> **Citation boundary**: the $\alpha$ here is **measured** (counted directly on the 1,803 of D-full), not
> extrapolated; but the **cost** ($\mathrm{ENR}$) on the $t\ge9$ rows is extrapolated — so "$t\ge9$ is
> what passes FDR" holds, while "what the wrongful-rejection rate at $t=9$ is" is not concluded.

> **One necessary clarification**: a screening task should not use FWER in the first place — driving the
> probability of "at least one error" to 5% is necessarily very conservative.
> **How large the price is can be read off a cell**: to reach $\mathrm{EPR}\le1.62\times10^{-4}$, $t$ must
> be pushed beyond 13, and the wrongful-rejection rate at $t=13$ is **35.56%** (Table 5.1, extrapolated).
> This paper's preferred reading is FDR: **of 20 declared sourced, one may be wrong; what is not
> acceptable is "nobody knows which one".**
> But that is **this paper's orientation**, not a standard the auditor has agreed to — they gave floors of
> the form "FP ≤ 3%" (§5.3) and **never gave an error-control convention**. This gap goes into §8.

### 5.8.3 Two free precision upgrades (no new data): simultaneous bands and conformal thresholds

**(i) The DKW inequality → a simultaneous confidence band on the whole $\mathrm{EPR}$ curve**

$$\Pr\Big(\sup_{t}\big|\hat F_{\text{neg}}(t)-F_{\text{neg}}(t)\big|\le\varepsilon\Big)\ge1-\delta,\qquad
\varepsilon=\sqrt{\frac{\ln(2/\delta)}{2n}}$$

| Negative family | $n$ | $\delta=0.05$ | $\delta=0.01$ |
|---|---:|---:|---:|
| D-full | 1,803 | **±3.20 points** | **±3.83 points** |
| D-pure | 721 | ±5.06 points | ±6.06 points |

> **This band immediately voids one phrasing in this chapter**: §5.4 says "at $t=7$, D-full 3.00% /
> B-main 2.56%", **a difference of only 0.44 points, far below the sum of two half-widths
> ($2\times3.83=7.66$ points)** — **so it cannot be taken as a difference.**
> **But the converse does not hold either**: overlapping bands only mean that **at this paper's precision
> one cannot claim they are separable**, **not that "they are the same number"**. Proving equality needs
> an **equivalence test** (e.g. TOST with a pre-specified equivalence margin), which was not done.
> Everywhere §5.4 and §5.5.2 read these two side by side as "different" must carry "within the band, not
> interpreted as a difference".
> Conversely, at $t=4$, 16.42% against 2.50% differs by **13.92 points, greater than the sum of two
> half-widths, 7.66** — so "**these two false-positive rates are statistically separable**" holds.
> **The criterion used must be named**: here it is "the two **single-sample** $\delta=0.01$ bands do not
> overlap", a **conservative** criterion — non-overlap suffices to establish a two-sample difference;
> overlap does not establish no difference. **±3.83 is a single-sample half-width, not a half-width for
> the difference** — comparing a single half-width to a difference loosens the criterion by half.
> **But it does not reach "nearly 10×"**: $16.42/2.50=6.57$. **"Nearly 10×" refers to a different pair,
> $24.79\%$ against $2.50\%$ ($=9.92×$)** — that pair must be used, not $16.42\%$.

**(ii) Conformal quantiles → upgrading "empirical EPR" to a construction with finite-sample coverage**

Taking $t$ as the $\lceil(n+1)(1-\alpha)\rceil$-th order statistic of the negatives, for a **new** unrelated
sentence,
$$\Pr(\mathrm{LCS}\ge t)\le\alpha+\frac{1}{n+1}$$

| Negative family | $\alpha=0.05$ | $\alpha=0.03$ | $\alpha=0.01$ | $\alpha=0.005$ |
|---|---:|---:|---:|---:|
| D-full | $t=6$ | **$t=7$** | $t=8$ | $t=9$ |
| D-pure | $t=3$ | $t=3$ | $t=4$ | $t=5$ |

→ **The deployed $t=7$ and the conformal threshold on D-full at $\alpha=3\%$ coincide numerically** —
**a direct consequence of the identity in §5.8.1 ($\mathrm{EPR}(t)=\alpha$), not independent validation.**
> **One may not write "this is no coincidence"**: that reads like "$t=7$ has the backing of conformal
> theory". **The numerical agreement comes from the identity; the guarantee does not transfer** — this
> paper reaches only "the approximation is more credible on D-full".
→ **Finite-sample false-positive upper bound at $t=7$**: D-full **3.05%** ($2.995\%+1/1804$); D-pure
**0.14%** ($0+1/722$).

> **A failure that must be stated first** (or one reviewer sentence dismantles it): the two cells in the
> D-pure column **are points where the conformal guarantee fails**.
> D-pure's $\mathrm{LCS}$ takes only a few integer values, with **many ties**. **Conformal coverage itself
> requires only exchangeability, not a continuous score**; what fails is **this paper's formula** — when
> translating a quantile into $P(\mathrm{LCS}\ge t)\le\alpha$, if the quantile falls inside a large tie
> block, the actual probability of $\ge t$ jumps to the step beyond the block. D-pure is exactly that
> case.
> The measured illustration: at $\alpha=0.05$ the formula yields $t=3$, but in D-pure the measured share
> with $\mathrm{LCS}\ge3$ is **34.95%**, far above the guaranteed 5.14%.
> **So: the conformal route is roughly usable on D-full; on D-pure it must be stated that "randomisation to
> break ties is required, and was not done here".**
>
> **But "D-full's scores are denser" is no exemption either**: $\mathrm{LCS}$ **always takes integer
> values**, and D-full has **26 items exactly equal to 7** ($54-28$, i.e. $\ge7$ minus $\ge8$) — **ties
> exist there too, only at a lower share**.
> The accurate statement: **on D-full the tie share is low and the conformal approximation more credible;
> but 3.05% is not a strict guarantee** — strictness would likewise require tie randomisation (not done).
> **The two families differ in degree here, not in kind.**
>
> Likewise, the guarantee requires the calibration set and the test sentence to be **exchangeable** —
> this paper's negatives are **constructed**, not drawn from a deployment distribution. **This is more
> fundamental than the ties**: conformal gives a guarantee for new samples from the same distribution, and
> this paper's distribution is self-made. **3.05% must not be read as "the post-deployment let-through
> rate is ≤ 3.05%".**

### 5.8.4 Explaining an old observation: why LCS between unrelated texts is naturally small

Among the 721 unrelated pairs of D-pure: **median LCS 2, maximum 5**. Chinese has thousands of common
characters, and two independent texts of length $\sim10^4$ are unlikely to share a long common substring
anyway.

The classical result is that the longest common substring of two independent random strings has
expectation growing **logarithmically** in string length (Arratia–Waterman / Erdős–Rényi lineage). This
explains why $t=7$ already sits in the far tail of the null — **not because 7 is special, but because
"two unrelated texts colliding on 7 characters" is intrinsically rare.**

> **The formula was not verified and does not enter the text**: the specific form of that classical result
> (something like $2\log_\sigma n$) was not checked paper by paper, and **by this paper's own rule it may
> only be described as "consistent in order of magnitude" in the discussion, not written as a formula**.
> This paper uses the **empirical null** (D-pure / D-full), not an analytic one — a choice to be stated at
> the boundary with §2: an analytic form supplies a prior on "how strict it should be", an empirical
> distribution supplies "how strict it is", and **only the latter is used here to conclude**.

---

---

## 5.9 Chapter conclusions

**Five things that can be said:**

1. **"No simultaneously acceptable operating point on the frontier" holds under two accountings only.**
   All three must be labelled —
   - FP ≤ 5% → wrongful rejections ≥ **26.57%** (point estimate; three-accounting spread 17.30–40.14%,
     not a confidence interval);
   - FN ≤ 25% → letting through ≥ **6.60%** (point estimate; 16.42% under the least favourable
     accounting).
   - **But the exception must follow immediately**: under the **lower accounting**, "FP ≤ 5% and
     FN ≤ 25%" **does have a solution** — the set $\{7,8\}$, and the deployed $t=7$ is in it. **So this
     paper may not say "both ends cannot both be acceptable under any accounting"; only "they cannot under
     the point and upper accountings".** This is the place in the chapter most easily overturned by a
     single reviewer sentence, and stating it first is better than being told.

2. **The deployed $t=7$ implies an unstated cost assumption**: letting through is **1.16–1.41 times**
   dearer than wrongful rejection (point estimate). The equal-weight optimum is $t=5$, but **$t=7$ differs
   from it by only 0.92 points**.

3. **Whether the same $t=7$ is defensible depends on the negative family**: family B-main requires
   "letting through 0.76–1.19× dearer", while on family D-pure **$t=7$ has no optimal interval** ($t=6$ is
   strictly better: identical false positives, lower wrongful rejection). (These two family results are
   unaffected by the labels.)

4. **With the human labels in, $t=7$ has an optimal interval under all three false-negative accountings**
   (upper 1.76–2.14 / point 1.16–1.41 / lower 0.76–0.92). This paper may now say: "**$t=7$ has a cost
   premise that can be written down**". It still may **not** say: "**$t=7$ is correct**" — because that
   cost ratio is a business judgement this paper has not measured.

5. **$t=7$ survives in 6 of the 9 cells** (§5.5.5), and the failures are very orderly: **all three
   "format only" cells drop out; the other six all remain.**
   So its reasonableness depends **mainly** on **one** condition — whether the false-positive accounting
   counts "Chinese terminology overlap" as letting through. Counted, it is in (requiring letting through
   0.76–7.94× dearer); not counted, it disappears.
   **But it is not "only" this one**: two others are equally necessary — **the negatives must be family
   D-full** (§5.5.2) and **the cost ratio $w$ must fall inside that cell's interval**. All three are
   indispensable; only the EPR accounting makes it **vanish outright**.

> The three multiplier ranges in item 2 (1.16–1.41 / 0.76–1.19 / 0.76–7.94) **must appear on the same
> screen as the conditions they depend on**: the negative family (§5.4 / §5.5.2), the false-negative
> accounting (§5.5.4), the three false-positive accountings (§5.7), and the nine-cell table (§5.5.5).
> **Citing a multiplier alone is a naked number divorced from its conditions**, and that is where this
> paper is most exposed to a reviewer.

> **§5.8 adds a limitation more fundamental than "is there a justification", and it must be stated
> alongside:**
> this paper makes **308 comparisons, not one**. Under both error-control standards $t=7$ **fails**, **but
> it fails in different ways, and the two must not be merged**:
> **FWER** (at least one error ≤ 5%, Bonferroni): each $\alpha\le1.62\times10^{-4}$ is required, while
> 1,803 negatives resolve only $5.5\times10^{-4}$ —
> **the conclusion can only be "the existing negatives cannot certify a non-zero level that small", not a
> statistical "no solution"** (which would assert that no valid test exists, something this paper has not
> proved);
> **FDR 5%** (a controllable error proportion): $t\ge9$ is required, and **the deployed $t=7$ is too
> permissive**.
> **$t=7$ was set on the scale of a single comparison at $\alpha=3.00\%$.** Cost justification (§5.5) and
> error control (§5.8) are two separate standards, and this paper has done only the former.

**Four things that must not be said:**

| Must not say | Why |
|---|---|
| "accuracy is only 33%" | $\mathrm{ENR}$ is not an error rate; its denominator is the positive subsample (B1) |
| "so the threshold should be changed to 5" | $t=5$ would raise letting-through from 54 to 160 items; choosing $t$ is a question of cost ratio, not of right and wrong |
| "we found that Jaccard / threshold sweeps are misleading" | the mechanism is known; this paper only measures the dose (B18 below) |
| "t=7 is the optimal threshold" | it is optimal only when $w\in[0.708,0.861]$, negatives are family D-full, **the false-positive rate is the 3.00% all-inclusive reading**, **and the false-negative rate is the point estimate** — **all four conditions must be stated**. Omit the false-positive accounting and the interval drifts to $w\in[0.191,0.226]$ or disappears entirely (§5.7 / §5.5.5) |

**What the next chapter (§6) needs:** the entire FN column of Table 5.1 — §6 shows how much is lost by
compressing that column into a single Jaccard number, and its arithmetic rests on the identity
$J(S(t_1),S(t_2))=|S(t_1)|/|S(t_2)|$.

---

# 6 Information Loss: Why Set-Overlap Metrics Cannot Score a Decision Rule

> This chapter answers one question: **if we evaluate the same decision rule with a "more general" metric
> (Jaccard, Dice, F1), would the conclusion change?**
> The answer is neither "yes" nor "no" — it is **that the metric cannot produce that number at all**,
> because it does not carry that information. The chapter gives recomputable counterexamples and states
> clearly **whose** this observation already is in the literature.

---

## 6.0 Setting the terms: this is not "one metric is not good enough"

An external novelty check assessed the claims of this chapter as **"a re-statement", not "a new
result"**. The reason is hard to argue with: **that overlap metrics degenerate on nested sets is
already has prior sources**.

- Baselga (2010, *GEB* 19(1):134–143) decomposes Jaccard exactly into a "turnover" and a "nestedness"
  component, and shows that under the nestedness component the reading **reflects only the ratio of
  set sizes**;
- Webber, Moffat & Zobel (2010, *TOIS* 28(4) Art. 20), in motivating RBO, state explicitly that
  **"Jaccard is degenerate on nested prefixes"**;
- Fagin et al. (2003, *SIDMA* 17(1):134–160) and Koleff et al. (2003) reach the same conclusion.

So this chapter **claims no new mechanism**. It claims exactly one thing:

> **Applying this known mechanism to the concrete setting of "threshold audit for a citation-checking
> decision rule", and measuring its dose there — how large it is, at which step it misleads, and
> whether it can be repaired.**

"Dose" is not a rhetorical word here. §6.2 gives the number: **the same Jaccard value of 0.500 can
correspond to false-negative rates of 18% and 80% under two different decision rules — a gap of
62 percentage points.**

### 6.0.1 The other half of the literature must be cited too: empirical results from NLP evaluation

The four citations above are **set-similarity theory** (they explain *why* degeneration happens). But
there is a second, **empirical** literature on the same fact, and there it is already common knowledge:

> **Sentence- and segment-level overlap metrics correlate only weakly with human judgements.**

| Work | Conclusion |
|---|---|
| Reiter & Belz 2009 (*CL* 35(4)) | Overlap metrics approximate linguistic quality only; they **do not measure content quality at all** |
| Reiter 2018 (*CL* 44(3)) | BLEU is suitable for development-time diagnostics only; **it should not be used for other tasks, nor at segment level** |
| Novikova et al. 2017 (SIGdial) | Segment-level correlation is poor |
| Kryscinski et al. 2019 (EMNLP) | ROUGE is **not significantly correlated with either relevance or faithfulness** |
| Bhandari et al. 2020 (EMNLP) | Extremely low segment-level correlation; **distributional metrics are no better than ROUGE** |
| Fabbri et al. 2021 (*TACL*) | Across 14 metrics, most correlate only weakly or moderately |

**And "deciding whether a sentence has a source" is itself an established research direction**
(NLG attribution / verifiability evaluation, see §2.2.1) — the AIS framework of Rashkin et al. 2023,
ALCE of Gao et al. 2023, and AttrScore of Yue et al. 2023. Their **task framing is the same as ours**;
the difference is that they work at the semantic level (NLI / QA / LLM judging), whereas this paper
works at the lexical level.

**The more uncomfortable part: this literature has already *measured* that overlap metrics are unreliable.**

Two pieces of published evidence make the boundary of this chapter narrower than one would like, and
they belong at the front rather than buried.

**① ALCE, Appendix A, Table 10: the metric barely moves while true quality falls to a quarter**

| System | ROUGE-L | claim recall |
|---|---:|---:|
| ChatGPT Vanilla | 20.6 | 12.0 |
| ChatGPT Oracle | 21.2 | 21.3 |
| LLaMa-13B Vanilla | 16.2 | 3.9 |
| **Top-1 passage** (returns only the best retrieved passage) | **19.1** | **3.0** |

The ROUGE-L of the degenerate baseline `Top-1 passage` is **only 7.3% below** ChatGPT's
($19.1$ versus $20.6$, **retaining 92.7%**), while its true claim recall is **down to 25%**
($3.0$ versus $12.0$, **a drop of 75%**).

**Read that sentence twice: the metric drops 7.3%, true quality drops 75%; read the other way, the
metric retains 92.7%, true quality retains only 25%.**
**Both readings point at the same thing: the metric hardly moved while true quality fell a long way.**

> **Both statements must be given in the same convention, or a reader will think the two magnitudes
> are incomparable**: both numbers are correct ($19.1/20.6=92.7\%$, $3.0/12.0=25\%$), but **one is a
> drop and the other is a remainder**, and mixing them in one sentence reads as though the ruler had been changed.
> So both are given in full: **drop 7.3% versus 75%, retention 92.7% versus 25%.**
>
> **The claim "either way of reading it gives a factor of ten" does not hold**: the ratio of drops is
> $75/7.3=\mathbf{10.3}$; the ratio of retentions is $92.7/25=\mathbf{3.7}$. **The two readings differ by
> a factor of three, not ten.** The only defensible common statement is: **the metric hardly moved
> (down 7.3%) while true quality fell a long way (down 75%)** — the factor depends on which reading is
> used.

**The authors' own words:**

> "We elect **not to use ROUGE-L** … since it does **not account for the different ways of expressing
> the same answer** and it can be **easily gamed**."

**② Krishna et al. 2021 (NAACL): replacing retrieved documents with random ones leaves ROUGE-L almost unchanged**

> "Results from both human A/B tests and automatic metrics like ROUGE-L demonstrate that conditioning
> on **random documents** has **almost no effect** on generated answer quality." (Figure 1c)
>
> "**ROUGE-L is not an informative metric of generated answer quality and can be easily gamed.**"

> **Together these two cut away another piece of what §6 can claim.**
> "Overlap metrics are unreliable" is not an observation of this paper: a proper controlled experiment
> was run in **2021** and the result was verified again in **2023**. And Krishna's setup is **stronger evidence**
> than our 127 pairs — this paper compares "same metric value, two decisions", whereas they
> **destroy the input entirely and the metric does not move**.

**So §6 can only position itself on three items, no more:**

| # | What is left to this paper | Why it is not already taken |
|---|---|---|
| 1 | **Structural explanation**: on nested decision sets $J$ is identically the size ratio $\hat F(t_1-1)/\hat F(t_2-1)$, and the denominator $n_1$ cancels | **The mechanism has prior sources; this paper only specialises it and measures the dose**: Baselga 2010 (*GEB*) already decomposes Jaccard exactly into turnover and nestedness and shows that under nestedness the reading reflects only the size ratio; Webber–Moffat–Zobel 2010 agree — **so we do not write "nobody has explained why"**. What this paper gives is **the explicit form of that degeneration on the special case where the decision sets are nested** ($J$ reduces to a size ratio and $n_1$ cancels), and where that forces the choice in a citation-checking setting. |
| 2 | **Cost and significance language**: connecting it to $t$, $w^*$, $\alpha$ | Those works report recall/precision; **they discuss neither cost nor a significance level** |
| 3 | **Frequency on real data**: 127 zero-distance counterexample pairs (§6.4.1) | They have not counted this on the citation-checking task |

> **Four sentences that must not be written**:
> ① not "we find that overlap metrics are unreliable" (ALCE / Krishna measured it);
> ② not "we find it can be gamed upward" (ALCE Table 11 measured it, see §8.6.1);
> ③ not "we are the first to give a counterexample" (Krishna's random-document control is a
> counterexample);
> ④ not "their method is better or worse" (no controlled experiment was run).

### 6.0.2 The dividing line between this paper and that literature: **non-identifiability, not weak correlation**

This is the only increment this chapter can still establish, and it must be stated clearly or the
chapter will be read as "a known conclusion said differently":

| | That literature says | This paper says |
|---|---|---|
| Wording | **weak correlation** | **non-identifiability** |
| What it is | A statistic: on average, the two do not line up | A structural property: **there exist cases with the same metric value and opposite decisions** |
| Form of evidence | A correlation coefficient (Pearson / Spearman) | **A constructive counterexample** plus **the number of zero-distance pairs in real data** |
| Can more data relieve it | **Yes** (larger samples give a steadier correlation estimate) | **It cannot change the structure, but it can improve the estimate** (see the note below) |
| Practical consequence | "Do not trust this metric too much" | **"No threshold based on it can certify 'has a source'"** ( scope of this sentence in the note below) |

> **"Cannot change the structure" and "can improve the estimate" must be kept apart**: more data
> **makes the estimate of $\mathrm{ENR}$ more accurate** (that is a question of statistical precision,
> and larger samples do help); but it **cannot change the fact that $J$ on nested sets contains no
> information about the denominator** — that is **structure**, independent of sample size.

**Evidence from real data (§6.4.1)**: on the 100 human-judged items of this paper, **127 sentence
pairs have exactly the same LCS while one is judged "findable" and the other "not findable"**; all ten
LCS buckets contain both A and B.

> **Two steps must be written out, or the reader cannot follow the connection**:
> ① **What these 100 items are**: they are the stratified sample of §4.6, a **subset of the 308**
> (not a separate draw), so the 127 pairs are a fact **about this dataset**, not something constructed.
> ② **The step from LCS to $J$**: two sentences with the same $\mathrm{LCS}$ have, for **any** $t$,
> exactly the same membership in $S(t)$; hence for **any** $t_1,t_2$, the three numbers
> $|S(t_1)|,|S(t_2)|,|S(t_1)\cap S(t_2)|$ are all identical; hence **$J$ is identical**.
> — **So "same $\mathrm{LCS}$, opposite decision" is equivalent to "same $J$, opposite decision".**
> This step is an identity and needs no extra assumption.

> **Put differently: it is not that "the metric is not accurate enough", it is that "at this point the
> metric does not contain that information".** The former can be improved; the latter **can only be
> addressed by switching to another class of set-overlap metric** ( scope in §8.4: non-overlap
> methods — semantic embeddings, entailment judges — **were not tested at all in this paper**) — which
> is exactly what the words "information loss" in the chapter title mean.

> **The level of the "certify" statement must be made explicit**:
> · **At the population level**: given a value of $J$, $\mathrm{ENR}$ can take any value in $(0,1]$
> (construction in §6.5);
> · **At the sentence level**: the real counterexamples of this paper are **127 pairs** (same
> $\mathrm{LCS}$ / same $J$, opposite decisions, above). They prove that **one reading can admit two
> answers** — **not** that every sentence is judged wrongly.
> So "cannot certify 'has a source'" means **"a single reading cannot be offered as a guarantee to a
> third party"**, **not** "this decision rule is useless overall" — as a **banding device** it still
> works, and §5 is entirely about measuring its banding error.

---

## 6.1 Writing the two quantities as computable expressions

We keep the notation of §3 (`S2`, §1):

| Symbol | Definition |
|---|---|
| $n_1$ | number of items in the positive subsample = **308** (the main dataset of this paper is entirely positive, $n_0=0$) |
| $\hat F(k)$ | $\#\{i: y_i=1,\ \mathrm{LCS}(x_i)\le k\}/n_1$, **defined on the positive subsample** |
| $\mathrm{ENR}(t)$ | $\hat F(t-1)$, i.e. the share of genuinely sourced items judged unsourced |
| $S(t)$ | $\{x:\mathrm{LCS}(x)<t\}$, **monotonically increasing in $t$** |

Because $S(t)$ is **nested** in $t$ ($t_1<t_2\Rightarrow S(t_1)\subseteq S(t_2)$), Jaccard degenerates
here into a pure ratio:

$$J\big(S(t_1),S(t_2)\big)=\frac{|S(t_1)\cap S(t_2)|}{|S(t_1)\cup S(t_2)|}=\frac{|S(t_1)|}{|S(t_2)|}=\frac{\hat F(t_1-1)}{\hat F(t_2-1)}\qquad(t_1\le t_2,\ \hat F(t_2-1)>0)$$

> **The condition on the right is not formalism: drop it and the expression is $0/0$.**
> $\hat F(t_2-1)=0$ means $S(t_2)=\varnothing$, and the Jaccard index of two empty sets is itself
> undefined. On the data of this paper $\hat F(6)=207/308\neq0$, so nothing goes wrong; but **the
> formula must carry the condition**, or it fails silently on sparse data.
> One further data-dependent assumption is worth naming: the step $|S(t)|/n_1=\hat F(t-1)$ requires
> that the counting universe of $S(t)$ be the counting universe of $\hat F$. Here $n_0=0$ (everything
> is positive), so it holds automatically; **it breaks on any dataset that contains negatives**.

This expression is the foundation of the whole chapter's argument. It says two things:

1. **Both numerator and denominator cancel $n_1$** — so $J$ **contains no information about "how large
   the denominator is"**. And the false-negative rate is precisely a numerator divided by a denominator.
2. **$J$ reads only two points** — $\hat F(t_1-1)$ and $\hat F(t_2-1)$. The full behaviour of the
   decision rule is twelve points ($t=2,\dots,13$, §5.1). **Representing twelve points by two loses the
   ten in between.**

**Checked digit by digit on the real data** (`S1·_反例_JvsENR.py`):

```
t=4: J=|S(4)|/|S(7)| = 120/207 = 0.5797   ← written as 0.580 in the text
t=6: J=|S(6)|/|S(7)| = 187/207 = 0.9034   ← written as 0.903 in the text
t=8: J=|S(7)|/|S(8)| = 207/223 = 0.9283   ← written as 0.928 in the text
```

> **The denominator follows the reference point; it is not a uniform $n_1=308$**: the first two have
> denominator $|S(7)|=207$ (both are measured against $S(7)$), the third has $|S(8)|=223$. So misusing
> $J(S(4),S(8))=120/223=0.5381$ yields $0.538$ rather than $0.580$ — this is the easiest place in this
> section to get a recomputation wrong, so it **must be stated explicitly** (`S2`, §1, last item).

---

## 6.2 Counterexample: pin $J$ and the false-negative rate can be anything

This is a construction, not an illustration. **Given $J$, construct two decision rules with exactly
equal $J$ and arbitrarily different $\mathrm{ENR}$.**

### 6.2.1 Pair A: $J=0.500$, $\mathrm{ENR}$ differs by 62 percentage points

Take $t_1=3$, $t_2=7$; then $J=\hat F(2)/\hat F(6)$; the error rate is reported at $t=7$, which reads
$\hat F(6)$ as well.

| Decision rule | $J$ | $\mathrm{ENR}(7)$ | minimum $n_1$ |
|---|---:|---:|---:|
| A1 | 0.500000 | **18.00%** | 100 |
| A2 | 0.500000 | **80.00%** | 5 |

**$J$ differs by $0.0\times10^{0}$; the false-negative rate differs by 62.00 percentage points.**

### 6.2.2 Pair B: a different set of values, to show this is not a coincidence

Take $J=2/9=0.2222$:

| Decision rule | $J$ | $\mathrm{ENR}(7)$ | minimum $n_1$ |
|---|---:|---:|---:|
| B1 | 0.222222 | **90.00%** | 10 |
| B2 | 0.222222 | **45.00%** | 20 |

**$J$ identical; the false-negative rate differs by 45.00 percentage points.**

### 6.2.3 The strongest form: fix $J=0.5$ and $\mathrm{ENR}$ can take any value in $(0,1]$

The construction is direct: **for any given $J$**, to make $\mathrm{ENR}$ equal any target $e$, set
$\hat F(t_2-1)=e$ and $\hat F(t_1-1)=J\cdot e$ — the ratio of the two is identically $J$, independent
of $e$. This section pins $J$ at $0.5$ only to keep the arithmetic in round numbers; **the $0.5$ is not
part of the construction.**

```
  target ENR= 1%  →  F(t1-1)=0.0050  F(t2-1)=0.0100  J actual=0.500000  min n1=200
  target ENR=50%  →  F(t1-1)=0.2500  F(t2-1)=0.5000  J actual=0.500000  min n1=4
  target ENR=99%  →  F(t1-1)=0.4950  F(t2-1)=0.9900  J actual=0.500000  min n1=200
```

> **At the population level, given $J$, $\mathrm{ENR}$ can take any value in $(0,1]$. The information
> content is zero.**

> **The nature of this sentence must be stated immediately, or what §6.6 concedes will make it read
> as bluffing**: it is a **general impossibility statement, and that statement has prior sources** (the
> decomposition of Baselga 2010 already implies it; see §6.0 and §6.6). **What this paper adds
> is not in that sentence, it is in the next one**: turning "any value" into **a pair of numbers
> concrete enough to be recomputed item by item** (same $J=0.5$, one 18% and one 80%), and pointing it
> at the specific setting of **citation-checking audit**. **The general conclusion belongs to others;
> this pair of numbers belongs to this paper.**

This is the strongest assertion in the chapter (the conclusion in §6.7 restates it at equal strength,
which is why it is not called "the only" one), and also the sentence a reviewer is most likely to
attack — §6.3 meets the most likely attack first.

---

## 6.3 A finite sample does not block this

The most natural objection is: *real samples are finite, so $\mathrm{ENR}$ can only take values on a
grid of $1/n_1$; surely $J$ still constrains it somewhat?*

**This is a question of granularity, not of direction.**

| $n_1$ | Grid spacing |
|---:|---:|
| 100 | 1.00 percentage point |
| **308** (this paper) | **≈0.32 percentage points** |

However fine the grid, it can only **approach** any point of $(0,1]$ — **a finite grid never "fills" an
interval**.

> **"Dense" must be used with care here**: **what can approach any point is the family of grids as
> $n_1 \to \infty$, not any one fixed finite grid** — a fixed set of 308 items has 308 grid points
> spaced about 0.32 percentage points apart; it is neither an interval nor dense in one. But
> **the conclusion is unaffected**: however large $n_1$ is, $J=0.5$ can be paired with a decision rule
> that is almost always right or almost always wrong. **Granularity decides how finely the interval can
> be cut, not how wide the interval is.**

> **A stricter side result**: the $1/n_1$ above is the grid spacing **without** pinning $J$. If
> $J=0.5$ must hold **exactly**, then $\hat F(t_1-1)=\hat F(t_2-1)/2$ must also fall on the grid, which
> requires the numerator of $\hat F(t_2-1)$ to be even, and **the attainable spacing of $\mathrm{ENR}$
> becomes $2/n_1$** (about 0.65 percentage points at $n_1=308$, twice as coarse as 0.32).
> **The conclusion does not change by a word**: the span can still be made arbitrarily fine — the lower
> end at $2/n_1$, the upper end at 1 (which **requires $n_1$ even**; for odd $n_1$ the upper end is
> $(n_1-1)/n_1$, because $\hat F(t_2-1)=1$ would require $\hat F(t_1-1)=1/2$, which is not on the grid.
> Here $n_1=308$ is even, so this does not bite) — it is only that "how fine" is a little coarser than
> the table suggests. This is included to close off the objection "your grid is finer than it can be".

> **What this blocks and what it does not — the distinction is essential, or the argument becomes
> circular**:
>
> **It meets the objection "this is an extreme construction".** The answer: a constructive claim is
> about **existence** — as long as one pair of decision rules exists with equal $J$ and a 62-point
> difference in $\mathrm{ENR}$, **the claim that "$J$ carries information about $\mathrm{ENR}$" has
> already been refuted**, and how "extreme" the pair is does not affect the refutation. To overturn it,
> a reviewer must show the pair is not admissible, not that it is unusual.
>
> **It does not meet the objection "then why are they aligned on the real data".** This paper cannot
> answer that; §6.4 can only establish that the alignment **is not guaranteed**, **not that it is pure
> coincidence** — the latter would require a second real dataset, which this paper does not have.
>
> The two must be written separately: **the first meets "the construction is too extreme", the second
> concedes "the real-data alignment is unexplained"**. Using the second to block the first ("we have
> only one sample, so it must be luck") defends a **weaker** claim in place of a **claim already
> established**, and goes round in a circle.

---

## 6.4 On the real data $J$ and $\mathrm{ENR}$ happen to move together — but this is not guaranteed

On the 308 real items of this paper, $J$ and $\mathrm{ENR}$ **do move together** ($\mathrm{ENR}$ taken
as a point estimate; all three accountings move together, in parentheses):

- $t$ from 7 down to 4: $J$ falls from $1.000$ to $0.580$, $\mathrm{ENR}$ falls from $26.57\%$ to
  $15.40\%$ (upper bound 40.14→23.27, lower bound 17.30→10.03);
- $t$ from 7 up to 8: $J$ falls to $0.928$, $\mathrm{ENR}$ rises to $28.63\%$ (upper bound 43.25,
  lower bound 18.64).

> **Those three values of $J$ do not move with the accounting** ($J$ is a ratio of set sizes and
> depends only on how many items are judged unsourced); **only $\mathrm{ENR}$ scales with it**. So the
> observation "they move together" holds under all three accountings — and **that is exactly its
> weakness**: the alignment comes from the scaling, not from two curves independently corroborating
> each other.

**But this is not a transferable property.** §6.2 has already shown: change the decision rule, keep $J$,
and $\mathrm{ENR}$ can be completely different.

> **There is a distinction here, and this paper claims only the weaker half:**
>
> - **The half this paper claims (weak, proven)**: alignment **is not guaranteed** — there exist
>   decision rules with identical $J$ and wildly different $\mathrm{ENR}$. This is constructive and it
>   holds.
> - **The half this paper does not claim (strong, unproven)**: that the alignment is **pure
>   coincidence** — saying "it is luck" would require a second real dataset showing that the alignment
>   disappears on other data. **This paper has one dataset, so that sentence is not written.**
>
> Every consequence below therefore rests only on the first half: **since it is not guaranteed, it
> cannot be written into an evaluation protocol** — which needs no proof that it is coincidental.

> **The misreading this section must resist**: **"but you measured it and they do move together, so
> why not just evaluate with $J$?"**
> Because the alignment is an **observation on this dataset**, not information **carried by $J$**.
> Writing an **unguaranteed** alignment into an evaluation protocol means it may fail on the next
> dataset, and **it will not raise an error when it does** — it will quietly return a number that looks
> entirely reasonable.
>
> This is why the chapter says **information loss** rather than "inaccuracy": **it does not compute a
> value wrongly; it simply does not have that value.**

### 6.4.1 It does happen in the real data: 127 pairs with the same LCS and opposite human judgements

The claim that "alignment is not guaranteed" is a **constructive** argument (two synthetic decision
rules). A reviewer will ask the obvious question:

> **"Has this possibility ever actually occurred on your 308 items?"**

It has. More than once. (`S1·_真实带宽.py`, 9 self-test assertions.)

Bucket the 100 P6 human-judged items by $\mathrm{LCS}$ and look at the judgements inside each bucket
(**A = findable in the source / B = not findable / C = cannot tell**; criteria in §4.6.1. The columns
`A|B|C` are raw counts of the three categories, **not** the upper-bound / point / lower-bound
accountings — those three are separate rules for how $C$ is merged):

| $\mathrm{LCS}$ | items | A | B | C | both A and B in the bucket? |
|---:|---:|---:|---:|---:|---|
| 2 | 22 | 4 | 7 | 11 | **yes** |
| 3 | 15 | 6 | 5 | 4 | **yes** |
| 4 | 11 | 4 | 5 | 2 | **yes** |
| 5 | 13 | 6 | 6 | 1 | **yes** |
| 6 | 10 | 1 | 1 | 8 | **yes** |
| 7 | 5 | 1 | 4 | 0 | **yes** |
| 8 | 3 | 1 | 2 | 0 | **yes** |
| 10 | 4 | 2 | 1 | 1 | **yes** |
| 12 | 2 | 1 | 1 | 0 | **yes** |
| 13 | 4 | 3 | 1 | 0 | **yes** |

**Ten buckets, every one of them containing both "findable" and "not findable".**

> **This table lists only the buckets where A and B coexist — another 11 items fall in buckets with a
> single judgement** (including the two at $\mathrm{LCS}=11$, both A, and those at $\mathrm{LCS}=0$ and
> $1$), so the item counts above sum to **89**, not 100. **Without this sentence a reader will take
> "ten buckets" for the whole sample and will be unable to reproduce the gap reported next.**

Counting every pair of sentences with the same $\mathrm{LCS}$ where one is A and the other B: **127
pairs.**

> **This is where the line between "non-identifiability" and "weak correlation" lands on the data.**
> A low correlation coefficient means "on average they do not line up"; what happens here is that **the
> metric takes one and the same value while the decision can go either way, and there are 127 pairs
> that do so**. The synthetic counterexample proves it **can** happen; these 127 pairs prove it **did**
> happen on this data.
>
> **One step is still missing between those two sentences**: the construction of §6.2 is about
> **two decision rules with the same $J$ and different $\mathrm{ENR}$**, whereas the 127 pairs are
> about **one LCS value with opposite human judgements** — **two different propositions**. The step
> joining them **is an identity**, and one sentence suffices:
> **swap the judgement labels pairwise among these 127 pairs to obtain two new datasets; the two have
> identical $\mathrm{LCS}$ distributions (because labels do not touch $\mathrm{LCS}$), so $J$ is
> identical; and as the number of A items changes, $\mathrm{ENR}$ differs.**
> — **So "same $\mathrm{LCS}$, opposite decision" is an instance in real data of "equal $J$, unequal
> $\mathrm{ENR}$".** This is not an analogy; it is the same fact stated two ways.

**A further, more striking number**: across adjacent $\mathrm{LCS}$ cells, the largest drop in the A
rate is

| accounting | adjacent cells | drop |
|---|---|---:|
| upper bound | $\mathrm{LCS}=6$ (90%, $n$=10) → $\mathrm{LCS}=7$ (20%, $n$=5) | **70 percentage points** |
| point | $\mathrm{LCS}=11$ (100%, $n$=2) → $\mathrm{LCS}=12$ (50%, $n$=2) | 50 percentage points |

> The $\mathrm{LCS}=11$ bucket of the second row **is not in the table above** (its two items are
> both A, with no B, so it does not enter the "coexisting" table) — which is exactly why the previous
> paragraph added that note. **Small-sample warning**: cells hold between 1 and 22 items, so a single-cell A rate fluctuates
widely; read these adjacent-cell differences **as an upper bound, not as point estimates**. What is
solid is the "ten buckets all coexist" table and the "127 pairs" — neither depends on any single-cell
estimate.

> This section moves the claim of §6 from "logically possible" to "actually happened on this data".
> It **does not change** the weak version of the conclusion: there is still only one dataset, and it
> still cannot be said that "another dataset would break the alignment".

---

## 6.5 Can it be repaired: can the whole $J$ curve recover $\hat F$?

If a single reading of $J$ will not do, can **sweeping a whole curve** save it? **Yes — but only with
two necessary conditions and two paper-level premises; drop any one and the expression fails.**

### 6.5.1 Condition one: the sweep must run in the direction $t_1>t_2$

Fix a reference $t_2$ and sweep $t_1>t_2$; then $J(S(t_2),S(t_1))=\hat F(t_2-1)/\hat F(t_1-1)$.

Once $t_1$ exceeds $\max\mathrm{LCS}$, $\hat F(t_1-1)=1$ — note this is an **exact equality, not an
approximation**: $\hat F$ is a step function and $\mathrm{LCS}$ is bounded above (§6.5.2), so after
finitely many $t$ it lands exactly on 1. Hence

$$J=\hat F(t_2-1)\quad\Longrightarrow\quad \hat F(t_1-1)=\frac{\hat F(t_2-1)}{J(t_1)}$$

**The reverse direction does not work.** Sweeping only $t_1<t_2$ determines $\hat F$ on
$(-\infty,t_2-1]$ only up to a constant multiple; the anchor $\hat F(\infty)=1$ is unavailable, and
nothing can be recovered.

### 6.5.2 Condition two: $\mathrm{LCS}$ must be bounded above

The step $\hat F(\infty)=1$ relies on $\mathrm{LCS}(x)\le\min(|x|,|s|)$ — that is, **for any sentence
paired with a finite source, LCS has a ceiling**. This holds on the data of this paper ($\mathrm{LCS}$
is bounded by construction), but with another similarity — cosine, say — it would have to be verified
again.

### 6.5.3 Paper-level premises three and four: a non-zero anchor, and $J$ taken exactly

**Premise three (identifiability)**: **$\hat F(t_2-1)>0$**. This is the same condition the expression
in §6.1 carries, appearing again in §6.5 in the role of an anchor — if the reference set is empty,
$J\equiv0$ along the whole curve and the inversion $\hat F(t_1-1)=\hat F(t_2-1)/J(t_1)$ is $0/0$;
**nothing can be recovered**.

**Premise four (precision)**: the formula uses the **exact** value of $J$, not a three-decimal rounding.
If $J$ is reported as $0.580$, the recovered $\hat F$ carries the corresponding precision only — this is
an arithmetic premise at the paper level, **not an estimation problem**, which is why it sits here and
not in the estimation layer.

> **The layers are kept explicit so that a defect of the estimation layer is not smuggled in to block
> this one**: as $t\to\infty$ the sample size goes to zero, and $J$ carries a small-sample bias —
> **both are estimation-layer problems, not conditions at this layer, and neither is added here**.
> And "sample size" does not exist at the paper level in the first place: $\hat F$ is a step function
> and $\mathrm{LCS}$ is bounded, so $\hat F$ is **exactly** 1 at finite $t$ (not approximately).

> **The practical meaning of the four taken together**: replacing a threshold sweep with a $J$ curve
> costs **sweeping toward $t\to\infty$** — and in citation checking, $t\to\infty$ means "almost any
> sentence counts as sourced", a range that does not overlap the audit target of this paper
> ($t\ge12$ already amounts to abolishing manual review, see §5.6 and §6.7).
>
> **So the statement is about "the ranges do not overlap", not "it is not worth it"** — the latter
> would need a comparison of sweep cost, human cost and benefit, and **this paper has computed none of
> the three**. This restriction is the same class of defect as "is $w<1$ reasonable" in §5 (an
> unverified piece of domain folklore written as a conclusion), which the first review round already
> caught once; here it is held back deliberately.

---

## 6.6 What this chapter does not claim

Per item B18 of this chapter's own list — the general conclusion must not be claimed as this
paper's own — none of the following may appear:

| Must not say | Because |
|---|---|
| "This paper is the first to reveal Jaccard degeneration on nested sets" | Has prior sources: Baselga 2010 already gives the exact decomposition; the motivation of RBO 2010 states it explicitly |
| "This paper finds that set-overlap metrics cannot be used for evaluation" | That conclusion is an existing consensus across several works; it is not this paper's |
| "$J=\lvert S(t_1)\rvert/\lvert S(t_2)\rvert$ is a finding of this paper" | It is a three-line fact about nested sets, textbook level |

**The increment this chapter can claim is exactly one item** (from the external novelty check):

> **The explicit adversarial construction "fix $J$ and build an 18% versus 80% pair of decision rules"**, counted as a
> narrow application-level increment — it turns an existing mechanism into **a recomputable
> counterexample** aimed at the concrete setting of **citation-checking audit**.

In one sentence: **the mechanism belongs to others, the dose belongs to this paper.**

> **So that this does not read as self-contradiction**: §6.2.3 states that "$\mathrm{ENR}$ can take
> any value in $(0,1]$", which sounds like a general impossibility theorem, while this section says the
> increment is "one pair of numbers". The two do not conflict, **because they are at two levels**: the
> general conclusion **belongs to the literature** (already implied by the decomposition of Baselga
> 2010), and what this paper does is **turn it into a dose that can be recomputed item by item and
> point it at the audit setting**. Nowhere in this chapter may the general conclusion be written as
> this paper's own — that would be claiming the general conclusion as ours (B18).

---

## 6.7 Conclusion of this chapter

**One sentence has to come first:**

> **"Sentence- and segment-level overlap metrics correlate only weakly with human judgement" is not a
> finding of this paper.** It is an established conclusion in NLP evaluation (Reiter & Belz 2009 /
> Reiter 2018 / Novikova 2017 / Kryscinski 2019 / Bhandari 2020 / Fabbri 2021), and on the specific
> task of "does this sentence have a source" a whole line of research already exists (AIS of Rashkin
> 2023 / ALCE of Gao 2023 / AttrScore of Yue 2023, see §2.2.1).
> **What this paper adds is not "it is also weak", but "it is weak to the point of carrying no
> information".** The difference is one word, and it is a whole class of conclusions: the former can be
> relieved with more data, the latter cannot.

**Five sentences that can be said:**

1. **On nested sets $J$ is identically the size ratio**, and the denominator $n_1$ on the $\hat F$ side
   cancels — **so it carries no information about "how large the denominator is"**, and that denominator
   is precisely what a false-negative rate is built on.
2. **Given $J=0.5$, the false-negative rate can be constructed as 18%, as 80%, or as any value in
   $(0,1]$.** A finite sample decides only the grid spacing and **does not narrow the span** — and the
   spacing requires two numbers: at $n_1=308$ it is **0.32 percentage points** when $J$ is not pinned,
   and **0.65 percentage points** ($2/n_1$, see the side result in §6.3) when $J=0.5$ is required to
   hold exactly. Quoting 0.32 without the setting would suggest that the side result applies only to
   §6.3 and not to the conclusion.
3. **It really happened on this data, more than once** (§6.4.1): among the $100$ human-judged items,
   **127 sentence pairs have exactly the same $\mathrm{LCS}$ and opposite judgements**, and **all ten
   $\mathrm{LCS}$ buckets contain both A and B**. This is not "low correlation", it is "one value, two
   answers".
4. **The alignment on the real data is an observation on a single sample, not a property of the
   metric.** Written into an evaluation protocol, **it will not raise an error when it fails.**
   > **This sentence does not contradict the chapter's ban on "false safety", but the difference
   > must be spelled out**:
   > · **"will not raise an error" is about the protocol layer**: no rule in an evaluation protocol
   >   reports anything when $J$ fails — that is a **structural fact**;
   > · **"a false safety signal" is about the direction of the bias**: it implies the metric leans
   >   toward "looking safer", which would require separately establishing the source of that bias,
   >   and this chapter cannot reach that.
   > So the first may be said, the second may not. **The difference is "is there any alarm at all"
   > versus "which way does it mislead".**
5. **The repair holds on paper (sweep $t_1>t_2$ + bounded $\mathrm{LCS}$ + non-zero anchor + exact
   $J$), but the evaluation range it requires does not overlap the audit target of this paper** — it
   must sweep toward $t\to\infty$, whereas this paper's setting already amounts to abolishing manual
   review at $t\ge12$. **This paper has not costed that repair; the claim is about ranges not
   overlapping, not about it being not worth it.**

**Five sentences that must not be said:**

| Must not say | Why |
|---|---|
| "This paper is the first to find that overlap metrics disagree with human judgement" | **The phenomenon has prior sources**: a decade of conclusions in NLP evaluation (§6.0.1). Saying it shows the homework was not done |
| "This paper is the first to reveal Jaccard degeneration on nested sets" | **Has prior sources**, see §6.6 |
| "$J$ gives a false safety signal / it reliably misleads you" | "False safety" implies a direction of bias, which needs a separately established source; this chapter only shows **information loss** |
| "So $J$ is the wrong metric" | It is not wrong; it **answers a different question**. Calling it wrong is dismantled by one sentence: "you changed the task" |
| "It aligns in our measurements, so it is usable in practice" | The alignment is an observation on this one sample (§6.4), not a transferable property |

> **How this chapter relates to §5**: §5 measures the cost point by point with a threshold sweep;
> this chapter explains **why the shortcut with $J$ cannot be taken**. Together they give the complete
> answer: **either run the full twelve-point sweep, or accept an information loss that raises no
> error.**

---

> Generation commands: `S1·_反例_JvsENR.py` (self-test **27 assertions**); real-data counterexample pairs:
> `S1·_真实带宽.py` (self-test **9 assertions**). The self-tests of the counterexample script include
> "the $J$ gap for pair A must be exactly 0", "the gap for pair B must be 45.00 points", "the three
> $J$ values must reproduce 120/207, 187/207, 207/223 digit for digit", "an empty anchor must raise an
> error", and "when $J=0.5$ holds exactly the $\mathrm{ENR}$ numerator must be even".

---

# 7 Discussion

> This chapter **does not restate the conclusions of any other chapter** (those are in §5.9, §6.7 and
> §8). It answers three questions only:
> **① How is this used in practice? ② How does it divide the work with the same-task literature?
> ③ What has this paper not done, that we already know how to do?**

---

## 7.0 The position of this paper in one paragraph

This paper has **not** built a better decision rule, and has **not** discovered that "overlap metrics
are unreliable" (that was properly measured as early as 2021, see §6.0.1).

What this paper does is **change the bookkeeping**: the sentence "the threshold is set to 7" is
translated into three things an auditor **can argue with**:

| What used to be a single number | What there is now |
|---|---|
| the threshold $t=7$ | ① **a cost ratio** $w^*=1.16$–$1.41$<br>② **a significance level** $\alpha=3.00\%$<br>③ **the interval spanned by the three accountings** (17.30%–40.14%) |

> **None of the three is "the right answer"** — they are **coordinates that make an argument possible
> at all**. An auditor can now say "I think letting something through costs five times as much", and
> the answer immediately becomes $t\ge9$; or "equal weights are fine", and it is $t=5$.
> **Before this, they did not even know what to object to.**

These three are not three parallel items but **one chain**:

> **The shortcut does not work** (**set-overlap** metrics are non-identifiable for $\mathrm{ENR}$, §6)
> → **since this family cannot be substituted, fall back to a point-by-point sweep** → **the sweep
> translates an integer into a cost ratio** (§5.5) → **and deployment means judging 308 items one by
> one, i.e. 308 comparisons** (§5.8) — **this last link is not on the chain; it comes with the task and
> no choice of metric avoids it.**

> **The restriction must be stated where the argument opens**: since the abstract and §9.2 both say
> "**this family** of metrics cannot be substituted", the opening must not read "the shortcut does not
> work, so only a point-by-point sweep remains" — that would hand the reader, at the outset, the very
> claim that has been withdrawn. Non-overlap methods (semantic embeddings, entailment judges) **were
> not tested at all in this paper** (§8.4).

**The head of the chain is a negative claim.** The sections that follow are all about things on the
chain (7.1 turns it into documentation, 7.2 divides the work with semantic-layer methods, 7.3 is the
human cost at the end of the chain, 7.5 is the boundary of the chain).

---

## 7.1 Three actionable recommendations (very cheap, clear payoff)

### Recommendation one: write $(t,\,w)$ into the system documentation, not a bare integer

System documentation today says "threshold 7". By §5.5 that sentence **is equivalent to** "we assume
that letting a fabricated sentence through costs 1.16–1.41 times as much as wrongly rejecting a genuine
one", while the authors of the system never wrote down the second half.

> **Fix**: add one line beside the threshold definition —
> " $t=7$, equivalent to assuming a cost ratio between the two error types of $w\in[1.16,1.41]$
> (basis: §5.5 of this paper). If the auditor specifies a different $w$, the optimal $t$ is given in
> Table 5.2."
>
> **Cost**: one line of text. **Payoff**: the assumption can now be discussed, challenged, and held
> to account.

### Recommendation two: report all three accountings, and state how "cannot tell" is counted

The false-negative rate of this paper has three values (17.30% / 26.57% / 40.14%), and the difference
comes **entirely** from how the "cannot tell" band is handled (§5.2.2).

> **Fix**: any error rate reported externally should be followed by "the 62 undecidable items were
> excluded from the denominator" or "were counted as not findable".
> **Reporting a single number without its convention is not acceptable** — that number moves by 23
> percentage points across conventions, more than most improvements.

### Recommendation three: record the threshold in $\alpha$ coordinates, not as "7"

$\mathrm{EPR}(t)$ is exactly the **type-I error rate** under the null hypothesis (§5.8). Hence:

> **"Nominal significance level" is the wrong term here**: "nominal" usually means a target set in advance
> that a discrete score may not attain, whereas the 2.995% here is a **measured rejection rate**. Three
> numbers must be kept apart: **target 3.00% / measured 2.995% / finite-sample upper bound 3.05%**
> (§5.8.1).

$$t=7\iff\alpha=3.00\%\qquad(\text{family D, all})$$

> **Fix**: write "$t=7$ ($\alpha=3.00\%$, family D, all)" in the documentation.
> **Payoff**: it immediately connects to a harder criticism — **this paper makes 308 comparisons, not
> one.** If the auditor requires a family-wise error rate of 5% (FWER), **a Bonferroni correction**
> needs $\alpha\le1.62\times10^{-4}$, whereas the negatives here can only resolve down to
> $5.5\times10^{-4}$: **the available data cannot certify it**. If only a 5% FDR is required, then
> $t\ge9$ is needed (§5.8.2).
> **That sentence matters more than "is there any basis for 7", but if it is hidden, nobody will ask.**

---

## 7.2 Division of work with semantic-layer methods: not a substitute, a division

The line of work in §2.2.1 (AIS / ALCE / AttrScore) has the **same task but a different level**. It
should not be read as a competitor:

| | **Lexical layer** (this paper) | **Semantic layer** (AIS / ALCE / AttrScore) |
|---|---|---|
| Depends on | **no model** (pure string matching) | an NLI model or an LLM |
| Can $w^*$ be inverted | **yes** | no (no absolute score) |
| Can it be read as $\alpha$ | **yes** | no (no null distribution) |
| Handles paraphrase | **no** | **yes** |
| Compute | about 2 seconds for 2,462 pairwise comparisons | one model inference per item |
| Agreement with human judgement | see §4 (AUC 0.812 / 0.705) | ALCE reports recall 85.1% / precision 77.6% |

**A reasonable combination (not measured in this paper; a design suggestion only)**:

> **The lexical layer as the first gate, the semantic layer as the second.**
> The first gate should have **high recall** (better to send too much to a human), at the cost of more
> false positives; the second filters before a human reads anything.
> The three-band structure of this paper fits that slot naturally: **only the "needs a human" band has
> to enter the second gate**, i.e. 62 of 308.

> **"This combination is better" cannot be said** — no such experiment was run here.
> Nor is "the lexical layer is cheaper" a virtue to claim — **the compute saved may be spent on human
> review instead** (see 7.3).

---

## 7.3 The "needs a human" band is not a fallback: it is a component that must be designed

This is the most easily overlooked part of the paper and, for practice, the most important.

**The "needs a human" band here holds 62 items (20.13%)** — that is, $7\le\mathrm{LCS}\le11$.

Intuitively many people would say "then loosen the threshold, judge fewer items as unsourced, and be
safer". **That intuition is wrong because it assumes the cost disappears.**

**The cost is merely shifted onto human review** (data from the per-$t$ table in §5.2):

| threshold | judged unsourced | **needs a human** | share needing a human |
|---|---:|---:|---:|
| $t=4$ (looser) | 120 | **149** | **48.4%** |
| $t=7$ (current) | 207 | **62** | 20.1% |
| $t=10$ (stricter) | 248 | **21** | 6.8% |

> **Lowering $t$ from 7 to 4 raises the human queue from 62 to 149 items — 87 more, a rise of 140%.**
> And §5.3 has already shown that at $t=4$ the false-negative rate on "judged unsourced" is still
> 15.40% (point estimate) with 16.42% false positives. **Compared with the current $t=7$** (26.57%
> false negatives, 3.00% false positives): **false negatives fall by 11.17 points and false positives
> rise by 13.42 points** — **one end improves, the other worsens. Taken together, the equal-weight
> expected error rate rises from 29.57% to 31.82%** ($15.40+16.42$), **so "it is not better" has an
> arithmetic basis, not a feeling**.
>
> **The definition of $E$ is given here so that a reader does not try to balance it as a simple average**:
> $E(t;w)=w\cdot\mathrm{ENR}(t)+\mathrm{EPR}(t)$ (§5.5). At $w=1$ it is **the two terms added, not
> divided by 2** — hence $E=26.57+3.00=29.57$ at $t=7$ and $E=15.40+16.42=31.82$ at $t=4$. (Halving
> each term would give 14.79→15.91; both conventions are internally correct, but **this paper uses the
> one without the division by 2**, consistently with §5.5.)
>
> **In other words: loosening the threshold buys not "more accuracy" but "shifting the error from one
> end to the other", while pushing more than twice the work onto people.**
>
> **"Better / not better" is a comparative judgement and cannot rest on the $t=4$ cell alone — a
> reference point must be given.**

**Why can this not be waved away with "someone will look at it anyway"?** Medical monitoring has a
mature research tradition on exactly this (alarm fatigue):

> The review by Cvach 2012 reports that the **overwhelming majority** of ICU monitor alarms require no
> clinical action, and that what follows is "alarm fatigue": people begin to ignore alarms
> systematically, **including the few that do require action**.

(Cvach, M. 2012, *Monitor alarm fatigue: an integrative review*, *BI&T* 46(4):268,
`10.2345/0899-8205-46.4.268`)

> **This paper draws no cross-domain inference**; it takes one design constraint from that literature:
> **whether the "needs a human" band is usable is decided by its absolute item count — the count, not
> the share.** 62 items and 149 items are two different systems under the same review capacity.

**So recommendation two in §7.1 has a further practical implication**: a report must give **the item
count in the human queue** as well, because that is the **only number that maps directly onto labour
cost**.

---

## 7.4 Three experiments the design already specifies: one done, two pending

§8 lists honestly what was not done. This section adds **how** — not a promise, but filling the hole
deep enough that someone else can pick it up.

| # | Experiment | Concrete design | What it resolves | Cost |
|---|---|---|---|---|
| **1** | **A second rater re-judges the existing 100 items** ✅ **done** | one new rater, the same criteria page, blinded to the first judgements, **no re-sampling and no re-stratification**; after locking, **report both raters' main figures side by side** — each one's main number, their difference, and the number of disagreements — **no averaging, no negotiation, no treating arbitration as ground truth** | the cross-rater sensitivity of §8.5 item 5 (**not** a variance decomposition; rater and pass remain inseparable) | completed: **100 items** × one pass |
| **2** | **Adversarial rewrites to measure EPR** | take 100 sentences **already judged sourced**, have a rewriter paraphrase under a "**maximise LCS**" strategy (ALCE's Top-1 passage is the extreme version), and measure $\mathrm{EPR}$ on that batch | turns the §8.6 statement "3.00% is a non-adversarial lower bound" **into a number** | 100 rewrites plus recomputation |
| **3** | **Familiarity effect** | the same sentences paired with two source documents, "one the rater has read" and "one they have not", comparing the share of A | §8.5.1 item 7 (a threat to **criterion validity**, not to reliability) | half a day of human time |

> **Why item 1 was designed as 100 and not 308**, for two reasons:
> ① **the bottleneck is measurement, not sampling** — re-judging another 208 items with the same person
> only computes unstable labels more precisely;
> ② **two raters each judging once still cannot separate rater effects from pass effects**, so spending
> 308×2 buys no variance decomposition;
> ③ one further point of convention: **if the target population is these 308 items, judging all of them
> is a census, and the within-pool sampling error is zero, not "smaller"**.
> → **Trading 100 items for "two people's own numbers" is worth more than trading 308 items for "one
> person's more precise number".**

> **It is done; the results are in §8.1.4**: rater 1 at 26.57% versus rater 2 at **31.39%** (point
> accounting, a difference of **4.82 points**); three-way disagreement **51%**, but **A/non-A agreement
> 74% (exactly the same as with the third rater)**.
> Toolchain: the judging page (100 items, source grouping order preserved, item numbers randomly
> permuted, progress key replaced; **the page itself is 1.6 MB and embeds the source texts, so it is not
> distributed with the paper**) / `S1·_复判_映射.json` (**not given to the rater**) /
> `S1·_两判者对照.py --判二 <result.json>`.
> One hard assertion in the self-test of that scoring script: **fed the first round's data it must
> reproduce 40.14 / 26.57 / 17.30** — it is not "a second formula I wrote", it is "the same one as
> §5.2.2".

> **What the three have in common**: none needs new data collection or a new model; each needs **a
> little more human time**.
> **But their completion status differs**: item 1 **is done** (results in §8.1.4) and what is
> described above is its design; items 2 and 3 **remain pending**. Writing all three as "pending" would
> let a reader mistake a cost already paid for a future cost.
> They are also **the only three limitations in this paper that come with a known fix** — the rest of
> §8 is either "conditional on resources" or "cannot be repaired in principle".

---

## 7.5 A larger question: should the threshold be stratified by source document?

This paper uses **one** $t$ throughout. But a footnote in §5.7 already flags a possible problem:

> The signal-to-noise ratio of $\mathrm{LCS}$ differs across source documents — in long source documents, and
> in terminology-dense ones, even an unrelated sentence can easily produce a long common substring.
> **A uniform threshold $t=7$ is therefore equivalent to "being systematically stricter on low-SNR
> sources".**

If that holds, the right move is not to tune $t$ but to **set thresholds per source stratum** (or
normalise by source length).

**This paper did not do it**, for two reasons:

1. **Not enough data**: 46 source documents and 308 sentences to check; after stratification each
   stratum holds single digits to the low teens in items, too few to estimate a within-stratum threshold;
2. **It turns the problem from one dimension into many**: once stratified, the tidy single-parameter
   conclusion "$t$ implies $w$" becomes "each stratum has its own $w$", and the whole cost analysis has
   to be redone.

> So this is listed as **a direction, not a conclusion**: **"a uniform threshold" is itself an
> unwritten assumption** — it assumes that the same LCS value means the same evidential strength across
> source documents. **That assumption was not tested here.**

---

## 7.6 Three things this chapter does not discuss (to avoid overreach)

| Not discussed | Why |
|---|---|
| **Whether the decision rule should be replaced wholesale by a semantic model** | That is a system design decision, not an audit question. This paper only notes that, after such a replacement, "invert $w^*$" and "read as $\alpha$" become impossible (7.2) |
| **How human review should be organised** (who judges, for how long, how to spot-check) | A management question. This paper contributes one input only: the item count in the human queue is the key quantity |
| **Liability for a wrong judgement** | A legal and ethical question, outside the technical scope. This paper provides only a recomputable record |

---

## 7.7 Summary of this chapter (three sentences)

1. **The value of this paper is not in a "discovery" but in "supplying coordinates one can argue
   with"** — a cost ratio, a significance level, and the interval spanned by the three accountings.
2. **The most useful practical change is three lines of documentation**: write $(t,w)$, state how
   "cannot tell" is counted, write $\alpha$. **All three are cheap, and they decide whether the whole
   thing can be audited at all.**
3. **The main thing to guard against is not "choosing the wrong threshold" but "shifting the cost onto human
   review and believing the problem is solved"** — lowering $t$ from 7 to 4 raises the human queue by
   140% and does not make the automatic decision any better.

---

*Every figure in this chapter traces back to §5 (the per-$t$ table, $w^*$, $\alpha$) and §6;
citations in §2.1–§2.5.*

---

# 8 Limitations

> The rule for this chapter: **every item must point back to a specific section of the chapter it
> belongs to**; boilerplate such as "this study has certain limitations" is not allowed.
> **Ordering principle: by "how easily a reviewer can dismantle it in one sentence", most fatal
> first.**

---

## 8.0 Overview in one table

The conclusions of this paper rest on **five independent foundations**; each has a hole, and **the
holes differ in position and in depth**:

| # | Foundation | Hole | How deep | Detail |
|---|---|---|---|---|
| 1 | The 100 human-judged items | **A judgements agree across raters (74%, proper containment), but the B/C boundary follows personal habit**; the drift within one person is of the same magnitude as the drift from changing raters | **Most fatal** — all three ENR numbers come from here | 8.1 / 8.1.1 / 8.1.2 |
| 2 | All analyses are static | **Once the threshold is public the text will drift; both curves move and the direction is unclear**; $\mathrm{EPR}=3.00\%$ is a **non-adversarial lower bound** | Deep — it undermines both $\mathrm{EPR}$ and the inverted $w^*$ | 8.6 |
| 3 | The cost basis for $t=7$ | It holds only at the scale of a **single $\alpha$** and **fails family-wise error control** | Deep — under a stricter standard, $t\ge9$ | 8.2 |
| 4 | The three-band structure (has a source / needs a human / no source) | It has the Fellegi–Sunter skeleton **but does not inherit its optimality guarantee** | Medium — the increment holds, optimality does not | 8.3 |
| 5 | Replacing the threshold sweep with $J$ | Only **information loss** is proven, not the **direction of bias** | Medium — narrow but defensible | 8.4 |

> **Item 1 was established in two steps**: first "the ruler itself is moving" was found (8.1.1),
> and only after a second rater was brought in could it be located as **the B/C boundary moving, not
> A** (8.1.2). **The conclusion cuts both ways**: **the bad part** is that two of the three bands are largely
> void; **the good part** is that
> **the numerator of the main figure happens to sit on the only stable band (A)**.
>
> **Item 2 came out of an economics literature check**: it was originally filed under "not enough
> data", but it is really **methodological** — any public deterministic decision rule induces text to
> move toward the rule's cheap side. See 8.6, plus §2.3.1 (burden of proof in law and economics) and
> §2.3.3 (the revealed-cost path).

---

## 8.1 Reliability of the human judgements: of the three accountings, the C band has no stable criterion

**This is the first thing this paper should say.** The three core numbers of §5 (40.14 / 26.57 / 17.30)
**all come from one person's single pass**, so we measured how hard they are.

**Method**: 50 items drawn at random from the first round of 100, **re-judged blind** (the page showed
no previous answer, item numbers were re-assigned, order was shuffled, and the source presentation was
identical to the first round). **All three cross-checks passed**: R→S→original answer had 0 errors and
the sentences matched 50/50 character for character — **the numbers are real; the package was not
corrupted.**

**Results**:

| Statistic | Value | Reading |
|---|---|---|
| Overall agreement | 54% ($\kappa=0.32$) | **fair, not substantial** |
| **A (findable) agreement** | **80%** | relatively reliable |
| **B (not findable) agreement** | **53%** | half reproduce |
| **C (cannot tell) agreement** | **33%** | **two thirds change their mind** |

Confusion matrix (rows = pass 1, columns = pass 2):

| pass 1 ＼ pass 2 | A | B | C |
|---|---:|---:|---:|
| A | **12** | 3 | 0 |
| B | 7 | **9** | 1 |
| C | 8 | 4 | **6** |

**Key structure: the flow out of C is essentially one-directional.** Pass 1 had 18 C items; by pass 2
**only 6 remain** (8 moved to A, 4 to B); **the 7 C items in pass 2 = those 6 plus 1 arriving from B**,
and **0 came from A in pass 1**.

→ **What is missing could be the criterion for when to choose C, or it could be rater skill — the flow
data cannot separate the two.**

> **It cannot be written directly as "what is missing is the criterion, not the rater's ability"**:
> the pattern "C flows one way, no A→C" **looks the same** under "the criterion is unclear" and under
> "the rater cannot tell the difference". What actually separates them is **the change-of-rater data in
> §8.1.2** (after the change, A judgements show proper containment with 0 in the reverse direction);
> this section was written before that evidence.

**The boundary of this limitation (stated clearly, without over-negating)**:

- **It does not overturn the main figure.** The upper-bound accounting (C counted as found) and the
  lower-bound one (C counted as not found) both have to assign C a value; **the point accounting
  (C dropped from the denominator) assigns C nothing** — so the main figure **is not swayed by the
  choice of "which band does C belong to"**. That is the **structural** reason the main figure uses the
  point accounting, not merely that it is "the middle one".
  > **But that does not mean "the point accounting resists drift best"**: across passes, its drift
  > (33.1 points) is **larger** than the upper-bound accounting's (14.0 points) — because a $C\to A$ or
  > $C\to B$ re-judgement still changes its numerator or denominator. **What the point accounting buys
  > is "no value has to be chosen for C", not "it stays put when C is unstable".**
- **But it changes how the span is read.** The two ends of 17.30–40.14 are **not "two reasonable
  accountings"**; they measure **how far things can drift when the criterion is unclear**. Of the
  23-point span, a large part comes from a band with 33% agreement.
- **Dropping C is not clean either**: 10 items cross between A and B, and the two-way agreement rate is
  only 67.7%.

** Two sentences that must not be written**:
① "the rater's single-pass error rate is 35.9%" — that conversion $p=(1-\sqrt{1-2q})/2$ assumes the two
judgements err **independently at random**, whereas the observed errors are **systematically
one-directional**; the premise fails;
② "so the human judgement is untrustworthy and the ENR conclusion is void" — over-reach, see the
boundary above.

**Two things not done (a real gap, not a disclaimer)**:

| Item | Cost | What it would fix |
|---|---|---|
| Write the C criterion as hard rules, then re-judge the 50 items | about 20 minutes | **the only way to actually fix C** |
| Bring in an independent rater for inter-rater agreement | needs another person | the only way to answer the "single rater" criticism |

> Detail in §4.6.6, `S2` §2.7.9, `S1·_重判_核分_20261108.md`.

### 8.1.1 The first of those was done, and the result matters more than not doing it: **the drift axis is at least as large as the accounting axis**

The first row of the "not done" table above was later done — hard criteria were written
(`S1·_判据_C档.md`) and the **same 50 items** were judged a third time. **There are three findings, each
worse than the last.**

**① The criteria did not meet the bar.** Of the three pre-committed standards, only one passed: no C
items appeared in this pass (required ≤10/50). A/B two-way agreement came in at **79.1%** against a
required 80%, and C-agreement could not be evaluated at all, there being no C to agree on. Worse, the
criteria **committed the error they forbade**: of the 16 B items in pass 2, **8 (50%) were absorbed into
A** — the "a synonymous paraphrase counts as found" clause over-applied, which is exactly the
"anything adjacent counts as A" that they explicitly banned. The full table is in §4.6.7.

**② The share of A rises monotonically over three passes (30% → 54% → 80%) while C falls
monotonically (36% → 14% → 0%).**

**This pattern has two opposite explanations, and the data here cannot separate them:**

| Explanation | What it says | If true, which way the main figure moves |
|---|---|---|
| Correcting over-strictness | many of those C items really were A, and the criteria put them back | ENR is **under**estimated |
| Criterion drift | the criteria licensed "a paraphrase counts", and the rater got looser as they went | ENR is **over**estimated |

> **There are two reasons they cannot be separated, both on this paper's side:**
> 1. **No external gold standard** — there is no index of true provenance independent of the rater;
> 2. **No control group** — **no criteria were added** between pass 1 and pass 2, yet A rose from 30%
>    to 54% (**+24 points**). So merely judging the same sentences again moved the standard. Of the
>    further 26-point rise in the third pass, how much is the criteria and how much is repetition
>    **cannot be separated**.
> **It cannot be said that "only one variable changed"** — this is a pre/post measurement, not a
>    controlled intervention.

**③ The two axes can only be compared on the same 50 items — and they are of the same magnitude.**

| Axis | Span in points on these 50 items |
|---|---:|
| drift across passes (same accounting, three passes) | upper bound 14.0 / point 33.1 / **lower bound 50.0** |
| span across accountings (same pass, three bands) | pass 1 36.0 / pass 2 14.0 / pass 3 0.0 |

**Largest drift 50.0 ≥ largest accounting span 36.0 — same magnitude, drift slightly larger.**

**This means the interval reported in §5.2.2 answers only "how should C be counted"; it cannot measure
the axis "the rater moves even on whether something counts as A at all".**

> **One bookkeeping discipline:**
> both numbers in that table are computed on **50 items**, so they may be compared.
> But the "**23 points**" used in the main text (§5.2.2) is the upper/lower gap on the **308-item
> stratified extrapolation** — **different base, different meaning, not subtractable**.
> **So writing "drift 50 points > the 23 points in the main text" is forbidden.**

**Disposition: the main figure does not change, but the error structure must be reported separately,
with bases labelled.**

The main figure of 26.57% still uses **Rater 1's set** (the stratified extrapolation from 100 items,
§4.6.5). There is exactly one reason, and it is observable: **that set was completed before the criteria
were introduced**, so it is not comparable with the standard of the later two passes — not because it is
more "correct". After the criteria were added in the third pass, the use of $A$ rose markedly (8 items
moved $B\to A$); **whether that rise is due to the criteria or to repetition, that version of the
standard is not the same ruler as the first two passes.** How much each contributes cannot be separated
here (§4.6.6).

| Component | Value | Base | Source |
|---|---:|---|---|
| sampling error | ±6.65 points | 308 items, stratified extrapolation | half-width of the 95% interval |
| accounting span (how C is counted) | 23 points | **308 items** | upper 40.14 − lower 17.30 |
| *(same-base reference)* | *36 points* | *50 items* | *three-band span within pass 1* |
| **rater drift** | **≤50 points, direction unknown** | **50 items** | lower-bound accounting across three passes, 30%→80% |

> **The three cannot be added** (different sources), but **must be reported separately with bases
> labelled**.
> **In particular: writing "26.57% ±6.65 points" suggests a total uncertainty only 13.3 points wide —
> the true figure is far larger.**
> **What can be said**: **on the same 50 items, drift and accounting span are the same magnitude** —
> the axis "the rater moves even on whether something counts as A" **is at least as large as "how C is
> counted"**, and the interval in the main text covers only the latter.
> **What cannot be said**: adding 50 points to 26.57 (**drift on the 308 items was not measured**;
> only 50 items were).
> So the conclusion is: **the core numbers of this paper can be read as magnitudes, not as precise
> values.** This is the hardest limitation in the paper.

> Detail in §4.6.7, `S2` §2.7.10, `S1·_三轮判断_20261108.md`, `S1·_三轮漂移.py` (7 self-test
> assertions).
> **The same person was not asked for a fourth pass**: the share of $A$ across three passes is
> 30%→54%→80%, **rising every time**. This paper has no group that separates "repetition" from
> "criteria", so **the rise cannot be attributed to either cause, nor can a fourth pass be predicted.**
> What can be said: **judging the same sentences repeatedly moves the standard** — which is itself the
> reason a single rater cannot serve as a gold standard.
> **But "changing the rater" is a different matter, and it was done**: a fourth person re-judged all
> 100 items, see **§8.1.4** — A/non-A reproduced at 74%, three-way disagreement 51%, a 4.82-point gap in
> the main figure. **Two measurements of the same dimension; neither is "one more pass".**

The "drift direction cannot be determined" of 8.1.1 came from **one person** judging three times, which
cannot separate "this person is moving" from "the task itself is undecidable". So **a second rater** was
brought in, using **the same criteria page** on **the same 50 items**. That separated them.

### 8.1.2 After bringing in a second rater: the ambiguity is **not "is it A", but "B or C"**

**Four-way distribution (same 50 items):**

| Pass | A | B | C | A share | C share |
|---|---:|---:|---:|---:|---:|
| 1 (Rater 1) | 15 | 17 | 18 | 30% | 36% |
| 2 (Rater 1, blind re-judge) | 27 | 16 | 7 | 54% | 14% |
| 3 (Rater 1 + criteria) | 40 | 10 | **0** | 80% | **0%** |
| **Rater 3 (+ same criteria)** | 27 | 7 | **16** | 54% | **32%** |

> Under the same criteria, **Rater 1 used 0 C while Rater 3 used 16 C (32%)** →
> **"the criteria closed off C" does not hold; it closed it off only in Rater 1's pass.**

**① Three pairings, each differing in exactly one factor:**

| Pairing | What differs | three-way agreement | κ | **A vs non-A agreement** |
|---|---|---|---:|---:|
| pass 1 vs pass 2 | **pure retest** (same person, no criteria) | 54% | 0.322 | **64%** |
| pass 2 vs pass 3 | retest + criteria | 68% | 0.365 | 70% |
| pass 3 vs Rater 3 | **pure change of rater** (different person, same criteria) | 60% | 0.259 | **74%** |

**② On A vs non-A the three pairings give 64% / 70% / 74% — but this is not a one-factor ladder.**

> **The sentence "changing the rater (74%) agrees better than retesting the same person (64%)"
> cannot be put that way**:
>
> | Pairing | What changed | A/non-A |
> |---|---|---:|
> | pass 1 vs pass 2 | same person, **no criteria** | 64% |
> | pass 2 vs pass 3 | same person, **criteria added** | **70%** |
> | pass 3 vs Rater 3 | **different person**, with criteria | **74%** |
>
> **Between 74% and 64% both "person" and "criteria" changed at once**, so those 10 points cannot be
> credited to the change of rater. **And none of the three pairings is "same person, same criteria,
> different occasion"** — so a same-criteria same-person baseline is not available here; the 70% row is
> the "criteria added" pairing, not a baseline.
> What can be said: **the A/non-A agreement of all three judgements lies between 64% and 74%, and none of
> them drags it down**; but **each of the two increments mixes in something else** (70% mixes in the
> criteria, 74% mixes in the change of rater), so **the separate contribution of "changing the rater"
> cannot be isolated**.
> → **Agreement did not fall after the change of rater (74%), but that does not support "it is more
> stable"** — the 4-point gap over the 70% row lies below the pass-to-pass noise of §8.1.5. **What can
> be said is "it was not dragged down", not "it is better".**

**③ The criteria improve "self-consistency", not "cross-rater consistency".**
**③ Of the 14 points the criteria added, part is drift from repetition itself — the whole amount cannot
be credited to the criteria.**

> 54% → 68% (same person, criteria added) looks like an improvement, but **this pairing changes both
> "retest" and "criteria"**, and §8.1.1 itself measured that **"pass 1 → pass 2" alone moved A from 30%
> to 54% (+24 points)**.
> So of the 14 points, **how much is the criteria and how much is repetition cannot be separated here**
> (the same section already conceded the missing control group).
> What can be said: **self-consistency is higher with the criteria (68% > 54%), but the attribution is
> not unique.**
> With a different person it is only 60% and **κ falls from 0.322 to 0.259** — that finding is not
> affected by the confound, and **the judgement "the criteria were internalised individually, not
> executed in common" still holds.**
> → **Each person internalised the criteria in their own way; they were not executed jointly.**

**④ The most valuable structure: A judgements form a proper containment, not random disagreement.**

| pass 3 ＼ Rater 3 | A | non-A | total |
|---|---:|---:|---:|
| **A** | **27** | 13 | 40 |
| **non-A** | **0** | 10 | 10 |

**Zero in the reverse direction — Rater 3's A set is a proper subset of Rater 1's third-pass A set.**
**Random error would produce mistakes in both directions; a difference in strictness produces only one.
Only one direction is observed → this looks more like a difference in strictness than random error.**

> **But this must not be read as "so nobody judged wrongly"**:
> ① one direction rules out **random error**, not **systematic offset** (two people each biased in one
> direction also produce one direction);
> ② **it cannot tell who is stricter** — "Rater 1 is looser" and "Rater 3 is stricter" **look identical**
> in a containment relation;
> ③ it also **cannot rule out both being biased** (just by different amounts).
> So this section can say "**the difference is in strictness — it does not look like random noise**"; it **cannot say "who
> is right"** — that needs an external gold standard, which this paper does not have (§8.1.1 ②).
> → It **can be acted on**, though: taking the intersection gives **27 "two-rater-confirmed A" items**
> (54% of 50).

**⑤ So this section walks back three overstated sentences:**

| Over-reading | The accurate statement here |
|---|---|
| "the human judgement data is untrustworthy" | **only the B/C boundary is; A is usable, and it can be reinforced** |
| "the rater moves even on whether something is A" | **"A judgements agree across raters (74%, proper containment); the drift happens mainly within one person (retest effect)"** |
| "the criteria closed off C" | **"only in Rater 1's pass; under the same criteria another person still used 32% C"** |

> **One number is enough to show how unreliable the B/C boundary is**: of the 10 B items in pass 3,
> **7 were judged C by Rater 3 and 0 were judged A**.
> In other words: **both agree "not A"; whether it is B or C is almost personal habit.**

**⑥ This also gives the main figure (point accounting) its third and hardest reinforcement:**

| Accounting | Definition | Dependence on A | Dependence on C |
|---|---|---|---|
| upper bound | $(A+C)/n$ | first order | **C enters the numerator directly** |
| **point** | $A/(A+B)$ | **first order (the numerator is A only)** | **C does not enter the numerator and is not in the denominator; but a C re-judged as A or B changes them** |
| lower bound | $A/n$ | first order | **C counted as "not A"** ($n$ is fixed at 50, so $B\leftrightarrow C$ swaps **do not** change $A/n$) |

Measured: **agreement on the A layer (74%) is higher than within B/C (46.8%)**, A shows one-directional
containment, and **C is the least stable** (0 versus 16 under the same criteria).

→ **The claim "the point accounting is the only one unaffected by C" must be withdrawn**: $A/(A+B)$
does contain no $C$, but **a $C\to A$ or $C\to B$ re-judgement changes the numerator or the denominator**.
And in the lower-bound $A/n$, $n$ is fixed, so **$B\leftrightarrow C$ swaps do not even move the
denominator** — on sensitivity to C re-judgements alone, the lower bound is no worse than the point
accounting.

> **The real reason this paper chooses the point accounting is something else**: the denominator of
> $\mathrm{ENR}$ **moves together with A**.
> Two people judging the same 100 items: **their A usage differs by 12 items** (Rater 1 uses 34, Rater 2
> uses 46; Rater 1 is stricter) and **their C by 20 items** (32 versus 12).
> **Under the lower-bound $A/n$**, A's systematic offset **enters the result in full** ($n$ is fixed at
> 100);
> **under the point $A/(A+B)$**, **the denominator $A+B$ moves with A, so the offset is partly offset**.
> → This is the mechanism behind the measured "the lower-bound ENR drifts 10.03 points, the point
> accounting only 4.82" in §8.1.4 (those two numbers are **after stratified extrapolation**, not the
> raw proportions in this batch).
> **It is not "the point accounting avoids C"; it is "its denominator moves along with it".**

**The main figure uses the point accounting, and the reason changes from "it is the middle one" to "it
lands on the most stable dichotomy (A vs non-A)".**

> **"Most stable" is not "unbiased"**: the point accounting dodges the axis **"how should C be
> counted"** (it assigns C no value); but **its numerator is A**, and §8.1.1 itself measured that **the
> share of A drifts from 30% to 80%** — **that axis is not dodged at all.**
> **It must not be called "the only reliable dichotomy"**: that would say the point accounting has
> passed every known error axis, whereas **the largest one was stepped around.**

**⑦ One more thing that could be done, and was later done (results in §8.1.4):**
**measuring cross-rater differences from two-rater data** is the only entry point for shrinking the
rater component. This section did 50 items only; **all 100 items were later re-judged by a different
person** (§8.1.4) — **not 308**, because the bottleneck is measurement, not sampling (item 1 of the
table in §7.4).

> Detail in §4.6.8, `S2` §2.7.11, `S1·_四方判断_20261108.md`, `S1·_四方对照.py` (7 self-tests).
> **The criteria road has run out; the change-of-rater road has not**: the criteria raise
> self-consistency (54%→68%), but cross-rater consistency cannot be raised by criteria (a different
> person gives 60%, and κ falls from 0.322 to 0.259). So the next move is not "one more round of
> criteria" but **"a new person, the full set"** — which is §8.1.4.

### 8.1.3 Turning "ambiguity" from a sentence into a distribution: 50 items × 4 passes

The previous section could only say "the B/C boundary follows personal habit". With 50 items × 4
judgements in hand, something harder can be computed: **how many of the 4 passes judged each sentence A**.

| number of passes judged A | sentences | reading |
|---:|---:|---|
| 4 | **11** | unambiguously "findable" |
| 3 | 15 | three A, one non-A |
| **2** | **5** | **split down the middle, most ambiguous** |
| 1 | 10 | one A, three non-A |
| 0 | 9 | not A in any pass — **but B and C may still be moving** |

- **"Is it A" agrees across all four passes for 20 items (40%)** ($11+9$); 30 items (60%) disagree.
  > **This is not the same as "all four three-way judgements agree"**: $11+9$ only says "A all four
  > times" or "not A all four times"; the latter may still be B, C, or a mix — **the number of items
  > with full three-way agreement was not computed** (that would require comparing B and C item by
  > item, whereas the classification here is by number of A judgements).
- **The range across passes is 25 items (50 percentage points)**: the loosest pass (the third) judged 40
  items A, the strictest (the first) judged 15.
- **Cochran's $Q=39.51$** (df=3, $\chi^2_{0.05}(3)=7.81$).

> The A column totals check out against §8.1.2: $4\times11+3\times15+2\times5+1\times10+0\times9=109$,
> which matches $15+27+40+27=109$ in the four-way table — **the two tables agree, item for item.**
> Recompute with `S1·_核8.1.3分布.py`.

> **$Q$ is descriptive only, not a test.** Cochran's $Q$ requires the "treatments" to be independent,
> but **3 of the 4 passes are the same person**; three judgements by one person are not independent
> (memory, familiarity and the previous answer all matter), so **there is no valid null distribution**.

**What this set of numbers changes:**

1. **"Ambiguity" now has a denominator** — 60% of sentences disagree on "is it A", so this is not "a few
   borderline cases";
2. **"Two-rater-confirmed A" can be graded in three levels** — stable across four passes (11 items),
   across three (15), or split two-two (5). Weighting by stability when citing is more honest than a
   flat cut;
3. **But pass effects and rater effects are now mixed together** — see the next item.

> **Why this cannot become a variance decomposition**: the standard tools in educational measurement
> for separating "rater severity" from "item difficulty" are **G-theory / many-facet Rasch**, and they
> need **multiple raters**. This paper has **2 raters** (Rater 1 and Rater 3), and 3 of the 4 passes are
> one person — **"rater" and "pass" are confounded, so no rater variance component can be estimated**.
> Hence the above is a **descriptive distribution**, not a variance decomposition. See §8.5 item 5.
>
> There is one place where a critic's own objection can be used in defence, and it belongs here:
> some would suggest "when in doubt, apply the precautionary principle". But **the most famous criticism
> of the precautionary principle is precisely that it is inoperable** — Sunstein, *Laws of Fear* (2005),
> argues that it is self-contradictory and leads to paralysis.
> **Precisely because it yields no executable rule on paper, the handling of C must be given an explicit
> cost argument** rather than waved through with "better safe than sorry". **The three accountings of
> this paper (upper bound / point / lower bound) are exactly that attempt at an explicit argument.**

> Detail in §4.6.8, `S2` §2.7.11 and `S2` §2.7.13, `S1·_一致度分解.py` (7 self-test assertions).

---

### 8.1.4 A fourth person: all 100 items, a different rater — **A is still the same A**

§8.1.2 used 50 items; §8.1.3 was still the same person. Both leave the same question unanswered:
**if a completely different person judges all 100 items, how far does the main figure move?**

That was done: **1 fully new rater × the existing 100 items × 1 pass** (nobody who had judged these 100
took part). The page had **exactly the same presentation as the first round** (auto-scroll plus
highlight, no full text), item numbers were **randomly permuted** (0 items in the same position), and
the progress key was replaced — otherwise opening it on the same machine would surface the first round's
answers.

> *Labels follow the tables throughout: **Rater 1** judged three passes of the 50-item set; **Rater 3**
> was the second person to judge (50 items, with the criteria page); **Rater 2** was the fourth person to
> judge (all 100 items, without the criteria page).*

**① Category usage: C usage differs by 20 items**

| | A findable | B not findable | C cannot tell |
|---|---:|---:|---:|
| Rater 1 | **34** | **34** | **32** |
| **Rater 2 (new)** | **46** | **42** | **12** |

**② Three-accounting main figures: reported side by side, not averaged, each with its own interval**

| Accounting | How C is counted | Rater 1 | Rater 2 | Difference |
|---|---|---:|---:|---:|
| upper bound | C counted as found | 40.14% [32.61, 47.67] | **36.51% [28.94, 44.07]** | −3.64 |
| **point (main)** | **C dropped** | **26.57% [19.93, 33.22]** | **31.39% [23.98, 38.80]** | **+4.82** |
| lower bound | C counted as not found | 17.30% [11.34, 23.26] | **27.32% [20.12, 34.53]** | **+10.03** |

> **Both intervals overlap, but overlap is not agreement.** Overlap says "on this data the two
> intervals intersect"; agreement is a matter of **item-by-item coincidence rates** (49% three-way, 74%
> A/non-A).
> The two intervals are **not stacked and not pooled**: they are not two samples from one population,
> but one set each from two raters.
> **Interval width is decided by $p$, not by the accounting**: the binomial variance is $p(1-p)/n$, so
> **the upper bound (40.14%, closest to 0.5) has the widest interval and the lower bound (17.30%,
> furthest) the narrowest**. Measured: Rater 1 gives **upper 15.06 points / point 13.29 / lower 11.92**;
> Rater 2 gives **upper 15.13 / point 14.82 / lower 14.41** — **for both people the upper-bound interval
> is wider than the point one.**
> **Interval width has no fixed relation to the accounting**: measured, both people's upper-bound
> interval is wider than their point one. The point accounting does have its own problem ($L_1$ drops
> from 18 items to 9, the denominator is small, the bootstrap distribution is skewed, and bootstrap
> differs from the analytic interval by 2.84 points, see §8.1.4 ①), **but that affects "whether the two
> methods agree", not "how wide the interval is"** — two different things.

> **The main figure stays at 26.57% (Rater 1); no averaging of the two, no negotiation.**
> The reason is not "Rater 1 is more authoritative" but that this is the judgement used inside the
> system, and replacing it would require **recomputing the cost ratio, the nine-cell table and the FDR
> in §5 in full**; and reporting the two numbers side by side already conveys the instability.
> **The two numbers are a range, not a measurement waiting to be averaged.**

**②b A fourth way to define A: the two-rater intersection (possible only because both judged the full set)**

Since both judged all 100 items, a different rule for "counting A" becomes available: **not "either one
says A" but "both say A".**

| | upper | point | lower | **intersection** |
|---|---:|---:|---:|---:|
| ENR(7) | 40.14% | 26.57% | 17.30% | **13.79%** |
| 95% interval | [32.61, 47.67] | [19.93, 33.22] | [11.34, 23.26] | **[7.94, 19.65]** |
| implied wrongly-rejected items | 124 | 82 | 53 | **42** |

**If the main figure were switched to the intersection, the consequences are concrete**
(`S1·_交集口径.py`, 12 self-tests):

| Item | Now (point, 26.57%) | Switched to intersection (13.79%) |
|---|---|---|
| cost ratio implied by $t=7$ (EPR fully counted) | **1.16–1.41** | **0.60–0.73** |
| equal-weight optimal $t$ | **5** | **8** |
| EPR, "format only" band | $t=7$ missing one cell | $t=7$ **never optimal** |

> **The main figure was not switched, for three reasons, none of them laziness:**
> ① **a lower intersection is not closer to the truth** — it only raises the bar for A; the two
> accountings **cannot be ranked**, only reported side by side (the fourth row of the table in §5.2.2);
> ② **switching would require recomputing all of §5**: the cost ratio, the nine-cell table, the FDR and
> every conclusion are tied to 26.57;
> ③ more fundamentally: **which accounting to choose is the auditor's decision** (§9.3). This paper's job
> is to compute the number for each accounting, not to choose on their behalf.
>
> **One fact that runs the other way has to be stated here**: the intersection interval [7.94, 19.65]
> is **11.7 points wide**, while the point accounting's is **13.29 points** — **the intersection is in
> fact 1.58 points narrower.** **On the metric "interval width", the intersection is more stable than
> the point accounting.** **This is something this paper should concede: switching would not be without
> statistical benefit; those benefits are simply outweighed by ①②③.** Hiding the reason would be more
> dangerous than admitting it — having admitted it, the reader sees that the authors know what they gave
> up.

**③ Confusion matrix (rows = Rater 1, columns = Rater 2)**

|  | Rater 2 A | Rater 2 B | Rater 2 C | total |
|---|---:|---:|---:|---:|
| **Rater 1 A** | **27** | 5 | 2 | 34 |
| **Rater 1 B** | 8 | **19** | 7 | 34 |
| **Rater 1 C** | 11 | 18 | **3** | 32 |
| total | 46 | 42 | 12 | 100 |

**④ 74% appears a second time — this time from another person, on all 100 items**

| Pairing | What differs | three-way agreement | **A vs non-A** |
|---|---|---:|---:|
| pass 1 vs pass 2 (§4.6.6) | same person, pure retest | 54% | 64% |
| pass 3 vs Rater 3 (§8.1.2) | different person, same criteria, **50 items** | 60% | **74%** |
| **pass 1 vs Rater 2 (this section)** | **different person, same page (no criteria), all 100 items** | **49%** | **74%** |

> **"Same criteria" does not apply here**: the hard criteria were introduced only in the third pass,
> whereas Rater 2 used **the first round's page** (auto-scroll plus highlight, no criteria page). So the
> only factor changed this time is **the person**; what was controlled is **page presentation**, and
> **the criteria were not controlled**.
> **That makes this row more worth reporting, not less**: the 74% in §8.1.2 came from Rater 3 **with
> criteria**, on 50 items; the 74% here came from Rater 2 **without criteria**, on all 100. **Two
> different pairings under different criteria conditions land on the same 74%** — this is not one
> experiment repeated, it is **the same agreement obtained once under each of two conditions**, which
> carries much more weight than a single repetition.

> **Two cross-rater pairings, A/non-A agreement 74% both times.** One on 50 items, one on all 100; two
> different people.
> **74% is an aggregate agreement rate, not an item-level conclusion**: it says **"with a different
> person, the overall grip on 'is/is not A' is about the same"**, and **must not be read as "an
> independent rater would produce the same set of A"** — three-way coincidence is only **49%**, so item
> by item half of them differ.
> The strongest defensible sentence is: **this layer's aggregate proportion lands at 74% in two
> independent pairings, so it is not one person's accident.** As to "any new rater would land near 74%"
> — **two samples cannot support that generalisation**, and this paper does not make it.
> What can be said: **the "is it A" layer has a stable core; bring an independent person in and it is
> still the same set.** What cannot: that 74% is a constant for this kind of task — it appears only in
> these three pairings.

**⑤ But three-way disagreement is 51%, higher than same-person retesting — this must not be glossed over**

- Only **49 items (49%)** coincide three-way; disagreement is **51%**, **higher** than the 46% of
  same-person retesting (§4.6.6);
- Of the 47 items both judged non-A, **only 46.8% coincide on B/C** (22/47);
- Of the 32 items Rater 1 called C, Rater 2 accepts only **3** (11 judged A, 18 judged B).

> **The honest reading**: **the uncertainty introduced by changing the rater is no smaller than by
> re-judging on another day.** The A layer is stable; inside non-A (B versus C) it is almost personal
> habit — and **the two people each have their own habit, rather than one being loose and the other
> strict**: in §8.1.2 Rater 3 used 32% C, here Rater 2 used only 12%. **C usage is a personal style, and
> its direction is not fixed.**

**⑥ This round measured "why the main figure uses the point accounting"**

With C usage differing by 20 items (32 versus 12):
- **the lower bound drifts 10.03 points** — because it assigns C to "not findable", so more C depresses it;
- **the point accounting drifts only 4.82 points** — because it drops C from the denominator and assigns C
  no value;
- the upper bound drifts 3.64 points — because assigning C to "findable" happens to cancel against "Rater
  1 uses more C, Rater 2 uses more A".

> So what can be said: **across raters, the upper and point accountings differ by 3.6–4.8 points and
> the lower bound by 10; the whole difference comes from "whether C has to be assigned a value".**
> **"Only the point accounting is stable" cannot be said** — on both axes the smallest is the upper
> bound; the point accounting's structural advantage is not "minimum drift" but "no value has to be
> chosen for C". Its advantage is that **the reason it is stable is structural, not contrived.**

**⑦ What this round did not achieve, stated plainly**

| Not done | Why |
|---|---|
| **Intervals** for Rater 2's three accountings | only the three point estimates were reported, no bootstrap was run. **No pretence of having them** |
| **Separating** rater effects from pass effects | a single-pass design cannot. Doing so needs multiple crossed passes by each of two raters (§8.5 item 5) |
| Switching the main figure to the "two-rater intersection" | the intersection contains only two-rater-confirmed A (27 items here), which would push the main figure lower and change the accounting used throughout. **Not done, and not intended** |

> Generated and checked with:
> `S1·_两判者对照.py --判二 S1·_人判_结果_第二名判者_20261108.json` (23 self-test assertions, one of them
> the hard assertion "fed the first round's data it must reproduce 40.14 / 26.57 / 17.30"); intervals
> with `S1·_区间_两判者.py` (22 self-test assertions).
> Results stored separately: `S1·_人判_结果_第二名判者_20261108.json`; mapping `S1·_复判_映射.json`.

---

### 8.1.5 Rater effects and pass effects: how far can they be separated

§8.5 item 5 has always said "a multi-rater variance decomposition cannot be done". **That sentence is
correct, but "cannot be done" does not mean "do nothing".** This section separates the part that can be
separated and explains why the rest cannot.

**Data**: five passes × the same item set (those 50 items, n=50). Rater 1 accounts for three passes, and
two other people one each.

| Pass | Who | A share |
|---|---|---:|
| 1 (first) | Rater 1 | **30.0%** |
| 2 (blind re-judge) | Rater 1 | **54.0%** |
| 3 (criteria added) | Rater 1 | **80.0%** |
| 4 | Rater 3 | **54.0%** |
| 5 | Rater 2 | **46.0%** |

> One look at this table is enough: **Rater 1 alone moves from 30% to 80% across three passes (a
> 50-point within-person span)**, while **the six cross-rater pairings differ by between 0 and 34
> points** — the smallest pair is identical (Rater 1's second pass 54.0% versus Rater 3's 54.0%) and the
> largest is 34 points (Rater 1's third pass 80% versus Rater 2's 46%).
> **Within-person 50 points > largest cross-rater 34 points**, and that direction is right; but **it
> cannot be written as "cross-rater is only 24 points"** (24 is just the 54−30 pairing, not the
> cross-rater maximum).
>
> **In other words: the same sentence can differ by 50 percentage points between passes, while changing
> the person differs by at most 34 — but changing the person is not free: four of the six pairs differ
> by 16 points or more.**
> Averaged over three passes (54.7%), it is almost identical to Rater 3's 54.0%; that is not "changing
> the rater has no effect", it is **the effect of changing the rater being buried by pass-to-pass noise**.

**① Nine pairings (hard: no model assumption needed)**

| Class | Pairing | three-way disagreement | A/non-A disagreement |
|---|---|---:|---:|
| within-person | 1-2 | 46.0% | 36.0% |
| within-person | 1-3 | 56.0% | 50.0% |
| within-person | 2-3 | 32.0% | 30.0% |
| | **within-person mean (3 pairs)** | **44.7%** | **38.7%** |
| cross-rater | 1-4 | 52.0% | 32.0% |
| cross-rater | 1-5 | 58.0% | 32.0% |
| cross-rater | 2-4 | 34.0% | 16.0% |
| cross-rater | 2-5 | 36.0% | 16.0% |
| cross-rater | 3-4 | 40.0% | 26.0% |
| cross-rater | 3-5 | 42.0% | 38.0% |
| | **cross-rater mean (6 pairs)** | **43.7%** | **26.7%** |

> **Three-way: within-person (44.7%) and cross-rater (43.7%) are almost the same — a 1-point
> difference.**
> **But on A/non-A, cross-rater (26.7%) is clearly better than within-person (38.7%)** — changing the
> person is more consistent, in the same direction as the 74% > 64% of §8.1.2.
> The two must be read together: **on the "is it A" layer, changing the person works; on the three-way
> layer, changing the person is as bad as changing the pass.**

**② Orthogonal-contrast decomposition (soft: gives magnitudes, not variance components)**

Arrange the A indicator (0/1) by "sentence × pass" and decompose the sum of squares for "pass main
effect" using four pairwise orthogonal contrasts:

| Contrast | Attributed to | SS | Effect (percentage points) |
|---|---|---:|---:|
| c1 pass 1 vs pass 2 | **within-person (pass only)** | 1.4400 | −24.00 |
| c2 mean of 1,2 vs pass 3 | **within-person (pass + criteria)** | 4.8133 | −38.00 |
| c3 Rater 1's mean vs Rater 3 | between-person | 0.0017 | **+0.67** |
| c4 mean of the first four vs Rater 2 | between-person | 0.2890 | **+8.50** |
| **total (within-person, df=2)** | | **6.2533** | |
| **total (between-person, df=2)** | | **0.2907** | |

> **Within-person is 21.5 times between-person** (MS 3.13 versus 0.15, each with df=2).
> **Conclusion: on this data, the fluctuation from "the same person under a different pass or a
> different set of criteria" is far larger than the fluctuation from "a different person".**
> c3 is only 0.67 points, which quantifies the earlier sentence "averaged over three passes it is almost
> the same as the other person".

**③ Three restrictions must travel with the conclusion**

| Must not say | Because |
|---|---|
| "the rater accounts for X% and the pass for Y%" | **the degrees of freedom are only 2**; any proportional variance component would be so wide as to be meaningless. This is not "a coarse estimate", it is **non-identifiability** |
| "G-theory has been done" | **It has not.** Three passes by Rater 1 and one each by two others is an **unbalanced design**; whether this data can support the intended decomposition is a substantive judgement that was not made here. (This is not "the tools always require full crossing": G-theory and many-facet Rasch both handle unbalanced designs, see the correction under §8.5 item 5) |
| "so the criteria were useless" | The opposite: c2 shows **the criteria pushed A from 42% to 80%** (mean of passes 1 and 2 $(30+54)/2=42\%$ → pass 3 $80\%$, **a 38-point difference**, exactly the $-38.00$ in the table). They work; they simply **get internalised by each person separately and produce no cross-rater consensus** (§8.1.2 ③) |

> In one sentence: **the "cannot be done" of §8.5 item 5 still stands; but "no three-way decomposition"
> does not mean "no idea which is larger" — which is larger is given above.**
> Generated with `S1·_判次判者分解.py` (10 self-test assertions, two of them hard assertions reproducing
> the 60% and 74% of §8.1.2).

---

## 8.2 Error control: cost justification and family-wise error are two different standards, and only the first was done

This paper **judges 308 items one at a time**, which is **308 comparisons**, not one. Under neither of
the two error-control standards does $t=7$ pass:

| Standard | Requirement | What this paper has | Verdict |
|---|---|---|---|
| **FWER 5%** (at least one error ≤ 5%) | $\alpha\le0.05/308=1.62\times10^{-4}$ | the 1,803 negatives resolve only to $5.5\times10^{-4}$ | **cannot be certified** — not enough negatives |
| **FDR 5%** (the proportion of errors is controlled) | $t\ge9$ | currently $t=7$ | **$t=7$ is too loose** |

**$t=7$ was set on the scale of "a single comparison at $\alpha=3.00\%$".**
§5.5 gives the **cost justification** (where $w$ falls) and §5.8 gives **error control** ($\alpha$ and
multiple comparisons) — **these are two standards; only the first was done, and it must not be used to
cover the second.**

**Why it was not done**: passing FDR 5% requires raising $t$ to 9, and $t\ge9$ is outside the range
covered by human judgement (which pins only $t=7$); beyond that everything rests on scaled extrapolation.
**It cannot be done, and that is stated plainly.**

> Detail in §5.8.2 and §5.8.3, `S2` §2.7.6.

---

## 8.3 The three-band structure has the skeleton but not the optimality guarantee

The three bands of this paper — "has a source / needs a human / no source" — **are** the match /
clerical review / non-match of Fellegi–Sunter (1969) record linkage. That framework supplies a theorem:
**given upper bounds on the two error rates, ordering by likelihood ratio and setting two cut-offs
minimises the share sent to clerical review.**

**The score here is $\mathrm{LCS}$, not a likelihood ratio.** §5.6.1 measures that $\Lambda(5)=4.71$
while $\Lambda(6)=1.80$: **LCS = 6 is actually weaker evidence than LCS = 5**.

→ **So the cut-offs here ($lo=6$, $hi=12$) are not optimal in the Fellegi–Sunter sense.**

> **But the claim stops at "not optimal"**: that means **there is no optimality proof**, **not that
> "more items than necessary were sent for review"**. Asserting the latter would require also computing
> the review volume under a $\Lambda$-ordering and comparing — **that was not done.**

**The boundary of this item (three layers, by how firm they are)**:

1. **Firm**: $\mathrm{LCS}$ and $\Lambda$ are not equivalent — this is a contribution this paper can claim;
2. **Half-firm**: the inversion is only a point estimate (20 positives, 65 negatives), so **it cannot be
   asserted to be real**;
3. **Not permitted**: it does **not** follow that 6 should be moved to 5 or 7 — under family D-pure,
   $\Lambda(6)=95.8$, so **the inversion itself is family-dependent**.

> Detail in §3.1.1, §5.6.1, `S2` §2.7.6.

---

## 8.4 Replacing the threshold sweep with $J$: information loss is proven, the direction of bias is not

The conclusion of §6 must be read narrowly: **switching to $J$ loses information** ($J$ identical while
$\mathrm{ENR}$ can differ by 62 points) — that is constructive and it holds.

**What is not proven** is **which direction** $J$ biases toward. Claiming "$J$ gives a false safety
signal" would require separately establishing a source of bias, and this paper has none. On the real data
$J$ and $\mathrm{ENR}$ happen to move together — **but that is an observation from a single sample, not a
transferable property**, and part of the alignment comes from scaling (§6.4 notes this).

**Consequence**: this paper can say "the sweep cannot be skipped by using $J$"; it **cannot say "the $J$
metric is bad"**. It is not wrong; it answers a different question.

> Detail in §6.4 and §6.6, and `S2` §2.12.

**One more scope restriction (the sentence in this paper most easily read too broadly lives here):**

This section and §6 both negate **set-overlap** metrics — the family of $J$, BLEU/ROUGE and LCS.
**Non-overlap methods** (semantic embedding similarity, entailment models) **were not tested at all**, and
this paper **has no view** on whether they can identify a false-negative rate.

> So "**the road of switching metrics is closed**" holds only within the **overlap family**.
> The accurate statement is "**switching to another set-overlap metric is a dead end**", which is how the
> abstract and §9.2 put it.
> **The broader sentence is a real risk**: to make the narrative one chain ("the shortcut fails, so
> only the sweep remains"), "cannot substitute within this family" was written as "switching metrics does
> not work". **One reviewer sentence — "what about a metric that is not an overlap metric?" — dismantles
> it.**

---

## 8.5 Seven items at the data and measurement level (shorter, but a reviewer will ask)

| # | Limitation | Why it is so | Detail |
|---|---|---|---|
| 1 | **A single dataset, a single source-document family** (308 sentences to check, 1,803 unrelated pairs, all from one content-production pipeline) | no second dataset was available | §4.1 |
| 2 | **The denoising road was tried and is closed**: recomputing after stripping digits and punctuation drops AUC from 0.812 / 0.705 to 0.778 / 0.677 | what was stripped may be exactly the discriminative signal | §4.5 |
| 3 | **L5 is not a pure gold standard either**: of the 12 items the machine called "sourced", a human called 3 "not findable" | both rulers carry noise | §4.6.5 |
| 4 | **In the EPR three-band set, "format + Chinese, 0.94%" is pending rather than settled**, reported as the interval 0.61–0.94 | whether terminology overlap counts as letting something through is a business judgement | §5.7 |
| 5 | **No multi-rater variance decomposition**: the mature tools are G-theory / many-facet Rasch; this paper has **3** independent raters, but the pass structure is unbalanced (Rater 1 has 3 passes, the other two 1 each, 5 in total) | **no variance decomposition was done** — not "the tools require full crossing" | §8.1.3 / §8.1.5 |
| **6** | **Negation and antonym rewrites pass straight through the decision rule, and the rule "picks its position"** | the rule reads only contiguous overlap; it cannot read "not / un- / antonym" | table below |
| **7** | **Rater familiarity with the material systematically inflates "findable" judgements** (illusory truth / fluency effect) | if the rater knows the source, they are more likely to accept a sentence it does not support — **this threatens the validity of the gold standard, not just its reliability** | see 8.5.1 |

> **"Unbalanced" does not mean "these tools cannot be used"**: both G-theory and many-facet Rasch have
> established treatments for unbalanced or incomplete designs; what matters is whether the design is
> **connected** and whether the target variance components are identifiable. That step was not taken, so
> the correct statement is **this paper did not do a variance decomposition**, **not** that the tools
> require full crossing and therefore it cannot be done.

### 8.5.1 Detail on the two newer limitations

**Item 6: negation and antonym rewrites defeat the rule by position** (`S1·_子串vs子序列.py`, 10 self-test
assertions)

| Source text | Sentence to check | What changed | $\mathrm{LCS}$ | Band here |
|---|---|---|---:|---|
| This study reduced latency by reordering candidate paragraphs | … paragraphs did **not** reduce latency | one character inserted mid-sentence | **11** | needs a human (7–11) ← **caught** |
| The method achieved the best results on all three datasets | … the **worst** results | trailing antonym swap | **15** | **sourced (≥12) ← missed** |
| The system judged all test samples as sourced | … judged all as **not** sourced | trailing antonym swap | **14** | **sourced (≥12) ← missed** |

> **How to read it**: the substring version (relative to the subsequence version) is **more
> conservative** — insert anything in the middle and it breaks, dropping automatically into "needs a
> human". **But it is not fail-safe**: with a long identical prefix and the change late in the sentence
> it still judges "sourced", while the meaning is inverted.
> **This and 8.6 are two faces of the same problem**: 8.6 says "adversarial rewriting will invalidate
> $\mathrm{EPR}=3.00\%$"; here is **one concrete, lowest-effort way to do it**.

**Item 7: the familiarity effect threatens validity, not reliability**

- Literature: Reber & Schwarz 1999, *Effects of Perceptual Fluency on Judgments of Truth*,
  **Consciousness and Cognition** 8(3), `10.1006/ccog.1999.0386` (checked against Crossref).
- Mechanism: the more fluently material is read, the more it tends to be judged true or supported —
  **independently of whether the content is actually supported**.
- **Why this is more serious than "low reliability"**: low reliability can be expressed as an interval
  (which this paper does); **a validity bias makes the interval itself biased**, and more samples will not
  correct it.
- **The experiment not done**: pair the same sentences with a "familiar source" and an "unfamiliar source"
  and see whether the A share differs systematically. **About half a day of human time; not done.**

> Item 5 must be stated on two layers; **neither is an excuse, but the responsibility differs**:
> · **The tool cannot be used (a condition)**: G-theory / many-facet Rasch need **multiple raters judging
>   in a crossed design**, and this paper has 2 raters with 3 of the 4 passes by the same person — **no
>   variance component can be computed from this design**, which is objective;
> · **Additional data were not collected (a choice)**: finding 2–3 more independent raters on the same
>   items is something **we know how to do and know what it would resolve** — **it was simply not done**.
> **The two coexist and each governs a layer**: the main text of item 5 says "cannot be done"
> (analytically), this says "was not done" (data collection) — a reader needs to know whom to charge this
> limitation to.
> **So both stand, each on its own layer: the analysis could not use it / the additional data were not
> collected.** This kind of limitation is much more honest than "the sample size is limited", and much
> easier for a reviewer to accept.

> **A companion document**: `_已检索领域清单.md` tabulates the results of eight rounds of
> cross-disciplinary searching (fifteen sub-disciplines plus linguistics and eight further directions)
> item by item. **It is the evidence behind "we looked, and there is nothing".** If a reviewer asks "did
> you consider discipline X", the answer is immediate; if they ask "how do you know there is nothing",
> that list is the answer.

---

## 8.6 All analyses are static: the **more public** the threshold, the **more the curve gets pushed**

> **This limitation has an exact name, from policy evaluation: Campbell's law** —
> **"The more any quantitative social indicator is used for social decision-making, the more subject it
> will be to corruption pressures and the more apt it will be to distort and corrupt the social processes
> it is intended to monitor."** (Campbell 1979, paraphrased)
> **The original is a matter of degree** ("The more X is used …, the more it will be subject to
> corruption pressure and the more it will tend to distort the social processes it is intended to
> monitor"), **not "once used, it necessarily fails"**. The difference is substantive: the former says
> **"risk rises with the extent of use"**, the latter says **"it necessarily loses its value"**.
> **It must not be written as an absolute proposition**: "once used for decisions it will be gamed and
> therefore lose its value as an indicator" turns **a claim about degree** into **a claim about
> necessity**.
> Once the $\mathrm{LCS}$ of this paper is used as a **basis for decisions** and the threshold is made
> public, it becomes **a target to be optimised** — **how far it is used and how widely it is disclosed
> determines how large that risk is**, which is exactly why §8.6.2 separates it out to be measured.

**This limitation affects both main quantities at once** ($\mathrm{ENR}$ and $\mathrm{EPR}$ both move with
the adversary's behaviour), which is why it can be singled out.
> What can be said: **its nature differs from the others — it does not improve with more samples or a
> different design.**
> **But it must not be called "the most fundamental"** (that would require ranking all the limitations,
> and no such comparison was made here), **nor "the only one"** (that is a universal claim requiring each
> of the six to be compared item by item, which was not done).
> What can be exhibited is **this item's own mechanism** (§8.6.1: the decision rule hands the adversary a
> clear optimisation target); **whether the other five could be relieved by collecting more data is not
> asserted item by item here.**

### 8.6.1 The decision rule itself hands the adversary a clear optimisation target

The decision here is $S(t)=\{x:\mathrm{LCS}(x)<t\}$ → judged "no source".
**So a higher LCS is safer** — for a writer who knows how they are being judged, the optimal strategy is
to **copy the source's own wording in**.

**And the false-positive rate here, $\mathrm{EPR}(7)=3.00\%$, was measured under non-adversarial
conditions**: the negatives are **randomly paired unrelated pairs** (families D-all / B / D-pure), **not
sentences designed to pass by copying source fragments**.

> **So 3.00% is a lower bound, not an estimate.** Under adversarial conditions $\mathrm{EPR}$ can only
> be higher, and the cost analysis (the $w^*$ interval of §5.5) **rests entirely on $\mathrm{EPR}$** — so
> it moves up with it.

#### This inference is not mine alone: ALCE Appendix D has already measured it

ALCE ran a "cheating baseline" experiment, and **the result matches the inference above exactly**
(`S1`):

| System (ASQA) | fluency MAUVE | accuracy EM | **citation recall** | **citation precision** |
|---|---:|---:|---:|---:|
| ChatGPT Vanilla | 66.6 | 40.4 | 73.6 | 63.0 |
| **Top-1 passage** (return the best retrieved passage verbatim as the answer, self-cited) | **20.8** | 35.1 | **99.4** | **99.4** |
| **First 2 sents** (copy the first two sentences of that passage) | 67.2 | **18.9** | **98.7** | **98.7** |

**Pure copying drives the citation score to near-perfect (99.4), while fluency is only 20.8 and accuracy
35.1.**

> **This upgrades the section from "my inference" to "measured"**: the previous paragraph says "the
> optimal strategy is to copy the source verbatim" — **ALCE's Top-1 passage is that strategy, scoring
> 99.4.** This paper cannot produce that number, because it holds only 308 **non-adversarial** samples.
> **So this item should cite ALCE, not my own reasoning.**
>
> Reference: Gao et al. 2023 (ALCE), EMNLP 2023, Appendix D Table 11,
> `10.18653/v1/2023.emnlp-main.398`.

> **A related caveat**: this is not "an insight of this paper". It is **an explanation of a phenomenon
> that has already been measured — and of an experiment that someone else ran first.**
> **The position of this paper here is**: explain why (LCS is monotone ⇒ copying is optimal), and what
> that does to $w^*$ (how far it moves up).

### 8.6.2 The inverted $w$ **interval** depends on "the distribution does not change with the threshold"

The $w^*=1.16$–$1.41$ of §5.5 is **a conditional interval inverted from the chosen $t=7$** (the
revealed-cost path, see §2.3.3) —
> **it does not yield a value of $w$, only "where $w$ must fall for $t=7$ to be optimal"**.

That inversion has one premise: **the threshold has not changed the distribution of the text being
checked.**

Once $t$ is public and writers begin targeted rewriting, the whole $\mathrm{LCS}$ distribution shifts,
**the two curves $\mathrm{ENR}(t)$ and $\mathrm{EPR}(t)$ move together, and the direction is unclear**
(an adversary may copy, or rewrite to escape; the two push in opposite directions).

> **So this sentence must travel with $w^*$**:
> **"1.16–1.41 is a value inverted from the system's current behaviour, under the conditions that the
> threshold is not public and the distribution has not drifted."** It is **a description of the status
> quo**, not **an estimate valid under arbitrary conditions**.

### 8.6.3 Why this paper does not add that model

A ready-made game structure could be applied: the audited party chooses a rewriting intensity (at
increasing cost), the auditor chooses $t$, and in equilibrium $t$ and the two error rates move together.
(The audit-game line: Allingham & Sandmo 1972; Graetz, Reinganum & Wilde 1986; Border & Sobel 1987.)

**But it was not done, for a firm reason**: **this paper has no data on actual rewriting behaviour** — no
observation of how writers change once they know the threshold, and no estimate of rewriting cost.
**A model added on that basis cannot be calibrated and would be illustrative narrative only**, which is
worse than not writing it (it would give the reader the illusion that something was computed).

**So it is written as one limitation plus one line of future work, not as a model.**

### 8.6.4 How this relates to §6

§6 says "paraphrase can escape lexical overlap", which is **from the perspective of the decision rule's
capability** (false negatives: a true statement rewritten beyond recognition).
8.6 here is **from the perspective of the adversary's behaviour** (false positives: a false statement
copied until it looks true).
**The two directions are opposite but the root is the same: any public deterministic decision rule
induces text to move toward the rule's cheap side.**

> Detail in `S2` §2.7.12.

---

## 8.7 What this chapter does not write (so that it does not become a universal disclaimer)

| Not written | Why |
|---|---|
| "This study has a limited sample size and its conclusions require further validation" | Boilerplate. Every limitation here points to a specific section; this sentence is not needed |
| "Future work will further improve" | Same. The two "not done" items in 8.1 already state their cost and benefit; no wishes needed |
| "Constrained by data availability" | Same. §8.5 item 1 already says "no second dataset was available" |

> **The test**: if a limitation can be deleted without changing the reader's confidence in the
> conclusions, it should not be in the paper.

---

## 8.8 Broader Impact

The object audited here is a class of decision rules for **automated citation checking** — tools that
**decide whether a sentence has a source by applying a lexical overlap threshold**. Two points about the mechanism
follow. **Both go only as far as "what could happen"; no actual consequence was measured** — there was
neither a deployment survey nor a user study.

### Impact one: the costs of the two error types may not fall on the same person

- **Wrongly rejected** ($\mathrm{ENR}$): genuinely sourced, judged unsourced. **If the check's output is
  used to require authors to justify themselves**, the cost of explanation may fall first on the author —
  who pays it for an error they did not commit. **The actual shape of this cost, and how it is shared
  among the parties, were not observed.**
- **Let through** ($\mathrm{EPR}$): fabricated, judged sourced. **If a nonexistent citation is taken at
  face value and carried into later text**, it may be cited again. **Neither the spread nor the
  accountability outcome was measured.**

**This asymmetry means the two costs must be discussed separately.** Whether $w$ equals 1, **this paper
does not judge** — that is an institution's own normative decision.

> One thing must be said, or the previous sentence will be read as an empirical conclusion: the $w$ of
> §5.5 is **defined within a cost-loss framework** ($w=C_{\text{wrongly rejected}}/C_{\text{let through}}$;
> see the opening of §5.5), **not an empirical quantity induced from the asymmetry in this section**. This
> paper **does not estimate $w$**; it only translates $t$ into "which interval $w$ must fall in".

**Reporting $t$ without the $w$ it corresponds to hides a premise.** That integer **already presupposes a
weighting** — whether or not whoever set it realised. Weighting is a normative judgement, not a technical
one: this paper only translates it into a discussable $w$, and **does not make that choice for anyone**.

### Impact two: once public, the threshold is no longer a neutral measure

§8.6.1 has shown: the decision rule is "a longest common substring with the source of length $\ge t$
means sourced", so a writer who knows the rule **optimises by copying the source verbatim** — this
requires no technical skill at all, only knowledge of what the rule looks like.

**This paper must state it rather than skip it.** Not because "disclosure is beneficial", but because
**skipping it achieves nothing**:

> This fragility **can be derived from the rule itself**, and the rule (an integer threshold) is public.
> So "we do not mention it" protects nothing — anyone who could derive it does not need this paper.
> **The places where it can actually be mitigated are two, and neither is at the disclosure layer:**
> **① the rule-design layer** — report $(t,\,w)$ rather than a bare integer, so the assumption is exposed
> (§7.1, recommendation one);
> **② the process layer** — keep human review in high-cost settings; do not treat the rule's output as
> final.
>
> What this paper **does not** do is quantify or refine: no scale for "how many copied characters count as copying",
> no comparison of which rewrite passes more easily, no template. **That kind of refinement would only
> lower the barrier to abuse, with no gain in understanding the mechanism.**

This paper therefore **recommends no specific threshold**. What an audit report should contain is given as
three suggestions in §7.1, of which **the first is directly relevant here**: write $(t,\,w)$, not a bare
integer — because when only $7$ is written, the reader cannot see that a cost assumption sits underneath.
And in the system documentation described in §7.1, **that assumption has never been written down** (whether
it is discussed elsewhere was not checked).

### Three things this paper explicitly does not do

> This subsection is **not a third type of impact**; it is a statement of boundaries — different in kind
> from the two above, hence listed separately.

| Not done | Reason |
|---|---|
| Not claiming this decision rule should be abandoned | As a **banding device** it still works (§5 is entirely about measuring its banding error); what is negated here is the use of **a single reading as a guarantee to a third party** (§6.0.2) |
| Not endorsing a threshold for any discipline | The cost ratio **may** differ across fields — that is a commonplace, and **no cross-domain comparison was made**; what is offered is **a method**, not **a number** |
| Not assessing the impact of such tools in non-academic settings (news verification, legal documents) | **Those settings were not analysed, so no conclusion about them is drawn.** The empirical analysis of this rule uses these 308 academic sentences |

> **In one sentence**: this paper makes "what this threshold means" into something that can be put on
> paper. **It does not tell anyone which number to choose — that is for whoever has to sign.**

> **Boundary statement for this section**: **it contains no measurement.** It does three things: it
> states on whom the two costs **might** fall (mechanism, not observation), states that $w$ is not
> estimated here, and states the three things this paper does not do. Wherever "what consequences would
> follow" is at issue, only the mechanism is given, never an estimate.

---

# 9 Conclusion

## 9.1 In one paragraph

**This paper does not make the decision rule more accurate; it makes it discussable.**

A threshold of $t=7$ looks like a single integer. This paper shows that it is **three things at once**:

| What the system never wrote down, but has in fact committed to | Value | Basis |
|---|---|---|
| **a cost judgement** | letting something through costs **1.16–1.41 times** as much as wrongly rejecting it | §5.5 |
| **a significance level** | $\alpha=3.00\%$ (single comparison) | §5.8 |
| **a human-machine division of labour** | 62 items (20.13%) go to human review | §7.3 |

**None of the three has ever been written down.** Yet they decide in what sense the system is
"correct".

> **But these three are not three parallel conclusions** — they are one chain: because this class of
> metric cannot be substituted (the main result), the threshold is the only thing left to tune;
> because of the sweep, the threshold becomes a cost ratio.
> **The chain has three links, and stops there.** The multiple-comparison section (§5.8) **is not on
> this chain** — **it comes with the task**: deploying means judging 308 items one at a time, i.e. 308
> comparisons, **which no choice of metric avoids**.
> **Do not merge "twelve thresholds" and "308 items to check" into one number**: the 308 comparisons
> come from **judging 308 items one at a time in deployment**, not from the sweep.
> **The position of the third row (the 62-item division of labour) must be stated**: **it is an
> output of the chain, not a fourth conclusion.** Once the chain reaches "threshold = cost ratio",
> **how the threshold is set directly determines how many items go to humans** — $t=7$ requires 62
> items (20.13%), $t=4$ requires **149** (§7.3). So 62 is not an independent finding; **it is the
> dial that shows how the human load moves when the cost ratio moves**, and the chain in §9.2 has only
> three links (main result / consequence one / consequence two), with 62 attached to the end of
> consequence one.
>
> **The head of the chain is a negative claim.** Reading it as "three parallel discoveries" would soften
> this paper.

---

## 9.2 One main result, two consequences

### Main result (negative): substituting another set-overlap metric is a dead end

On nested decision sets the set-overlap metric $J$ is identically the ratio of two empirical
distributions, and **the denominator cancels** (§6.1). The consequence is not "it is inaccurate" but
**"it does not contain that information"**: with $J$ fixed, the false-negative rate can take any point
in the range (§6.2); and on the 100 human-judged items of this paper, **127 sentence pairs have
identical overlap and opposite human judgements** (§6.4.1).

> **This must be read together with its boundary**: "overlap metrics are unreliable" is not a
> finding of this paper — a proper controlled experiment dates from 2021 (Krishna et al., NAACL) and
> ALCE measured it again in 2023 (§6.0.1); "overlap metrics degenerate on nested sets" is older still,
> established by Baselga 2010 and Webber 2010 (§6.0).
> **The increment here is only three things**: why (a structural property called "non-identifiability",
> not "weak correlation"), how much (the cost and significance in this setting), and how often on this
> data (127 pairs).

### Consequence one: this family cannot be substituted, leaving only the threshold — and the sweep translates that integer into a cost

The paper gives a recomputable path: from a chosen $t$, invert the **interval** of cost ratios $w^*$
that makes $t$ optimal (§5.5), and give the full step function of optimal $t$ as $w$ varies (Table 5.2).
This is not a new theorem (§2.3 concedes the lineage: Neyman–Pearson → law and economics →
cost-sensitive learning); what this paper supplies is **an instance and numbers in this setting**.

### Consequence two: the sweep also runs into a harder constraint

$\mathrm{EPR}(t)$ is exactly the $\alpha$ under the null hypothesis (§5.8.1). Following that coordinate
gives the least forgiving result in the paper:

> **This paper makes 308 comparisons, not one.**
> Controlling FWER at 5%, **a Bonferroni correction** requires $\alpha\le1.62\times10^{-4}$
> ($=0.05/308$; with Šidák it is $1.665\times10^{-4}$ — **both are sufficient bounds, not necessary
> ones**, and the more conservative is used); but the 1,803 negatives only resolve down to
> $5.5\times10^{-4}$ — **the available data cannot certify it**.
> Controlling FDR at 5% requires $t\ge9$ — **the current $t=7$ is too loose**.

**This is not "the threshold is not chosen well", it is "this configuration cannot deliver a
family-wise guarantee on error".** Cost justification (§5.5) and error control (§5.8) are two separate
standards, and **this paper completes only the first half** — the second half gives thresholds
(FWER not certifiable; FDR requires $t\ge9$) that the current $t=7$ satisfies none of.

> **This is a consequence of consequence one, not of the main result** — if the shortcut worked (one
> $J$ value, one comparison), the problem would not arise.
> **This chain is the real claim of the paper; the three numbers are merely its three links.**

---

## 9.3 Five sentences that must not be said (more important to write here than the contributions)

| Must not say | Because |
|---|---|
| **"$t=7$ is wrong"** | It is equivalent to a cost assumption that is **statable and not unreasonable** (1.16–1.41). This paper did not measure that cost ratio — **that is a business judgement** |
| **"$t=4$ is better"** | $t=4$ pushes the human queue from 62 items to **149** (up 140%) and the automatic decision does not improve (§7.3). **That is not an improvement, it is a transfer** |
| **"This paper finds that a threshold implies a cost assumption"** | An instantiation of a known theorem, not a discovery; and within this paper it **has been demoted to consequence one**, not the main result. Lineage in §2.3 and §2.6 |
| **"This paper proves $J$ is non-identifiable"** | The proof has prior sources (Baselga 2010 / Webber 2010). What this paper does is **apply it to this setting, measure the dose, and state where it forces the choice** |
| **"No substitute metric works"** | Only the **set-overlap class** ($J$ / BLEU / ROUGE / LCS) is shown non-identifiable; **non-overlap methods** (semantic embeddings, entailment judges) **were not tested at all** (§8.4). Extending the claim to "switching metrics" in general is a sentence that was written too broadly |

**A sixth (not in the table, only half a sentence)**: this paper **does not** say "which $t$ should be
chosen". A recommendation would first require picking an accounting (**three false-negative accountings
× three false-positive accountings = nine cells**; the negative-sample family is **a separate
dimension** and is not multiplied in — see §5.5.2 and §5.5.5). **Which cell to pick is the auditor's
decision, not this paper's.**

---

## 9.4 How hard these conclusions hold — one sentence that must travel with them

The core number of this paper (false-negative rate **26.57%**; the three-accounting span of 17.30–40.14%
is **not a confidence interval** — its own 95% interval is [19.93%, 33.22%]) rests on one set of human
labels, and what **five judging passes** establish about those labels (three by the same person plus
two by different people, one of which is **a second rater re-judging all 100 items**) is:

- intra-rater agreement 54% ($\kappa=0.32$);
- **the share of "A (findable)" can move between 30% and 80%**;
- but **A judgements agree 74% across raters** (the three pairings give 64% / 70% / 74%, all in the
  same band, and no single item drags agreement down; **no separate contribution of "changing raters"
  can be isolated**); and rater 2's A set is a **proper subset** of rater 1's A set (0 in the reverse
  direction) — **the ambiguity lies at the B/C boundary, not at A**;
- **a completely new rater re-judging all 100 items (§8.1.4)**: only **49%** of three-way judgements
  coincide (**51%** disagreement, higher than same-person retesting), but **A/non-A is still 74%**; the
  point-accounting main figure moves from **26.57% to 31.39% (a difference of 4.82 points)**.

So the main figure uses the point accounting (dropping "cannot tell") not because it is "the middle
one": it **is the only rule that assigns no value to the least stable band**, and it matches how the
AIS framework handles "the source does not clearly support this" (§4.6.8).
This round also measured why: when the two raters differ by 20 items in their use of C, **the
lower-bound accounting moves by 10.03 points while the point accounting moves by only 4.82** — because
the former has to assign a value to C and the latter does not.

> **One more thing worth remembering (§8.1.5)**: once the five judging passes are separated,
> **the variance component from "same person, different pass or different criteria" is 21.5 times that
> from "different person"** (ratio of orthogonal-contrast SS: within-person 6.2533 versus between-person
> 0.2907, each with df=2).
> **The 21.5 is a ratio of variances, not of percentage points** — in percentage points, the
> within-person span is 50 points (30%→80%) while the largest between-person gap is 34 points. **The
> two quantities must not be mixed.**
> Rater 1's own three passes move 30%→54%→80% in A share (a 50-point internal span), while the six
> between-person pairings differ by **0 to 34 points**; **averaged over three passes (54.7%) it is
> almost the same as the other person's 54.0%.**
> **So the uncertainty in this data comes mainly not from "who is judging" but from "when, and under
> which version of the criteria".**

> **The error is reported in three dimensions and four rows, and must not be collapsed into one $\pm$:**
> sampling $\pm6.5$ points (308 items) | accounting span 23 points (308 items) | rater ·
> **category share** $\le50$ points (50 items, direction unknown) | rater · **main figure**
> **4.82 points** (100 items, cross-rater, point accounting).
> **The last two rows belong to the same "rater" dimension but have different bases and measure
> different things** (one measures how the A share drifts, the other how far the main figure moves);
> **they are neither interconvertible nor additive.**
> **The core numbers of this paper can be read as magnitudes only, not as precise values.**

---

## 9.5 One open question (left for others)

This paper uses **one** $t$ throughout. But the signal-to-noise ratio differs across source documents,
and that itself rests on an assumption:

> **"A uniform threshold" is itself an unwritten assumption: it assumes that the same $\mathrm{LCS}$
> value means the same evidential strength across source documents.**

That assumption **was not tested** here (§7.5). If it fails, the right move is not to tune $t$ but to
**set thresholds per source stratum**. That would break the tidy single-parameter conclusion of this
paper into "each stratum has its own $w^*$", and the cost analysis would have to be redone.

**This is probably the most worthwhile next question on this line.**

---

## 9.6 Closing

A threshold is at once **a cost judgement, a significance level, and a human-machine division of
labour**.

Putting the three on the table costs this: **we find that the system is not as reliable as it believed
itself to be** — the cost ratio it requires was never written down, it fails family-wise error control,
and how it handles the "cannot tell" band swings 23 percentage points.

What it buys is this: **the system can now be discussed.** For the first time an auditor can say "I
disagree with your cost assumption" — because they can finally see where that assumption is.

> **This is the minimum this paper asks of "honesty": not to make the system better, but to stop it
> hiding its assumptions.**

---

## Use of Language Models

The pipeline audited in this paper is a human--LLM writing workflow; describing it requires naming the
roles the model played, which we do throughout. In preparing the manuscript itself, we used language
models as adversarial reviewers: several rounds of critique were solicited, and every point raised was
checked against the underlying numbers before being accepted or rejected. **No number in this paper was
produced by a language model.** All labels were assigned by the raters described in §1.6, and every
figure is recomputable from the released scripts.

---

## Broader Impact



---

## References

Allingham, M. G., & Sandmo, A. (1972). Income tax evasion: A theoretical analysis.
*Journal of Public Economics*, 1(3–4), 323–338. doi:10.1016/0047-2727(72)90010-2

Altschul, S. F., Gish, W., Miller, W., Myers, E. W., & Lipman, D. J. (1990). Basic local
alignment search tool. *Journal of Molecular Biology*, 215(3), 403–410.
doi:10.1016/S0022-2836(05)80360-2

Angoff, W. H. (1971). Scales, norms, and equivalent scores. In R. L. Thorndike (Ed.),
*Educational Measurement* (2nd ed.). American Council on Education.

Axelsson, S. (2000). The base-rate fallacy and the difficulty of intrusion detection.
*ACM Transactions on Information and System Security*, 3(3), 186–205. doi:10.1145/357830.357849

Bhandari, M., Gour, P. N., Ashfaq, A., Liu, P., & Neubig, G. (2020). Re-evaluating evaluation
in text summarization. In *Proceedings of EMNLP 2020*, 9347–9359.
doi:10.18653/v1/2020.emnlp-main.751

Baselga, A. (2010). Partitioning the turnover and nestedness components of beta diversity.
*Global Ecology and Biogeography*, 19(1), 134–143. doi:10.1111/j.1466-8238.2009.00490.x

Border, K. C., & Sobel, J. (1987). Samurai accountant: A theory of auditing and plunder.
*Review of Economic Studies*, 54(4), 525–540. doi:10.2307/2297481

Campbell, D. T. (1979). Assessing the impact of planned social change. *Evaluation and Program
Planning*, 2(1), 67–90. doi:10.1016/0149-7189(79)90048-X

Cvach, M. (2012). Monitor alarm fatigue: An integrative review. *Biomedical Instrumentation &
Technology*, 46(4), 268–277. doi:10.2345/0899-8205-46.4.268

Drummond, C., & Holte, R. C. (2006). Cost curves: An improved method for visualizing classifier
performance. *Machine Learning*, 65(1), 95–130. doi:10.1007/s10994-006-8199-5

Elias, J. E., & Gygi, S. P. (2007). Target-decoy search strategy for increased confidence in
large-scale protein identifications by mass spectrometry. *Nature Methods*, 4(3), 207–214.
doi:10.1038/nmeth1019

Elkan, C. (2001). The foundations of cost-sensitive learning. In *Proceedings of IJCAI 2001*,
973–977.

Fabbri, A. R., Kryscinski, W., McCann, B., Xiong, C., Socher, R., & Radev, D. (2021). SummEval:
Re-evaluating summarization evaluation. *Transactions of the ACL*, 9, 391–409.
doi:10.1162/tacl_a_00373

Fagin, R., Kumar, R., & Sivakumar, D. (2003). Comparing top $k$ lists. *SIAM Journal on Discrete
Mathematics*, 17(1), 134–160. doi:10.1137/S0895480102412856

Fellegi, I. P., & Sunter, A. B. (1969). A theory for record linkage. *Journal of the American
Statistical Association*, 64(328), 1183–1210. doi:10.1080/01621459.1969.10501049

Gao, T., Yen, H., Yu, J., & Chen, D. (2023). Enabling large language models to generate text with
citations (ALCE). In *Proceedings of EMNLP 2023*. doi:10.18653/v1/2023.emnlp-main.398

Graetz, M. J., Reinganum, J. F., & Wilde, L. L. (1986). The tax compliance game: Toward an
interactive theory of law enforcement. *Journal of Law, Economics, & Organization*, 2(1), 1–32.
doi:10.1093/oxfordjournals.jleo.a036900

Green, D. M., & Swets, J. A. (1966). *Signal Detection Theory and Psychophysics*. Wiley.

Hand, D. J. (2009). Measuring classifier performance: A coherent alternative to the area under
the ROC curve. *Machine Learning*, 77(1), 103–123. doi:10.1007/s10994-009-5119-5

Kaplow, L. (2012). Burden of proof. *Yale Law Journal*, 121(4), 738–859. SSRN preprint version:
doi:10.2139/ssrn.1954006

Kaplow, L., & Shavell, S. (1994). Accuracy in the determination of liability. *Journal of Law and
Economics*, 37(1), 1–15. doi:10.1086/467304

Karlin, S., & Altschul, S. F. (1990). Methods for assessing the statistical significance of
molecular sequence features by using general scoring schemes. *PNAS*, 87(6), 2264–2268.
doi:10.1073/pnas.87.6.2264

Krishna, K., Roy, A., & Iyyer, M. (2021). Hurdles to progress in long-form question answering.
In *Proceedings of NAACL 2021*, 4940–4957. doi:10.18653/v1/2021.naacl-main.393

Koleff, P., Gaston, K. J., & Lennon, J. J. (2003). Measuring beta diversity for presence–absence
data. *Journal of Animal Ecology*, 72(3), 367–382. doi:10.1046/j.1365-2656.2003.00710.x

Kryscinski, W., Keskar, N. S., McCann, B., Xiong, C., & Socher, R. (2019). Neural text
summarization: A critical evaluation. In *Proceedings of EMNLP 2019*, 540–551.
doi:10.18653/v1/D19-1051

Lin, C.-Y. (2004). ROUGE: A package for automatic evaluation of summaries. In *Text Summarization
Branches Out (WAS 2004)*, 74–81.

Lin, C.-Y., & Och, F. J. (2004). Automatic evaluation of machine translation quality using longest
common subsequence and skip-bigram statistics. In *Proceedings of ACL 2004*.
doi:10.3115/1218955.1219032

Manning, C. D., Raghavan, P., & Schütze, H. (2008). *Introduction to Information Retrieval*.
Cambridge University Press. doi:10.1017/CBO9780511809071

Neyman, J., & Pearson, E. S. (1933). On the problem of the most efficient tests of statistical
hypotheses. *Philosophical Transactions of the Royal Society A*, 231, 289–337.
doi:10.1098/rsta.1933.0009

Novikova, J., Dušek, O., Cercas Curry, A., & Rieser, V. (2017). Why we need new evaluation metrics
for NLG. In *Proceedings of EMNLP 2017*, 2241–2252. doi:10.18653/v1/D17-1238

Prechelt, L., Malpohl, G., & Philippsen, M. (2002). Finding plagiarisms among a set of programs
with JPlag. *Journal of Universal Computer Science*, 8(11), 1016–1038.
doi:10.3217/jucs-008-11-1016

Provost, F., & Fawcett, T. (2001). Robust classification for imprecise environments. *Machine
Learning*, 42(3), 203–231. doi:10.1023/A:1007601015854

Rashkin, H., Nikolaev, V., Lamm, M., Aroyo, L., Collins, M., Das, D., Petrov, S., Tomar, G. S.,
Turc, I., & Reiter, E. (2023). Measuring attribution in natural language generation models.
*Computational Linguistics*, 49(4), 777–840. doi:10.1162/coli_a_00486

Reber, R., & Schwarz, N. (1999). Effects of perceptual fluency on judgments of truth.
*Consciousness and Cognition*, 8(3), 338–342. doi:10.1006/ccog.1999.0386

Reiter, E. (2018). A structured review of the validity of BLEU. *Computational Linguistics*,
44(3), 393–401. doi:10.1162/coli_a_00322

Reiter, E., & Belz, A. (2009). An investigation into the validity of some metrics for automatically
evaluating natural language generation systems. *Computational Linguistics*, 35(4), 529–558.
doi:10.1162/coli.2009.35.4.35405

Robertson, S. E., & Walker, S. (1994). Some simple effective approximations to the 2-Poisson model
for probabilistic weighted retrieval. In *Proceedings of SIGIR '94*, 232–241.
doi:10.1007/978-1-4471-2099-5_24

Robertson, S., & Zaragoza, H. (2009). The probabilistic relevance framework: BM25 and beyond.
*Foundations and Trends in Information Retrieval*, 4(1), 1–174. doi:10.1561/1500000019

Schleimer, S., Wilkerson, D. S., & Aiken, A. (2003). Winnowing: Local algorithms for document
fingerprinting. In *Proceedings of SIGMOD 2003*, 76–85. doi:10.1145/872757.872770

Sunstein, C. R. (2005). *Laws of Fear: Beyond the Precautionary Principle*. Cambridge University
Press. doi:10.1017/CBO9780511790850

Volokh, A. (1997). $n$ guilty men. *University of Pennsylvania Law Review*, 146(1), 173–216.
doi:10.2307/3312707

Webber, W., Moffat, A., & Zobel, J. (2010). A similarity measure for indefinite rankings. *ACM
Transactions on Information Systems*, 28(4), Article 20. doi:10.1145/1852102.1852106

Wise, M. J. (1996). YAP3: Improved detection of similarities in computer program and other texts.
In *Proceedings of SIGCSE 1996*, 130–134. doi:10.1145/236452.236525

Youden, W. J. (1950). Index for rating diagnostic tests. *Cancer*, 3(1), 32–35.
doi:10.1002/1097-0142(1950)3:1<32::AID-CNCR2820030106>3.0.CO;2-3

Yue, X., Wang, B., Chen, Z., Zhang, K., Su, Y., & Sun, H. (2023). Automatic evaluation of
attribution for large language models (AttrScore). In *Findings of EMNLP 2023*.
doi:10.18653/v1/2023.findings-emnlp.307