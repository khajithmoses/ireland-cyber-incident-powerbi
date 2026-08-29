# Findings

All figures below describe the 30 incidents in this dataset only. None of these statistics should be read as claims about the prevalence, distribution, or handling of cyber incidents across Ireland generally — the dataset is a research/demonstration sample, not a representative survey. See `limitations.md` for the full scope discussion.

## Operational Response

- Average detection time across all 30 incidents: **18.4 hours**. Average response time: **5.58 hours**.
- Detection and response by severity both increase from SEV1 through SEV3 in this dataset (visible in the "Detection & Response Hours by Severity" chart on Pages 1 and 2) — lower-severity incidents here are not detected or responded to noticeably faster than higher-severity ones, which is a pattern a lean SOC would likely want to investigate rather than assume triage is already working as intended.
- The Detection Hours vs. Response Hours scatter plot (Page 2) shows detection and response time broadly moving together across incidents, with SEV3 incidents (orange) appearing at some of the highest detection-hour values in the dataset.

## Incident Distribution

- Phishing (6) and Ransomware (5) are the two largest categories in this dataset, together accounting for **36.7%** of the 30 incidents.
- SEV2 is the largest severity group (14 of 30 incidents, 46.7%), roughly double the count of either SEV1 or SEV3 (8 each, 26.7%).
- The nine-category taxonomy (see `methodology.md` §5) spreads the remaining incidents across Insider Threat (4), Cloud Incidents (3), Credential-Based (3), Malware (3), DoS/DDoS (3), SQL Injection (2), and Web Defacement (1).

## Governance and Documentation

- Documentation Rate: **73.33%** — roughly three in four incidents in this dataset have a documented record.
- Post-Incident Review Completion Rate: **30.00%** — considerably lower. The dashboard's Page 3 makes this gap visible directly: the same incident can be marked "documented" without a completed post-incident review, meaning documentation practice in this dataset is materially ahead of review practice.
- Broken down by severity (Page 3's stacked columns), SEV2 — the largest severity group — also shows the **lowest share** of completed Post-Incident Reviews among the three severity tiers. In a real SOC, the highest-volume severity tier having the weakest review completion would be a natural place to focus governance effort, since it affects the largest number of incidents.

## Confidence Distribution

- Source-attribution confidence across the 30 incidents skews toward **High** and **Medium**, with comparatively few incidents rated **Low** confidence (visible on Page 3's Confidence Level Distribution chart). This reflects how well-sourced each incident record is, not how severe or how well-handled the incident itself was — the two should not be conflated (see `limitations.md`).

## Exploratory: Risk Factors vs. Response Performance

The Analysis Factor Field Parameter on Page 2 allows switching the risk-comparison chart's axis between Asset Value, Data Sensitivity, User Role, and Alert Timing. This is presented as an **exploratory comparison tool**, not as a validated statistical finding: no correlation or regression analysis was run to confirm which of these four factors, if any, is associated with detection or response time. The chart lets a user visually compare average detection/response hours across levels of each factor; drawing a firm conclusion from it (e.g. "higher asset value causes slower response") would require statistical testing beyond the scope of this dashboard.