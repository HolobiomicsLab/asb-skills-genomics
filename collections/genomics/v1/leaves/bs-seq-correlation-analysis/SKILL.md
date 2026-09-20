---
name: bs-seq-correlation-analysis
description: Use when you have nanopore R10.4 BAM datasets with CpG modification calls
  and matched whole-genome BS-seq methylation frequency tracks, and you need to benchmark
  caller-derived methylation frequencies against bisulfite ground truth at controlled
  depths (e.g. 10x/30x).
when_to_use_negative:
- Evaluating single-molecule methylation calling accuracy (AUROC/F1) — use a labeled-site
  benchmark instead of population-level frequency correlation with BS-seq
- Data lacks matched BS-seq methylation frequency tracks, so no correlation ground
  truth can be computed
- Calling non-CpG methylation (other motif types than CG) — the workflow specifies
  motif_type=CG only
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0622
tools:
- name: DeepBAM
  role: Primary CpG methylation caller; extract_and_call_mods mode (motif_type=CG,
    kmer_size=51, batch_size=1024, num_workers=10) produces per-site methylation rates
    correlated against BS-seq
  repo: https://github.com/huicongyao/DeepBAM.git
- name: Dorado
  role: Baseline ONT methylation caller run on the same down-sampled datasets to provide
    comparable CpG modification calls
- name: samtools
  role: Down-samples each nanopore BAM to 10x and 30x coverage using samtools view
    -s
- name: Oxford Nanopore Technology (ONT)
  role: Sequencing platform whose R10.4 nanopore data and signal features underlie
    both callers' modification calls
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae413
    title: 'DeepBAM: a high-accuracy single-molecule CpG methylation detection tool
      for Oxford nanopore sequencing'
  source_content_hash: sha256:104b3797b929aeef662054e7fdc45aeb8f5864a7ff556b0c10a24ccb3960aee2
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
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# bs-seq-correlation-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies how well genome-wide CpG methylation frequency profiles from nanopore methylation callers (DeepBAM vs Dorado) agree with matched BS-seq tracks by computing Pearson correlations across down-sampled R10.4 human and plant datasets at 10x and 30x coverage.

## When to use

Use when you have nanopore R10.4 BAM datasets with CpG modification calls and matched whole-genome BS-seq methylation frequency tracks, and you need to benchmark caller-derived methylation frequencies against bisulfite ground truth at controlled depths (e.g. 10x/30x).

## When NOT to use

- Evaluating single-molecule methylation calling accuracy (AUROC/F1) — use a labeled-site benchmark instead of population-level frequency correlation with BS-seq
- Data lacks matched BS-seq methylation frequency tracks, so no correlation ground truth can be computed
- Calling non-CpG methylation (other motif types than CG) — the workflow specifies motif_type=CG only

## Inputs

- Nanopore R10.4 BAM alignment files (human cell lines including HG001 and plant species including O. sativa)
- Reference genome (FASTA) for feature extraction and modification calling
- Matched whole-genome BS-seq methylation frequency tracks
- Dorado methylation calling software
- DeepBAM with extract_and_call_mods mode

## Outputs

- Per-site CpG methylation rates from DeepBAM and Dorado for each down-sampled dataset
- Genome-wide CpG methylation frequency profiles per caller and coverage
- bs_seq_correlation_table.csv — Pearson correlation matrix of BS-seq vs each caller at 10x and 30x

## How to apply

Down-sample each nanopore BAM (five datasets: human cell lines including HG001 and plant species including O. sativa) to 10x and 30x coverage with samtools view -s. For each down-sampled dataset, run DeepBAM's extract_and_call_mods mode against the reference genome with motif_type=CG, kmer_size=51, batch_size=1024, num_workers=10, so features are asynchronously extracted and passed to the model to produce per-site modification calls; run Dorado methylation calling on the same data for comparison. Aggregate per-site methylation rates from both callers into genome-wide CpG methylation frequency profiles, then compute Pearson correlation coefficients against the matched BS-seq methylation frequency tracks. Assemble all correlations into a single table (bs_seq_correlation_table.csv) with rows for both coverages and both callers; the rationale is that population-level frequency agreement with BS-seq complements single-molecule AUROC/F1 evaluation, and DeepBAM was designed to be more stable across heterogeneous human and plant datasets than Dorado.

## Related tools

- **DeepBAM** (Primary CpG methylation caller; extract_and_call_mods mode (motif_type=CG, kmer_size=51, batch_size=1024, num_workers=10) produces per-site methylation rates correlated against BS-seq) — https://github.com/huicongyao/DeepBAM.git
- **Dorado** (Baseline ONT methylation caller run on the same down-sampled datasets to provide comparable CpG modification calls)
- **samtools** (Down-samples each nanopore BAM to 10x and 30x coverage using samtools view -s)
- **Oxford Nanopore Technology (ONT)** (Sequencing platform whose R10.4 nanopore data and signal features underlie both callers' modification calls)

## Examples

```
samtools view -s 0.1 -b HG001.bam > HG001_10x.bam; DeepBAM extract_and_call_mods --motif_type CG --kmer_size 51 --batch_size 1024 --num_workers 10
```

## Evaluation signals

- bs_seq_correlation_table.csv exists in CSV format with rows for both coverages (10x and 30x) and both callers
- DeepBAM correlations exceed 0.95 on four of five datasets (HG001 is the exception at 10x, 0.9032)
- DeepBAM correlations exceed Dorado in every dataset and depth comparison
- Correlation values fall in the reported ranges: 10x from 0.9032 (HG001) to 0.9847 (O. sativa); 30x from 0.9212 (HG001) to 0.9872 (O. sativa)
- Down-sampled BAMs verify at approximately 10x and 30x mean coverage after samtools view -s

## Limitations

- Dorado methylation calling performance differs significantly across heterogeneous human and plant datasets, so Dorado correlations are unstable baselines
- Correlations are population-level frequency agreements; they do not certify single-molecule calling accuracy, and ongoing optimization of DeepBAM is acknowledged as necessary
- No accession IDs or hosting locations are given for the three plant datasets or human cell line datasets, limiting reproducibility of the down-sampling inputs

## Evidence

- [abstract] whole genome methylation frequencies have achieved >0.95 correlations with BS-seq on four of five datasets, outperforming Dorado in all instances: "whole genome methylation frequencies have achieved >0.95 correlations with BS-seq on four of five datasets, outperforming Dorado in all instances"
- [methods] asynchronously extract features and pass data to model to get modification result: "asynchronously extract features and pass data to model to get modification result"
- [discussion] indicated by correlation coefficients greater than 0.95 in most datasets, rivals that of BS-seq: "indicated by correlation coefficients greater than 0.95 in most datasets, rivals that of BS-seq"
- [readme] We've enhanced the feature extraction mechanism, resulting in significantly faster processing times for modification calling while also marghinally boosting calling accuracy.: "We've enhanced the feature extraction mechanism, resulting in significantly faster processing times for modification calling while also marghinally boosting calling accuracy."
