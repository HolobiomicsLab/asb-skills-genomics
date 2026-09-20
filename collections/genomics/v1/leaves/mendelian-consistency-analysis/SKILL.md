---
name: mendelian-consistency-analysis
description: Use when comparing SNV variant callers on parent-offspring trio data
  (e.g., GIAB HG002/HG003/HG004) and a genotyping-consistency metric such as Mendelian
  Consistency Rate is needed to complement per-sample F1/precision/recall benchmarks.
when_to_use_negative:
- Input data is not a parent-offspring trio (no parental genotypes available to check
  child genotypes against).
- VCFs contain indels or multiallelic variants only — this skill is defined on biallelic
  SNV genotypes.
- You need site-level accuracy against a truth set (use GIAB benchmarking with stratifications
  such as GIAB v3.3 difficult/low-complexity regions instead of trio consistency).
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3577
tools:
- name: miniSNV
  role: SNV caller whose trio VCFs are evaluated; achieved highest F1-score and recall
    (>99.7%) on HG002, HG003, HG004 sup-datasets and highest MCR alongside Longshot
  repo: https://github.com/CuiMiao-HIT/miniSNV
- name: Clair3
  role: Comparison SNV caller (pileup + full-alignment) whose trio VCFs are evaluated;
    baseline MCR about 3 points below miniSNV and Longshot
  repo: https://github.com/nanoporetech/rerio/tree/master/clair3_models
- name: Longshot
  role: Diploid pair-HMM SNV caller whose trio VCFs are evaluated; achieves an MCR
    roughly 3% above Clair3 on the sup-dataset trio
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae473
    title: 'miniSNV: accurate and fast single nucleotide variant calling from nanopore
      sequencing data'
  source_content_hash: sha256:17cd106170e8d8553f7bef053fa28f4bf554709694a7fe948db743a8c9fd0e97
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/CuiMiao-HIT/miniSNV
  repo_url: https://github.com/CuiMiao-HIT/miniSNV
  tools:
  - Clair3
  - Longshot
  - miniSNV
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# mendelian-consistency-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates trio SNV callers by computing the Mendelian Consistency Rate (MCR): the fraction of child biallelic SNV genotypes (0/0, 0/1, 1/1) consistent with parental genotypes under Mendelian inheritance rules. Used to benchmark miniSNV, Clair3, and Longshot on the GIAB Ashkenazi Trio (HG002/HG003/HG004) ONT sup-datasets.

## When to use

Use when comparing SNV variant callers on parent-offspring trio data (e.g., GIAB HG002/HG003/HG004) and a genotyping-consistency metric such as Mendelian Consistency Rate is needed to complement per-sample F1/precision/recall benchmarks.

## When NOT to use

- Input data is not a parent-offspring trio (no parental genotypes available to check child genotypes against).
- VCFs contain indels or multiallelic variants only — this skill is defined on biallelic SNV genotypes.
- You need site-level accuracy against a truth set (use GIAB benchmarking with stratifications such as GIAB v3.3 difficult/low-complexity regions instead of trio consistency).

## Inputs

- Per-caller VCF files of trio SNV genotypes for HG002, HG003, and HG004 (miniSNV, Clair3, Longshot) generated on the GIAB Ashkenazi Trio sup-datasets

## Outputs

- Per-caller Mendelian Consistency Rate values written to mcr_table.csv

## How to apply

Load the per-caller trio VCFs (HG002 child, HG003/HG004 parents) produced by miniSNV, Clair3, and Longshot on the GIAB Ashkenazi Trio ONT sup-datasets (kit v14 chemistry, R10.4.1 flowcells, Dorado basecalled). Parse each VCF to extract biallelic SNV genotypes for all three trio members per caller. For each child genotype (0/0, 0/1, 1/1), apply Mendelian inheritance rules against the two parental genotypes to classify the child SNV as Mendelian-consistent or not. Compute each caller's MCR as the fraction of child SNVs consistent with the parental genotypes, then write per-caller MCR values to mcr_table.csv. Validation: the table should show miniSNV and Longshot MCRs exceeding Clair3's by roughly 3 percentage points, consistent with the paper's trio benchmark where miniSNV achieved the highest metrics on HG002, HG003, and HG004 sup-datasets.

## Related tools

- **miniSNV** (SNV caller whose trio VCFs are evaluated; achieved highest F1-score and recall (>99.7%) on HG002, HG003, HG004 sup-datasets and highest MCR alongside Longshot) — https://github.com/CuiMiao-HIT/miniSNV
- **Clair3** (Comparison SNV caller (pileup + full-alignment) whose trio VCFs are evaluated; baseline MCR about 3 points below miniSNV and Longshot) — https://github.com/nanoporetech/rerio/tree/master/clair3_models
- **Longshot** (Diploid pair-HMM SNV caller whose trio VCFs are evaluated; achieves an MCR roughly 3% above Clair3 on the sup-dataset trio)

## Examples

```
python compute_mcr.py --trio HG002 HG003 HG004 --vcfs miniSNV.vcf Clair3.vcf Longshot.vcf --out mcr_table.csv
```

## Evaluation signals

- The regenerated mcr_table.csv contains one MCR value per caller (miniSNV, Clair3, Longshot).
- miniSNV and Longshot MCRs exceed Clair3's MCR by approximately 3 percentage points.
- miniSNV's MCR is the highest among the three callers, consistent with its highest F1-score and recall (>99.7%) on HG002, HG003, and HG004 sup-datasets.
- Each MCR is a fraction in [0, 1] computed only over biallelic SNV child genotypes classified against both parental genotypes.
- All three callers are evaluated on the same trio (HG002 child; HG003/HG004 parents) from the same sup-dataset VCFs so the MCRs are directly comparable.

## Limitations

- MCR measures trio genotype consistency, not absolute variant-calling accuracy; a caller could appear consistent yet disagree with a truth set.
- Clair3 model compatibility depends on the basecalling model version — rerio Clair3 models are compatible with Clair3 v1.x but not v2.x, so caller versions must be pinned for reproducible comparisons.
- Filtering choices upstream of MCR (e.g., MAPQ < 20, R_alt thresholds in miniSNV) affect which variants enter the trio comparison and thus the reported MCR.

## Evidence

- [results] Trio dataset description: [evidence span withheld — anchor status: not_found]
