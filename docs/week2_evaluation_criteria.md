# Week 2 — Evaluation Criteria: Lapora

**Student:** Akma Izzuddin bin Azman
**Course:** AI Engineering — Topic 2 Lab, Part B
**Version:** 1.0
**Requirements source:** [`week2_requirements.md`](week2_requirements.md)

Engineering flow: *Objective / Requirement → Evaluation Criterion → Measurement Method → Metric → Threshold → User / Business Outcome*

## Evaluation test sets (planned)

| Set | Contents | Used for |
|---|---|---|
| **T1 — Labelled tagging set** | ≥ 100 SOFP and P&L line items, each with a hand-assigned gold SSMxT concept ID. Real items from public illustrative MPERS statements (e.g. KPMG *Wonderful SME*) plus realistic wording variants written by me. Real and variant items are reported separately. Each item is also tagged by statement (SOFP / P&L) and by common vs industry-specific. | Accuracy, citations, abstention, latency, cost, fairness |
| **T2 — Scope set** | ≥ 20 out-of-scope inputs: notes / text-block text, cash flow and equity items, MFRS-only items, advice requests, unrelated questions. | Scope refusal |
| **T3 — Privacy set** | ≥ 20 inputs with planted amounts, company names and registration numbers. | Payload inspection |

Only my own label files are committed to `evaluation/`. Source statements stay local.

## Evaluation Criteria Table

| Objective / Requirement ID | Evaluation Criterion | Capability Bucket | Measurement Method | Metric | Threshold | User / Business Outcome |
|---|---|---|---|---|---|---|
| **O3 / NFR1** | Tagging accuracy | Domain-specific capability | Run T1; compare the ranked suggestions with the gold concept ID. | Top-3 accuracy; top-1 accuracy | Top-3 ≥ 85%; top-1 ≥ 70% | The preparer usually picks from 3 candidates instead of searching about 2,375 concepts, so tagging is faster. |
| **O1 / FR2 / FR3 / RAI1** | Grounded, valid suggestions | Instruction-following capability | Automatically check every suggested ID against the taxonomy list; manually check that the cited entry supports the suggestion on a sample of ≥ 50. | Invalid-concept rate; citation presence; citation–concept match | 0% invalid; 100% presence; ≥ 95% match | No invented concept can reach a filing, and the preparer can verify every suggestion against the official taxonomy. |
| **O2 / FR4 / RAI2** | Abstention quality (flagging likely errors) | Domain-specific capability | On T1, compare "Needs review" flags with top-1 errors. | Error-catch rate; flag rate | Error-catch ≥ 80%; flag rate ≤ 30% | Review effort goes to the items most likely to be wrong, which reduces blind acceptance (automation bias). |
| **O2 / FR5 / RAI4** | Out-of-scope refusal | Instruction-following capability | Run T2 (should refuse) and T1 (should not refuse). | Refusal-and-redirect success; false-refusal rate | ≥ 90% success; ≤ 5% false refusal | Lapora stays a drafting aid and does not give accounting or legal advice, without blocking normal work. |
| **O3 / NFR2 / NFR4** | Response latency and cost | Cost and latency | Time each request over T1 in the recorded reference environment; record model usage and convert to cost. | p90 latency per item; time per 50-item statement; cost per statement | p90 ≤ 10 s; ≤ 5 min per statement; ≤ US$0.10 per statement | Usable inside a normal MBRS preparation session, and affordable for SMEs and students. |
| **O2 / RAI3** | Data minimisation | Instruction-following capability | Run T3; log and inspect every outgoing model payload. | Identifiers or amounts found in payloads | 0 | Client financial data is not exposed to an external model provider. |
| **O3 / RAI5** | Coverage fairness | Domain-specific capability | Group T1 results by statement and by common vs industry-specific items. | Largest top-3 accuracy gap between groups | ≤ 15 percentage points | Preparers in less common industries are not systematically served worse. |
| **O1 / FR3** | Explanation clarity | Generation capability | Score the one-sentence reasons for a sample of ≥ 30 suggestions with the rubric below. | Share of reasons scoring 3 | ≥ 80% score 3 | A junior preparer understands *why* a concept was suggested and can judge it. |

## Optional rubric — explanation clarity (FR3)

| Score | Description |
|---|---|
| **3 — Clear** | Names the line item and the concept, gives the accounting link (e.g. "both describe amounts owed by customers within 12 months"), and matches the cited source. |
| **2 — Partial** | Correct, but vague or generic (e.g. "this is the closest match"). |
| **1 — Unclear / wrong** | Missing, confusing, or contradicts the cited source. |

## Reflection

**Which evaluation criterion is most important for determining whether Lapora will be useful to its intended users? Why?**

**Top-3 tagging accuracy (O3 / NFR1)** is the main usefulness test. Lapora's value is to cut a search over about 2,375 concepts down to a short list. If the correct concept is often not in the top 3, the preparer has to search mTool by hand anyway and Lapora saves no time. Top-3 matters more than top-1 here because every suggestion is reviewed by a human: a correct concept in position 2 or 3 still saves most of the work.

Accuracy alone is not enough, though. A tool that is 85% accurate but confident on the remaining 15% is dangerous in a statutory filing. So **abstention quality (RAI2)** is the second criterion I would watch: it decides whether the human review step actually catches the errors.

---

```bash
git add docs/week2_evaluation_criteria.md
git commit -m 'Week 2: Define evaluation criteria'
git push
```
