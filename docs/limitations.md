# Limitations

## Sample Size and Representativeness

This dataset contains 30 incidents. It is a research/demonstration dataset built to support MSc-level analysis and this Power BI portfolio piece, not a statistically representative or randomly sampled survey of cyber incidents in Ireland. No finding in this repository should be generalised to claim a rate, proportion, or pattern applies to Irish organisations, Irish cyber incidents, or any population beyond these 30 records. Where the README or `findings.md` state a percentage (e.g. "36.7% of these 30 incidents"), that percentage describes the dataset itself, not the wider world.

## Field Scale Inconsistency

Asset Value and Data Sensitivity are scored 3–9; User Role is scored 2–9 (with one decimal value present, 4.5); Alert Timing is scored 1–5. The MSc dissertation's narrative describes these as being on a consistent 1–10 scale, but Alert Timing in the actual source spreadsheet does not follow that range. Anyone using the Analysis Factor Field Parameter to compare across these four fields should be aware that Alert Timing's shorter scale is not directly comparable to the others without normalisation, which this dashboard does not apply.

## Incident Category Assignment

The `Incident Category` field is derived by keyword pattern-matching (`Text.Contains`) against the free-text `Type` column, not by manual expert classification of each incident. This works reliably for this dataset's 30 relatively distinct incident descriptions, but the approach has two known weaknesses:

- An incident involving multiple attack techniques (e.g. a phishing email that led to credential theft and later ransomware deployment) can only be assigned to one category, based on whichever keyword pattern matches first in the ordered logic.
- The taxonomy was designed against this specific dataset's vocabulary; applying the same Power Query logic to a differently-worded incident log would likely require adjusting or extending the pattern list.

## Taxonomy Divergence from the MSc Report

As documented in `methodology.md`, this dashboard's taxonomy keeps Credential-Based incidents as a distinct category, while the MSc dissertation's Fig 3.1 folded them into Phishing and Malware. Category-level percentages in this repository will not match Fig 3.1's percentages one-for-one for those three categories. This is a disclosed, deliberate difference, not a data error.

## Confidence Is Not Severity

The `Confidence` field (High/Medium/Low) describes how well-attributed or well-sourced an incident record is — essentially, how much trust to place in the reported details — not how severe the incident was or how well it was handled. The two fields are independent in the data model and should not be read as correlated or substitutable. A Low-confidence record can describe a SEV1 incident, and a High-confidence record can describe a SEV3 incident.

## PIR Relabeling

The source column header reads *"Priority Intelligence Requirements (PIR) Done"*, which was renamed to `Post-Incident Review Done` during Power Query (see `methodology.md` §4). The underlying Yes/No values were not altered — only the column label was corrected to match this project's intended meaning of PIR. Anyone tracing this dashboard's numbers back to the raw `Dataset.xlsx` file should expect to find the column under its original header text, not the renamed one.

## No Statistical Testing on the Field Parameter Comparison

The Analysis Factor visual on Page 2 allows visual comparison of average detection/response hours across Asset Value, Data Sensitivity, User Role, and Alert Timing. No regression, correlation coefficient, or significance test was run to establish whether any of these factors actually predicts detection or response time. Any pattern visible in that chart should be treated as a starting point for further investigation, not as a demonstrated relationship.

## Single Point-in-Time Snapshot

The dataset represents 30 incidents as a static snapshot; it does not track how an individual incident's status (documentation, review, confidence) may have changed over time between initial logging and eventual closure. The dashboard therefore reports current-state rates (e.g. Documentation Rate, Post-Incident Review Completion Rate) rather than time-to-completion metrics for those governance activities.