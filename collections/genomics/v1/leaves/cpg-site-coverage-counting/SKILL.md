---
name: cpg-site-coverage-counting
description: Use when comparing the CpG coverage breadth of nanopore modification-call
  TSVs (read_id, chromosome, pos_in_strand, strand, methylation_rate) against BS-seq
  methylation calls on matched human or plant datasets.
when_to_use_negative:
- The goal is methylation accuracy (AUROC/F1) at single-molecule level rather than
  coverage breadth counting
- Input is whole-genome methylation frequency tables for BS-seq correlation analysis
  (>0.95 coefficient comparison) instead of per-site call records
- Datasets are not matched between nanopore and BS-seq (different genomes or assemblies),
  since distinct (chromosome, pos_in_strand) counts would not be comparable
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0622
tools:
- name: DeepBAM
  role: Source of the nanopore CpG methylation call outputs (TSV) whose unique (chromosome,
    pos_in_strand) pairs are counted
  repo: https://github.com/huicongyao/DeepBAM
- name: Dorado
  role: Alternative ONT methylation caller producing comparable per-dataset modification
    call outputs for the coverage comparison
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae413
    title: 'DeepBAM: a high-accuracy single-molecule CpG methylation detection tool
      for Oxford nanopore sequencing'
  source_content_hash: sha256:1b9e00fe9341272170d3416b833f9397ede538d5a97858dbaeba56cf4b62391f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/xiaochuanle/DeepBAM
  repo_url: https://github.com/huicongyao/DeepBAM
  tools:
  - DeepBAM
  - Dorado
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# cpg-site-coverage-counting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Counts unique genomic CpG sites covered by nanopore methylation calls (DeepBAM/Dorado TSV outputs) versus matched BS-seq calls, per dataset, to quantify which platform resolves more CpG sites despite lower nanopore depth.

## When to use

Use when comparing the CpG coverage breadth of nanopore modification-call TSVs (read_id, chromosome, pos_in_strand, strand, methylation_rate) against BS-seq methylation calls on matched human or plant datasets.

## When NOT to use

- The goal is methylation accuracy (AUROC/F1) at single-molecule level rather than coverage breadth counting
- Input is whole-genome methylation frequency tables for BS-seq correlation analysis (>0.95 coefficient comparison) instead of per-site call records
- Datasets are not matched between nanopore and BS-seq (different genomes or assemblies), since distinct (chromosome, pos_in_strand) counts would not be comparable

## Inputs

- DeepBAM/Dorado per-dataset modification call TSV files (columns: read_id, chromosome, pos_in_strand, strand, methylation_rate) for HG001, HG002, A. thaliana, B. vulgaris, O. sativa
- Matched BS-seq methylation call tables for the same five datasets

## Outputs

- cpg_coverage_comparison.tsv with per-dataset nanopore CpG counts, BS-seq CpG counts, and differences
- Validation verdict: nanopore covers more CpGs than BS-seq on HG001, A. thaliana, B. vulgaris, O. sativa; HG002 BS-seq (~246x) exceeds 39x nanopore by ~0.06M CpGs (GRCh38)

## How to apply

Load each per-dataset nanopore modification call output (TSV with columns read_id, chromosome, pos_in_strand, strand, methylation_rate) with pandas and aggregate distinct (chromosome, pos_in_strand) pairs to count unique covered CpG sites. Apply the identical distinct-position aggregation to the matched BS-seq methylation calls for each dataset. Tabulate nanopore vs BS-seq CpG coverage per dataset (HG001, HG002, A. thaliana, B. vulgaris, O. sativa) and compute per-dataset differences. Rationale: nanopore runs are much lower depth (e.g., 39x on HG002) than BS-seq (e.g., ~246x), yet single-molecule reads can resolve more distinct CpGs; the count comparison tests this trade-off directly. Write the tabulated comparison to cpg_coverage_comparison.tsv.

## Related tools

- **DeepBAM** (Source of the nanopore CpG methylation call outputs (TSV) whose unique (chromosome, pos_in_strand) pairs are counted) — https://github.com/huicongyao/DeepBAM
- **Dorado** (Alternative ONT methylation caller producing comparable per-dataset modification call outputs for the coverage comparison)

## Examples

```
import pandas as pd; df = pd.read_csv('HG001_deepbam_calls.tsv', sep='\t'); n_cpg = df[['chromosome','pos_in_strand']].drop_duplicates().shape[0]
```

## Evaluation signals

- Nanopore unique CpG count exceeds BS-seq count on HG001, A. thaliana, B. vulgaris, and O. sativa
- On HG002, BS-seq (~246x) exceeds 39x nanopore by approximately 0.06M CpG sites using GRCh38
- Counts are based on distinct (chromosome, pos_in_strand) pairs, deduplicated across reads before comparison
- Output file cpg_coverage_comparison.tsv exists and contains all five datasets with nanopore, BS-seq, and difference columns

## Limitations

- HG002 is an edge case: higher-depth BS-seq (~246x) can outperform lower-depth nanopore (39x), so 'nanopore always covers more' is not a valid assumption
- Coverage comparison depends on matched reference genome (e.g., GRCh38) between the two platforms
- Dorado's performance and coverage vary significantly across heterogeneous human and plant datasets, so caller choice affects counts

## Evidence

- [results] Nanopore profiling resolved substantially more CpG sites than BS-seq on HG001, A. thaliana, B. vulgaris, and O. sativa: [evidence span withheld — anchor status: not_found]
- [methods] count unique CpG sites by aggregating distinct (chromosome, pos_in_strand) pairs: [evidence span withheld — anchor status: not_found]
- [methods] tsv files with read_id, chromosome, pos_in_strand, strand, methylation_rate: [evidence span withheld — anchor status: not_found]
