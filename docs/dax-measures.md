# DAX Measures

All nine measures live in a single table, `CyberIncidents`. This document explains not just what each measure does, but the filter-context reasoning behind the less obvious ones — the `+ 0` fix and the `REMOVEFILTERS` rate pattern — since those are the parts that distinguish a measure written with an understanding of DAX's evaluation model from one that happens to produce the right number by luck.

## Simple aggregates

```dax
Total Incidents = COUNTROWS(CyberIncidents)
Average Detection Hours = AVERAGE(CyberIncidents[Detection Hours])
Average Response Hours = AVERAGE(CyberIncidents[Response Hours])
```

Nothing unusual here — these three respond to whatever filter context (slicers, cross-filtering) is active on the page, which is exactly the desired behaviour for KPI cards and axis values.

## Conditional counts, and the blank-vs-zero problem

```dax
SEV1 Incidents =
    CALCULATE(COUNTROWS(CyberIncidents), CyberIncidents[Severity] = "SEV1") + 0

Documented Incidents =
    CALCULATE(COUNTROWS(CyberIncidents), CyberIncidents[Documented] = "Yes") + 0

Post-Incident Reviews Completed =
    CALCULATE(COUNTROWS(CyberIncidents), CyberIncidents[Post-Incident Review Done] = "Yes") + 0
```

Each measure wraps `COUNTROWS` inside `CALCULATE` with an explicit boolean condition. In DAX, a boolean filter argument passed directly into `CALCULATE` (e.g. `CyberIncidents[Severity] = "SEV1"`) doesn't just add a filter — it **replaces** any existing filter on that same column that came from the surrounding context (a slicer, a visual's axis, another `CALCULATE`). This is why `SEV1 Incidents` returns 8 regardless of what severity a user has clicked elsewhere: the explicit condition inside the measure always wins over the external filter on that column.

**The `+ 0` fix.** During testing, these three measures rendered as a **blank card** — not a `0` — whenever the active filter context left zero matching rows (for example, filtering the Incident Category slicer to a category with no SEV1 incidents in it). This is standard DAX behaviour: `CALCULATE(COUNTROWS(...), <condition>)` returns `BLANK()`, not `0`, when the filtered table has no rows, and a card visual displays a `BLANK()` result as an empty box rather than a zero. That's indistinguishable from a broken visual to anyone looking at the dashboard. Appending `+ 0` forces DAX to evaluate `BLANK() + 0`, which resolves to `0`, so the card always shows a real number. This was diagnosed by building a temporary diagnostic table showing the raw (pre-fix) measure output across every slicer combination, confirming the blank only appeared at zero-row filter contexts, then applying and re-testing the fix on all three affected measures.

## Rate measures, and why `REMOVEFILTERS` is necessary

```dax
SEV1 % =
    VAR SEV1Count = CALCULATE(COUNTROWS(CyberIncidents), CyberIncidents[Severity] = "SEV1")
    VAR TotalIgnoringSeverity = CALCULATE(COUNTROWS(CyberIncidents), REMOVEFILTERS(CyberIncidents[Severity]))
    RETURN DIVIDE(SEV1Count, TotalIgnoringSeverity, 0)

Documentation Rate =
    VAR DocumentedCount = CALCULATE(COUNTROWS(CyberIncidents), CyberIncidents[Documented] = "Yes")
    VAR TotalIgnoringDocumented = CALCULATE(COUNTROWS(CyberIncidents), REMOVEFILTERS(CyberIncidents[Documented]))
    RETURN DIVIDE(DocumentedCount, TotalIgnoringDocumented, 0)

Post-Incident Review Completion Rate =
    VAR PIRCount = CALCULATE(COUNTROWS(CyberIncidents), CyberIncidents[Post-Incident Review Done] = "Yes")
    VAR TotalIgnoringPIR = CALCULATE(COUNTROWS(CyberIncidents), REMOVEFILTERS(CyberIncidents[Post-Incident Review Done]))
    RETURN DIVIDE(PIRCount, TotalIgnoringPIR, 0)
```

Each rate measure needs a numerator (rows meeting the target condition) and a denominator representing all relevant incidents in the current analytical context. The denominator therefore removes only the filter from the column defining the rate while preserving filters from other dimensions such as Incident Category and Confidence. For example, `REMOVEFILTERS(CyberIncidents[Documented])` ensures that a filter on documentation status does not collapse the denominator to only documented or undocumented records, while other active report filters remain in effect. This allows the measure to answer: "Of the incidents currently in scope, what proportion are documented?" `DIVIDE(..., ..., 0)` safely returns zero if no incidents remain in scope.

## Design decision: no Severity slicer on Page 1

`SEV1 Incidents` (and its underlying logic) always evaluates strictly against `Severity = "SEV1"`, regardless of any Severity filter present elsewhere on the page — that's the same explicit-filter-context behaviour described above. A Severity slicer sitting next to a KPI card labelled "SEV1 Incidents" would therefore let a user select "SEV2" and watch the card keep reporting 8, which reads as a bug even though the measure is behaving exactly as designed. Page 1 avoids that confusion by offering only Incident Category and Confidence slicers. Pages 2 and 3 include a Severity slicer because none of their visuals carry a hard-coded severity condition — Severity there behaves as an ordinary filterable dimension.

## Formatting

| Measure | Format |
|---|---|
| Total Incidents, SEV1 Incidents, Documented Incidents, Post-Incident Reviews Completed | Whole number |
| Average Detection Hours | 1 decimal |
| Average Response Hours | 2 decimals |
| SEV1 %, Documentation Rate, Post-Incident Review Completion Rate | Percentage, 2 decimals |
