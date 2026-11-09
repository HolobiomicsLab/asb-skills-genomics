---
name: single-molecule-methylation-metrics-evaluation
description: Use when comparing per-read 5mC callers on ONT R10.4 plant data against
  a bisulfite ground truth of fully methylated/unmethylated CHH sites (>=5x coverage)
  to compute F1, accuracy, recall, precision, AUC, and AP.
when_to_use_negative:
- The goal is whole-genome methylation frequency correlation with BS-seq (aggregated
  per-site levels) rather than per-read (single-molecule) classification metrics
- Data come from the R9.4 FlowCell platform; tools such as Tombo, Megalodon, and DeepSignal-Plant
  cannot be directly applied to R10.4 and vice versa
- The evaluation set is not balanced between fully methylated (100%) and unmethylated
  (0%) CHH sites, which the reported F1 gains assume
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: DeepPlant
  role: Produces per-read, per-site CHH methylation rates via extract_and_call_mods
    using the pretrained bilstm model; evaluated against Dorado with single-molecule
    metrics
  repo: https://github.com/xiaochuanle/DeepPlant
- name: Dorado
  role: Baseline ONT 5mC caller for the R10.4 platform used as the comparison method
    in the single-molecule benchmark
  repo: https://github.com/nanoporetech/dorado
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-58576-x
    title: Accurate cross-species 5mC detection for Oxford Nanopore sequencing in
      plants with DeepPlant
  source_content_hash: sha256:ebe291d41fa44ce642850807469f465f29564dd0d582311b938cc67f70d24831
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xiaochuanle/DeepPlant
  tools:
  - DeepPlant
  - Dorado
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# single-molecule-methylation-metrics-evaluation

## Summary

Benchmark single-molecule 5mC detection performance (accuracy, precision, recall, F1, AUC, AP) of plant methylation callers such as DeepPlant and Dorado by intersecting per-read, per-site methylation calls with BS-seq-derived high-confidence CHH sites that are fully methylated (100%) or unmethylated (0%).

## When to use

Use when comparing per-read 5mC callers on ONT R10.4 plant data against a bisulfite ground truth of fully methylated/unmethylated CHH sites (>=5x coverage) to compute F1, accuracy, recall, precision, AUC, and AP.

## When NOT to use

- The goal is whole-genome methylation frequency correlation with BS-seq (aggregated per-site levels) rather than per-read (single-molecule) classification metrics
- Data come from the R9.4 FlowCell platform; tools such as Tombo, Megalodon, and DeepSignal-Plant cannot be directly applied to R10.4 and vice versa
- The evaluation set is not balanced between fully methylated (100%) and unmethylated (0%) CHH sites, which the reported F1 gains assume

## Inputs

- Per-read, per-site methylation rate TSV from DeepPlant extract_and_call_mods (chh_kmer_size=13, batch_size=1024, /DeepPlant/model/bilstm)
- Per-read modified-base calls from Dorado R10.4 5mC model
- BS-seq-derived high-confidence CHH positive site list (fully methylated, 100%, >=5x coverage)
- BS-seq-derived high-confidence CHH negative site list (unmethylated, 0%, >=5x coverage)
- Sorted BAM alignments, reference genome, and POD5 read files

## Outputs

- Per-species single-molecule accuracy, precision, recall, F1-score, AUC, and AP for each caller
- Side-by-side caller comparison table (e.g., single_molecule_metrics.tsv) stratified by CHH 9-mer context where applicable

## How to apply

Run DeepPlant's extract_and_call_mods on POD5 plus sorted BAM against the reference genome with the pretrained Bi-LSTM model directory /DeepPlant/model/bilstm, setting chh_kmer_size=13 (default cpg/chg kmer size 51) and batch_size=1024 to obtain per-read, per-site methylation rates in TSV. Run Dorado with its R10.4 5mC model on the same reads as the baseline. Intersect both tools' called cytosine positions with BS-seq-derived positive (100%) and negative (0%) high-confidence CHH site lists at >=5x coverage to obtain balanced labels per species. Compute per-species single-molecule accuracy, precision, recall, and F1 for each tool over the balanced CHH site sets, stratifying by CHH 9-mer context where applicable, and write a side-by-side comparison TSV. Validate by checking that the reproduced metrics reproduce the paper's Table 2 pattern of DeepPlant outperforming Dorado.

## Related tools

- **DeepPlant** (Produces per-read, per-site CHH methylation rates via extract_and_call_mods using the pretrained bilstm model; evaluated against Dorado with single-molecule metrics) — https://github.com/xiaochuanle/DeepPlant
- **Dorado** (Baseline ONT 5mC caller for the R10.4 platform used as the comparison method in the single-molecule benchmark) — https://github.com/nanoporetech/dorado

## Examples

```
docker run -i -t -v data_path:data_path:rw --gpus all chenhx26/deepplant:1.1.0 /bin/bash
```

## Evaluation signals

- DeepPlant's single-molecule F1 exceeds Dorado's on balanced CHH test sets across species, with reported gains such as 6.8% (O. sativa), 5.94% (B. vulgaris), and 5.48% (S. lycopersicum)
- Reproduced metrics are consistent with the paper's Table 2 pattern of superior DeepPlant single-molecule accuracy and F1 across evaluated species
- Each ground-truth CHH site is restricted to high-confidence BS-seq calls with >=5x coverage and binary labels (100% or 0%)
- Metrics are computed per species over balanced positive/negative site sets and stratified by CHH 9-mer context where applicable
- All compared cytosine positions are intersected identically against the same site lists for both callers

## Limitations

- DeepPlant is less computationally efficient than Dorado and took much longer to call methylation on the same dataset (Supplementary Data 10)
- The benchmark relies on species with naturally abundant high-methylation CHH sites, which may not capture the full diversity of methylation patterns across all plant species
- Scarcity of high-methylation CHH positive samples, although partially mitigated in this study, continues to pose challenges for model training and generalization

## Evidence

- [abstract] DeepPlant also demonstrates superior single-molecule accuracy and F1 score: "DeepPlant also demonstrates superior single-molecule accuracy and F1 score, offering strong generalization for plant epigenetics research"
- [methods] extract_and_call_mods usage: "Usage: extract_and_call_mods [--help] [--version] pod5_dir bam_path reference_path ref_type write_dir model_dir"
- [readme] Dorado as the only R10.4 plant 5mC baseline: "Dorado, the only tool for plant 5mC detection on the R10.4 platform, lacks extensive species testing."
- [discussion] DeepPlant runtime disadvantage: "The computational benchmark showed that DeepPlant is less computationally efficient than Dorado as it took much longer to call methylation on the same dataset"
