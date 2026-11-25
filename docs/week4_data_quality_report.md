# Week 4 Data Quality Report: Lapora (Topic 4)

**Project:** Lapora, an MBRS XBRL tagging assistant (SSMxT / MPERS concepts, citations, abstention)
**Author:** Akma Izzuddin bin Azman · **Input:** Topic 3 hand-off, 744 chunks from 3 SSM documents
**Evidence:** `notebooks/week4_quality.ipynb`, `docs/week4_validation_failures.csv`, `docs/week4_duplicate_review.csv`, `docs/week4_quarantine.csv`, `docs/week4_coverage_by_source.png`
**Reviewed corpus:** 678 chunks (`data/prepared_chunks_clean.jsonl`, `data/chunks_clean.pkl`; reloaded and verified)

## 1. Data Quality Scorecard

Rating 1 (poor) to 5 (strong), or **N.V.** (Not Verified). Ratings rest on the evidence shown, not on a pass rate.

| Dimension | Rating | Evidence | Limit of the evidence |
|---|---|---|---|
| Completeness | 3 | 0 missing text, 0 missing provenance (R1, R2). But 19 content chunks are under 40 characters (R5) and the FAQ "Other" rule is split over two chunks (S01-0073/0074) | Checked structure, not whether every needed concept is present |
| Accuracy | 3 | 15 of 15 sampled chunks found in the raw source (S01 also on the stated page); 0 corrupted characters (R4) | Small sample (5 per source) |
| Consistency | 2 | FAQ carries two version strings (39 pages "1.0 Jun 2022", 3 pages "2.0 Jun 2022"; PDF title "update v2 31052022"); 258 chunks (34.7%) cut a word at the boundary (R6) | Mid-word count is a heuristic; Topic 3 counted 252, a gap of 6 not explained (N.V.) |
| Timeliness | N.V. | Versions known: mTool v2.4 (Sept 2024), SSMxT_2022 v1.0 (no date), FAQ conflicting | Not compared with newer SSM publications (site unreachable from my environment) |
| Uniqueness | 4 | 0 exact duplicates, 0 shared 100-character openings | Exact hashing is not semantic uniqueness; paraphrased overlap between FAQ and mTool is not measured |
| Coverage | 2 | Corpus is 60.0% S03, 23.2% S02, 16.8% S01; core statements thin (14 chunks mention financial position, 9 profit or loss) | Keyword proxy, not concept-level coverage |
| Provenance | 4 | SHA-256, version and URL recorded for all three sources | Rights terms N.V. |

## 2. Validation and Data-Defect Findings

| Rule | Failures / 744 | Decision |
|---|---|---|
| R1 text present | 0 | Hard fail |
| R2 provenance complete | 0 | Hard fail |
| R3 page in range | 0 | Hard fail |
| R4 no corrupted characters | 0 | Hard fail |
| R5 content chunk ≥ 40 characters | 19 (S01 1, S02 6, S03 12) | Review |
| R6 no word cut at boundary | 258 (S01 11, S02 49, S03 198) | Hard fail (pipeline defect); kept and flagged |
| R7 not a contents chunk | 66 (S01 21, S02 19, S03 26) | Quarantine |
| R8 no noisy whitespace | 0 | Warning |

The text is clean at character level; the defects are structural (500-character hard cuts). Also found: 53 S03 chunks where a long element name is wrapped across lines and split. Triage of 5 cases (retain / correct / remove / quarantine / review) is in the notebook, section A5. **Pandera was not available in my environment; R1-R4 ran in pandas, and the Pandera path is unverified.**

## 3. Duplicate and Anomaly Findings

- Exact-hash duplicates: **0**. Shared 100-character openings: **0**.
- This does not prove semantic uniqueness: the FAQ and mTool manual describe overlapping filing steps in different words.
- Anomalies: 3 organisational emails and 3 phone numbers in S01-S03 text (flagged `contact_detail`); chunk S01 email list ends with a stray full stop; FAQ version conflict.

## 4. Representation, Coverage, and Bias Assessment

Potential representation risks only; no fairness claims (Topic 8).

| Risk | Evidence | Propagation path | Mitigation / test |
|---|---|---|---|
| Source imbalance (S03 = 60.0%) | 407 / 157 / 114 chunks | Source selection → S03 dominates corpus → S03 dominates retrieval → fewer plain FAQ-style answers | Per-source cap; test top-5 source share on 20 FAQ questions |
| Unequal boundary damage | Mid-word rate S03 46.4%, S02 29.9%, S01 7.0% | 500-character cut → S03/S02 chunks damaged more → worse citations for those topics | Sentence-aware chunking; target < 5% per source |
| Version / historical bias | 2022 and 2024 guidance only | Old versions → corpus → outdated retrieval → wrong tag suggested | Version check; show version in every citation |

Unequal chunk counts are a signal, not proof: S03 is the longest document. Coverage vs scope: MPERS 33 and MFRS 37 chunks; the "Other" rule has only 2 chunks (both damaged); the SSMxT concept list itself is not in the corpus.

## 5. Privacy, Consent, Rights/Access, Governance

| Item | Record |
|---|---|
| Owner | Student (Akma Izzuddin); source owner is Suruhanjaya Syarikat Malaysia (SSM) |
| Version / provenance | FAQ conflicting versions; mTool v2.4 Sept 2024; SSMxT_2022 v1.0; SHA-256 and URL stored in `data/week3_manifest.json` |
| Rights / access | Publicly accessible documents. **Redistribution terms N.V.** Publicly accessible does not mean public domain |
| Personal data | 0 NRIC-like numbers; 3 organisational emails and 3 phone numbers, not needed for tagging, to be excluded before indexing |
| Consent basis | Not applicable (no individuals' data) |
| Update / retention | Re-validate on a new SSM version or any chunking change; raw files stay out of Git |

**Open issue:** this lab requires committing the chunk files, which contain SSM text. This conflicts with my Week 3 decision not to publish source text. The files can be removed from Git, or the repository made private, if SSM terms do not allow redistribution.

## 6. Mitigation Plan and Residual Risk

| Issue | Evidence | Action | Owner / timing | Residual risk |
|---|---|---|---|---|
| Mid-word chunk cuts | R6: 258 (34.7%) | Re-chunk sentence-aware with overlap; re-run R6 | Student, before Topic 7 (Week 9) | Medium until fixed; flagged meanwhile |
| FAQ "Other" rule split | S01-0073/0074 | Re-chunk so the rule stays whole | Student, with re-chunking | High for FR4 if left |
| Contents pages | R7: 66 chunks | Quarantined (kept in log, not indexed) | Done | Low |
| Short fragments | R5: 19 | Review after re-chunking | Student | Low |
| FAQ version conflict / timeliness | 1.0 vs 2.0 strings; newer versions N.V. | Confirm with SSM; check newer versions | Student, before final evaluation | Medium |
| Source imbalance | 60.0% S03 | Per-source retrieval cap; add concept list as audited source (FR2) | Student, Topic 7 | Medium |
| Rights N.V. | No terms found | Confirm reuse terms; remove files from Git if needed | Student, now | Medium |

## 7. Impact on Topic 2 Requirements / Evaluation Criteria

- **FR3 (citations):** provenance is complete and pages are valid, so citations are possible. Half-word chunks weaken citation quality for S03/S02.
- **FR4 (Needs review / "Other" rule):** the rule's evidence is split, so abstention tests on "Other" should be expected to fail until re-chunking.
- **RAI1:** timeliness is N.V., so Lapora should show the document version and say it may be outdated.
- **NFR5:** the 15-chunk accuracy sample passed, but the sample is small.
- Evaluation: add per-source retrieval share and R6 rate as tracked metrics.

## 8. Quality Gate Decision

**GO WITH CONDITIONS** (internal use and development only).

Conditions before Topic 7 indexing:
1. Re-chunk (sentence-aware, overlap) and pass R6 (< 5% per source), keeping the "Other" rule whole.
2. Confirm the FAQ version, check for newer SSM versions, and confirm reuse rights.
3. Add the SSMxT concept list as an audited source (FR2).

Unresolved issues stay in the record: mid-word cuts (flagged in `quality_flags`), timeliness N.V., rights N.V.
