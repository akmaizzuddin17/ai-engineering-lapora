# Lapora — AI-Assisted MBRS XBRL Tagging Assistant

> Suggests the correct SSM taxonomy (SSMxT) concepts for each line item in a Malaysian private company's MPERS financial statements, with citations to official SSM/IFRS documents, for preparers filing through SSM's MBRS.

## Problem Statement

Malaysian companies must file their financial statements with SSM in XBRL. Preparers have to map every line item to a concept in the SSM Taxonomy by hand: about 2,375 concepts for MPERS alone, with no automatic mapping in SSM's mTool. Because company wording rarely matches taxonomy labels, keyword search is not enough. A retrieval-grounded AI assistant can suggest likely concepts, cite the evidence for each one, and flag uncertain items for human review.

## Evidence of the Problem

- SSM states that preparers must map line items to taxonomy concepts and that *"auto population feature for financial statements is not available"*. Items with no matching concept go under generic "Other" elements. — SSM, *MBRS FAQ*, §1.7, §2.3.1.4, §2.3.1.11 ([link](https://www.ssm.com.my/Pages/Register_Business_Company_LLP/Company/document/FAQ-MBRS.pdf))
- Malaysian practitioners' understanding, readiness and intention to use MBRS need improvement, and discomfort and insecurity affect adoption. — Ilias et al. (2020), *Humanities and Social Sciences Letters, 8*(2), 202–214 ([doi](https://doi.org/10.18488/journal.73.2020.82.202.214))
- SSMxT 2022 contains 8,531 reporting concepts, about 2,375 of them for MPERS financial statements. — SSM, *SSMxT 2022 Architecture Document* ([link](https://www.ssm.com.my/Pages/Register_Business_Company_LLP/Company/document/SSMxT2022_Architecture_Document.pdf))

## Initial Related Work / Existing Approaches

- **SSM mTool** (official, free Excel tool) — [User Manual](https://www.ssm.com.my/Pages/Register_Business_Company_LLP/Company/document/User_manual_mTool_v2_FS2016.pdf). *Limitation:* concept selection is fully manual, and its validation catches structural errors only, not a wrong concept choice.
- **Outsourced XBRL conversion** (audit firms and vendors, e.g. [KPMG Malaysia](https://assets.kpmg.com/content/dam/kpmg/my/pdf/simplifying-your-mbrs-transition-2026.pdf), [DataTracks](https://www.datatracks.com/?p=22944)). *Limitation:* paid and human-bound, so costly for SMEs, and the process is still largely manual.
- **LLM XBRL-tagging research**: *FinTagging* ([Wang et al., 2025](https://arxiv.org/abs/2505.20650)) and *XBRLTagRec* ([Hu et al., 2026](https://arxiv.org/abs/2603.25263)). *Limitation:* LLMs struggle with fine-grained taxonomy concept linking, both target US GAAP rather than SSMxT/MPERS, and neither is a preparer-facing tool.

## Engineering Gap / Limitation

No existing approach gives Malaysian preparers grounded, explainable concept suggestions for the SSMxT taxonomy. Lapora restricts suggestions to real SSMxT concepts, cites the taxonomy entry and SSM guidance behind each one, and abstains or flags an item for review when confidence is low.

## Project Objectives

- **O1:** To design a retrieval-augmented tagging assistant that recommends SSMxT (MPERS) concepts for financial-statement line items, grounded only in official SSM and IFRS taxonomy documents.
- **O2:** To develop the assistant with a document pipeline, vector retrieval, LLM ranking, and guardrails (valid-concept constraint, mandatory citation, low-confidence abstention, scope check).
- **O3:** To evaluate top-1 / top-3 accuracy, groundedness, invalid-concept rate, abstention quality, and latency on a hand-labelled test set built from public illustrative MPERS financial statements.

## Target Users

Junior accountants, audit associates and company-secretarial staff preparing MBRS financial-statement filings for private companies under MPERS, plus accounting students learning MBRS.

> ⚠️ Lapora is a **drafting aid**, not accounting or legal advice. A qualified preparer stays responsible for every filing.

## Data Sources

- SSMxT 2022 Architecture Document (SSM) — https://www.ssm.com.my/Pages/Register_Business_Company_LLP/Company/document/SSMxT2022_Architecture_Document.pdf
- MBRS FAQ (SSM) — https://www.ssm.com.my/Pages/Register_Business_Company_LLP/Company/document/FAQ-MBRS.pdf
- mTool v2 User Manual, Financial Statements CA 2016 (SSM) — https://www.ssm.com.my/Pages/Register_Business_Company_LLP/Company/document/User_manual_mTool_v2_FS2016.pdf
- IFRS Accounting Taxonomy, IFRS for SMEs (IFRS Foundation) — https://www.ifrs.org/issued-standards/ifrs-taxonomy/
- *Evaluation input (not committed):* KPMG *Wonderful SME Sdn Bhd Illustrative Financial Statements (MPERS)* — https://assets.kpmg.com/content/dam/kpmg/my/pdf/Wonderful-SME-Sdn-Bhd-Illustrative-Financial-Statements-2016.pdf

Source documents are referenced by URL and are not redistributed here. No client or confidential financial data is stored in this repository.

## Repository Structure

```
docs/         Week-by-week engineering documents (observation sheet, system map, scope, ...)
data/         Data documentation only (raw source files are kept local, not committed)
src/          Pipeline and application code
notebooks/    Colab notebooks (from Week 3)
evaluation/   Labelled test set and evaluation results
prompts/      Versioned prompt templates
```

## Week 1 Documents

- [Observation sheet — ChatGPT (group, Part A)](docs/week1_observation_sheet.md)
- [Conceptual AI system map — ChatGPT (group, Part B)](docs/week1_system_map.png)
- [Project scope — Lapora (individual, Part C)](docs/week1_project_scope.md)

## Weekly Progress

| Week | Focus | Deliverable | Status |
|------|-------|-------------|--------|
| W1 | System map + project scope | README + docs | ✅ |
