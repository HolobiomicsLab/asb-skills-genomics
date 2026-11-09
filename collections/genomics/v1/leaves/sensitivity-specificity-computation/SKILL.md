---
name: sensitivity-specificity-computation
description: Use when you have a variant benchmark set with binary ground-truth pathogenicity
  labels and classifier output (Pathogenic/Likely Pathogenic vs rest), and need to
  quantify and compare detection performance across algorithms.
when_to_use_negative:
- The task concerns somatic/tumor variant interpretation (PCGR territory), not germline
  cancer predisposition classification
- Input variants are not PASS-filtered SNVs/InDels from a single sample in VCF v4.2
  format, so classifier input assumptions do not hold
- No gold-standard pathogenicity labels exist, making sensitivity/specificity denominators
  (105 LP/P, 683 nonpathogenic) unavailable
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_0622
tools:
- name: CPSR
  role: Automated ACMG/AMP germline variant classification of the benchmark VCF, assigning
    Pathogenic-to-Benign labels used as the predictions being scored
  repo: https://github.com/sigven/cpsr
- name: Docker
  role: Containerized execution environment for running the CPSR workflow on the benchmark
    VCF
- name: Python
  role: Extraction and binarization of per-variant classifications and computation
    of sensitivity/specificity metrics
- name: R
  role: CPSR report generation and visualisation (cpsr/pcgrr R package); also usable
    for metric tabulation
  repo: https://github.com/sigven/pcgr/tree/master/pcgrr
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/ijc.33749
    title: 'Cancer Predisposition Sequencing Reporter ( <scp>CPSR</scp> ): A flexible
      variant report engine for high‐throughput germline screening in cancer'
  source_content_hash: sha256:bbd1580ca628ede9c57e5452bb815cf1ade6685b89cbcaf6c09e25b80f304772
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
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# sensitivity-specificity-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes sensitivity (fraction of LP/P benchmark variants classified pathogenic) and specificity (fraction of nonpathogenic variants not classified pathogenic) for automated germline variant classifiers such as CPSR's ACMG/AMP implementation, and compares them against published comparator rates (CharGer custom and ACMG-based algorithms) on the PCGP gold-standard benchmark.

## When to use

Use when you have a variant benchmark set with binary ground-truth pathogenicity labels and classifier output (Pathogenic/Likely Pathogenic vs rest), and need to quantify and compare detection performance across algorithms.

## When NOT to use

- The task concerns somatic/tumor variant interpretation (PCGR territory), not germline cancer predisposition classification
- Input variants are not PASS-filtered SNVs/InDels from a single sample in VCF v4.2 format, so classifier input assumptions do not hold
- No gold-standard pathogenicity labels exist, making sensitivity/specificity denominators (105 LP/P, 683 nonpathogenic) unavailable

## Inputs

- 788-variant PCGP benchmark set with ground-truth pathogenicity labels (105 LP/P, 683 nonpathogenic) from Zenodo deposit 10.5281/zenodo.5035949 (Supplementary Table 3)
- Single-sample germline VCF (>= v4.2, PASS-filtered SNVs/InDels) derived from the benchmark set
- Published comparator sensitivity/specificity rates (CharGer custom 72.4%/97.2%; CharGer ACMG-based 56.2%/98.1%)

## Outputs

- Per-variant classifier labels (Pathogenic/Likely Pathogenic/VUS/Likely Benign/Benign) for all 788 benchmark variants
- Binarized predictions (pathogenic vs non-pathogenic) per variant
- benchmark_results.csv with one row per algorithm and columns: algorithm, sensitivity, specificity

## How to apply

Obtain the 788-variant PCGP benchmark (105 LP/P, 683 nonpathogenic; Zenodo deposit 10.5281/zenodo.5035949, Supplementary Table 3) and format it as a single-sample germline VCF (v4.2, PASS-filtered SNVs/InDels). Run the classifier under test — e.g. the CPSR workflow via Docker or the pcgr/cpsr conda environments — so its automated ACMG/AMP procedure (more than 30 refined criteria, point-based, ClinGen SVI groupmax gnomAD approach) assigns Pathogenic/Likely Pathogenic/VUS/Likely Benign/Benign labels to each variant. Extract per-variant classifications and binarize predictions as pathogenic (Pathogenic/Likely Pathogenic) versus non-pathogenic. Compute sensitivity as the fraction of the 105 LP/P benchmark variants classified pathogenic and specificity as the fraction of the 683 nonpathogenic variants not classified pathogenic. Retrieve published comparator rates for the same benchmark (CharGer custom 72.4%/97.2%, ACMG-based 56.2%/98.1%) and write one row per algorithm to benchmark_results.csv with columns algorithm, sensitivity, specificity.

## Related tools

- **CPSR** (Automated ACMG/AMP germline variant classification of the benchmark VCF, assigning Pathogenic-to-Benign labels used as the predictions being scored) — https://github.com/sigven/cpsr
- **Docker** (Containerized execution environment for running the CPSR workflow on the benchmark VCF)
- **Python** (Extraction and binarization of per-variant classifications and computation of sensitivity/specificity metrics)
- **R** (CPSR report generation and visualisation (cpsr/pcgrr R package); also usable for metric tabulation) — https://github.com/sigven/pcgr/tree/master/pcgrr

## Examples

```
docker run -v $PWD:/work pcgr/cpsr cpsr.py --input /work/benchmark.vcf.gz --cpsr_config /work/cpsr.toml --genome_assembly grch38 --output_dir /work/results
```

## Evaluation signals

- Sensitivity for the tested classifier falls in the reported neighborhood (CPSR 74.3%, CharGer custom 72.4%, CharGer ACMG-based 56.2%); large deviations indicate binarization or label-extraction errors
- Sensitivity denominator is exactly 105 LP/P variants and specificity denominator exactly 683 nonpathogenic variants — totals must sum to 788
- benchmark_results.csv contains exactly one row per algorithm with columns algorithm, sensitivity, specificity and comparator values matching published rates (72.4/97.2 and 56.2/98.1)
- All 788 variants receive a classifier label; unclassified or missing classifications must be resolved before computing metrics
- Input VCF records are PASS-filtered, single-sample, and decomposed if multi-allelic sites were present

## Limitations

- Nonpathogenic variants defaulting to VUS vs Benign is a key specificity driver — binarizing VUS as pathogenic vs not will change results; CPSR's report treats classification as Pathogenic-to-Benign with VUS in between
- Classifier performance depends on the bundled annotation databases (ClinVar, gnomAD, cancerhotspots.org) and their versions, so re-runs on newer data releases may shift rates
- Comparators (CharGer custom and ACMG-based) must be evaluated on the same benchmark set and binarization rule for the comparison to be valid
- If no PASS variants are present in the input VCF, processing may be skipped (as noted for the PCGR framework), yielding no predictions

## Evidence

- [results] CPSR achieved a sensitivity of 74.3% for LP/P variants (n = 105), higher than CharGer's custom (72.4%) and ACMG-based (56.2%) algorithms.: [evidence span withheld — anchor status: not_found]
