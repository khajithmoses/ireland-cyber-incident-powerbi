# Methodology

This document traces the full path from source spreadsheet to published dashboard: source validation, Power Query transformation, taxonomy design, field typing, the Field Parameter, and the validation checks run before any visual was built.

## 1. Source Validation

Before any transformation, `Dataset.xlsx` was checked field-by-field against the statistics reported in the MSc dissertation, to confirm the file being loaded into Power BI was the same dataset the original research was built on:

| Check | Result |
|---|---|
| Record count | 30 |
| Average detection hours | 18.4 |
| Average response hours | 5.583 |
| SEV1 count | 8 |
| Severity split (SEV1 / SEV2 / SEV3) | 26.7% / 46.7% / 26.7% |
| Documentation rate | 73.33% |
| Post-Incident Review rate | 30% |
| Null values | 0 |
| Duplicate IDs | 0 |
| Duplicate rows | 0 |

All figures matched. This step exists to catch a stale or corrupted source file before it propagates into every downstream measure — validating the dashboard's numbers after the fact is much harder than confirming the input is correct first.

## 2. Schema Notes

Two aspects of the raw schema needed to be understood correctly before modelling, rather than assumed from the dissertation narrative alone:

- **Field ranges are not uniform.** Asset Value and Data Sensitivity run 3–9; User Role runs 2–9 (with one decimal value, 4.5); Alert Timing runs 1–5. The dissertation narrative implies a 1–10 scale throughout, but Alert Timing in the actual source data is scored 1–5. This is documented rather than silently corrected, since it affects how the Field Parameter's axis values should be interpreted.
- **Column naming mismatch.** The source column is headed *"Priority Intelligence Requirements (PIR) Done"*, while this project (and the MSc framework it extends) uses PIR to mean *Post-Incident Review*, in line with standard IR terminology. See §4 below and the README's "Documented data decisions" section for the resolution.

## 3. Power Query Pipeline

Built in Power BI Desktop's Power Query Editor:

1. **Import** `Dataset.xlsx` → query renamed `CyberIncidents_Raw`. This query is left untouched after import (an auto-inserted `Changed Type` step from the import wizard was manually removed) so it remains a faithful, auditable copy of the source file.
2. **Reference, not Duplicate.** `CyberIncidents` is created as a *Reference* of `CyberIncidents_Raw`, not a *Duplicate*. A Reference builds on top of the Raw query's steps rather than copying them, keeping the relationship between source and transformed data explicit in the query dependency tree.
3. **Explicit type casting** (`Changed Type` step): ID, Type, Severity, Documented, Post-Incident Review Done, Confidence, Source → Text; Detection Hours, Response Hours, User Role → Decimal Number; Asset Value, Data Sensitivity, Alert Timing → Whole Number. (An earlier pass had incorrectly typed every column as Text; this was caught and corrected before proceeding.)
4. **Column rename.** The PIR column is renamed to `Post-Incident Review Done` at this stage — see §4.
5. **Text trimming** applied to `Type` and `Source` to remove inconsistent leading/trailing whitespace that would otherwise cause visually identical values to be treated as distinct categories.
6. **Custom column — `Incident Category`.** Added via M code using `Text.Contains` pattern matching against the `Type` field, in a deliberately ordered sequence of conditions (more specific patterns checked before broader ones) to avoid one category's keywords accidentally matching rows that belong to another. See §5.
7. **Close & Apply.**

## 4. PIR Column Resolution

The source column header — *"Priority Intelligence Requirements (PIR) Done"* — uses a different, though also legitimate, meaning of the PIR acronym than the one this project's framework relies on. Rather than leave the column under a label that would silently mismatch the dissertation's own definition of PIR as *Post-Incident Review*, the column was renamed to `Post-Incident Review Done` during Power Query. This is a disclosed transformation: the underlying Yes/No data is untouched, only the column's label was corrected to match the intended meaning used throughout this project and its source research.

## 5. Incident Category Taxonomy

The custom `Incident Category` column maps each incident's free-text `Type` field into one of nine categories via ordered `Text.Contains` matching:

| Category | Count |
|---|---|
| Phishing | 6 |
| Ransomware | 5 |
| Insider Threat | 4 |
| Cloud Incidents | 3 |
| Credential-Based | 3 |
| Malware | 3 |
| DoS/DDoS | 3 |
| SQL Injection | 2 |
| Web Defacement | 1 |
| **Total** | **30** |

This taxonomy deliberately keeps **Credential-Based** incidents (credential stuffing, brute-force attacks) as their own category, whereas the MSc dissertation's category breakdown (Fig 3.1) folded these into the broader Phishing and Malware categories. Both groupings are analytically valid; this dashboard uses the finer split because it makes credential-related attack patterns visible as their own line item rather than absorbed into larger categories. Anyone comparing this dashboard's category chart directly against the dissertation's Fig 3.1 should expect this difference and not read it as an inconsistency in the underlying data.

## 6. Data Model

A single flat table, `CyberIncidents`, with nine calculated measures layered on top (see `dax-measures.md`). No star schema or separate dimension tables were introduced — with one incident-level fact table and no repeating dimensions (dates, regions, multiple fact grains), a single-table model is the appropriate level of complexity for this dataset's size and shape, rather than an unnecessary star schema imposed for its own sake.

## 7. Field Parameter

A Field Parameter named **Analysis Factor** was created over four numeric fields — Asset Value, Data Sensitivity, User Role, Alert Timing — with "Add slicer to this page" enabled. This lets Page 2's risk-comparison chart swap its X-axis dynamically via a single slicer selection, instead of requiring four separate static charts (one per field) to cover the same comparison.

## 8. Validation After Modelling

Once the model and measures were built, the dashboard's own numbers were checked against the source-validation figures in §1 (unfiltered baseline: 30 total incidents, 18.40 average detection hours, 5.58 average response hours, 8 SEV1 incidents, 73.33% documentation rate, 30.00% Post-Incident Review rate) — confirming the DAX layer reproduces the same figures as the raw file, not a transformed or miscounted variant of them.