---
name: cross-species-methylation-benchmarking
description: Use when evaluating whether a nanopore CpG methylation calling model
  generalizes across human and plant R10.4 datasets, using bisulfite-seq ground truth
  and AUC/F1/balanced accuracy metrics.
when_to_use_negative:
- Data is from older non-R10.4 flowcells — the article notes only a small number of
  tools are compatible with R10.4 (DeepMod2 and Dorado), so benchmark expectations
  may not transfer
- You need whole-genome methylation frequency correlation rather than single-molecule
  calling performance — a separate evaluation (>0.95 correlation with BS-seq) is reported
  for that
- Input datasets lack bisulfite-defined ground-truth methylation sites, making AUC/F1/balanced
  accuracy against high-confidence sites impossible
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0622
tools:
- name: DeepBAM
  role: Primary CpG methylation caller run via 'extract_and_call_mods' on POD5/BAM/reference
    inputs to produce per-read methylation TSVs
  repo: https://github.com/xiaochuanle/DeepBAM
- name: Dorado
  role: Baseline methylation caller (official ONT model) whose cross-dataset performance
    is being benchmarked
- name: DeepBAM (huicongyao repo)
  role: Cloned source used to build and run the DeepBAM executable
  repo: https://github.com/huicongyao/DeepBAM
- name: libtorch
  role: Deep learning runtime supporting DeepBAM's neural-network methylation calling
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae413
    title: 'DeepBAM: a high-accuracy single-molecule CpG methylation detection tool
      for Oxford nanopore sequencing'
  source_content_hash: sha256:b9f20e2ff9a36d9f17d346d69228d0fa344f5c201016e54dad35fd6c6f045c2c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/xiaochuanle/DeepBAM
  repo_url: https://github.com/xiaochuanle/DeepBAM
  tools:
  - DeepBAM
  - DeepBAM (huicongyao repo)
  - Dorado
  - libtorch
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cross-species-methylation-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks single-molecule CpG methylation callers (DeepBAM vs Dorado) across heterogeneous human and plant Oxford Nanopore R10.4 datasets using ROC AUC, F1, and balanced accuracy against bisulfite-defined ground truth, exposing dataset-dependent variability of the official Dorado model.

## When to use

Use when evaluating whether a nanopore CpG methylation calling model generalizes across human and plant R10.4 datasets, using bisulfite-seq ground truth and AUC/F1/balanced accuracy metrics.

## When NOT to use

- Data is from older non-R10.4 flowcells — the article notes only a small number of tools are compatible with R10.4 (DeepMod2 and Dorado), so benchmark expectations may not transfer
- You need whole-genome methylation frequency correlation rather than single-molecule calling performance — a separate evaluation (>0.95 correlation with BS-seq) is reported for that
- Input datasets lack bisulfite-defined ground-truth methylation sites, making AUC/F1/balanced accuracy against high-confidence sites impossible

## Inputs

- POD5 nanopore signal files (five R10.4 datasets: human cell lines and plant samples)
- Sorted BAM alignments for each dataset
- Reference genome FASTA for each dataset
- Pre-trained DeepBAM 51-mer model module
- High-confidence bisulfite (BS-seq)-defined positive and negative CpG methylation sites per dataset

## Outputs

- Per-read CpG methylation TSVs from DeepBAM
- CpG methylation calls from Dorado
- methylation_calling_metrics.tsv — per-dataset, per-tool table of ROC AUC, balanced accuracy, and F1 score

## How to apply

Run DeepBAM's 'extract_and_call_mods' executable on each POD5 dataset with its sorted BAM, reference genome, and the pre-trained 51-mer module (kmer_size=51, motif_type=CG, num_workers=10, sub_thread_per_worker=4, batch_size=1024) to obtain per-read CpG methylation TSVs, then run Dorado on the same POD5 files as the baseline caller. Align each tool's called CpG sites to high-confidence bisulfite-defined positive and negative methylation sites per dataset, balance the positive/negative CpG classes, and compute ROC AUC, balanced accuracy, and F1 per tool per dataset. Compile the metrics into a single comparison table (e.g. methylation_calling_metrics.tsv) and judge reproduction against the reported ranges: DeepBAM AUCs >=0.956 (average 98.47%), F1 average 94.97%, balanced accuracy 92.20%-98.96%, and Dorado AUCs within 0.8825-0.9852. The rationale is that Dorado's performance varies significantly across heterogeneous human and plant datasets, so cross-dataset comparison reveals robustness that single-dataset evaluation hides.

## Related tools

- **DeepBAM** (Primary CpG methylation caller run via 'extract_and_call_mods' on POD5/BAM/reference inputs to produce per-read methylation TSVs) — https://github.com/xiaochuanle/DeepBAM
- **Dorado** (Baseline methylation caller (official ONT model) whose cross-dataset performance is being benchmarked)
- **DeepBAM (huicongyao repo)** (Cloned source used to build and run the DeepBAM executable) — https://github.com/huicongyao/DeepBAM
- **libtorch** (Deep learning runtime supporting DeepBAM's neural-network methylation calling)

## Examples

```
extract_and_call_mods --pod5 <dataset>.pod5 --bam <sorted.bam> --ref <genome.fa> --mod_file <51mer.module> --kmer_size 51 --motif_type CG --num_workers 10 --sub_thread_per_worker 4 --batch_size 1024
```

## Evaluation signals

- DeepBAM reproduced AUCs are >=0.956 on every dataset with average AUC 98.47% and up to 7.36% improvement over Dorado
- DeepBAM average F1 is 94.97% (up to 16.24% improvement) and balanced accuracies fall within 92.20%-98.96%
- Dorado AUCs fall within the reported 0.8825-0.9852 range, confirming dataset-dependent variability
- methylation_calling_metrics.tsv contains one row per dataset-tool pair covering all five R10.4 datasets with all three metrics populated
- Positive and negative CpG classes are balanced per dataset before metric computation

## Limitations

- Dorado's methylation calling performance varies significantly across heterogeneous human and plant datasets, so single-dataset results are not generalizable
- No accession IDs or hosting locations are given for the three plant datasets or human cell line datasets in the article text
- Quantitative speed/memory comparisons are deferred to supplementary material not present in the text
- The article acknowledges ongoing optimization of DeepBAM is necessary, implying incomplete current performance

## Evidence
