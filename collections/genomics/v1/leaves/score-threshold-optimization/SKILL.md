---
name: score-threshold-optimization
description: Use when a five-level germline variant classification scheme (P/LP, VUS,
  LB/B) needs its numeric score boundaries calibrated against a curated reference
  set such as ClinVar, with per-class concordance as the evaluation metric.
when_to_use_negative:
- The task concerns somatic/oncogenicity classification of tumor variants (PCGR's
  VICC/CGC/ClinGen and AMP/ASCO/CAP frameworks) rather than germline ACMG/AMP classification
  of cancer predisposition variants
- The reference set consists of ClinVar variants without the minimum two-star review
  status, so concordance estimates would not be comparable to the curated two-star
  calibration
- The goal is running CPSR's classification on a single patient sample to produce
  a report, not optimizing or evaluating the underlying score thresholds
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_2640
tools:
- name: CPSR
  role: Provides the point-based ACMG/AMP classification workflow whose five-level
    score thresholds are calibrated; implemented in Python/R and available via Docker
  repo: https://github.com/sigven/cpsr
- name: pcgrr
  role: R package in which the CPSR classification logic used for scoring calibration
    variants is implemented
  repo: https://github.com/sigven/pcgr/tree/master/pcgrr
- name: Python
  role: Language of the CPSR tool and the calibration scripts used to sweep score
    cut-offs and compute per-class concordance
- name: R
  role: Language of the cpsr/pcgrr packages and report generation used in the classification
    logic under calibration
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/ijc.33749
    title: 'Cancer Predisposition Sequencing Reporter ( <scp>CPSR</scp> ): A flexible
      variant report engine for high‐throughput germline screening in cancer'
  source_content_hash: sha256:b1969bc9c76196c4414a30789756a07779ab5458c21a5f28592a179749fcee65
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
  - Python
  - R
  - pcgrr
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# score-threshold-optimization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Calibrates the five score-to-classification cut-offs of CPSR's point-based ACMG/AMP variant classification logic against curated ClinVar assertions in cancer predisposition genes, and quantifies per-class concordance between the automated scores and ClinVar review-starred classifications.

## When to use

Use when a five-level germline variant classification scheme (P/LP, VUS, LB/B) needs its numeric score boundaries calibrated against a curated reference set such as ClinVar, with per-class concordance as the evaluation metric.

## When NOT to use

- The task concerns somatic/oncogenicity classification of tumor variants (PCGR's VICC/CGC/ClinGen and AMP/ASCO/CAP frameworks) rather than germline ACMG/AMP classification of cancer predisposition variants
- The reference set consists of ClinVar variants without the minimum two-star review status, so concordance estimates would not be comparable to the curated two-star calibration
- The goal is running CPSR's classification on a single patient sample to produce a report, not optimizing or evaluating the underlying score thresholds

## Inputs

- ClinVar 20210328 release subset restricted to variants with minimum review status of two stars (55,841 variants in 335 cancer predisposition genes)
- Calibration datasets and scripts from Zenodo deposit doi 10.5281/zenodo.5035949
- Germline variant calls (SNVs/InDels) in VCF format
- CPSR point-based ACMG/AMP classification logic from the cpsr/pcgrr Python/R code

## Outputs

- Calibration table (TSV/CSV) with five threshold cut-off rows (P, LP, VUS, LB, B boundaries)
- Per-class concordance values: 92.8% P/LP, 95.7% VUS, 96.3% B/LB

## How to apply

Download the calibration datasets and scripts from the Zenodo deposit 10.5281/zenodo.5035949 accompanying the CPSR classification calibration. Build the calibration subset from the ClinVar 20210328 release, retaining variants with a minimum review status of two stars — 55,841 variants across 335 cancer predisposition genes. Score each variant with the CPSR point-based ACMG/AMP classification logic implemented in the Python/R cpsr/pcgrr code (which evaluates more than 30 refined ACMG criteria, including PVS1 decision-tree evaluation of loss-of-function variants). Sweep the five score-to-classification cut-offs (P, LP, VUS, LB, B boundaries) and compute per-class concordance between CPSR assertions and ClinVar assertions. Verify the recomputed concordance against the reported 92.8% for P/LP, 95.7% for VUS, and 96.3% for B/LB within a small tolerance, then write the five threshold cut-offs plus per-class concordance values to a single calibration table in TSV/CSV format.

## Related tools

- **CPSR** (Provides the point-based ACMG/AMP classification workflow whose five-level score thresholds are calibrated; implemented in Python/R and available via Docker) — https://github.com/sigven/cpsr
- **pcgrr** (R package in which the CPSR classification logic used for scoring calibration variants is implemented) — https://github.com/sigven/pcgr/tree/master/pcgrr
- **Python** (Language of the CPSR tool and the calibration scripts used to sweep score cut-offs and compute per-class concordance)
- **R** (Language of the cpsr/pcgrr packages and report generation used in the classification logic under calibration)

## Evaluation signals

- Output calibration table contains exactly five threshold cut-off rows (P, LP, VUS, LB, B boundaries)
- Per-class concordance values match the reported 92.8% (P/LP), 95.7% (VUS), and 96.3% (B/LB) within a small tolerance
- Calibration subset size matches 55,841 variants across 335 cancer predisposition genes from the ClinVar 20210328 release at minimum two review stars
- Each calibration variant receives a CPSR assertion from the point-based ACMG/AMP logic (more than 30 refined ACMG criteria including PVS1 decision-tree handling of LOF variants)
- Calibration table is written as a single TSV/CSV file combining cut-offs and per-class concordance

## Limitations

- Concordance results are tied to a specific ClinVar release (20210328) and the two-star minimum review trust level; re-running on other releases or looser review criteria may shift the values
- Calibration only covers the 335 curated cancer predisposition genes, so thresholds may not generalize to other gene sets
- ClinVar variants with non-standard significance levels have historically caused issues in CPSR parsing (a 2025 bug fix addressed 'ClinVar variants with non-standard significance levels')

## Evidence
