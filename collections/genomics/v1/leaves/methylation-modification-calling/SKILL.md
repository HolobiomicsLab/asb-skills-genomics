---
name: methylation-modification-calling
description: Use when you have plant ONT R10.4 POD5 reads with a sorted BAM and reference
  genome and need per-read or genome-wide 5mC calls in CHH (or CpG/CHG) contexts,
  particularly where Dorado's CHH accuracy is insufficient.
when_to_use_negative:
- Data are from the older R9.4 FlowCell — tools like Tombo, Megalodon, and DeepSignal-Plant
  were built for R9.4 and DeepPlant targets R10.4 data
- Non-plant (non-RdDM) systems where CHH methylation biology differs — the model was
  trained on plant species screened for abundant high-methylation CHH sites
- Runtime-constrained pipelines — DeepPlant is less computationally efficient than
  Dorado and took much longer to call methylation on the same dataset
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_3674
tools:
- name: DeepPlant
  role: Primary 5mC detection tool; runs extract_and_call_mods on POD5/BAM/reference
    inputs with the pretrained Bi-LSTM model to produce per-read methylation calls
  repo: https://github.com/xiaochuanle/DeepPlant
- name: Dorado
  role: Baseline ONT basecaller with R10.4 5mC models used for comparison and cross-checking
    methylation calls
  repo: https://github.com/nanoporetech/dorado
- name: DeepSignal-Plant
  role: Prior R9.4-era plant 5mC detection tool; superseded by DeepPlant for R10.4
    FlowCell data
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-58576-x
    title: Accurate cross-species 5mC detection for Oxford Nanopore sequencing in
      plants with DeepPlant
  source_content_hash: sha256:ea5284e897f9cafc9d06d5bbc39f54436e4bddc0a2d3f1a2c47cebe707b1e0dc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xiaochuanle/DeepPlant
  tools:
  - DeepPlant
  - DeepSignal-Plant
  - Dorado
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_3674
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# methylation-modification-calling

## Summary

Detect 5-methylcytosine (5mC) modifications in CpG, CHG, and CHH contexts from Oxford Nanopore R10.4 plant sequencing data using DeepPlant (Bi-LSTM + Transformer), producing per-read, per-site methylation calls comparable to or better than Dorado, especially for CHH.

## When to use

Use when you have plant ONT R10.4 POD5 reads with a sorted BAM and reference genome and need per-read or genome-wide 5mC calls in CHH (or CpG/CHG) contexts, particularly where Dorado's CHH accuracy is insufficient.

## When NOT to use

- Data are from the older R9.4 FlowCell — tools like Tombo, Megalodon, and DeepSignal-Plant were built for R9.4 and DeepPlant targets R10.4 data
- Non-plant (non-RdDM) systems where CHH methylation biology differs — the model was trained on plant species screened for abundant high-methylation CHH sites
- Runtime-constrained pipelines — DeepPlant is less computationally efficient than Dorado and took much longer to call methylation on the same dataset

## Inputs

- POD5 directory of Oxford Nanopore R10.4 reads
- Sorted BAM alignment file against a plant reference genome
- Reference genome FASTA with reference type specification
- Pretrained DeepPlant model directory (e.g. /DeepPlant/model/bilstm)
- Optional: BS-seq-derived high-confidence methylated/unmethylated site lists (>=5x coverage) for validation

## Outputs

- Per-read, per-site 5mC methylation rates in TSV format for CpG, CHG, and CHH contexts
- Validation metrics (accuracy, precision, recall, F1, AUC, AP) against BS-seq ground truth if evaluated

## How to apply

Pull the DeepPlant Docker image (chenhx26/deepplant:1.1.0), verify GPU availability with nvidia-smi inside the container, and mount the data directory read-write. Run DeepPlant's extract_and_call_mods with the POD5 directory, sorted BAM, reference genome, and the pretrained model directory /DeepPlant/model/bilstm, setting chh_kmer_size=13 (cpg/chg default kmer size 51) and batch_size=1024 to emit per-read, per-site methylation rates as a TSV. As a baseline or cross-check, run Dorado with its R10.4 5mC model on the same data. To validate single-molecule performance, intersect called cytosine positions with bisulfite-seq-derived high-confidence positive (100% methylated) and negative (0%) site lists at >=5x coverage, then compute accuracy, precision, recall, F1, AUC, and AP per context (stratifying by CHH 9-mer context where applicable). DeepPlant reportedly improves whole-genome CHH methylation frequency correlation with BS-seq to 0.705-0.838 across nine species, a 23.4-117.6% gain over Dorado.

## Related tools

- **DeepPlant** (Primary 5mC detection tool; runs extract_and_call_mods on POD5/BAM/reference inputs with the pretrained Bi-LSTM model to produce per-read methylation calls) — https://github.com/xiaochuanle/DeepPlant
- **Dorado** (Baseline ONT basecaller with R10.4 5mC models used for comparison and cross-checking methylation calls) — https://github.com/nanoporetech/dorado
- **DeepSignal-Plant** (Prior R9.4-era plant 5mC detection tool; superseded by DeepPlant for R10.4 FlowCell data)

## Examples

```
docker run -i -t -v /data:/data:rw --gpus all chenhx26/deepplant:1.1.0 /bin/bash -c "extract_and_call_mods /data/pod5 /data/aligned.bam /data/ref.fa plant /data/out /DeepPlant/model/bilstm --chh_kmer_size 13 --batch_size 1024"
```

## Evaluation signals

- Per-site methylation output is a TSV with per-read, per-site methylation rates covering CpG, CHG, and CHH contexts
- Whole-genome CHH methylation frequency correlation with BS-seq should approach the reported 0.705-0.838 range across the nine species, exceeding Dorado by 23.4-117.6%
- Single-molecule F1 against BS-seq-derived balanced CHH ground truth (>=5x coverage, 100%/0% sites) should match or exceed Dorado, e.g. gains of ~6.8% in O. sativa, 5.94% in B. vulgaris, 5.48% in S. lycopersicum
- nvidia-smi inside the container confirms GPU access before running inference
- Called cytosine positions intersect non-trivially with both the fully-methylated and unmethylated high-confidence site lists (no empty intersection)

## Limitations

- DeepPlant is less computationally efficient than Dorado — it took much longer to call methylation on the same dataset (Supplementary Data 10)
- Scarcity of high-methylation CHH samples, although partially mitigated, continues to pose challenges for model training and generalization
- Training relied on species with naturally abundant CHH methylation, which may not capture the full diversity of methylation patterns across all plant species
- Potential overfitting to specific sequence patterns underscores the need for careful model optimization and validation
- In vitro enzyme-catalyzed DNA methylation is ineffective for generating CHH positive samples because CHH methylation depends on RdDM with DRM1/DRM2

## Evidence

- [methods] Docker image and inference parameters: "docker pull chenhx26/deepplant:1.1.0"
- [methods] extract_and_call_mods CLI signature: "Usage: extract_and_call_mods [--help] [--version] pod5_dir bam_path reference_path ref_type write_dir model_dir"
- [abstract] Whole-genome CHH correlation findings: "DeepPlant achieves high whole-genome methylation frequency correlations (0.705-0.838) with BS-seq data on CHH, improved by 23.4- 117.6% compared to Dorado"
- [abstract] Dorado as the only R10.4 plant alternative: "Dorado, the only tool for plant 5mC detection on the R10.4 platform, lacks extensive species testing"
- [abstract] Single-molecule performance claim: "DeepPlant also demonstrates superior single-molecule accuracy and F1 score, offering strong generalization for plant epigenetics research"
- [discussion] Runtime limitation: "DeepPlant is less computationally efficient than Dorado as it took much longer to call methylation on the same dataset (Supplementary Data 10)"
- [intro] R9.4 tools incompatible with R10.4: "methylation detection tools developed for the R9.4 platform, such as Tombo17, Megalodon18, and DeepSignal-Plant16, cannot be directly applied to the R10.4 FlowCell"
- [discussion] Training 9-mer coverage: "Our comprehensive training dataset now covers 97.2% of all possible 9-mer CHH contexts, averaging over 9225 samples per context"
