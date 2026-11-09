---
name: tabular-aggregation-across-datasets
description: Use when comparing coverage breadth (unique genomic positions) between
  two or more platforms — e.g., DeepBAM/Dorado nanopore TSV calls vs BS-seq methylation
  tables — across multiple matched datasets.
when_to_use_negative:
- The goal is per-read single-molecule methylation state calling rather than aggregating
  calls to unique genomic CpG positions
- The comparison is of methylation frequency correlation with BS-seq (a different
  skill; DeepBAM targets >0.95 correlation there), not coverage breadth
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3678
tools:
- name: DeepBAM
  role: Produces the nanopore CpG methylation calls (TSV outputs) that are aggregated
    to unique CpG sites
  repo: https://github.com/huicongyao/DeepBAM
- name: Dorado
  role: ONT's official methylation caller whose per-dataset outputs can be aggregated
    the same way as the DeepBAM calls
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae413
    title: 'DeepBAM: a high-accuracy single-molecule CpG methylation detection tool
      for Oxford nanopore sequencing'
  source_content_hash: sha256:0e642292c3ef52e84fcb3b8943bc308abcb1a87e128a4f90f223673cadc19008
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3678
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# tabular-aggregation-across-datasets

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aggregates per-dataset tabular methylation call outputs into a unified CpG coverage comparison table, counting unique CpG sites per platform across human and plant datasets to test whether nanopore resolves more CpGs than BS-seq.

## When to use

Use when comparing coverage breadth (unique genomic positions) between two or more platforms — e.g., DeepBAM/Dorado nanopore TSV calls vs BS-seq methylation tables — across multiple matched datasets.

## When NOT to use

- The goal is per-read single-molecule methylation state calling rather than aggregating calls to unique genomic CpG positions
- The comparison is of methylation frequency correlation with BS-seq (a different skill; DeepBAM targets >0.95 correlation there), not coverage breadth

## Inputs

- DeepBAM/Dorado modification call TSV files with columns read_id, chromosome, pos_in_strand, strand, methylation_rate (per nanopore dataset)
- Matched BS-seq methylation call tables per dataset (HG001, HG002, A. thaliana, B. vulgaris, O. sativa)

## Outputs

- Tabulated nanopore vs BS-seq unique CpG site counts per dataset
- Per-dataset coverage differences (nanopore minus BS-seq) in units of M CpG sites
- cpg_coverage_comparison.tsv

## How to apply

Load the DeepBAM/Dorado per-dataset modification call outputs (TSV files with columns read_id, chromosome, pos_in_strand, strand, methylation_rate) with pandas. For each nanopore dataset, count unique CpG sites by aggregating distinct (chromosome, pos_in_strand) pairs, which collapses multi-read observations into single genomic positions. Load the matched BS-seq methylation calls for each of the five datasets (HG001, HG002, A. thaliana, B. vulgaris, O. sativa) and count unique covered CpG genomic positions identically so the two platforms are tabulated on a common definition. Compute per-dataset differences (nanopore minus BS-seq) and write the table to cpg_coverage_comparison.tsv. The rationale is that sequencing depth alone is misleading: DeepBAM's single-molecule CpG calls on ONT R10.4 data can resolve more genomic CpGs than even much deeper bisulfite data on most genomes.

## Related tools

- **DeepBAM** (Produces the nanopore CpG methylation calls (TSV outputs) that are aggregated to unique CpG sites) — https://github.com/huicongyao/DeepBAM
- **Dorado** (ONT's official methylation caller whose per-dataset outputs can be aggregated the same way as the DeepBAM calls)

## Examples

```
import pandas as pd; df = pd.read_csv('deepbam_HG001.tsv', sep='\t'); n_cpg = df[['chromosome','pos_in_strand']].drop_duplicates().shape[0]
```

## Evaluation signals

- Nanopore unique CpG counts exceed BS-seq counts on HG001, A. thaliana, B. vulgaris, and O. sativa
- On HG002 the ~246x BS-seq data marginally outperforms 39x nanopore by about 0.06M CpG sites using GRCh38
- Counts are computed over distinct (chromosome, pos_in_strand) pairs, so the same key definition applies to both platforms
- Output file cpg_coverage_comparison.tsv is written with per-dataset rows and computed differences

## Limitations

- Result is reference-genome dependent (the HG002 comparison is stated using GRCh38); switching assemblies changes CpG counts
- Coverage comparison is sensitive to sequencing depth on the BS-seq side — at ~246x, BS-seq can exceed 39x nanopore
- No accession IDs or hosting locations are given for the three plant datasets or human cell line datasets in the article, so raw inputs must be sourced elsewhere

## Evidence

- [other] Nanopore profiling resolved substantially more CpG sites than BS-seq on HG001, A. thaliana, B. vulgaris, and O. sativa, while on HG002 the ~246× BS-seq data marginally outperformed 39× nanopore by 0.06M CpG sites using GRCh38.: [evidence span withheld — anchor status: not_found]
- [other] TSV column layout for modification calls.: [evidence span withheld — anchor status: not_found]
