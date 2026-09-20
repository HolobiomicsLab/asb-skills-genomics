---
name: clinvar-concordance-evaluation
description: Use when you need to calibrate or validate automated germline variant
  pathogenicity score thresholds against ClinVar expert-reviewed assertions and report
  per-class concordance.
when_to_use_negative:
- The variants of interest are somatic/tumor variants interpreted for oncogenicity
  and actionability (PCGR's VICC/CGC/ClinGen and AMP/ASCO/CAP scope), not germline
  cancer predisposition classification.
- ClinVar assertions lack review status (fewer than two stars) — unreviewed assertions
  are too noisy to serve as a calibration ground truth.
- You only need to run CPSR's classification on a sample VCF without evaluating or
  recalibrating the score thresholds themselves.
edam_topics:
- http://edamontology.org/topic_3053
tools:
- name: CPSR
  role: Provides the point-based ACMG/AMP classification logic whose score thresholds
    are calibrated and whose per-class concordance against ClinVar is measured
  repo: https://github.com/sigven/cpsr
- name: pcgrr
  role: R package containing the classification implementation (cpsr/pcgrr code) used
    to score each calibration variant
  repo: https://github.com/sigven/pcgr/tree/master/pcgrr
- name: Python
  role: Language in which the classification/calibration logic is implemented and
    the score cut-off sweep is executed
- name: R
  role: Runtime for the cpsr package classification logic and calibration scripts
- name: ClinVar
  role: Source of expert-reviewed (two-star minimum) reference assertions used as
    calibration ground truth
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/ijc.33749
    title: 'Cancer Predisposition Sequencing Reporter ( <scp>CPSR</scp> ): A flexible
      variant report engine for high‐throughput germline screening in cancer'
  source_content_hash: sha256:a5f9c856dfe1e50a12a59ba091326f2cb6be367bffb17b89758ee05a8b5165c1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/sigven/cpsr
  repo_url: https://github.com/sigven/cpsr
  tools:
  - CPSR
  - ClinVar
  - Python
  - R
  - pcgrr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# clinvar-concordance-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Calibrates and validates a variant classification scheme (e.g. CPSR's point-based ACMG/AMP implementation) against high-confidence ClinVar assertions, measuring per-class concordance for P/LP, VUS, and B/LB calls across five score-to-classification cut-offs.

## When to use

Use when you need to calibrate or validate automated germline variant pathogenicity score thresholds against ClinVar expert-reviewed assertions and report per-class concordance.

## When NOT to use

- The variants of interest are somatic/tumor variants interpreted for oncogenicity and actionability (PCGR's VICC/CGC/ClinGen and AMP/ASCO/CAP scope), not germline cancer predisposition classification.
- ClinVar assertions lack review status (fewer than two stars) — unreviewed assertions are too noisy to serve as a calibration ground truth.
- You only need to run CPSR's classification on a sample VCF without evaluating or recalibrating the score thresholds themselves.

## Inputs

- ClinVar release (e.g. 20210328) variant records with clinical significance assertions
- ClinVar subset restricted to variants with minimum review status of two stars in 335 cancer predisposition genes (55,841 variants in the reference run)
- Calibration scripts and datasets from Zenodo deposit 10.5281/zenodo.5035949
- Germline variant calls scored by the CPSR/pcgrr ACMG/AMP classification logic

## Outputs

- Calibration table in TSV/CSV format with five score-to-classification threshold cut-off rows (P, LP, VUS, LB, B boundaries)
- Per-class concordance values for P/LP, VUS, and B/LB classes (expected reference values 92.8%, 95.7%, 96.3%)

## How to apply

Download the calibration datasets and scripts from the Zenodo deposit 10.5281/zenodo.5035949 accompanying the CPSR classification calibration. Build a calibration subset from a fixed ClinVar release (here 20210328), retaining only variants with a minimum review status of two stars — 55,841 variants across 335 cancer predisposition genes in the reference run. Score every calibration variant with the CPSR point-based ACMG/AMP classification logic (30+ refined criteria, PVS1 decision-tree handling of loss-of-function variants) implemented in the Python/R cpsr/pcgrr code. Sweep the five score-to-classification cut-offs (P, LP, VUS, LB, B boundaries) and compute per-class concordance between CPSR assertions and ClinVar assertions with the calibration scripts. Accept the result only if the output calibration table contains five threshold cut-off rows and concordance values matching the reported figures (92.8% P/LP, 95.7% VUS, 96.3% B/LB) within a small tolerance, and write it as a single TSV/CSV table.

## Related tools

- **CPSR** (Provides the point-based ACMG/AMP classification logic whose score thresholds are calibrated and whose per-class concordance against ClinVar is measured) — https://github.com/sigven/cpsr
- **pcgrr** (R package containing the classification implementation (cpsr/pcgrr code) used to score each calibration variant) — https://github.com/sigven/pcgr/tree/master/pcgrr
- **Python** (Language in which the classification/calibration logic is implemented and the score cut-off sweep is executed)
- **R** (Runtime for the cpsr package classification logic and calibration scripts)
- **ClinVar** (Source of expert-reviewed (two-star minimum) reference assertions used as calibration ground truth)

## Evaluation signals

- The calibration table contains exactly five threshold cut-off rows (P, LP, VUS, LB, B boundaries).
- Per-class concordance matches the reported values — 92.8% for P/LP, 95.7% for VUS, 96.3% for B/LB — within a small tolerance.
- The calibration subset size and composition reproduce the reference: 55,841 two-star ClinVar variants across 335 cancer predisposition genes from the 20210328 release.
- Output is written as a single TSV/CSV table combining the five cut-offs and the three concordance values.
- All scored variants pass through the same CPSR ACMG/AMP logic (30+ refined criteria, PVS1 decision tree) as production classification.

## Limitations

- Concordance is measured only against ClinVar variants with at least two review stars; performance on unreviewed or one-star assertions is not established.
- The calibration is specific to a fixed ClinVar release (20210328) and 335 cancer predisposition genes; later ClinVar or gene-panel updates may shift the optimal cut-offs.
- Concordance inherits any errors in the underlying annotation resources (ClinVar, gnomAD, dbNSFP, Cancer Hotspots), which are versioned independently of the classification logic.

## Evidence

- [full_text] Using ClinVar variants in cancer predisposition genes (minimum two review stars, 55,841 variants total), the calibrated thresholds achieved concordance of 92.8% for P/LP variants, 95.7% for VUS variants, and 96.3% for B/LB variants.: [evidence span withheld — anchor status: not_found]
