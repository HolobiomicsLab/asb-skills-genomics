---
name: variant-classification-algorithm-comparison
description: Use when a germline VCF (SNVs/InDels, single sample) must be automatically
  classified Pathogenic-to-Benign and the classifier's sensitivity/specificity must
  be compared against published algorithms on a labeled benchmark.
when_to_use_negative:
- Input contains somatic tumor variants — use PCGR's oncogenicity/actionability classification
  instead of CPSR's germline ACMG/AMP classification
- The goal is to re-run CharGer itself rather than to reuse its published comparator
  rates (72.4%/97.2% custom, 56.2%/98.1% ACMG-based) for the same benchmark
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_0199
tools:
- name: CPSR
  role: Automated germline variant classifier applying >30 refined ACMG/AMP criteria
    (ClinVar, gnomAD, cancerhotspots.org) whose sensitivity/specificity is measured
  repo: https://github.com/sigven/cpsr
- name: PCGR
  role: Framework underlying CPSR; provides the shared infrastructure for annotation
    and classification
  repo: https://github.com/sigven/pcgr
- name: Python
  role: Extract and binarize CPSR classifications and compute sensitivity/specificity
    over the benchmark labels
- name: R
  role: Report generation and visualisation of classification results via the cpsr/pcgrr
    packages and Quarto
  repo: https://github.com/sigven/cpsr
- name: Docker
  role: Containerized execution environment for running the CPSR classification workflow
    on the benchmark VCF
  repo: https://github.com/sigven/cpsr
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/ijc.33749
    title: 'Cancer Predisposition Sequencing Reporter ( <scp>CPSR</scp> ): A flexible
      variant report engine for high‐throughput germline screening in cancer'
  source_content_hash: sha256:f85359c4587dc9c19366a3b363fba5545ad1692fc7584b4df837367bbf1285a4
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
  - Docker
  - PCGR
  - Python
  - R
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# variant-classification-algorithm-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark automated germline variant classification algorithms (CPSR's >30 refined ACMG/AMP criteria vs CharGer custom and ACMG-based algorithms) by computing sensitivity and specificity on a gold-standard set of 788 PCGP variants with ground-truth pathogenicity labels.

## When to use

Use when a germline VCF (SNVs/InDels, single sample) must be automatically classified Pathogenic-to-Benign and the classifier's sensitivity/specificity must be compared against published algorithms on a labeled benchmark.

## When NOT to use

- Input contains somatic tumor variants — use PCGR's oncogenicity/actionability classification instead of CPSR's germline ACMG/AMP classification
- The goal is to re-run CharGer itself rather than to reuse its published comparator rates (72.4%/97.2% custom, 56.2%/98.1% ACMG-based) for the same benchmark

## Inputs

- Single-sample germline VCF (>= v4.2) with PASS-filtered SNVs/InDels
- PCGP benchmark set of 788 variants with ground-truth pathogenicity labels (105 LP/P, 683 nonpathogenic) from Zenodo deposit 10.5281/zenodo.5035949 (Supplementary Table 3)

## Outputs

- Per-variant CPSR classification (Pathogenic/Likely Pathogenic/VUS/Likely Benign/Benign) for each of the 788 benchmark variants
- benchmark_results.csv with columns algorithm, sensitivity, specificity (one row per algorithm)
- Interactive CPSR HTML report of germline variant classifications

## How to apply

Download the 788-variant PCGP benchmark (105 LP/P, 683 nonpathogenic) from Zenodo deposit 10.5281/zenodo.5035949 and format it as a single-sample germline VCF (>= v4.2, PASS-filtered SNVs/InDels). Run the CPSR workflow via its Docker image so its point-based ACMG/AMP classification (>30 refined criteria, ClinGen SVI groupmax gnomAD approach, PVS1 decision-tree for loss-of-function variants) labels each variant Pathogenic/Likely Pathogenic/VUS/Likely Benign/Benign. Binarize predictions into pathogenic (Pathogenic/Likely Pathogenic) versus non-pathogenic, then compute sensitivity as the fraction of the 105 LP/P variants classified pathogenic and specificity as the fraction of the 683 nonpathogenic variants not classified pathogenic. Retrieve the published comparators for the same set — CharGer custom (72.4% sensitivity / 97.2% specificity) and ACMG-based (56.2% / 98.1%) — and write one row per algorithm to benchmark_results.csv.

## Related tools

- **CPSR** (Automated germline variant classifier applying >30 refined ACMG/AMP criteria (ClinVar, gnomAD, cancerhotspots.org) whose sensitivity/specificity is measured) — https://github.com/sigven/cpsr
- **PCGR** (Framework underlying CPSR; provides the shared infrastructure for annotation and classification) — https://github.com/sigven/pcgr
- **Python** (Extract and binarize CPSR classifications and compute sensitivity/specificity over the benchmark labels)
- **R** (Report generation and visualisation of classification results via the cpsr/pcgrr packages and Quarto) — https://github.com/sigven/cpsr
- **Docker** (Containerized execution environment for running the CPSR classification workflow on the benchmark VCF) — https://github.com/sigven/cpsr

## Evaluation signals

- Sensitivity of CPSR for LP/P variants on the PCGP benchmark is 74.3% (n = 105), higher than CharGer custom (72.4%) and ACMG-based (56.2%)
- Specificity values reproduce the published pattern (CharGer custom 97.2%, ACMG-based 98.1%) computed over the 683 nonpathogenic variants
- Binarization invariant: sensitivity + false-negative rate over the 105 LP/P variants equals 1.0, and specificity + false-positive rate over the 683 nonpathogenic variants equals 1.0
- All benchmark VCF records are PASS-filtered SNVs/InDels and each of the 788 variants receives exactly one CPSR ACMG classification label
- benchmark_results.csv contains exactly three rows (CPSR, CharGer custom, CharGer ACMG-based) with numeric sensitivity and specificity columns

## Limitations

- Benchmark and inputs are restricted to germline SNVs/InDels from a single sample designated PASS in the VCF FILTER column; other variant types or filtered records are excluded from reporting
- Multi-allelic sites in the input VCF are subject to decomposition before classification, which can shift variant-level comparisons
- Sensitivity gains can come at a specificity cost: CharGer's ACMG-based algorithm trades 72.4%->56.2% sensitivity for 97.2%->98.1% specificity, so no single metric suffices

## Evidence
