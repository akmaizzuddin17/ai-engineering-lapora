# Week 2 — Requirements Specification: Lapora

**Student:** Akma Izzuddin bin Azman
**Course:** AI Engineering — Topic 2 Lab, Part A (individual semester project)
**Version:** 1.0
**Builds on:** [`week1_project_scope.md`](week1_project_scope.md)

---

## Section 0 — Project Framing & Application Scope Checkpoint

| Element | Response |
|---|---|
| **Problem Evidence** | **E1 — Manual mapping is required by the regulator.** SSM's MBRS FAQ states that preparers must map each financial-statement line item to a taxonomy concept, that the *"auto population feature for financial statements is not available"*, and that items with no matching concept go under generic "Other" elements (SSM, *MBRS FAQ*, §1.7, §2.3.1.4, §2.3.1.11). <br>**E2 — Large search space and low practitioner readiness.** SSMxT 2022 has 8,531 concepts, about 2,375 of them for MPERS financial statements (SSM, *SSMxT 2022 Architecture Document*). A survey of Malaysian practitioners found that their understanding, readiness and intention to use MBRS still needed improvement (Ilias et al., 2020). <br>*Together:* preparers must make about 2,375-way choices by hand, and many lack the expertise to do it confidently. |
| **Existing Approaches** | **1. SSM mTool** (official, free): template forms, a taxonomy browser and structural validation. *Already does:* authoritative templates and catches structural errors such as an unbalanced SOFP. <br>**2. Outsourced XBRL conversion** (audit firms such as KPMG Malaysia; vendors such as DataTracks). *Already does:* expert-quality tagging as a paid service. <br>**3. LLM XBRL-tagging research** (FinTagging, Wang et al., 2025; XBRLTagRec, Hu et al., 2026). *Already does:* benchmarks and retrieval + LLM re-ranking for US GAAP tagging, and shows that LLMs extract values well but are weak at fine-grained concept linking. <br>**4. General-purpose chatbots** (observed in the Week 1 lab, ChatGPT). *Already does:* explains accounting terms in plain language. In Week 1 it gave confident answers without sources, so it is not grounded in the SSMxT taxonomy. |
| **Project / Engineering Gap** | A **practical limitation**, not a novelty claim: no available tool gives Malaysian MPERS preparers **grounded, verifiable concept suggestions restricted to real SSMxT concepts**. mTool is fully manual. Outsourcing is costly. Research systems target US GAAP and are not preparer tools. Chatbots give no citations and can name concepts that do not exist. The remaining gaps are **grounding, source traceability, and safe handling of uncertain items**. |
| **Objective O1** | **To design** a retrieval-grounded tagging assistant that recommends SSMxT (MPERS) concepts for Statement of Financial Position (SOFP) and Statement of Profit or Loss (P&L) line items, using only official SSM and IFRS taxonomy sources as its knowledge base. |
| **Objective O2** | **To develop** the assistant with a taxonomy knowledge pipeline, concept retrieval, LLM ranking, and guardrails: a valid-concept constraint, mandatory citation, low-confidence abstention, a scope check, and data minimisation. |
| **Objective O3** | **To evaluate** the assistant on a hand-labelled test set built from public illustrative MPERS financial statements, measuring top-1 / top-3 accuracy, citation groundedness, invalid-concept rate, abstention quality, scope refusal and latency. |
| **Intended Users** | **Primary:** junior accountants, audit associates and company-secretarial staff preparing MBRS financial-statement filings for private companies under MPERS. **Secondary:** accounting students learning MBRS, and SME owner-preparers. |
| **Supported Tasks** | 1. Suggest the top 3 SSMxT concepts for a single SOFP or P&L line item. <br>2. Process a list of line items (one statement) in one batch. <br>3. Explain each suggestion with a citation to the taxonomy entry and its source document. <br>4. Flag uncertain items, and items that likely belong under an "Other" element, for review. <br>5. Let the preparer accept, change or reject each suggestion, and export the confirmed mapping. |
| **Information / Knowledge Domain** | The SSMxT 2022 taxonomy, MPERS financial-statement entry point (concept IDs, standard labels, definitions, presentation hierarchy for SOFP and P&L); SSM MBRS guidance (FAQ, mTool manual); the IFRS for SMEs taxonomy (labels and references SSMxT builds on). |
| **Operating Context** | A desktop web tool used by a preparer **after** the financial statements are finalised and **before** keying the data into mTool. A typical session has 30–80 line items. The output is a draft mapping that a human reviews; the confirmed mapping is then used manually in mTool. |
| **Important Constraints** | **Time:** one semester, one student. **Budget/compute:** free or low-cost compute (Google Colab; optional local GPU); no paid enterprise services. **Data:** public documents only; no client or confidential data; SSM/KPMG documents referenced by URL, not redistributed. **Technical:** the taxonomy is published in XBRL format and needs parsing; English labels are primary. **Policy:** output is a drafting aid; the preparer and directors stay legally responsible for the filing. |
| **Explicit Out-of-Scope Tasks** | Generating the XBRL instance file; submitting to SSM / MBRS portal; MFRS entities; notes, text-block disclosures, statement of cash flows and statement of changes in equity; judging whether the financial statements are correct or compliant; giving accounting, tax, audit or legal advice; processing client-identifying data (company name, registration number, amounts are not needed). |
| **Evaluation Focus** | *Does Lapora place the correct SSMxT concept in its top 3 suggestions often enough, with a verifiable citation and never an invalid concept, that a junior preparer could tag faster without introducing more errors — and does it flag the items it is likely to get wrong instead of presenting them confidently?* |

**Traceability check:** O1 → FR2, FR3, RAI1 (grounded design). O2 → FR4, FR5, FR6, RAI2–RAI4 (guardrails built). O3 → NFR1–NFR4, RAI5 (measured on the labelled test set).

---

## Section 1 — Functional Requirements

| # | Requirement — The system SHALL | Scope Boundary — SHALL NOT (where relevant) |
|---|---|---|
| **FR1** | Accept SOFP or P&L line items, either one at a time or as a list, where each item has its label text, its statement (SOFP or P&L) and, optionally, its parent heading (e.g. *"Current assets"*), and return suggestions for every item. | Require amounts, company name, registration number or other client-identifying data to produce a suggestion. |
| **FR2** | Return up to 3 ranked SSMxT MPERS concepts for each line item, each showing the concept ID, its standard label and a confidence score. | Suggest any concept ID that does not exist in the loaded SSMxT taxonomy version or that sits outside the MPERS financial-statement entry point. |
| **FR3** | Give every suggestion a citation to its source (concept ID, label or definition, source document and taxonomy version) and a one-sentence reason linking the line item to the concept. | Present a citation or reason that is not supported by the retrieved taxonomy or guidance content. |
| **FR4** | Mark a line item **"Needs review"** when the top suggestion's confidence is below the set threshold, when no suitable concept is found, or when SSM guidance points to a generic "Other" element. | Silently present a weak best guess as if it were a confident answer. |
| **FR5** | Detect inputs outside the supported scope (notes / text-block disclosures, cash flow or equity statement items, MFRS-only items, requests for advice, unrelated questions) and reply with a short message stating what Lapora supports. | Give accounting, tax, audit or legal advice, or judge whether the financial statements are correct. |
| **FR6** | Let the preparer accept, change or reject each suggestion, and export the reviewed mapping (line item, chosen concept ID, citation, reviewer decision) as a table. | Mark any item as confirmed without an explicit user action, generate an XBRL instance file, or submit anything to SSM. |
| **FR7** | Show the taxonomy version used and a "drafting aid — preparer remains responsible" notice on every results screen and export. | — |

---

## Section 2 — Non-Functional Requirements

| # | Quality Attribute | Requirement | Measurement Method | Metric | Threshold |
|---|---|---|---|---|---|
| **NFR1** | Correctness | The correct SSMxT concept shall appear among the suggestions for most in-scope line items. | Run the hand-labelled test set (≥ 100 SOFP/P&L line items: real items from public illustrative MPERS statements plus realistic wording variants, reported separately) and compare suggestions with the gold concept ID. | Top-3 accuracy; top-1 accuracy | Top-3 ≥ 85%; top-1 ≥ 70% *(initial targets; revisit after the first baseline run)* |
| **NFR2** | Performance | Suggestions shall be returned fast enough for interactive use and for a full statement in one sitting. | Time each request end-to-end on the evaluation workload in the recorded reference environment. | p90 latency per line item; total time per 50-item statement | p90 ≤ 10 s per item; ≤ 5 min per 50-item statement |
| **NFR3** | Reliability | The same input shall give the same top suggestion and a well-formed output across repeated runs. | Run the full test set 3 times; check output schema and compare top-1 concepts across runs. | Top-1 agreement across runs; schema-valid output rate | Agreement ≥ 95%; schema-valid = 100% |
| **NFR4** | Cost | Processing one statement shall stay within a student / SME budget. | Record model usage (tokens or compute time) over the evaluation workload and convert to cost. | Cost per 50-item statement | ≤ US$0.10 per statement (US$0 marginal if run on a free/local model) |
| **NFR5** | Maintainability | A new taxonomy version or guidance document shall be loaded without changing application code. | Controlled update test: replace one source document / taxonomy file and re-run the knowledge pipeline. | Update time; code changes needed | < 30 min; 0 code changes |

---

## Section 3 — Responsible AI Requirements

| # | Responsible AI Risk | Requirement | Measurement Method | Metric | Threshold |
|---|---|---|---|---|---|
| **RAI1** | Hallucination / Transparency (wrong or invented tag reaches a statutory filing) | Every suggestion shall be a real SSMxT concept and shall carry a citation that supports it. | Check every suggested concept ID against the taxonomy list automatically; manually inspect cited sources for a sample of ≥ 50 suggestions. | Invalid-concept rate; citation presence rate; citation–concept match rate | 0% invalid; 100% presence; ≥ 95% match |
| **RAI2** | Automation bias / human oversight (junior preparers accept the top suggestion without checking) | The system shall flag the items it is likely to get wrong, and shall not export any item without an explicit reviewer decision. | Compare "Needs review" flags against top-1 errors on the test set; run an export test with unreviewed items. | Error-catch rate (share of top-1 errors that were flagged); flag rate (share of all items flagged); exports with unreviewed items | Error-catch ≥ 80%; flag rate ≤ 30%; 0 exports with unreviewed items |
| **RAI3** | Privacy / confidentiality (client financial data sent to an external model) | The system shall send only line-item labels and statement context to the language model — no amounts, company names or registration numbers. | Run ≥ 20 privacy test cases with planted amounts and identifiers; log and inspect every outgoing model payload. | Number of identifiers or amounts found in outgoing payloads | 0 |
| **RAI4** | Safety / scope (advice or answers outside the drafting-aid role) | The system shall refuse out-of-scope inputs and explain what it supports, without refusing valid line items. | Run a scope test set of ≥ 20 out-of-scope inputs and the in-scope test set. | Refusal-and-redirect success rate; false-refusal rate on in-scope items | ≥ 90% correct refusal; ≤ 5% false refusal |
| **RAI5** | Fairness / coverage bias (worse results for less common industries) | Accuracy shall not drop beyond the agreed tolerance for SOFP vs P&L items or for industry-specific items (e.g. biological assets, construction contracts). | Tag each test item by statement and by common vs industry-specific; compare accuracy per group. | Largest top-3 accuracy gap between groups | ≤ 15 percentage points |

---

## Reflection

**Which Responsible AI requirement will be the most difficult to measure or enforce technically? Why?**

**RAI2 (automation bias)** is the hardest. I can measure whether the system *flags* risky items (error-catch rate and flag rate), and I can block exports that have no reviewer decision. But the actual risk is human behaviour: a preparer can still click "accept" on every item without reading it. The metrics are proxies for real oversight. Measuring true over-reliance would need a user study (e.g. timing reviews, or seeding known-wrong suggestions to see if users catch them), which is outside this semester's scope. There is also a tension inside the requirement: flagging more items catches more errors, but if too many items are flagged, users stop paying attention to flags. That is why it has both an error-catch target and a maximum flag rate.

RAI1's citation–concept match is the second hardest, because checking whether a citation *supports* a tag needs accounting judgement and is done manually on a sample.

---

```bash
git add docs/week2_requirements.md
git commit -m 'Week 2: Requirements specification (FR, NFR, RAI)'
git push
```
