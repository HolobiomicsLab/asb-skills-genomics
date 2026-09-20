---
name: reported-yield-extraction-from-methods-text
description: Use when a paper's methods (or its Transparent Methods supplemental file)
  reports numeric detection/acquisition yields per slide condition, and those values
  must be extracted into a structured comparison table such as yield_comparison.csv.
when_to_use_negative:
- Raw slide images and image-analysis outputs are available, so yields can be recomputed
  directly rather than transcribed from methods prose
- The methods text describes a workflow qualitatively but contains no numeric yield
  or proportion values to extract
- The task requires recomputing detection rates from OpenHiCAMM/ROIfinder module outputs
  instead of extracting the paper's reported figures
edam_operation: http://edamontology.org/operation_0306
tools:
- name: OpenHiCAMM
  role: The imaging-acquisition system whose reported per-slide object-acquisition
    yields (75%-85% on high-quality CRM slides vs 55% on GMR33E04) are the extraction
    targets
  repo: https://github.com/bdgp/OpenHiCAMM
- name: pandas
  role: Used to write the parsed (condition, yield_range, degradation caveat) rows
    to yield_comparison.csv, one row per slide condition
- name: manual expert review
  role: Used to locate and validate the yield statements in the methods text and to
    preserve the paper's degradation caveat verbatim
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2018.03.017
    title: 'OpenHiCAMM: High-Content Screening Software for Complex Microscope Imaging
      Workflows'
  source_content_hash: sha256:c316ffd50cf1f04755d39caa6c9258c8a2ec673fbbe9b57f8321737948a44c25
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bdgp/OpenHiCAMM
  tools:
  - OpenHiCAMM
  - manual expert review
  - pandas
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# reported-yield-extraction-from-methods-text

## Summary

Parses quantitative object-acquisition yields reported in a paper's methods text (specifically the Transparent Methods supplemental file of the OpenHiCAMM paper) into a structured per-condition comparison table. It preserves the authors' stated degradation caveats alongside each yield value rather than reporting numbers out of context.

## When to use

Use when a paper's methods (or its Transparent Methods supplemental file) reports numeric detection/acquisition yields per slide condition, and those values must be extracted into a structured comparison table such as yield_comparison.csv.

## When NOT to use

- Raw slide images and image-analysis outputs are available, so yields can be recomputed directly rather than transcribed from methods prose
- The methods text describes a workflow qualitatively but contains no numeric yield or proportion values to extract
- The task requires recomputing detection rates from OpenHiCAMM/ROIfinder module outputs instead of extracting the paper's reported figures

## Inputs

- Transparent Methods supplemental file (supplemental procedures document) of the OpenHiCAMM paper
- Methods-section text containing per-condition object-acquisition yield statements
- Slide-condition identifiers (e.g., high-quality CRM reporter-construct slides; age-degraded GMR33E04 slide)

## Outputs

- yield_comparison.csv with columns (condition, yield_range, degradation caveat), one row per slide condition

## How to apply

First load the Transparent Methods supplemental file referenced by the methods section, since the OpenHiCAMM procedures including yield figures reside there rather than in the main text. Scan the methods text for acquisition-yield statements per slide condition: for the high-quality CRM reporter-construct slides the reported proportion of objects acquired is 75%-85%, while the age-degraded GMR33E04 slide yielded 55% of total objects detected. Parse each condition into three fields — (condition, yield_range, degradation caveat) — taking care to carry the paper's own attribution for the degraded slide (age-related degradation producing low contrast) into the caveat field rather than discarding it. Write one row per slide condition to yield_comparison.csv using pandas so the two-row contrast is directly machine-readable. Judge success by verifying that the CSV contains exactly one row per reported condition, that the yield ranges match the paper verbatim, and that each degraded-condition row retains the authors' caveat text.

## Related tools

- **OpenHiCAMM** (The imaging-acquisition system whose reported per-slide object-acquisition yields (75%-85% on high-quality CRM slides vs 55% on GMR33E04) are the extraction targets) — https://github.com/bdgp/OpenHiCAMM
- **pandas** (Used to write the parsed (condition, yield_range, degradation caveat) rows to yield_comparison.csv, one row per slide condition)
- **manual expert review** (Used to locate and validate the yield statements in the methods text and to preserve the paper's degradation caveat verbatim)

## Examples

```
import pandas as pd; pd.DataFrame([{'condition':'high-quality CRM slides','yield_range':'75%-85%','degradation caveat':''},{'condition':'GMR33E04','yield_range':'55%','degradation caveat':'age-related degradation (low contrast)'}]).to_csv('yield_comparison.csv', index=False)
```

## Evaluation signals

- yield_comparison.csv contains exactly two rows, one per slide condition (high-quality slides and GMR33E04)
- High-quality-slide row reports the 75%-85% acquisition range exactly as stated in the methods text
- GMR33E04 row reports 55% of total objects detected and includes the age-related degradation (low contrast) caveat field
- Schema check: all three fields (condition, yield_range, degradation caveat) are non-empty for every row
- Re-reading the Transparent Methods file reproduces the same values, confirming no paraphrased or rounded numbers were introduced

## Limitations

- The lower GMR33E04 yield is attributed by the paper to age-related degradation (low contrast) but no quantitative degradation threshold is given, so the caveat is qualitative
- Only one age-degraded slide (GMR33E04) is reported, so the 55% figure is a single-condition observation without replication
- Values are transcriptions of paper-reported yields, not recomputed measurements; they inherit any reporting errors in the source text

## Evidence

- [intro] OpenHiCAMM controls optical microscopes and interfaces with an automated slide loader: "The software, OpenHiCAMM (Open Hi Content Acquisition for μManager) controls optical microscopes and interfaces with an automated slide loader"
- [abstract] Embryos detected at low resolution and re-imaged at high resolution, the workflow whose yields are reported: "we imaged Drosophila embryos, detecting the embryos at low resolution, followed by re-imaging the detected embryos at high resolution"
- [results] Workflow imaged slide areas and captured coordinates of detected embryos, the detection step whose yield is quantified: "We ran the workflow to first image manually selected areas of the slide occupied by the coverslip as tiled images, followed by an image analysis module to capture the coordinates of the detected"
- [readme] README points to docs folder with detailed usage instructions for the system under study: "More detailed usage instructions can be found in the `docs/` folder."
