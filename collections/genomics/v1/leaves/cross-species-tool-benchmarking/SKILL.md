---
name: cross-species-tool-benchmarking
description: Use when comparing methylation callers (e.g., DeepPlant vs Dorado) across
  multiple plant species on R10.4 nanopore data, using bisulfite-sequencing high-confidence
  sites as balanced ground truth for per-read metrics.
when_to_use_negative:
- The flow cell is the older R9.4 platform — tools such as Tombo, Megalodon, and DeepSignal-Plant
  were developed for R9.4 and cannot be directly applied to R10.4, so this benchmarking
  design does not transfer
- The goal is runtime-efficient methylation calling rather than accuracy benchmarking
  — DeepPlant is less computationally efficient than Dorado (Supplementary Data 10)
- The context of interest is CpG or CHG only — DeepPlant's key gains are in CHH detection,
  where positive training samples are scarce
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_0622
tools:
- name: DeepPlant
  role: Primary methylation caller; Bi-LSTM + Transformer deep learning model run
    via extract_and_call_mods to produce per-read 5mC calls in CHH contexts
  repo: github.com/xiaochuanle/DeepPlant
- name: Dorado
  role: Baseline methylation caller; ONT's tool with R10.4 5mC models, used as the
    comparison standard across all species
  repo: https://github.com/nanoporetech/dorado
- name: DeepSignal-Plant
  role: Prior R9.4-era plant 5mC detection tool; contextual predecessor, not applicable
    to R10.4 data in this benchmark
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-58576-x
    title: Accurate cross-species 5mC detection for Oxford Nanopore sequencing in
      plants with DeepPlant
  source_content_hash: sha256:4f40c64797775056fc96b7b7b818de4428da1ae993e6b95f1f7f31f6afd76ac9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepPlant
  - DeepSignal-Plant
  - Dorado
  edam_operation: http://edamontology.org/operation_3204
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cross-species-tool-benchmarking

## Summary

Benchmark DeepPlant against Dorado for single-molecule and whole-genome 5mC detection in plant CHH contexts on ONT R10.4 data, computing F1, accuracy, recall, precision, AUC, and AP per species against BS-seq-derived ground truth.

## When to use

Use when comparing methylation callers (e.g., DeepPlant vs Dorado) across multiple plant species on R10.4 nanopore data, using bisulfite-sequencing high-confidence sites as balanced ground truth for per-read metrics.

## When NOT to use

- The flow cell is the older R9.4 platform — tools such as Tombo, Megalodon, and DeepSignal-Plant were developed for R9.4 and cannot be directly applied to R10.4, so this benchmarking design does not transfer
- The goal is runtime-efficient methylation calling rather than accuracy benchmarking — DeepPlant is less computationally efficient than Dorado (Supplementary Data 10)
- The context of interest is CpG or CHG only — DeepPlant's key gains are in CHH detection, where positive training samples are scarce

## Inputs

- POD5 signal directory per species (ONT R10.4 reads)
- Sorted BAM alignment file per species
- Reference genome FASTA per species
- BS-seq-derived high-confidence CHH site lists (positive fully methylated 100% and negative unmethylated 0%, >=5x coverage)
- Pretrained DeepPlant Bi-LSTM model directory (/DeepPlant/model/bilstm)

## Outputs

- Per-read, per-site methylation rate TSV from extract_and_call_mods
- Side-by-side per-species metric comparison (F1, accuracy, recall, precision, AUC, AP) written to single_molecule_metrics.tsv
- Whole-genome CHH methylation frequency correlation values vs BS-seq per tool per species

## How to apply

Pull the DeepPlant Docker image (chenhx26/deepplant:1.1.0), mount the data directory with --gpus all, and verify GPU availability via nvidia-smi inside the container. For each species, run DeepPlant's extract_and_call_mods with the POD5 directory, sorted BAM, reference genome, and pretrained model /DeepPlant/model/bilstm, setting chh_kmer_size=13 (default cpg/chg kmer sizes of 51) and batch_size=1024 to obtain per-read, per-site methylation rates in a TSV; run Dorado with its R10.4 5mC model as the baseline. Intersect both tools' called cytosines with BS-seq-derived positive (fully methylated, 100%) and negative (unmethylated, 0%) high-confidence CHH site lists (>=5x coverage) to obtain balanced labels, then compute per-species single-molecule accuracy, precision, recall, and F1, stratifying by CHH 9-mer context. Also compare whole-genome CHH methylation frequency correlations with BS-seq, where DeepPlant reports 0.705-0.838 across nine species (23.4-117.6% improvement over Dorado).

## Related tools

- **DeepPlant** (Primary methylation caller; Bi-LSTM + Transformer deep learning model run via extract_and_call_mods to produce per-read 5mC calls in CHH contexts) — github.com/xiaochuanle/DeepPlant
- **Dorado** (Baseline methylation caller; ONT's tool with R10.4 5mC models, used as the comparison standard across all species) — https://github.com/nanoporetech/dorado
- **DeepSignal-Plant** (Prior R9.4-era plant 5mC detection tool; contextual predecessor, not applicable to R10.4 data in this benchmark)

## Examples

```
docker run -i -t -v data_path:data_path:rw --gpus all chenhx26/deepplant:1.1.0 /bin/bash -c "extract_and_call_mods pod5_dir bam_path reference_path ref_type write_dir /DeepPlant/model/bilstm"
```

## Evaluation signals

- Reproduced single-molecule metrics are consistent with the reported Table 2 pattern: DeepPlant exceeds Dorado in single-molecule accuracy and F1 across evaluated species
- On balanced CHH testing sets, DeepPlant's single-molecule F1 gains reach 6.8% (O. sativa), 5.94% (B. vulgaris), and 5.48% (S. lycopersicum)
- Whole-genome CHH methylation frequency correlations with BS-seq fall within the reported 0.705-0.838 range across nine species
- Extracted cytosine positions for both tools successfully intersect the >=5x-coverage high-confidence BS-seq positive (100%) and negative (0%) site lists, yielding balanced class counts
- Container setup is validated by successful nvidia-smi output inside the chenhx26/deepplant:1.1.0 container before running inference

## Limitations

- DeepPlant is less computationally efficient than Dorado, taking much longer to call methylation on the same dataset (Supplementary Data 10)
- Scarcity of high-methylation CHH samples continues to pose challenges for model training and generalization
- Reliance on species with naturally abundant CHH methylation may not capture the full diversity of methylation patterns across all plant species
- Potential for overfitting to specific sequence patterns requires careful model optimization and validation
- In vitro enzyme-catalyzed DNA methylation is ineffective for generating CHH positive samples because CHH methylation depends on RdDM; PCR with modified base substitutions also has difficulty producing high-purity 5mC

## Evidence

- [abstract] Evaluated across nine species, DeepPlant achieves high whole-genome methylation frequency correlations (0.705-0.838) with BS-seq data on CHH, improved by 23.4- 117.6% compared to Dorado: "Evaluated across nine species, DeepPlant achieves high whole-genome methylation frequency correlations (0.705-0.838) with BS-seq data on CHH"
- [methods] Docker commands: "docker run -i -t -v data_path:data_path:rw --gpus all chenhx26/deepplant:1.1.0 /bin/bash"
- [methods] extract_and_call_mods usage: "Usage: extract_and_call_mods [--help] [--version] pod5_dir bam_path reference_path ref_type write_dir model_dir"
- [abstract] Dorado is the only R10.4 plant 5mC tool: "Dorado, the only tool for plant 5mC detection on the R10.4 platform, lacks extensive species testing"
- [discussion] DeepPlant less efficient than Dorado: "The computational benchmark showed that DeepPlant is less computationally efficient than Dorado as it took much longer to call methylation on the same dataset"
- [intro] R9.4 tools incompatible with R10.4: "methylation detection tools developed for the R9.4 platform, such as Tombo17, Megalodon18, and DeepSignal-Plant16, cannot be directly applied to the R10.4 FlowCell"
