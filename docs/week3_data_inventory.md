# Week 3 — Data Inventory: Lapora

**Student:** Akma Izzuddin bin Azman · **Course:** AI Engineering — Topic 3 Lab · **Pathway:** A (PDF / TXT)

All three sources are official SSM publications. Raw files are kept local (`data/raw/`, git-ignored) because redistribution terms were not verified. Figures below come from the executed notebook `notebooks/week3_pipeline.ipynb`.

## Source register

| ID | File | Type | Owner | Pages | Characters | SHA-256 (first 12) | Declared version |
|---|---|---|---|---|---|---|---|
| S01 | `FAQ-MBRS.pdf` | PDF | SSM | 43 | 58,285 | `ef1fb2647ca1` | Title/cover: Version 2.0 Jun 2022. 39 of 42 page headers say Version 1.0 Jun 2022 (inconsistent, recorded as found) |
| S02 | `User_manual_mTool_v2_FS2016.pdf.txt` | TXT conversion of a PDF | SSM | 105 | 63,881 | `157e9fabaf78` | Version 2.4 September 2024 |
| S03 | `SSMxT2022_Architecture_Document.pdf.txt` | TXT conversion of a PDF | SSM | 111 | 187,796 | `a2d15a2f3b5a` | Taxonomy SSMxT_2022 v1.0; document date not stated |

S02 and S03 are text conversions supplied by me, not the original PDFs. Page numbers were recovered from the repeated page header (strictly increasing, no gaps).

## Requirement → data mapping

| Requirement | Information needed | Source | Status |
|---|---|---|---|
| FR3 (citations) | Source, page and version per passage | S01, S02, S03 | Covered: every chunk carries `source_id`, `page`, `source_sha256`, `source_version` |
| FR4 (flag "Other" elements) | FAQ rule about "Other" elements | S01 | Covered: found in chunk `S01-0073`, page 25 |
| RAI1 / NFR5 (grounded, reloadable) | Checksums, version, processing record | All | Covered by the two manifests |
| FR2 (the ~2,375 MPERS concepts) | Concept list | SSMxT taxonomy XBRL files | **Not covered.** No lab pathway parses XBRL/XML. This is the next source. |

## Inspection findings

| Check | S01 | S02 | S03 |
|---|---|---|---|
| Low-text pages (<100 chars) | 1 (cover) | 24, 25, 26, 46, 98 | 1 (cover) |
| Table-of-contents pages | 2–4 | 2–5 | 3–6 |
| Repeated header lines removed | 2 rules (title; version + page no.) | page-header line used as page marker | page-header line used as page marker |
| Characters removed (headers + whitespace) | 6,312 | 889 | 1,700 |
| Pages starting mid-sentence | 4 | 5 | 25 |

Issues that affect later steps:
1. **FAQ version inconsistency** (above). Recorded as found; to confirm with SSM before it is cited as authoritative.
2. **S03 is a technical architecture document**, useful for taxonomy structure and terms, not a concept list.
3. **S02 has five near-empty pages** (screenshots in the original); no text is lost by keeping them.
4. S02 describes the mTool Auto Tag feature in v2.4. My earlier Week 1/2 wording that tagging in mTool is "fully manual" needs correcting.

## Prepared output

| Source | Chunks | Chunks flagged as table of contents | Prepared file |
|---|---|---|---|
| S01 | 135 | 21 | `data/processed/prepared_text_S01_faq.jsonl` |
| S02 | 176 | 19 | `data/processed/prepared_text_S02_mtool.jsonl` |
| S03 | 433 | 26 | `data/processed/prepared_text_S03_taxonomy_arch.jsonl` |

Record fields: `record_id, source_id, source_file, source_sha256, source_version, page, is_toc, processing, chars, text`. Table-of-contents chunks are flagged, not deleted; retrieval should exclude them.

## Lineage and rights
- Part B manifest: `data/week3_manifest.json` (3 records, verified by Cell 6B).
- Part C manifests: `data/pipeline_v1/documents_{faq,mtool,taxonomy_arch}/pipeline_manifest.json`.
- Access rights: public SSM documents, redistribution not verified. Raw files and prepared text are not committed; the notebook regenerates them.
