# How Many Times Does One Incident Count?

### Statistical Unit and the Sensitivity Region of LLM Service Failure Attribution on a Single Machine

**Abstract only** · the full manuscript (Chinese, ~18,300 characters, nine sections) is **not released**.
What is public here is the abstract, in both languages. Every number below is taken from the
frozen data snapshot described at the end of this page.

> 本页只放摘要。中文全稿（约 1.83 万字符、九节）**未公开**。

---

## Abstract

LLM services exhibit a failure mode that raises no error: the API returns HTTP 200, usage fields are recorded normally, but `choices` is an empty array — a call that succeeds at the interface level and returns no content. Working from **104,812 generations** and **147 such events** observed on a single Windows workstation, we examine a more basic question than "what caused this": under single-machine observation (n = 1), **can between-group differences in these events be attributed to specific factors?**

Three findings.

**First, these events are not independent.** The median gap between consecutive events is **2.02 minutes**; **72.6%** of gaps fall below 10 minutes and **85.6%** below one hour; and **47 of the 147 events** fall inside a single cluster. Against an independent Poisson baseline — under which only 3.55% of gaps would fall below 10 minutes — the observed share is **20.4 times** the baseline.

**Second, the choice of statistical unit is sufficient to reverse the conclusion.** On the same data and with the same denominators, the event-rate ratio between local-direct and platform-hosted access is **9.37** counted per event; counted per incident cluster it ranges from **1.20 to 5.54** across six gap thresholds (1 to 120 minutes). At 1 minute the ratio is 5.54 with a 95% CI of **[2.91, 10.57], excluding 1**; at 60 minutes it is 1.20 with a 95% CI of **[0.52, 2.78], including 1**. The crossover lies between 10 and 30 minutes.

**Third, the three candidate factors constructible from local fields — access route, context length, time of day — yield ratios spanning 1.00 to 38.38** depending on which days are excluded from the sample. The original exclusion rule produced a ratio of 1.00, but that rule selects days by their event rate, that is, **by the outcome variable itself**; once replaced by exposure-only rules, all three candidates remain significant (access route 6.85–10.33; context length 22.36–38.38; time of day 17.20–25.22).

We therefore neither claim a root cause nor deny one. Our claim is narrower and easier to test:

> **On this data, whether the two groups differ depends on a quantity the data does not supply — how long one incident lasts. And in this data that quantity falls inside the region where the conclusion flips.**

We stress that this is **not non-identifiability**: at the finest unit the effect is estimable. Nor is it **robustness**: at coarser units it is not. What is missing is a definition of "one incident", which single-machine logs do not provide.

To check that the criterion carries information, we ran a simulation calibrated to this data (Neyman–Scott cluster process, 200 runs per world). At a true rate ratio of 5, the number of thresholds whose interval contains 1 was **0 in all 200 runs**; at a true ratio of 1, that number was 6 in **91.0%** of runs. Our data gives **3** — a low-probability outcome under both pure worlds.

We offer **four reusable pre-checks**, each costing a few commands: the inter-event gap distribution; recomputing under an alternative statistical unit; the treatment-by-stratum cell support (exposure count and expected events per cell); and whether a removal rule uses the outcome variable.

**Keywords**: silent failure; statistical unit; sensitivity region; crossover point; positivity violation; LLM service reliability

---

## 摘要（中文）

LLM 服务存在一种不报错的失败：HTTP 200、有 usage 记录、但 `choices` 为空——接口正常，内容没有。本文以一台 Windows 工作站上连续观测的 **104,812 次生成、147 条此类事件**为对象，检验一个更基础的问题：**在单机（n=1）观测条件下，这类事件的组间差异能否被归因到具体因素**。

我们发现三件事。

**第一，这类事件不是独立发生的。** 相邻事件间隔的中位数为 **2.02 分钟**，**72.6%** 的相邻间隔小于 10 分钟，**85.6%** 小于 1 小时；147 条事件中有 **47 条**落在同一个簇内。与独立泊松基线对照（该基线下仅 3.55% 的间隔会小于 10 分钟），实测是基线的 **20.4 倍**。

**第二，统计单位的选择直接决定结论。** 同一批数据、同一分母，按"事件条数"计算，本机直连与平台托管的事件率之比为 **9.37 倍**；按"事故簇"计算，该比值降到 **1.20–5.54 倍**（六个间隔阈值）。1 分钟簇下为 5.54，95% 置信区间 **[2.91, 10.57]，不含 1**；60 分钟簇下为 1.20，95% 置信区间 **[0.52, 2.78]，含 1**。翻转点在 **10–30 分钟**之间。

**第三，三个候选因素（接入路由、上下文长度、时段）的组间倍数随"剔除哪些天"的规则在 1.00–38.38 之间漂移。** 其中原规则给出的"倍数塌到 1.00"可追溯到该规则**使用了结局变量本身**；换成只看暴露侧的规则后，三个候选都仍显著（接入路由 6.85–10.33；上下文长度 22.36–38.38；时段 17.20–25.22）。

由此，本文既不说"找到了根因"，也不说"没有任何原因"。本文的主张更窄、也更容易被检验：

> **在这份数据上，"两组是否有差异"这个结论取决于一个数据本身不提供的量——一次事故持续多久。而这个量恰好落在能让结论翻转的区间里。**

这个主张**不是"不可识别"**——按最细的单位，效应是估得出来的；它说的是**估出的值依赖一个未被给定的尺度，而这个尺度正好在敏感区**。本文用模拟验证了这一判据有区分度：在真率比 = 5 的簇过程上运行 200 次，六档中"置信区间含 1"的档数恒为 0；在真率比 = 1 上该值为 6 的次数占 91.0%（182/200）；而本数据为 3——在两种纯情形下都是小概率。

本文给出**四项可复用的前置检查**，每项只需几条命令：事件间隔分布、换统计单位重算、每格曝光数、剔除规则是否用了结局变量。

**关键词**：静默失败；统计单位；结论敏感性；转折点；正值性违反；LLM 服务可靠性

---

## What this is

This is the second of four papers built around **a system that reports success it cannot substantiate**, and it is the only one in the set whose conclusion is negative.

The first paper counted the failure and found that one word, *stop*, carried at least four numbers with four different denominators. This paper takes the next step — attribution — and finds that **the step cannot be completed on this kind of data**, not because the numbers are too few but because the counting unit has no observable definition.

The remaining two papers take up remediation. **They cannot assume a root cause**, because this paper shows none can be identified here. That is the point of publishing a negative result in the middle of a series: it constrains what the later papers are allowed to claim.

The methodological position is stated in the manuscript as a distinction that is easy to blur:

| | what it means | does this data fit |
|---|---|---|
| **non-identifiability** | the effect cannot be estimated at all — structurally, some cells have no observations | **No** — at the finest unit, 5.54 with CI [2.91, 10.57] is estimable |
| **unit-dependent sensitivity** | the effect can be estimated, but the estimate depends on a quantity nobody supplied — and that quantity sits where the conclusion flips | **Yes** |

## Data and reproducibility

| Item | State |
|---|---|
| Observation window | 2026-10-15 → 11-13 (local time, UTC+8) |
| Trace files | **6,913** |
| Generations with usage statistics | **108,867** (main-session **104,812**) |
| Empty `choices` responses | **148** (main-session **147**) |
| Snapshot fingerprint | `8e70ca5bea3d24ed101ffe8a228ee06280be2a8af7812b030d78f6f7d3027fc1` |
| Snapshot time | 2026-11-13T12:12:38 |
| Manifest | 6,913 rows (relative path / bytes / mtime) |
| Intermediate data | event-aggregate table and daily table, written to disk at freeze time |
| Scripts | 17 counting, adjudication and simulation scripts, each with its own self-test |

**Two counting rules, never mixed.** All analysis in this paper uses the **main-session** rule (104,812 generations / 147 events). The all-sessions rule (108,867 / 148) is quoted only where it must be reconciled with the earlier paper. The difference is 4,055 sub-agent generations carrying 1 event.

**On the gap between this page and paper 03.** Paper 03 was frozen at 2026-11-13T09:11:41 and reports 6,893 files / 108,304 generations / 148 events. This page is frozen three hours later and reports 6,913 files / 108,867 generations / 148 events. **Both are correct; they are different sampling instants.** The number of empty responses did not change across the three hours (+563 generations, +0 events) — which is itself the reason the figure 148 is treated as stable in paper 03.

The fingerprint covers **metadata**, not file content: it fixes *which* files the numbers were computed from, and it changes if files are added, removed or modified. It does not guarantee byte-identical content.

## Status

| Item | State |
|---|---|
| Manuscript (Chinese) | Complete — ~18,300 characters, nine sections, four external audit passes (structure / literal / fact / blind) and one blind replication |
| This page | Abstract only |
| Gate status | Delivery gate passed (`sha16 = caa13d5e67bd88e4`): audited by an independent literal reviewer, an independent structural reviewer, and a fact-check pass that re-verified every cited claim against the source papers |
| English edition | Not planned as a full translation |
| Replication package | Scripts, manifest and intermediate tables held locally; released on request |

## Two things worth reading past the abstract for

**The result was nearly the opposite.** An earlier draft of this paper argued that the attribution was structurally impossible, on the basis of a "placebo test" that appeared to fail. That test was wrong: it compared a *removal rule* (drop the weekend days, then recompute) with a *placebo* (group by weekend, then compare). The real placebo gives a ratio of 0.12 with p = 0.803 — it passes. The structural argument was withdrawn and replaced by the sensitivity argument above.

**A negative control that passes is still not a proof.** "Cannot reject no difference" is not "proved no difference". Both statements appear in the manuscript at the places where the distinction matters.

---

*This paper studies the counting failures of the system that runs this account. That is why the
case is n = 1, and why it is honest to publish a null result from it.*
