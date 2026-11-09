---
name: acmg-amp-variant-classification-calibration
description: Use when calibrating or reproducing score-to-classification cut-offs
  for an automated ACMG/AMP germline variant classifier against ClinVar assertions,
  or validating per-class concordance of a five-level pathogenicity scheme (P/LP,
  VUS, LB/B).
when_to_use_negative:
- You only need to run CPSR on a patient germline VCF with the already-published thresholds
  — this skill is about recalibrating the score cut-offs, not applying them in reporting.
- The truth set lacks ClinVar review-status metadata, so the two-star calibration
  subset (55,841 variants, 335 genes) cannot be faithfully reconstructed.
- The data are somatic tumor variants classified for oncogenicity under VICC/CGC/ClinGen
  guidelines (PCGR) — that is a different classification framework from germline ACMG/AMP
  calibration.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_0199
tools:
- name: CPSR
  role: Point-based ACMG/AMP classification logic (more than 30 refined criteria,
    PVS1 decision tree) whose five score cut-offs are swept and calibrated against
    ClinVar
  repo: https://github.com/sigven/cpsr
- name: pcgrr
  role: R package implementing the variant classification logic scored across the
    calibration subset
  repo: https://github.com/sigven/pcgr/tree/master/pcgrr
- name: Python
  role: Implementation language of the cpsr classification code and the calibration
    scripts that sweep cut-offs and compute concordance
- name: R
  role: Implementation language of the pcgrr variant classification code exercised
    during threshold calibration
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/ijc.33749
    title: 'Cancer Predisposition Sequencing Reporter ( <scp>CPSR</scp> ): A flexible
      variant report engine for high‐throughput germline screening in cancer'
  source_content_hash: sha256:f0d6b992534fb9c35bcd0395df024adf357f395406a1ad6f3ed45b503ba7e387
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# acmg-amp-variant-classification-calibration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Calibration of CPSR's point-based ACMG/AMP germline classifier thresholds: score ClinVar two-star variants in 335 cancer predisposition genes with the cpsr/pcgrr logic, sweep the five P/LP–VUS–LB/B cut-offs, and report per-class concordance (92.8% P/LP, 95.7% VUS, 96.3% B/LB).

## When to use

Use when calibrating or reproducing score-to-classification cut-offs for an automated ACMG/AMP germline variant classifier against ClinVar assertions, or validating per-class concordance of a five-level pathogenicity scheme (P/LP, VUS, LB/B).

## When NOT to use

- You only need to run CPSR on a patient germline VCF with the already-published thresholds — this skill is about recalibrating the score cut-offs, not applying them in reporting.
- The truth set lacks ClinVar review-status metadata, so the two-star calibration subset (55,841 variants, 335 genes) cannot be faithfully reconstructed.
- The data are somatic tumor variants classified for oncogenicity under VICC/CGC/ClinGen guidelines (PCGR) — that is a different classification framework from germline ACMG/AMP calibration.

## Inputs

- ClinVar 20210328 release variant records with clinical significance assertions in cancer predisposition genes
- ClinVar review-status (star) metadata used to retain variants with a minimum of two review stars
- Calibration datasets and scripts from Zenodo deposit doi:10.5281/zenodo.5035949

## Outputs

- TSV/CSV calibration table with five threshold cut-off rows (P, LP, VUS, LB, B boundaries)
- Per-class concordance values between CPSR assertions and ClinVar assertions (92.8% P/LP, 95.7% VUS, 96.3% B/LB)

## How to apply

Download the calibration datasets and scripts from the Zenodo deposit 10.5281/zenodo.5035949, then build the gold-standard subset from the ClinVar 20210328 release restricted to cancer predisposition genes and variants with a minimum review status of two stars (55,841 variants across 335 genes), since two-star status screens out low-confidence assertions. Score every calibration variant with the CPSR point-based ACMG/AMP classification logic — the >30 refined ACMG criteria including the PVS1 decision tree for loss-of-function variants — as implemented in the Python/R cpsr/pcgrr code. Sweep the five score-to-classification boundaries (P, LP, VUS, LB, B) and compute per-class concordance between CPSR assertions and ClinVar assertions with the calibration scripts. Verify the recomputed concordance against the reported 92.8% for P/LP, 95.7% for VUS and 96.3% for B/LB within a small tolerance. Write the five threshold cut-off rows together with the per-class concordance values to a single calibration table in TSV/CSV format.

## Related tools

- **CPSR** (Point-based ACMG/AMP classification logic (more than 30 refined criteria, PVS1 decision tree) whose five score cut-offs are swept and calibrated against ClinVar) — https://github.com/sigven/cpsr
- **pcgrr** (R package implementing the variant classification logic scored across the calibration subset) — https://github.com/sigven/pcgr/tree/master/pcgrr
- **Python** (Implementation language of the cpsr classification code and the calibration scripts that sweep cut-offs and compute concordance)
- **R** (Implementation language of the pcgrr variant classification code exercised during threshold calibration)

## Evaluation signals

- The output calibration table contains exactly five threshold cut-off rows (P, LP, VUS, LB, B boundaries).
- Recomputed per-class concordance matches the reported 92.8% (P/LP), 95.7% (VUS) and 96.3% (B/LB) figures within a small tolerance.
- The calibration subset equals 55,841 ClinVar variants across 335 cancer predisposition genes at a minimum review status of two stars.
- Subset derivation is traceable to the ClinVar 20210328 release, and variant scoring reproduces assertions from the unmodified cpsr/pcgrr classification code.

## Limitations

- Ground truth is restricted to ClinVar assertions with at least two review stars, so single-submitter or conflicting assertions are excluded and concordance may be optimistic.
- Concordance values and the chosen cut-offs are tied to the ClinVar 20210328 release and the 335-gene cancer predisposition panel; newer ClinVar or gnomAD data may shift them.
- Because the cut-offs are tuned to maximize agreement with ClinVar, the classifier inherits ClinVar curation biases rather than being measured against an independent standard.

## Evidence

- [abstract] Automated ACMG/AMP classification is integral to the CPSR workflow: "A fully automated variant classification procedure based on more than 30 refined American College of Medical Genetics and Genomics (ACMG) criteria represents an integral part of the workflow"
- [methods] CPSR classification relies on ClinVar and other public resources: "CPSR classifies germline variants against ACMG/AMP criteria using public resources (ClinVar, gnomAD, cancerhotspots.org)"
- [results] Per-class concordance achieved by the calibrated thresholds: [evidence span withheld — anchor status: not_found]
- [methods] Definition of the ClinVar calibration subset: [evidence span withheld — anchor status: not_found]
- [other] Acceptance criterion for the calibration reproduction: [evidence span withheld — anchor status: not_found]
- [readme] Five-level classification scope in the CPSR README: "**Variant classification** (*Pathogenic* to *Benign*) through a dedicated implementation of [ACMG/AMP guidelines]"
