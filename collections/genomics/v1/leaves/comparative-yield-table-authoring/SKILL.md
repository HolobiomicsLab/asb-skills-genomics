---
name: comparative-yield-table-authoring
description: Use when a paper reports separate object-acquisition/detection yield
  percentages for two or more slide or sample conditions and the goal is a machine-readable
  side-by-side yield comparison preserving stated caveats.
when_to_use_negative:
- The paper reports only a single aggregate yield with no contrast between conditions
  — a two-row comparison table would be misleading.
- Yield values are only present in figures/figure legends rather than reported numerically
  in the methods text — do not eyeball percentages from plots.
- You need software performance metrics (e.g. version numbers, changelog, benchmarks)
  — the OpenHiCAMM discussion explicitly notes no changelog, no version number, and
  no quantitative benchmark results are available; this skill extracts biological
  acquisition yields, not software validation metrics.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: OpenHiCAMM
  role: The imaging platform whose slide-object acquisition yields (75%-85% high-quality
    vs 55% GMR33E04) are being tabulated; developed as a Fiji/μManager module with
    a Prior PL-200 slide loader
  repo: github.com/bdgp/OpenHiCAMM
- name: manual expert review
  role: Used to verify that each parsed (condition, yield_range, caveat) row matches
    the methods text, including the age-degradation attribution for GMR33E04
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2018.03.017
    title: 'OpenHiCAMM: High-Content Screening Software for Complex Microscope Imaging
      Workflows'
  source_content_hash: sha256:9a27bf61a8c48791c7fa0d289cce4741c8b05100e84f758e11de2f5efe1d4f5f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - OpenHiCAMM
  - manual expert review
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# comparative-yield-table-authoring

## Summary

Extracts condition-specific object-acquisition yield values reported in a paper's methods/supplemental text (e.g. 75%–85% for high-quality CRM reporter-construct slides vs 55% for the age-degraded GMR33E04 slide in OpenHiCAMM) and writes them as a per-condition comparison table (yield_comparison.csv) with pandas.

## When to use

Use when a paper reports separate object-acquisition/detection yield percentages for two or more slide or sample conditions and the goal is a machine-readable side-by-side yield comparison preserving stated caveats.

## When NOT to use

- The paper reports only a single aggregate yield with no contrast between conditions — a two-row comparison table would be misleading.
- Yield values are only present in figures/figure legends rather than reported numerically in the methods text — do not eyeball percentages from plots.
- You need software performance metrics (e.g. version numbers, changelog, benchmarks) — the OpenHiCAMM discussion explicitly notes no changelog, no version number, and no quantitative benchmark results are available; this skill extracts biological acquisition yields, not software validation metrics.

## Inputs

- Transparent Methods supplemental file (referenced from the methods section of the OpenHiCAMM iScience 2018 paper)
- Methods text passages reporting acquisition yield percentages per slide condition (high-quality CRM reporter-construct slides; age-degraded GMR33E04 slide)

## Outputs

- yield_comparison.csv with one row per slide condition, fields (condition, yield_range, degradation_caveat): e.g. high-quality slides → 75%-85%; GMR33E04 → 55% with age-related low-contrast caveat
- Structured (condition, yield_range, degradation caveat) records suitable for downstream joining with workflow-phase metadata

## How to apply

First load the paper's Transparent Methods supplemental file (referenced from the methods section), since the OpenHiCAMM procedures and yield values live there rather than in the main text. Locate each reported object-acquisition yield: high-quality CRM reporter-construct slides are reported at 75%–85% of objects acquired, while the age-degraded GMR33E04 slide yielded detection of 55% of total objects, with the paper attributing the lower yield to age-related degradation (low contrast). Parse each condition into structured fields — (condition, yield_range, degradation caveat) — one row per slide condition, preserving the paper's own attribution of the degraded slide's lower yield. Write the rows to yield_comparison.csv with pandas, one row per condition, so downstream agents can join yields against the corresponding workflow phases (5× tiled survey pass, then 20× re-imaging of detected embryos).

## Related tools

- **OpenHiCAMM** (The imaging platform whose slide-object acquisition yields (75%-85% high-quality vs 55% GMR33E04) are being tabulated; developed as a Fiji/μManager module with a Prior PL-200 slide loader) — github.com/bdgp/OpenHiCAMM
- **manual expert review** (Used to verify that each parsed (condition, yield_range, caveat) row matches the methods text, including the age-degradation attribution for GMR33E04)

## Examples

```
import pandas as pd; pd.DataFrame([{'condition':'high-quality CRM slides','yield_range':'75%-85%','caveat':''},{'condition':'GMR33E04','yield_range':'55%','caveat':'age-related degradation (low contrast)'}]).to_csv('yield_comparison.csv', index=False)
```

## Evaluation signals

- yield_comparison.csv contains exactly two rows, one per slide condition (high-quality; GMR33E04).
- High-quality condition row records the full reported range 75%-85%, not a single point value.
- GMR33E04 row records 55% of total objects detected and retains the paper's caveat that the lower yield is attributed to age-related degradation (low contrast).
- Every yield value in the CSV can be traced to a verbatim sentence in the Transparent Methods supplemental file (no inferred or interpolated numbers).
- CSV loads cleanly with pandas.read_csv and the column names match (condition, yield_range, degradation caveat) schema.

## Limitations

- Yields are reported as ranges (75%-85%) for high-quality slides and a single percentage (55%) for GMR33E04, so per-slide or per-experiment variance cannot be recovered from the published text.
- The 55% GMR33E04 yield is confounded by slide age/low contrast; the paper's attribution is qualitative and no matched-age control is reported.
- The OpenHiCAMM discussion section reports no quantitative benchmark or validation results, so the yields cannot be cross-checked against published system-level metrics; only a GitHub source (github.com/bdgp/OpenHiCAMM) is cited, with no DOI or archival data repository.

## Evidence

- [supplementary] two-row yield comparison: [evidence span withheld — anchor status: not_found]
- [supplementary] load Transparent Methods and parse conditions: [evidence span withheld — anchor status: not_found]
- [readme] protocol documentation location: "More detailed usage instructions can be found in the `docs/` folder."
