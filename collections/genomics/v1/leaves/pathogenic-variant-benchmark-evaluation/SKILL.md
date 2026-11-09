---
name: pathogenic-variant-benchmark-evaluation
description: Use when you have a gold-standard germline variant set with ground-truth
  pathogenicity labels (e.g. 105 LP/P vs 683 nonpathogenic variants) and need to quantify
  an automated classifier's sensitivity/specificity.
when_to_use_negative:
- Input variants are somatic/tumor-derived — PCGR (somatic variant interpretation)
  is the intended tool in this framework, not CPSR's germline classification
- The variant set has no ground-truth pathogenicity labels, so sensitivity/specificity
  cannot be computed
- Input is not a PASS-filtered single-sample germline VCF of SNVs/InDels (e.g. CNAs
  or RNA fusions are outside CPSR's scope)
edam_operation: http://edamontology.org/operation_3225
edam_topics:
- http://edamontology.org/topic_0199
tools:
- name: CPSR
  role: Automated ACMG/AMP germline variant classification engine applied to the benchmark
    VCF to produce per-variant pathogenicity labels
  repo: https://github.com/sigven/cpsr
- name: Docker
  role: Containerized deployment mechanism for running the CPSR workflow reproducibly
    on the benchmark set
- name: R
  role: Underlying implementation language of CPSR reporting (pcgrr R package) used
    to extract per-variant classification calls
  repo: https://github.com/sigven/pcgr/tree/master/pcgrr
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/ijc.33749
    title: 'Cancer Predisposition Sequencing Reporter ( <scp>CPSR</scp> ): A flexible
      variant report engine for high‐throughput germline screening in cancer'
  source_content_hash: sha256:f9bdc9a6ba0e547afa114cef4ca8709b88050d7067a5b0fbd56d5ea42688126a
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
  - Python
  - R
  edam_operation: http://edamontology.org/operation_3225
  edam_topics:
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# pathogenic-variant-benchmark-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a germline variant classification algorithm (e.g. CPSR's automated ACMG/AMP procedure with >30 refined criteria) against a gold-standard set of variants with known pathogenicity, computing sensitivity for LP/P variants and specificity for nonpathogenic variants, and compare with published comparator algorithms such as CharGer.

## When to use

Use when you have a gold-standard germline variant set with ground-truth pathogenicity labels (e.g. 105 LP/P vs 683 nonpathogenic variants) and need to quantify an automated classifier's sensitivity/specificity.

## When NOT to use

- Input variants are somatic/tumor-derived — PCGR (somatic variant interpretation) is the intended tool in this framework, not CPSR's germline classification
- The variant set has no ground-truth pathogenicity labels, so sensitivity/specificity cannot be computed
- Input is not a PASS-filtered single-sample germline VCF of SNVs/InDels (e.g. CNAs or RNA fusions are outside CPSR's scope)

## Inputs

- Benchmark VCF of germline SNVs/InDels from a single sample with PASS filters (e.g. 788 variants, VCF >= v4.2)
- Ground-truth pathogenicity labels per benchmark variant (105 LP/P, 683 nonpathogenic; Supplementary Table 3, Zenodo 10.5281/zenodo.5035949)
- CPSR installation: Docker image or conda environments pcgr/cpsr with reference data bundle

## Outputs

- Per-variant CPSR ACMG/AMP classification (Pathogenic/Likely Pathogenic/VUS/Likely Benign/Benign) for each of the 788 benchmark variants
- Sensitivity and specificity per algorithm (e.g. benchmark_results.csv with columns: algorithm, sensitivity, specificity)

## How to apply

Format the benchmark variants (788 SNVs/InDels from Zenodo deposit 10.5281/zenodo.5035949, Supplementary Table 3) as a single-sample germline VCF (>= v4.2) retaining only PASS-filtered records, since CPSR only reports variants designated 'PASS' in the VCF FILTER column. Run the CPSR workflow (Docker image or the pcgr/cpsr conda environments) so its point-based ACMG/AMP classification — using ClinVar, gnomAD, and cancerhotspots.org resources plus a refined PVS1 decision tree for loss-of-function variants — assigns Pathogenic/Likely Pathogenic/VUS/Likely Benign/Benign labels. Extract the CPSR class per variant and binarize predictions into pathogenic (Pathogenic/Likely Pathogenic) vs non-pathogenic. Compute sensitivity as the fraction of the 105 LP/P benchmark variants classified pathogenic and specificity as the fraction of the 683 nonpathogenic variants not classified pathogenic; then retrieve published comparator rates (CharGer custom 72.4%/97.2%, ACMG-based 56.2%/98.1%) for direct comparison.

## Related tools

- **CPSR** (Automated ACMG/AMP germline variant classification engine applied to the benchmark VCF to produce per-variant pathogenicity labels) — https://github.com/sigven/cpsr
- **Docker** (Containerized deployment mechanism for running the CPSR workflow reproducibly on the benchmark set)
- **R** (Underlying implementation language of CPSR reporting (pcgrr R package) used to extract per-variant classification calls) — https://github.com/sigven/pcgr/tree/master/pcgrr

## Evaluation signals

- All 788 benchmark variants receive a CPSR classification; any variant without a call indicates a VCF formatting or PASS-filter failure
- Sensitivity for LP/P variants (n = 105) falls in a plausible range relative to published comparators (CharGer custom 72.4%, ACMG-based 56.2%; CPSR reported at 74.3%)
- Specificity for nonpathogenic variants (n = 683) is high (>97%), consistent with comparator rates of 97.2–98.1%
- The output CSV contains exactly one row per algorithm with columns algorithm, sensitivity, specificity, and values in [0,1]
- The input VCF validates as single-sample, VCF >= v4.2, SNVs/InDels only, with multi-allelic sites decomposed before classification

## Limitations

- Only PASS-designated variants are used for reporting, so unfiltered or FAIL-flagged benchmark records are silently excluded
- Multi-allelic sites are decomposed, which can change variant representation relative to the source benchmark table
- Classifier output depends on the reference data bundle version (ClinVar, gnomAD r2.1.1/r4.1, CancerMine, CIViC release dates), so re-running at a later date may shift sensitivity/specificity away from published values

## Evidence
