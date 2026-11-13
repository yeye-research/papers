# One Word, Four Kinds of Stop

### Measurement-Mismatch in LLM Service Failure Rates and Their Denominators

**Abstract only** · the full manuscript (Chinese, ~46,400 characters, six sections) is **not released**.
What is public here is the abstract, in both languages. Every number below is taken from the
frozen data snapshot described at the end of this page.

> 本页只放摘要。中文全稿（约 4.64 万字符、六节）**未公开**。

---

## Abstract

A task is dispatched, the interface reports "completed" within seconds to tens of seconds, and the body is empty. It is not an error, not a timeout, and not "still thinking": it looks finished, it just says nothing.

This paper does not explain **why** it stops. It asks one question: **when it stops, what exactly has been counted?** The material comes from a single continuously running machine, observed from 2026-10-15 to 11-13: **6,893** trace files and **108,304** generations carrying usage statistics (**104,833** of them main-session).

Three findings.

**First, "stop" corresponds to at least four countable quantities** — 4,176 dialogue-gap segments, 148 empty `choices` responses at the generation level, 48 "containing a stop" on the shift-report, and 15 cancelled UI states. **All four have different denominators; only two observation windows are involved, and no two of them share a denominator.** Between the second and the third there is a **structural 1:1 correspondence** — one empty generation *is* one stop-containing turn under that report's definition — which is precisely why it is so easily misread as corroboration. The sharpest instance: on a single shift-report line, "containing a stop: 48" sits next to "empty generations: 48 / 2,889". It reads as two sources confirming each other. It is **one 48 recorded twice in two units**.

**Second, one denominator is eliminated by the outcome itself.** Restricting to sessions with at least two generations makes the first-generation empty rate **identically zero** — that is a definition, not a finding. Restoring single-turn sessions to the denominator gives **1.9295%** (75/3,887) for the first generation and **0.1205%** (60/49,807) thereafter, a **16.02-fold** difference. A previous manuscript explained this pattern as "earlier is more fragile" and then withdrew that claim — **the withdrawal was correct, but it also withdrew the start-up failure rate**; the two are different quantities.

**Third, the same empty response is judged three different ways — and a fourth at the billing layer.** An empty `choices` array has **no minimum-item constraint in the interface specification**; client libraries treat a non-empty `choices` array as **mandatory** and raise a server-side fault (HTTP 502 semantics); the local harness records `status: ok` and the interface shows "completed". At the billing layer it is simpler still: the provider's usage export contains request counts, token volumes and tool-call counts, and **no failure field at all**. The 148 empty responses carry output tokens, are billed, and are counted in the request total — **on that ledger they are indistinguishable from a normal request**. Whether a call counts as a failure has no single definition to begin with.

The contribution stops at reporting, not at remediation: a failure rate must state **its denominator, its criterion, and its window**. Without all three, the number is not comparable.

**Keywords**: measurement definition; silent failure; denominator; survivorship bias; observability

---

## 摘要（中文）

使用者把一件活派下去，界面几秒到几十秒就显示"已完成"，正文却是空的。这不是报错，不是超时，也不是"正在思考"：它看起来就是干完了，只是什么都没说。

本文不解释"为什么会停"，只追问一件事：**当它停的时候，究竟被数成了什么。** 材料来自一台持续运行的机器，观测窗 2026-10-15 至 11-13，共 6,893 个 trace 文件、**108,304** 次带用量统计的生成（主会话 **104,833**）。

三个发现：

**其一，"停"在日志侧至少对应四个可数量**——对话停顿 4,176 段、生成级返回空 148 条、收工条含停机 48、界面取消 15 张。**四者的分母互不相同，时间窗只有两种**：`4,176` 与 `148` 同为全库窗，`48` 与 `15` 只覆盖 11-12 当天的一小段；**没有任何两个共享同一分母**。其中②与③之间还存在一层**构造性的 1:1 对应**（一次空即一个含停机轮次）——**恰恰因为这一层，最容易被误读成"两个来源互证"**。 其中一处最刺眼：同一张收工条上「含停机 48」与「空生成 48／2,889」并列，看似两个来源互相印证，实为**同一个 48 按两种单位各记了一次**——**在该收工条的口径下，「含停机轮次」就是"轮内出现过一次空"的轮次**，所以两者数值必然相等；这是**口径使然，不是独立印证**。（空之后也可能重试，重试属新的一轮，见 §3.2.2 的锚点事件。）

**其二，有一个分母被结果本身删掉了**。若只统计"有 ≥2 次生成"的会话，第 1 次生成的空率**恒为 0**——这是定义，不是发现。把单轮会话补回分母后，第 1 次生成 **1.9295%**（75/3,887），第 2 次及以后 **0.1205%**（60/49,807），相差 **16.02 倍**。此前稿件曾以"越靠前越脆弱"解释这组数据，后来撤回——**撤回是对的，但它把"会话启动那一次的失败率"一并撤掉了**；两者是不同的量。

**其三，同一个空，三层三种判定，到账务层是第四种**。`choices` 空数组在接口规范里**未受最小项约束**，客户端库却把它当**强制契约**（判为服务端故障、502 语义），而本机宿主记 `status: ok`、界面显示"已完成"。**"这次算不算失败"这件事本身，就没有统一口径。** 到了账务层就更彻底：服务商导出的用量账单里只有请求数、token 量与工具调用次数，**没有任何失败字段**；那 **148** 条空响应有 token、被计费、并计入请求数，**在账上与一次正常请求无法区分**。

本文的结论止于"如何报告"，不涉及"如何治理"：报一个失败率时，**分母、判据、窗口**三个字段必须同时在场，缺一个，这个数就不可比。

**关键词**：失败率口径；静默失败；分母；生存偏差；可观测性

---

## What this is

This is the third paper in a series that examines **a system that reports success it cannot
substantiate**. The first looked at a lexical threshold that says *sourced* on the strength of a
handful of shared characters. The second looked at a guardrail that passes a well-formed empty
response and has it recorded as a finished answer.

This one steps back from both. It does not ask why the failure happens; it asks **how the failure
gets counted** — and finds that a single word, *stop*, is carrying at least four numbers with four
different denominators, none of which can be converted into another.

## Data and reproducibility

| Item | State |
|---|---|
| Observation window | 2026-10-15 → 11-13 (local time, UTC+8) |
| Trace files | **6,893** |
| Generations with usage statistics | **108,304** (main-session **104,833**) |
| Empty `choices` responses | **148** (main-session **147**) |
| Snapshot fingerprint | `148b317f63a267287eb089bdf82489455f942afdadf04cafdb276c8a1cca1d3b` |
| Snapshot time | 2026-11-13T09:11:41 |
| Manifest | 6,893 rows (relative path / bytes / mtime) |
| Scripts | 14 counting and adjudication scripts, each with its own self-test |

The fingerprint covers **metadata**, not file content: it fixes *which* files the numbers were
computed from, and it changes if files are added, removed or modified. It does not guarantee
byte-identical content — that would require per-file hashes.

## Status

| Item | State |
|---|---|
| Manuscript (Chinese) | Complete — 46,400 characters, six sections, three external audit passes |
| This page | Abstract only |
| English edition | Not planned as a full translation |
| Replication package | Scripts and manifest held locally; released on request |

---

*This paper studies the counting failures of the system that runs this account. That is why the
case is n = 1, and why the trace exists at all.*
