---
name: genomic-position-deduplication
description: Use when you hold per-read methylation call tables (read_id, chromosome,
  pos_in_strand, strand, methylation_rate) or BS-seq calls and need unique-site CpG
  coverage counts per dataset rather than per-read statistics.
when_to_use_negative:
- When the goal is methylation frequency accuracy (e.g. AUROC/F1 or correlation with
  BS-seq, where DeepBAM achieved 98.47% AUROC and >0.95 correlations) rather than
  site-coverage counting.
- When positions are not CpG genomic coordinates — deduplicating arbitrary per-read
  rows without the (chromosome, pos_in_strand) key would inflate counts by read multiplicity.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: DeepBAM
  role: Produces the per-read CpG methylation call TSVs (read_id, chromosome, pos_in_strand,
    strand, methylation_rate) whose genomic positions are deduplicated and counted
  repo: https://github.com/huicongyao/DeepBAM
- name: Dorado
  role: Alternative ONT methylation caller whose per-dataset modification outputs
    can serve as the nanopore side of the coverage comparison
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae413
    title: 'DeepBAM: a high-accuracy single-molecule CpG methylation detection tool
      for Oxford nanopore sequencing'
  source_content_hash: sha256:b4d52a3d61f90fba47dfd419bf6205391fe8b051a4d841bc713b5730802b531e
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# genomic-position-deduplication

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Counts unique covered CpG genomic sites by deduplicating distinct (chromosome, pos_in_strand) pairs from DeepBAM/Dorado per-read methylation call TSVs and matched BS-seq calls, enabling like-for-like CpG coverage comparison across nanopore and bisulfite datasets.

## When to use

Use when you hold per-read methylation call tables (read_id, chromosome, pos_in_strand, strand, methylation_rate) or BS-seq calls and need unique-site CpG coverage counts per dataset rather than per-read statistics.

## When NOT to use

- When the goal is methylation frequency accuracy (e.g. AUROC/F1 or correlation with BS-seq, where DeepBAM achieved 98.47% AUROC and >0.95 correlations) rather than site-coverage counting.
- When positions are not CpG genomic coordinates — deduplicating arbitrary per-read rows without the (chromosome, pos_in_strand) key would inflate counts by read multiplicity.

## Inputs

- DeepBAM/Dorado per-dataset modification call TSV files (columns: read_id, chromosome, pos_in_strand, strand, methylation_rate)
- Matched BS-seq methylation calls per dataset (HG001, HG002, A. thaliana, B. vulgaris, O. sativa)

## Outputs

- Per-dataset unique CpG site counts for nanopore and BS-seq
- Per-dataset coverage differences (nanopore minus BS-seq)
- cpg_coverage_comparison.tsv

## How to apply

Load each DeepBAM/Dorado modification call output (TSV with columns read_id, chromosome, pos_in_strand, strand, methylation_rate) with pandas, then aggregate distinct (chromosome, pos_in_strand) pairs to obtain the number of unique CpG sites covered; do the same on the matched BS-seq methylation calls. Because the same CpG can appear on both strands and multiple reads, deduplication must key on the genomic position pair, not read_id. Tabulate nanopore vs BS-seq unique CpG counts per dataset (HG001, HG002, A. thaliana, B. vulgaris, O. sativa) and compute per-dataset differences. The rationale is that nanopore depths (e.g. 39x on HG002) differ from BS-seq depths (e.g. ~246x), so coverage must be compared at the resolved-site level to interpret whether lower-depth nanopore data nonetheless resolves more CpGs.

## Related tools

- **DeepBAM** (Produces the per-read CpG methylation call TSVs (read_id, chromosome, pos_in_strand, strand, methylation_rate) whose genomic positions are deduplicated and counted) — https://github.com/huicongyao/DeepBAM
- **Dorado** (Alternative ONT methylation caller whose per-dataset modification outputs can serve as the nanopore side of the coverage comparison)

## Examples

```
import pandas as pd; df = pd.read_csv('HG001.mods.tsv', sep='\t'); sites = df[['chromosome','pos_in_strand']].drop_duplicates(); print(len(sites))
```

## Evaluation signals

- Counts equal the number of drop_duplicates() rows over the (chromosome, pos_in_strand) column pair — any count exceeding the distinct-pair total indicates incomplete deduplication.
- Reproduction shows nanopore covering more CpGs than BS-seq on HG001, A. thaliana, B. vulgaris, and O. sativa.
- On HG002, the ~246x BS-seq data outperforms the 39x nanopore data by about 0.06M CpG sites using GRCh38.
- cpg_coverage_comparison.tsv contains one row per dataset with both technology counts and the computed difference.
- Whole-genome methylation frequencies computed on the deduplicated sites correlate >0.95 with BS-seq on four of five datasets.

## Limitations

- Coverage outcomes are depth-dependent: on HG002 the high-depth (~246x) BS-seq exceeds 39x nanopore, so the nanopore-resolves-more-CpGs finding does not hold universally.
- No accession IDs or hosting locations are given for the three plant datasets or the human cell line datasets, hindering direct re-acquisition.
- No software version or release identifier is stated for DeepBAM, so deduplication on outputs from other releases may not match reported counts.

## Evidence

- [other] For each nanopore dataset, count the number of unique CpG sites by aggregating distinct (chromosome, pos_in_strand) pairs.: [evidence span withheld — anchor status: not_found]
