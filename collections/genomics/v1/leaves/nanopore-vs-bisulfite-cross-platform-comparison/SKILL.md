---
name: nanopore-vs-bisulfite-cross-platform-comparison
description: Use when comparing CpG site coverage between nanopore modification-call
  outputs (tsv with read_id, chromosome, pos_in_strand, strand, methylation_rate)
  and matched BS-seq calls across matched human or plant whole-genome datasets.
when_to_use_negative:
- The task is single-molecule methylation accuracy evaluation (AUROC/F1) against high-confidence
  bisulfite-validated sites, not whole-genome CpG coverage counting
- Datasets are not matched across platforms (no corresponding BS-seq run for the nanopore
  sample), so a coverage comparison would be confounded
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: DeepBAM
  role: Produces the nanopore CpG methylation call TSVs (read_id, chromosome, pos_in_strand,
    strand, methylation_rate) that are aggregated into unique CpG site counts
  repo: https://github.com/huicongyao/DeepBAM
- name: Dorado
  role: Alternative ONT methylation calling model whose outputs can serve as the nanopore
    side of the CpG coverage comparison
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae413
    title: 'DeepBAM: a high-accuracy single-molecule CpG methylation detection tool
      for Oxford nanopore sequencing'
  source_content_hash: sha256:e2eb1634be552f3544b86e49be5ad5908eaa30c3b71a27dea6ab082f9626251b
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
  edam_operation: http://edamontology.org/operation_3204
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# nanopore-vs-bisulfite-cross-platform-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies and compares the number of unique genomic CpG sites resolved by nanopore methylation calling (DeepBAM/Dorado) versus matched BS-seq methylation calls across human and plant datasets, testing whether nanopore's lower depth still yields broader CpG coverage.

## When to use

Use when comparing CpG site coverage between nanopore modification-call outputs (tsv with read_id, chromosome, pos_in_strand, strand, methylation_rate) and matched BS-seq calls across matched human or plant whole-genome datasets.

## When NOT to use

- The task is single-molecule methylation accuracy evaluation (AUROC/F1) against high-confidence bisulfite-validated sites, not whole-genome CpG coverage counting
- Datasets are not matched across platforms (no corresponding BS-seq run for the nanopore sample), so a coverage comparison would be confounded

## Inputs

- DeepBAM/Dorado per-dataset modification call TSVs with columns read_id, chromosome, pos_in_strand, strand, methylation_rate
- Matched BS-seq methylation calls for HG001, HG002 (GRCh38), A. thaliana, B. vulgaris, O. sativa

## Outputs

- Per-dataset table of unique CpG site counts for nanopore vs BS-seq with differences
- cpg_coverage_comparison.tsv

## How to apply

Load the DeepBAM/Dorado per-dataset modification call TSVs with pandas; for each nanopore dataset count unique CpG sites by aggregating distinct (chromosome, pos_in_strand) pairs. Load the matched BS-seq methylation calls (HG001, HG002 on GRCh38; A. thaliana, B. vulgaris, O. sativa) and count unique covered CpG genomic positions identically. Tabulate nanopore vs BS-seq coverage per dataset and compute the differences, then validate against the reported pattern: nanopore resolves substantially more CpGs than BS-seq on HG001, A. thaliana, B. vulgaris, and O. sativa, while on HG002 the ~246x BS-seq marginally outperforms the 39x nanopore data by about 0.06M CpG sites. Rationale: single-molecule nanopore readouts can resolve more genomic CpG sites despite lower depth, and DeepBAM's whole-genome methylation frequencies correlate >0.95 with BS-seq on four of five datasets, making the coverage comparison meaningful across platforms.

## Related tools

- **DeepBAM** (Produces the nanopore CpG methylation call TSVs (read_id, chromosome, pos_in_strand, strand, methylation_rate) that are aggregated into unique CpG site counts) — https://github.com/huicongyao/DeepBAM
- **Dorado** (Alternative ONT methylation calling model whose outputs can serve as the nanopore side of the CpG coverage comparison)

## Examples

```
import pandas as pd; df = pd.read_csv('hg001_deepbam_mods.tsv', sep='\t'); n_cpg = df[['chromosome','pos_in_strand']].drop_duplicates().shape[0]
```

## Evaluation signals

- Nanopore unique (chromosome, pos_in_strand) counts exceed BS-seq counts on HG001, A. thaliana, B. vulgaris, and O. sativa
- On HG002 the ~246x BS-seq data exceed the 39x nanopore data by approximately 0.06M CpG sites using GRCh38
- Output table cpg_coverage_comparison.tsv contains exactly the five datasets (HG001, HG002, A. thaliana, B. vulgaris, O. sativa) with per-platform counts and differences
- Counting uses distinct (chromosome, pos_in_strand) pairs only, so no duplicate positions inflate coverage

## Limitations

- On HG002 the high-depth (~246x) BS-seq marginally outperforms the 39x nanopore data, so nanopore does not universally resolve more CpGs
- No accession IDs or hosting locations are given for the three plant datasets or human cell line datasets, complicating raw-data reuse
- Dorado methylation calling performance differs significantly across heterogeneous human and plant datasets, so coverage numbers are caller-dependent

## Evidence
