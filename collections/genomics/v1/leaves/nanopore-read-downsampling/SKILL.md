---
name: nanopore-read-downsampling
description: Use when comparing methylation callers or basecalling models on nanopore
  data and you need standardized, depth-matched BAM inputs (e.g., 10x and 30x coverage)
  from high-coverage R10.4 datasets.
when_to_use_negative:
- The input BAM is already at or below the target depth — further sampling would erode
  coverage and bias frequency estimates
- The analysis goal is single-molecule per-read methylation accuracy benchmarking
  against ground-truth labels, where down-sampling frequency estimates to genome-wide
  tracks is not the target
- Reference genomes or matched BS-seq tracks are unavailable, so depth-controlled
  downstream correlation cannot be computed
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: samtools
  role: Performs read-level down-sampling of nanopore BAM files to 10x and 30x coverage
    via `samtools view -s`
- name: DeepBAM
  role: Downstream methylation caller run on the down-sampled BAMs (extract_and_call_mods,
    motif_type=CG, kmer_size=51, batch_size=1024, num_workers=10)
  repo: https://github.com/huicongyao/DeepBAM.git
- name: Dorado
  role: Comparison methylation caller run on the identical down-sampled BAMs
- name: Oxford Nanopore Technology (ONT)
  role: Source platform for the R10.4 nanopore reads being down-sampled
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae413
    title: 'DeepBAM: a high-accuracy single-molecule CpG methylation detection tool
      for Oxford nanopore sequencing'
  source_content_hash: sha256:259122870af89696beeb434fb9196991eb6911acef0803d245c0ef463167fc6c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/xiaochuanle/DeepBAM
  repo_url: https://github.com/huicongyao/DeepBAM.git
  tools:
  - DeepBAM
  - Dorado
  - Oxford Nanopore Technology (ONT)
  - samtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# nanopore-read-downsampling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Down-samples ONT nanopore BAM alignments to controlled sequencing depths (10x and 30x in the DeepBAM study) using samtools view -s, enabling depth-controlled benchmarking of methylation callers against BS-seq.

## When to use

Use when comparing methylation callers or basecalling models on nanopore data and you need standardized, depth-matched BAM inputs (e.g., 10x and 30x coverage) from high-coverage R10.4 datasets.

## When NOT to use

- The input BAM is already at or below the target depth — further sampling would erode coverage and bias frequency estimates
- The analysis goal is single-molecule per-read methylation accuracy benchmarking against ground-truth labels, where down-sampling frequency estimates to genome-wide tracks is not the target
- Reference genomes or matched BS-seq tracks are unavailable, so depth-controlled downstream correlation cannot be computed

## Inputs

- High-coverage Oxford Nanopore R10.4 BAM alignment files (sorted/indexed)
- Target coverage depths (e.g., 10x and 30x)

## Outputs

- Down-sampled BAM files at each target depth per source dataset
- Depth-matched input sets for methylation callers (DeepBAM, Dorado)
- Correlation table (e.g., bs_seq_correlation_table.csv) of caller vs BS-seq Pearson correlations per depth and dataset

## How to apply

Take each high-coverage nanopore BAM dataset (in the DeepBAM study, five human and plant R10.4 datasets including HG001 and O. sativa) and down-sample it to target coverage with `samtools view -s`, producing 10x and 30x coverage variants. Choose the -s sampling fraction so that the resulting mean depth approximates the target depth for each dataset, since original coverage differs between runs. Use identical down-sampled BAM sets for all callers being compared (e.g., DeepBAM's extract_and_call_mods and Dorado) so downstream correlations with matched BS-seq tracks reflect caller differences rather than input differences. Aggregate per-site methylation calls into genome-wide CpG methylation frequency profiles and compute Pearson correlations against BS-seq to confirm depth effects — the DeepBAM study reported 10x correlations of 0.9032 (HG001) to 0.9847 (O. sativa), improving at 30x (0.9212–0.9872).

## Related tools

- **samtools** (Performs read-level down-sampling of nanopore BAM files to 10x and 30x coverage via `samtools view -s`)
- **DeepBAM** (Downstream methylation caller run on the down-sampled BAMs (extract_and_call_mods, motif_type=CG, kmer_size=51, batch_size=1024, num_workers=10)) — https://github.com/huicongyao/DeepBAM.git
- **Dorado** (Comparison methylation caller run on the identical down-sampled BAMs)
- **Oxford Nanopore Technology (ONT)** (Source platform for the R10.4 nanopore reads being down-sampled)

## Examples

```
samtools view -s <fraction> -b dataset.bam > dataset_10x.bam
```

## Evaluation signals

- Down-sampled BAM mean coverage is close to the target depth (10x or 30x) for each dataset
- Down-sampled BAMs remain sorted/indexed and reference-consistent with the original genome build
- Same down-sampled BAM set feeds both DeepBAM and Dorado so caller comparisons are paired
- Resulting genome-wide CpG methylation frequency correlations with BS-seq fall in the reported ranges (DeepBAM 10x: 0.9032–0.9847; 30x: 0.9212–0.9872) and exceed Dorado in all dataset/depth instances
- DeepBAM correlations exceed 0.95 on four of five datasets, consistent with the published finding

## Limitations

- Read-sampling fractions must be tuned per dataset because original coverage differs; a single -s fraction will not hit target depth uniformly
- Down-sampling reduces per-site molecule counts, lowering methylation frequency precision (correlations drop from 30x to 10x, e.g., 0.9212 vs 0.9847 for O. sativa)
- Human cell line datasets are relatively homogeneous in CpG methylation frequency compared with the heterogeneous plant datasets, so depth effects may not transfer uniformly across species
- No software version or release identifier for DeepBAM was stated, and dataset accession IDs/hosting locations were not given, complicating exact reproduction

## Evidence

- [methods] Downsample each of the five nanopore BAM datasets ... to 10x and 30x coverage using samtools view -s: [evidence span withheld — anchor status: not_found]
