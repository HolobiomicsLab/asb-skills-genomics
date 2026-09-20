---
name: nanopore-cpg-methylation-calling
description: Use when evaluating whether a CpG methylation caller is robust across
  heterogeneous human/plant nanopore R10.4 datasets, or when calling per-read CpG
  methylation states from POD5 + sorted BAM data.
when_to_use_negative:
- Data come from earlier nanopore flowcells (pre-R10.4) — DeepBAM and Dorado compatibility
  was demonstrated specifically for the R10.4 system
- Methylation calls already exist and only whole-genome methylation frequency correlation
  (not single-molecule benchmarking) is required — use a downstream correlation skill
  instead
- Targeting non-CpG motifs (e.g. CHH/CHG in plants) — the pipeline uses motif_type=CG
  only
edam_topics:
- http://edamontology.org/topic_0160
tools:
- name: DeepBAM
  role: Deep neural network CpG methylation caller; its 'extract_and_call_mods' executable
    asynchronously extracts features from POD5/BAM and calls CpG methylation states
  repo: github.com/huicongyao/DeepBAM.git
- name: Dorado
  role: ONT's officially supplied methylation calling model; run on the same POD5
    files and references to produce the baseline per-read CpG calls for comparison
- name: DeepMod2
  role: Alternative R10.4-compatible methylation caller mentioned as one of few tools
    compatible with R10.4
- name: libtorch
  role: Deep learning runtime used by DeepBAM for model inference during modification
    calling
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae413
    title: 'DeepBAM: a high-accuracy single-molecule CpG methylation detection tool
      for Oxford nanopore sequencing'
  source_content_hash: sha256:fbd6208bee9be8820e355d186e145edb7f07af31bcc6dae2cd873d8036c13163
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/xiaochuanle/DeepBAM
  tools:
  - DeepBAM
  - DeepMod2
  - Dorado
  - libtorch
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# nanopore-cpg-methylation-calling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A skill for benchmarking single-molecule CpG methylation calling on Oxford Nanopore R10.4 data, using DeepBAM's asynchronously-executed extract-and-call pipeline against ONT's official Dorado model, validated against high-confidence bisulfite (BS-seq) sites.

## When to use

Use when evaluating whether a CpG methylation caller is robust across heterogeneous human/plant nanopore R10.4 datasets, or when calling per-read CpG methylation states from POD5 + sorted BAM data.

## When NOT to use

- Data come from earlier nanopore flowcells (pre-R10.4) — DeepBAM and Dorado compatibility was demonstrated specifically for the R10.4 system
- Methylation calls already exist and only whole-genome methylation frequency correlation (not single-molecule benchmarking) is required — use a downstream correlation skill instead
- Targeting non-CpG motifs (e.g. CHH/CHG in plants) — the pipeline uses motif_type=CG only

## Inputs

- POD5 nanopore read files (R10.4 datasets)
- Sorted BAM alignment file
- Reference genome FASTA
- Pre-trained DeepBAM 51-mer model module
- High-confidence bisulfite-defined methylation positive/negative site lists for validation

## Outputs

- Per-read CpG methylation TSV files from DeepBAM
- Per-read CpG methylation calls from Dorado (baseline)
- Per-dataset, per-tool metrics table (ROC AUC, balanced accuracy, F1) e.g. methylation_calling_metrics.tsv

## How to apply

Run DeepBAM's 'extract_and_call_mods' executable on each POD5 dataset with the sorted BAM, reference genome, pre-trained 51-mer module, kmer_size=51, motif_type=CG, num_workers=10, sub_thread_per_worker=4, and batch_size=1024 to asynchronously extract features and obtain per-read CpG methylation TSVs. Run Dorado (ONT's official methylation caller) on the same POD5 files with the same references as the baseline. Align each tool's called CpG sites to high-confidence bisulfite-defined positive and negative methylation sites, balance the positive/negative classes per dataset, and compute ROC AUC, balanced accuracy, and F1 score. Compile per-dataset, per-tool metrics into a single comparison table (e.g. methylation_calling_metrics.tsv). The rationale is that Dorado's methylation model was shown to perform significantly differently across datasets, while DeepBAM achieves stable single-molecule performance (AUC >= 0.956).

## Related tools

- **DeepBAM** (Deep neural network CpG methylation caller; its 'extract_and_call_mods' executable asynchronously extracts features from POD5/BAM and calls CpG methylation states) — github.com/huicongyao/DeepBAM.git
- **Dorado** (ONT's officially supplied methylation calling model; run on the same POD5 files and references to produce the baseline per-read CpG calls for comparison)
- **DeepMod2** (Alternative R10.4-compatible methylation caller mentioned as one of few tools compatible with R10.4)
- **libtorch** (Deep learning runtime used by DeepBAM for model inference during modification calling)

## Examples

```
git clone https://github.com/huicongyao/DeepBAM.git
```

## Evaluation signals

- Reproduced DeepBAM ROC AUCs should be >= 0.956 across all five datasets, with average AUC 98.47%
- DeepBAM average F1 score should reach ~94.97% (up to 16.24% improvement over Dorado)
- DeepBAM balanced accuracies should fall within 92.20%-98.96% across datasets
- Dorado AUCs should fall within 0.8825-0.9852, confirming dataset-dependent variability
- Whole-genome methylation frequencies should correlate > 0.95 with BS-seq on four of five datasets

## Limitations

- Dorado's performance varies significantly across heterogeneous human and plant datasets, so a single-dataset evaluation may mislead
- No software version or release identifier for DeepBAM is stated in the article
- No accession IDs or hosting locations are given for the three plant datasets or human cell line datasets
- Quantitative speed/memory comparison is deferred to supplementary material not present in the main text
- The authors acknowledge ongoing optimization of DeepBAM is necessary, implying incomplete current performance

## Evidence
