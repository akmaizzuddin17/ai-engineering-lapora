# Week 2 — V1 Architecture Decisions & Traceability: Lapora

**Student:** Akma Izzuddin bin Azman
**Course:** AI Engineering — Topic 2 Lab, Part C
**Version:** 1.0
**Diagram:** [`week2_architecture_v1.png`](week2_architecture_v1.png) (editable source: `week2_architecture_v1.drawio`)

> V1 is a **conceptual** architecture. Components are labelled by function. The specific models, retrieval method, storage and UI framework will be chosen in later topics.

---

## C1 — Simplest architecture

**Preparer → Lapora → Language model → Suggested tag**

- **Main user interaction:** a preparer enters SOFP / P&L line items and gets back suggested SSMxT concepts.
- **AI capability needed:** match the meaning of a company's own wording (e.g. *"Amount due from a director"*) to the right concept among about 2,375 MPERS concepts, and explain the match.
- **Why it is not enough:** a model on its own can name concepts that do not exist in SSMxT (RAI1), cannot cite the taxonomy entry (FR3), gives no signal when it is unsure (FR4 / RAI2), lets client data and advice requests through (RAI3 / RAI4 / FR5), and gives no way to measure accuracy (NFR1 / O3).

## C2 — Architecture needs from requirements

| Requirement ID | What does the requirement need? | Architecture Response |
|---|---|---|
| FR2 / RAI1 / NFR1 | Suggestions must be real SSMxT concepts and the correct one must usually appear in the top 3. | **Approved Knowledge Source** (SSMxT taxonomy + SSM/IFRS guidance) and a **Knowledge Access** capability that finds candidate concepts before the model ranks them |
| FR3 / RAI1 | Every suggestion needs a verifiable citation. | Knowledge Access returns the source entry with each candidate; **Output Validation** rejects any suggestion without one |
| RAI1 | No invented concept ID may reach the user. | **Output Validation**: deterministic check of every ID against the loaded taxonomy list |
| FR4 / RAI2 | Uncertain items must be flagged, not presented confidently. | **Output Validation** applies a confidence threshold and sets the "Needs review" flag |
| FR6 / RAI2 | No item may be exported without a human decision. | **Human Review & Confirmation** step between suggestions and export |
| FR5 / RAI4 | Out-of-scope inputs must be refused with a scope message. | **Input & Scope Check** before any model call |
| RAI3 | Amounts and client identifiers must not reach the model. | **Input & Scope Check** strips amounts and identifiers (data minimisation) |
| NFR1–NFR4 / RAI1–RAI5 / O3 | Accuracy, citations, abstention, latency, cost and fairness must be measurable. | **Evaluation & Logging** point plus **Labelled test sets** (T1–T3) |
| NFR2 | Responses must meet the latency threshold. | Short linear path; the model only ranks a small candidate list instead of the whole taxonomy |
| NFR5 | Taxonomy updates without code changes. | Knowledge source kept separate from application logic (re-load the source, not the code) |

## C3 — V1 diagram

See [`week2_architecture_v1.png`](week2_architecture_v1.png). Main flow:

**Preparer → Lapora Tagging Interface → Input & Scope Check → Knowledge Access (← Official taxonomy & guidance) → Language Model Service → Output Validation → Ranked Suggestions → Human Review & Confirmation → Reviewed mapping export**

Control and evaluation flows: out-of-scope inputs are refused back to the interface; invalid or uncited suggestions are dropped and re-ranked; the scope check, validation and review steps log to the Evaluation point, which scores results against the labelled test sets.

---

## C4 — Architecture Decision Record (ADR)

| Decision | Requirement / Constraint | Alternatives Considered | Selected Option | Trade-off Accepted |
|---|---|---|---|---|
| **ADR1 — Ground suggestions in the official taxonomy through a knowledge-access step** | FR2, FR3, RAI1, NFR1; LLMs are weak at fine-grained concept linking on their own (FinTagging, 2025) | (a) Base model only; (b) keyword search over taxonomy labels; (c) classifier fine-tuned on labelled data; (d) find candidate concepts by meaning, then let the model rank them | **(d) Knowledge access + model ranking** | More components to build and test. If the right concept is not among the retrieved candidates, the model cannot recover it, so retrieval recall caps top-3 accuracy. Fine-tuning (c) needs labelled data I do not have yet; it is left for the FYP. |
| **ADR2 — Separate, rule-based Output Validation instead of relying on prompt instructions** | RAI1 (0% invalid concepts), FR3, FR4 | (a) Instruct the model to use valid IDs and cite; (b) explicit validation component that checks IDs against the taxonomy list, requires a citation and applies a confidence threshold | **(b) Explicit validation** | Extra processing and a re-rank loop. Some correct suggestions will be flagged as low-confidence, which adds review work. In return, an invalid concept can never reach the user, which a prompt alone cannot guarantee. |
| **ADR3 — Human confirmation before export; no auto-accept** | FR6, RAI2; statutory filing, preparer legally responsible | (a) Auto-accept suggestions above a high confidence; (b) review only flagged items; (c) explicit decision on every item | **(c) Explicit decision on every item**, with flagged items highlighted | Slower than auto-accept, and users may still "click through". Accepted because a wrong tag in a statutory filing costs more than review time. Revisit (b) once evaluation shows how reliable high-confidence suggestions are. |
| **ADR4 — Data minimisation before the model** | RAI3 (0 identifiers or amounts in payloads); confidentiality of client data | (a) Send the full statement; (b) send label + statement + parent heading only; (c) local model only | **(b) Labels and statement context only**, keeping a locally hosted model as an option | The model loses the amount (sign and size can sometimes help, e.g. separating income from expense). A local model may be less accurate or slower than a hosted one. |
| **ADR5 — Suggest concepts only; no XBRL generation in V1** | Scope and time constraint (one semester); out-of-scope list | (a) Full pipeline to an XBRL file, as in the FYP; (b) tagging core only | **(b) Tagging core only** | The preparer still keys values into mTool by hand, so time saving is partial. Keeps V1 small enough to evaluate properly. |

---

## Initial Requirements Traceability Matrix

| Objective | Requirement | Evaluation Criterion | Architecture Element | Later Evidence |
|---|---|---|---|---|
| **O1** | FR2, FR3, RAI1 | Grounded, valid suggestions (0% invalid, 100% citation, ≥ 95% citation–concept match) | Official taxonomy & guidance; Knowledge Access; Output Validation | Automated ID check on T1; manual citation review of ≥ 50 suggestions |
| **O1** | FR3 | Explanation clarity (≥ 80% score 3 on rubric) | Language Model Service | Rubric scoring of ≥ 30 reasons |
| **O2** | FR4, RAI2 | Abstention quality (error-catch ≥ 80%, flag rate ≤ 30%) | Output Validation (confidence threshold); Human Review & Confirmation | Flag vs error analysis on T1; export test with unreviewed items |
| **O2** | FR5, RAI4 | Out-of-scope refusal (≥ 90% success, ≤ 5% false refusal) | Input & Scope Check | T2 scope test results |
| **O2** | RAI3 | Data minimisation (0 identifiers in payloads) | Input & Scope Check; Evaluation & Logging | T3 privacy test with payload logs |
| **O3** | NFR1 | Tagging accuracy (top-3 ≥ 85%, top-1 ≥ 70%) | Knowledge Access + Language Model Service; Labelled test sets | Accuracy report on T1 (real vs variant items) |
| **O3** | NFR2, NFR4 | Latency and cost (p90 ≤ 10 s; ≤ US$0.10 per statement) | Evaluation & Logging | Timing and usage logs over T1 |
| **O3** | RAI5 | Coverage fairness (gap ≤ 15 pp) | Labelled test sets (grouped); Evaluation & Logging | Per-group accuracy table |

---

## Final review checklist

| Question | Answer |
|---|---|
| Does each major component support a requirement or evaluation need? | Yes. Every added (dashed) component in the diagram shows the requirement IDs that justify it; see the C2 table. |
| Is any important data source, safety control, or evaluation point missing? | Data source, scope/privacy control, output validation, human review and evaluation are all present. **Open item:** the concept list must come from the SSMxT taxonomy files themselves, not only the Architecture Document; to confirm in Week 3. |
| Have I added any technology or component that is not yet justified? | No. No specific model, vector database, framework or UI library is named. FYP-only parts (trial-balance mapping, adjustments, XBRL generation, fine-tuning, monitoring dashboard) are deliberately left out. |
| Is there a simpler architecture that could still satisfy the requirements? | Input & Scope Check could be merged into the interface, but keeping it separate makes RAI3/RAI4 easier to test. Removing Knowledge Access or Output Validation would break RAI1 and FR3, so those stay. |

## Reflection

**Which architecture decision involved the most important trade-off? Which requirement drove it, and what trade-off did you accept?**

**ADR1 (knowledge access before model ranking)** has the most important trade-off. It was driven by **FR2 / RAI1** (only real SSMxT concepts) and **NFR1** (top-3 accuracy). Retrieving candidate concepts first means the model chooses from a short, real list with its source text, which makes citations possible and makes invented IDs easy to block. The cost is that the whole system now depends on retrieval: if the correct concept is not retrieved, no amount of model reasoning can fix it, so retrieval recall becomes the ceiling on accuracy. I accepted this because it is measurable — I can report retrieval recall@k separately in evaluation to see whether errors come from retrieval or from ranking — and because it is much easier to fix than a model that confidently invents concepts.

---

```bash
git add docs/week2_architecture_v1.png docs/week2_architecture_v1.drawio docs/week2_architecture_decisions.md
git commit -m "Week 2: Add V1 architecture, ADR, and traceability"
git push
```
