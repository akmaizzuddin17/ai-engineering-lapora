# Week 4 — Data Quality & Bias Report: Lapora

**Student:** Akma Izzuddin bin Azman · **Course:** AI Engineering — Topic 4 Lab · **Audit date:** 22 Nov 2026
**Labels used:** **Observed** (I checked it) · **Inferred risk** (plausible, not proven) · **N.V.** (Not Verified)

---

## 1. Project and Data Overview

| Field | Evidence |
|---|---|
| Project | **Lapora**: retrieval-grounded assistant that suggests SSMxT (MPERS) concepts for SOFP / P&L line items, with citations and a "Needs review" flag. |
| Two Topic 2 requirements | **FR3** (every suggestion cites its source document and taxonomy version): needs source ID, page and version on every guidance passage. **FR4** (mark "Needs review" when SSM guidance points to a generic "Other" element): needs the FAQ "Other" rule available as complete, retrievable text. |
| Topic 3 pathway | A (PDF/TXT). Unit of inspection: **chunk** of prepared text. |
| Audited output | **Part C**, `data/pipeline_v1/documents_{faq,mtool,taxonomy_arch}/prepared_text.jsonl`. Matching manifests: `data/pipeline_v1/documents_*/pipeline_manifest.json`. These files are byte-identical to the Part B files under `data/processed/` (Topic 3 Cell 9B), so `data/week3_manifest.json` traces them to the same sources. |
| Sources | S01 `FAQ-MBRS.pdf` · S02 `User_manual_mTool_v2_FS2016.pdf.txt` · S03 `SSMxT2022_Architecture_Document.pdf.txt` (SHA-256 in `data/week3_manifest.json`). Official SSM publications. Rights: **N.V.** |
| Population and sample | **Population:** 744 chunks (S01 135, S02 176, S03 433), of which 66 are flagged table-of-contents, leaving 678 content chunks. V1, V2, uniqueness and coverage use the **whole population**. Accuracy uses a **sample of 15 chunks** (5 per source: 2 suspected-problem chunks chosen from the inspection, 3 random with seed 4). |
| Limits | Prepared text only. The SSMxT **concept list** needed for FR2 is not in any source, so it is **not audited** (no data exists yet). Page numbers of S02 and S03 follow the supplied text conversions, not the original PDFs (**N.V.**). |

**Checkpoint A1:** If the FAQ "Other" rule is cut or missing, FR4 cannot flag generic-element items and error-catch (O2) falls. If page or version is wrong, FR3 citations cannot be verified.

## 2. Five-Dimension Quality Scorecard

| Dimension | Finding | Sample / evidence | Impact | Decision |
|---|---|---|---|---|
| Completeness | The FR4 "Other" rule is present once (FAQ 2.3.1.4, page 25) but is **split across chunks `S01-0073` and `S01-0074`**. "Statement of financial position / SOFP" appears in only 1 FAQ chunk (S02: 3, S03: 4). The FR2 concept list is absent from all sources. | Whole population (keyword match) | A retrieved half-rule may not trigger "Needs review". FR2 cannot be tested on this data. | **Review** |
| Consistency | FAQ version conflict: cover says **Version 2.0 Jun 2022**; 39 of 42 page headers say **Version 1.0 Jun 2022**. Recorded as found in the manifest. | Topic 3 inspection (Cell A3) | Citation may name the wrong version (FR3). | **Warning** |
| Accuracy | Text of **15 / 15** sampled chunks found in the raw source. Page confirmed for **9 / 10** txt-source chunks by unique match; 1 is **N.V.** (2-character fragment `S02-0126` matches everywhere). | Sample n = 15 (not the whole dataset) | Prepared text is faithful to the supplied files. Original-PDF page numbers for S02/S03 are N.V. | **Pass** (S01) · **N.V.** (S02/S03 page = original PDF) |
| Timeliness | S01 Jun 2022 (or 2.0), S02 v2.4 Sept 2024, S03 SSMxT_2022 v1.0, no document date. Whether newer versions exist: **N.V.** (not checked; the shell cannot reach ssm.com.my). | `docs/week3_data_inventory.md` | Guidance may be superseded. | **N.V.** |
| Uniqueness | **0** duplicate record IDs; **0** duplicate texts among 678 content chunks; **0** chunks duplicated across sources (>80 chars). | Whole population | No duplicate evidence inflating retrieval. | **Pass** |

## 3. Validation Findings

| Rule | Risk / requirement | Condition | Tested | Observed | Action if failed |
|---|---|---|---|---|---|
| **V1** Source reference complete | Uncitable passage, FR3 / RAI1 | `source_id`, `source_file`, `source_version` non-empty; `page` an integer inside the source's real page range; `source_sha256` equals the manifest value; text non-empty | 744 chunks | **744 pass, 0 fail** | Hard fail: quarantine the chunk |
| **V2** No word cut at a chunk boundary | Topic 3 decision D2 risk (fixed-size fallback for paragraphs over 500 characters) → incomplete FR4 rule, RAI1 | A chunk boundary created by the fixed-size fallback must not fall inside a word (alphanumeric on both sides) | 744 chunks | **252 boundaries fail** (33.9%): S01 11, S02 45, S03 196. Example: `S01-0073` → `S01-0074` splits the "Other" rule mid-sentence. | Hard fail for the retrieval corpus: re-chunk before embedding (Section 7) |

V2 is a **defect in the preparation method**, not bad source text. The evidence is the Topic 3 comparison (Cell 5A1) and the recount above.

## 4. Duplicate, Anomaly and Coverage Findings

**Duplicates:** none found (Section 2).
**Anomalies (Observed):** 19 of 678 content chunks are shorter than 40 characters (S01 1, S02 6, S03 12), e.g. `S02-0126` ("he", 2 characters) and `S03-0051` ("ns").", 5 characters), which are word fragments from V2. `S01-0126` ("d) Others (if any) PDF") is a genuine table cell, so it is an **exception, not a defect**; none were removed. Lowercase-start chunks: 17/114 (S01), 60/157 (S02), 258/407 (S03).

| Case needed | Expected evidence | Observed (content chunks) | Possible effect |
|---|---|---|---|
| FR4: "Other" element guidance | Complete FAQ rule | 1 FAQ section (p25), split over 2 chunks (`S01-0073`, `S01-0074`). The word "Other" appears in 14 / 14 / 61 content chunks (S01 / S02 / S03). | A retrieval miss on either half weakens the "Needs review" trigger. |
| SOFP and P&L statements | Guidance for both statements | SOFP term: 1 / 3 / 4; P&L term: 3 / 3 / 3 (S01 / S02 / S03). Keyword presence only; I did not read for adequacy. | Thin FAQ coverage for the task's two core statements. |

Limit: keyword counts show presence, not quality.

## 5. Bias Assessment

- **Pipeline-induced (Observed cause, Inferred effect):** paragraphs over 500 characters are cut by fixed slicing (S01 11, S02 47, S03 108 paragraphs; 252 mid-word cuts). The cut falls where the text is longest, which is often where explanations are given. **Effect:** rules and definitions may be retrieved as fragments, so FR3 citations support less and FR4 flags are missed. **Action:** re-chunk (sentence-aware with overlap); re-run V2.
- **Representation (Inferred risk):** S03 (architecture document) is **60.0%** of content chunks (S02 23.2%, S01 16.8%). **Effect:** retrieval for a line item may favour taxonomy-structure wording over the FAQ rules that FR4 depends on. **Action:** keep `source_id` as a filter and report source mix per retrieved top-k in Topic 5+.
- **Historical:** all guidance dates from 2022 to 2024 (Inferred risk: later rule changes are not reflected).
No claim is made about system fairness or accuracy; that is measured later (O3 / RAI5).

## 6. Privacy, Rights and Governance

| Question | Record |
|---|---|
| Provenance / version | Source IDs S01–S03, owner SSM, checksums in `data/week3_manifest.json`; versions as declared (FAQ conflict flagged). |
| Rights | Public SSM documents. Licence or permission for reuse: **N.V.** Sharing: **N.V.** |
| Privacy | **Observed:** no NRIC-like numbers. 3 organisational email addresses (`enquiry@ssm.com.my`, `refund@ssm.com.my`, `customercare@digicert.com.my`) and 2 phone numbers in S03. Whether they identify individuals: **N.V.** They are not needed for Lapora. |
| Consent / basis | Not applicable: public guidance, no data collected from individuals. |
| Maintenance | Owner: me (Akma). Revalidate on a new SSM document version or any change to the chunking rules. |
| Safeguard (required) | Raw files and prepared text stay out of GitHub (`.gitignore`); only manifests and checksums are committed. Strip or exclude contact lines before the index is built. |

## 7. Mitigation and Impact on Requirements

| # | Priority improvement | Owner / when | Requirements affected |
|---|---|---|---|
| 1 | Replace the fixed-size fallback with sentence-aware splitting plus overlap; re-run V1 and V2 (target 0 mid-word cuts) and confirm `S01-0073/0074` are retrievable as one rule. | Akma, **before Topic 5 embeddings** | FR3, FR4, RAI1; O1 citation validity, O2 error-catch |
| 2 | (a) Ask SSM which FAQ version is current and whether newer versions of S01–S03 exist; confirm reuse rights. (b) Add the SSMxT XBRL concept list as a new audited source. | Akma (with supervisor), before any FR2 or NFR1 testing | FR2, FR3, NFR1, RAI1 |

**Residual risks:** guidance may be out of date (N.V.); S02/S03 pages may differ from the original PDFs (N.V.); sample accuracy (n = 15) does not establish whole-corpus accuracy; keyword coverage is not adequacy.

## 8. Quality-Gate Decision

**GO WITH CONDITIONS** for the scope audited: the three prepared guidance-text files (S01–S03), for **internal** use in the next stage. This does **not** cover the FR2 concept list, which has no data yet, and does not allow sharing the texts.

**Evidence:** V1 passed 744/744, uniqueness is clean, and 15/15 sampled chunks match the source. **But** V2 failed on 252 boundaries, including the FR4 rule.
**Conditions:** (1) pass V2 after re-chunking before embeddings; (2) rights and version confirmation before anything is shared or cited as authoritative.
**Re-review triggers:** new SSM document versions; any change to chunking; adding the concept list; V1 or V2 failing on re-run.

---

### Appendix — evidence
- Topic 3: `notebooks/week3_pipeline.ipynb` (Cells A3, 5A1, 6B, 9), `docs/week3_data_inventory.md`, `docs/week3_pipeline_decisions.md`, `data/week3_manifest.json`.
- Topic 4 checks (recomputed from the Part C JSONL with the Topic 3 functions; no new pipeline): V1 744/744; V2 252 (11 / 45 / 196); duplicates 0; accuracy sample `S01-0073, 0126, 0052, 0060, 0035`; `S02-0126, 0027, 0122, 0144, 0060`; `S03-0051, 0169, 0074, 0062, 0037`.
