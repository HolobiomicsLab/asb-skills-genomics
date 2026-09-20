---
name: multi-dataset-benchmark-comparison
description: Use when evaluating whether a nanopore methylation caller's performance
  generalizes across diverse datasets (e.g., human cell lines and plant species) by
  comparing its methylation frequencies against a bisulfite-sequencing ground truth
  at controlled coverage depths.
when_to_use_negative:
- The task is single-molecule per-read methylation calling accuracy (AUROC/F1 evaluation)
  rather than genome-wide frequency-level comparison against BS-seq
- The caller being evaluated is trained specifically for plant species only (e.g.,
  DeepSignal-plant), so cross-kingdom benchmarking against human datasets is out of
  scope
- Input data is from an older pore chemistry (pre-R10.4); the article notes only a
  small number of tools including DeepMod2 and Dorado are compatible with R10.4
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: DeepBAM
  role: Primary CpG methylation caller run in extract_and_call_mods mode (CG motif,
    kmer_size=51) on down-sampled BAMs to produce per-site methylation rates
  repo: https://github.com/huicongyao/DeepBAM
- name: Dorado
  role: Baseline ONT methylation caller run on the same down-sampled datasets for
    head-to-head comparison
- name: samtools
  role: Down-samples nanopore BAM datasets to 10x and 30x coverage using 'samtools
    view -s'
- name: Oxford Nanopore Technology (ONT)
  role: Sequencing platform generating the R10.4 nanopore datasets under evaluation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae413
    title: 'DeepBAM: a high-accuracy single-molecule CpG methylation detection tool
      for Oxford nanopore sequencing'
  source_content_hash: sha256:a5c6e813483bd27146dd79f7878f14f07e5b2a7ab1deda02ea47bfd76f2fc746
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
  - Oxford Nanopore Technology (ONT)
  - samtools
  edam_operation: http://edamontology.org/operation_3204
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# multi-dataset-benchmark-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a methylation caller (e.g., DeepBAM vs Dorado) across heterogeneous human and plant nanopore R10.4 datasets by down-sampling BAMs to 10x-30x coverage, computing genome-wide CpG methylation frequencies, and correlating them against matched BS-seq tracks to expose dataset-dependent performance differences.

## When to use

Use when evaluating whether a nanopore methylation caller's performance generalizes across diverse datasets (e.g., human cell lines and plant species) by comparing its methylation frequencies against a bisulfite-sequencing ground truth at controlled coverage depths.

## When NOT to use

- The task is single-molecule per-read methylation calling accuracy (AUROC/F1 evaluation) rather than genome-wide frequency-level comparison against BS-seq
- The caller being evaluated is trained specifically for plant species only (e.g., DeepSignal-plant), so cross-kingdom benchmarking against human datasets is out of scope
- Input data is from an older pore chemistry (pre-R10.4); the article notes only a small number of tools including DeepMod2 and Dorado are compatible with R10.4

## Inputs

- Nanopore BAM files from five human and plant R10.4 datasets (e.g., HG001 human cell line and O. sativa plant data)
- Reference genome FASTA for each dataset
- Dorado methylation calls on the same down-sampled datasets
- Matched BS-seq (bisulfite sequencing) methylation frequency tracks per dataset

## Outputs

- bs_seq_correlation_table.csv with rows for 10x and 30x coverages and both callers
- Genome-wide CpG methylation frequency profiles for DeepBAM and Dorado per down-sampled dataset
- Pearson correlation coefficients per dataset-depth-caller combination

## How to apply

Down-sample each nanopore BAM dataset to 10x and 30x coverage using 'samtools view -s' so caller performance is compared at controlled depths. For each down-sampled dataset, run DeepBAM's extract_and_call_mods mode with the CG motif (motif_type=CG, kmer_size=51, batch_size=1024, num_workers=10) to asynchronously extract features and call CpG modifications, and run Dorado on the same inputs for comparison. Aggregate per-site methylation rates into genome-wide CpG methylation frequency profiles for each caller. Load matched BS-seq methylation frequency tracks and compute Pearson correlation coefficients between BS-seq and each caller's frequencies for every dataset-depth pair, then assemble the values into a single CSV table. Rationale: Dorado's methylation calling model was shown to perform significantly differently across heterogeneous human and plant R10.4 datasets, so multi-dataset evaluation with a consistent ground truth is required to demonstrate universal robustness.

## Related tools

- **DeepBAM** (Primary CpG methylation caller run in extract_and_call_mods mode (CG motif, kmer_size=51) on down-sampled BAMs to produce per-site methylation rates) — https://github.com/huicongyao/DeepBAM
- **Dorado** (Baseline ONT methylation caller run on the same down-sampled datasets for head-to-head comparison)
- **samtools** (Down-samples nanopore BAM datasets to 10x and 30x coverage using 'samtools view -s')
- **Oxford Nanopore Technology (ONT)** (Sequencing platform generating the R10.4 nanopore datasets under evaluation)

## Examples

```
samtools view -s 0.1 input.bam > downsampled_10x.bam && deepbam extract_and_call_mods --motif_type CG --kmer_size 51 --batch_size 1024 --num_workers 10 -i downsampled_10x.bam -r reference.fa
```

## Evaluation signals

- bs_seq_correlation_table.csv exists with rows for both coverages (10x, 30x) and both callers
- DeepBAM Pearson correlations exceed 0.95 on four of five datasets (10x range: 0.9032 HG001 to 0.9847 O. sativa; 30x range: 0.9212 HG001 to 0.9872 O. sativa)
- DeepBAM correlations outperform Dorado in every dataset and depth instance
- Both callers are evaluated on identical down-sampled BAMs at the same coverage (10x and 30x) so correlations are directly comparable
- Correlations increase or stay comparable from 10x to 30x depth, consistent with expected depth-frequency stability

## Limitations

- Dorado methylation calling performance differs significantly across heterogeneous human and plant datasets, so single-dataset benchmarks are misleading
- Genome-wide frequency correlations do not reach 0.95 on all five datasets (HG001 is below 0.95 at both 10x and 30x)
- Dataset accessions and hosting locations for the three plant datasets and human cell line datasets are not provided in the article
- Quantitative speed/memory comparison is deferred to supplementary material not present in the main text
- Ongoing optimization of DeepBAM is acknowledged as necessary, implying incomplete current performance

## Evidence
