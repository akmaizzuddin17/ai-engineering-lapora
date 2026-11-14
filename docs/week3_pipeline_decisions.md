# Week 3 — Pipeline Decisions: Lapora

**Student:** Akma Izzuddin bin Azman · **Course:** AI Engineering — Topic 3 Lab
**Diagram:** [`week3_pipeline_diagram.png`](week3_pipeline_diagram.png)

## Decisions

| # | Decision | Alternatives | Evidence | Why |
|---|---|---|---|---|
| D1 | **Pathway A (PDF/TXT)** for all three sources | Tabular, image pathways | All sources are prose guidance documents | Lapora must cite guidance text (FR3). The taxonomy concept list (XBRL) fits no lab pathway and is deferred. |
| D2 | **Paragraph-aware chunks within each page**, 500 characters | Fixed 500-character slices; paragraph-aware over the whole document | Mid-word cuts on S01: 66 (fixed) vs 11. On S03: 237 vs 196. Chunks starting lowercase on S01: 78 vs 21. Only the per-page method keeps a page for every chunk (135/135, 176/176, 433/433). | Fixed slices cut words and lose the page. Page-aware chunking costs a few sentences split across pages (4, 5 and 25 pages begin mid-sentence), which I accept for citations. |
| D3 | **Keep page numbers** (PDF-native; recovered from page header for TXT) | Lab default `page = None` | Page numbers strictly increasing in all three sources | FR3 needs a citable page. |
| D4 | **Remove repeated page headers and extra whitespace only** | Aggressive cleaning; no cleaning | S01 header lines are boilerplate on every page | Content lines are never dropped. The rule is stored per source. |
| D5 | **Flag table-of-contents chunks (`is_toc`), do not delete** | Delete them | Dotted-leader pages: S01 2–4, S02 2–5, S03 3–6 | A flag is reversible and lets retrieval filter them. |
| D6 | **Batch ETL**, not streaming, not ELT | Streaming; load raw first then transform | Three static documents, versioned a few times a year | Changes are rare, so a re-run is enough. Transforming before storage keeps the stored chunks small and ready for later steps. |
| D7 | **One prepared file and one manifest per source** | One merged file | Sources have different versions and page-recovery methods | Per-source lineage; a new version reloads one source (NFR5). |
| D8 | **Record declared versions as found, with a flag** | Pick one version silently | FAQ says 2.0 on the cover and 1.0 in 39 page headers | Not hidden, not guessed. |
| D9 | **Do not commit raw or prepared text** | Commit everything | Redistribution terms not verified | Notebook regenerates both; manifests with checksums are committed. |

## Stage contracts

| Stage | Input | Output | Checked by | Stops when |
|---|---|---|---|---|
| Ingest | Exact relative path | One file path | `resolve_source` | File missing |
| Inspect | File | Pages, characters, low-text and TOC pages | `inspect_documents` | No text, or page numbers not increasing |
| Prepare | File + config | Chunk records | `build_records` | No chunks |
| Validate | Prepared file | Pass / fail | B3 contract + content check | Missing, empty, chunk without page, "Other" rule not found |
| Store | Records | JSONL | `write_jsonl` | — |
| Record lineage | Source + output | Manifests with SHA-256 | Cells 6B and 9 | Hash mismatch |

## Evidence of repeatability
- Cell 6B: `COMMON LINEAGE VALIDATION: PASS`, 3 records.
- Cell 9: all three pipelines passed (source fingerprints, output hashes and mappings match on re-run).
- Cell 9B: missing input and empty text both stop the pipeline; Part B and Part C outputs are byte-identical for all three sources.

## Limitations
- Validation checks structure and one known rule, not retrieval quality. That is measured later against the Week 2 criteria.
- S02/S03 are text conversions; table layout may be lost.
- Version labels come from the documents themselves and are not confirmed with SSM.
- Notebook outputs in the repo come from a sandbox run; re-run in Colab for a Colab-run record.
