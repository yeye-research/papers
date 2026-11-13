# What a Lexical Threshold Commits You To

### Auditing Set-Overlap Metrics in Citation Verification

**Abstract only** · the full manuscript (~68,200 words, nine sections, 123 pp typeset) is **not released**.
What is public here is the summary below and the results it reports.

> 本页只放摘要与结果。英文全稿（约 6.82 万词、九节、排版 123 页）**未公开**。

---

## The one-paragraph version

A citation checker decides whether a sentence is supported by a source by computing a lexical
overlap score and comparing it against a threshold — a bare integer, `7`, in the system audited
here. This paper asks what that integer commits its operator to, and reports three results:
the obvious fix does not work, the threshold turns out to be a cost claim nobody wrote down,
and the error control that a claim of this kind needs is not achievable with this data.

## The three results

**Main result — negative.** Substituting a set-overlap index (Jaccard and relatives) for a
threshold sweep does not work. On nested decision sets the index reduces to a ratio of two
empirical distributions, and the denominator cancels; the quantity of interest *is* that
denominator. In the human-labelled sample, **127 sentence pairs share an identical overlap
value and received opposite labels**.

The distinction matters for what comes next: this is **non-identifiability, not weak
correlation**. Weak correlation yields to more data. Non-identifiability does not, because the
index does not contain the information at that value. Scope is stated in the paper — what is
ruled out is *this family* of metrics; embedding-based and entailment-based judges are not
tested and are not covered by the claim.

**Consequence one — the threshold is a cost claim.** With that family closed off, the threshold
is the only knob left, and scanning it converts the bare integer into a statement of relative
cost. The deployed `t = 7` is equivalent to asserting that missing a fabricated claim is
**1.16–1.41×** more costly than wrongly flagging a genuine one. Under equal weighting the
optimum is `t = 5`, which differs from `t = 7` by **0.92 percentage points**.

**Consequence two — error control is not met.** `EPR(t) = α` under the null, so `t = 7` is
`α = 3.00%`. But the task involves 308 comparisons: family-wise control at 5% under Bonferroni
requires `α ≤ 1.62×10⁻⁴`, while 1,803 negative samples resolve only `5.5×10⁻⁴` — not
certifiable with this data. FDR control at 5% requires `t ≥ 9`. Cost justification and error
control are separate standards, and only the first is achieved.

## What the paper says against itself

Three constraints are reported in the abstract, not buried in limitations: the 3.00%
false-positive rate is a **non-adversarial lower bound**; the 23-point spread of the interval
comes entirely from how "unclear" labels are treated; and category proportions move between
30% and 80% across four labelling passes. A fully independent rater re-labelling all 100
sentences moves the central estimate from 26.57% to 31.39% — a 4.82-point gap — so the central
figure is a **magnitude, not a precise value**.

## How the numbers were produced

Every table and every figure cites the script that generates it, and the scripts carry
self-tests. Where a number was produced by hand, the paper says so. An independent
re-computation in MATLAB accompanies the statistics, so no figure rests on a single
implementation.

The replication package is **not in this repository**. It is held with the manuscript and released
on request — open an issue if you need it.

---

*Corrections are welcome as issues. A correction that changes a number will be recorded in the
commit message, not silently amended.*
