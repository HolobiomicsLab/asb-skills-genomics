---
name: nanopore-raw-signal-basecalling
description: Use when raw nanopore fast5 files must be translated into basecalled
  reads (FASTQ/FASTA) with a deep learning basecaller, either with a pretrained model
  (DNA_default/RNA_default) or a custom-trained model, and accuracy should rival Albacore
  v2.0.1.
when_to_use_negative:
- The input is already basecalled (FASTQ/FASTA) reads — alignment or assembly tools,
  not a basecaller, are needed.
- Segregated-event or k-mer-segmentation-based basecallers (nanonet, Albacore pre-v2.0.1,
  DeepNano, BasecRAWller) are required for legacy two-stage pipeline compatibility.
- Only a CPU is available and large-scale throughput is required — Chiron runs at
  only ~21 bp/s greedy on CPU vs 1652 bp/s on a K80 GPU, and is substantially more
  computationally expensive than Albacore.
edam_operation: http://edamontology.org/operation_3458
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3474
tools:
- name: Chiron
  role: Primary CNN+RNN+CTC basecaller that translates raw nanopore signal directly
    to DNA sequence without a segmentation step; provides call (with greedy/beam decoders)
    and train modes
  repo: https://github.com/haotianteng/chiron
- name: TensorFlow
  role: Deep learning backend (v1.15 or tensorflow-gpu==1.15) required to run inference
    and training
- name: h5py
  role: Dependency for reading fast5 HDF5 raw signal files
  repo: https://github.com/h5py/h5py
- name: BoostNano
  role: Preprocessing of fast5 files to segment/trim adapter, polyA stall and transcript
    signal before RNA basecalling
  repo: https://github.com/haotianteng/BoostNano
- name: graphmap
  role: Aligns basecalled reads to the reference genome with default parameters to
    compute per-read accuracy
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_006
  - task_003
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giy037
    title: 'Chiron: translating nanopore raw signal directly into nucleotide sequence
      using deep learning'
  source_content_hash: sha256:0388208416e6e180f1f86a86adbc6e035945d03eb4f75f8065424fe0030a7b35
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haotianteng/chiron
  tools:
  - Albacore
  - BasecRAWller
  - BoostNano
  - Chiron
  - Metrichor
  - Miniasm
  - Minimap2
  - Racon
  - TensorFlow
  - graphmap
  - h5py
  edam_operation: http://edamontology.org/operation_3458
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# nanopore-raw-signal-basecalling

## Summary

End-to-end basecalling of Oxford Nanopore raw signal (fast5) directly into nucleotide sequence using Chiron's CNN+RNN+CTC deep learning architecture, bypassing the error-prone segmentation step used by prior two-stage basecallers.

## When to use

Use when raw nanopore fast5 files must be translated into basecalled reads (FASTQ/FASTA) with a deep learning basecaller, either with a pretrained model (DNA_default/RNA_default) or a custom-trained model, and accuracy should rival Albacore v2.0.1.

## When NOT to use

- The input is already basecalled (FASTQ/FASTA) reads — alignment or assembly tools, not a basecaller, are needed.
- Segregated-event or k-mer-segmentation-based basecallers (nanonet, Albacore pre-v2.0.1, DeepNano, BasecRAWller) are required for legacy two-stage pipeline compatibility.
- Only a CPU is available and large-scale throughput is required — Chiron runs at only ~21 bp/s greedy on CPU vs 1652 bp/s on a K80 GPU, and is substantially more computationally expensive than Albacore.

## Inputs

- Folder of Oxford Nanopore raw signal fast5 files (DNA or direct-RNA)
- Pretrained model folder, e.g. chiron/model/DNA_default or chiron/model/RNA_default
- Optional custom-trained model checkpoint in log_dir/model_name/ (TensorFlow)

## Outputs

- Per-read FASTQ/FASTA basecalls in <output_folder>/result (one file per fast5 input)
- Output subfolders raw/, segments/, meta/, reference/
- Per-read accuracy metrics (identity, insertion, deletion, mismatch rates) after graphmap alignment to a reference genome

## How to apply

Install Chiron via pip (with tensorflow==1.15, or tensorflow-gpu==1.15 for GPU) plus h5py for reading fast5 files. Basecall with `chiron call -i <input_fast5_folder> -o <output_folder> -m chiron/model/DNA_default --preset dna-pre`, which applies the dna-pre preset (batch_size 400, segment_len 400, jump 390, beam 30) and writes raw, result, segments, meta, and reference subfolders containing per-read FASTQ/FASTA. Choose the decoder deliberately: beam search (--beam <width>, default 30) gives higher accuracy, while greedy decoding (--beam 0) is faster — e.g. 1652 bp/s greedy vs 1204 bp/s beam on an Nvidia K80 GPU, vs only 21/17 bp/s on CPU. Reduce batch_size (-b 200) when GPU RAM (~8 GB recommended for training) or RAM is limited. For direct RNA, switch to the RNA_default model with --mode rna --preset rna-pre (batch_size 300, segment_len 2000, jump 1900), optionally preprocessing fast5 files with BoostNano to trim adapter and polyA tails. Downstream quality is assessed by aligning basecalled reads to a reference genome (e.g. Phage Lambda NC_001416.1) with graphmap and computing per-read identity, insertion, deletion, and mismatch rates.

## Related tools

- **Chiron** (Primary CNN+RNN+CTC basecaller that translates raw nanopore signal directly to DNA sequence without a segmentation step; provides call (with greedy/beam decoders) and train modes) — https://github.com/haotianteng/chiron
- **TensorFlow** (Deep learning backend (v1.15 or tensorflow-gpu==1.15) required to run inference and training)
- **h5py** (Dependency for reading fast5 HDF5 raw signal files) — https://github.com/h5py/h5py
- **BoostNano** (Preprocessing of fast5 files to segment/trim adapter, polyA stall and transcript signal before RNA basecalling) — https://github.com/haotianteng/BoostNano
- **graphmap** (Aligns basecalled reads to the reference genome with default parameters to compute per-read accuracy)

## Examples

```
chiron call -i <input_fast5_folder> -o <output_folder> -m chiron/model/DNA_default --preset dna-pre
```

## Evaluation signals

- Output folder contains the five expected subfolders (raw, result, segments, meta, reference) and result holds a FASTQ/FASTA file per input fast5.
- Read-level identity rate on bacterial/viral datasets (Phage Lambda NC_001416.1, E. coli, M. tuberculosis PRJNA386696) should match Albacore v2.0.1; on Human NA12878 it should be within ~2% of Albacore.
- Speed sanity check: GPU greedy decoding ~1652 bp/s and beam search (width 50) ~1204 bp/s on K80; CPU rates ~21 and 17 bp/s; greedy must be faster than beam on each device.
- A model trained on only ~4,000 reads (2,000 E. coli + 2,000 Lambda, 3 epochs, ~10 h on a K80) should still yield state-of-the-art read-level accuracy on previously unseen species.
- Increasing beam width should not decrease accuracy, and larger batch_size should slightly improve accuracy and inference speed.

## Limitations

- Chiron is substantially more computationally expensive than Albacore and somewhat more expensive than BasecRAWller; CPU-only throughput (~21 bp/s greedy) is impractical at scale.
- A ~2% read-level accuracy gap versus Albacore remains on Human DNA, since the default DNA model was trained only on a mix of Lambda and E. coli R9.4 data.
- Increasing the sliding window size improves accuracy at the cost of increased memory and running time; batch_size must be reduced if GPU RAM is limited.
- Training labels were generated by Metrichor plus Nanoraw with alignment-based error correction, so residual Metrichor errors can bias training if not corrected.
- The RNA mode benefits from BoostNano preprocessing and older TensorFlow 1.15 dependency may be hard to install on modern systems.

## Evidence

- [abstract] we report Chiron, the first deep learning model to achieve end-to-end basecalling and directly translate the raw signal to DNA sequence without the error-prone segmentation step: "we report Chiron, the first deep learning model to achieve end-to-end basecalling and directly translate the raw signal to DNA sequence without the error-prone segmentation step"
- [readme] Beam search decoder: chiron call -i <input> -o <output> --beam <beam_width>: "Beam search decoder: chiron call -i <input> -o <output> --beam <beam_width>
Greedy decoder: chiron call -i <input> -o <output> --beam 0"
- [readme] 'start':0,'batch_size':400,'segment_len':400,'jump':390,'threads':0,'beam':30: "**DNA preset arguments value(dna-pre):**
'start':0,'batch_size':400,'segment_len':400,'jump':390,'threads':0,'beam':30"
- [discussion] aligned to the reference genome using graphmap[33] with the default parameters: "aligned to the reference genome using graphmap[33] with the default parameters"
- [abstract] Chiron achieves basecalling speeds of more than 2,000 bases per second using desktop computer graphics processing units.: "Chiron achieves basecalling speeds of more than 2,000 bases per second using desktop computer graphics processing units."
