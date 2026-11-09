---
name: bilstm-methylation-model-training
description: Use when re-implementing or training a signal-based methylation classifier
  from POD5/FAST5-derived feature matrices, specifically DeepMod2-style 21-bp window
  features centered on CpG sites.
when_to_use_negative:
- Input is already a per-read methylation prediction file or modBAM — inference/post-processing
  skills apply instead of training.
- The goal is methylation calling for modifications other than 5mCpG — the paper notes
  DeepMod2 only provides models for 5mCpG due to lack of training data for other modifications.
- Feature matrices have not been generated from signal (no move-table BAM / POD5 available)
  — the feature-extraction stage must run first.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_3474
tools:
- name: DeepMod2
  role: Provides sample training data (v0.3.1 release), generate_features.py for 21-bp
    CpG-window feature extraction, and the reference BiLSTM architecture being re-implemented
  repo: https://github.com/WGLab/DeepMod2
- name: Dorado
  role: Basecalls modified/unmodified POD5 slices with --emit-moves and --reference
    to produce move-table BAMs aligned to GRCh38 for signal-to-base alignment
  repo: https://github.com/nanoporetech/dorado
- name: PyTorch
  role: Framework used to re-implement and train the bidirectional LSTM classifier
    (2 recurrent layers, 32 hidden nodes, batch_size 128, 10 epochs, lr 0.01, L2 0.01,
    seed 0)
- name: samtools
  role: Converts move-table BAMs (e.g. via samtools fastq -T mv,ts) for realignment
    while retaining move tables
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-45778-y
    title: A signal processing and deep learning framework for methylation detection
      using Oxford Nanopore sequencing
  source_content_hash: sha256:c44a9b380cea86bcf6aa29721aa7838c55d436761034c8d775fa8bbac036c692
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WGLab/DeepMod2
  tools:
  - DeepMod2
  - Dorado
  - PyTorch
  - samtools
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# bilstm-methylation-model-training

## Summary

Train a BiLSTM classifier that consumes per-base signal feature matrices from Oxford Nanopore reads to output a per-read 5mC methylation probability at CpG sites, mirroring the DeepMod2 model architecture.

## When to use

Use when re-implementing or training a signal-based methylation classifier from POD5/FAST5-derived feature matrices, specifically DeepMod2-style 21-bp window features centered on CpG sites.

## When NOT to use

- Input is already a per-read methylation prediction file or modBAM — inference/post-processing skills apply instead of training.
- The goal is methylation calling for modifications other than 5mCpG — the paper notes DeepMod2 only provides models for 5mCpG due to lack of training data for other modifications.
- Feature matrices have not been generated from signal (no move-table BAM / POD5 available) — the feature-extraction stage must run first.

## Inputs

- DeepMod2 v0.3.1 sample training dataset (mod/unmod POD5 signal files plus coordinate label files)
- GRCh38 reference genome FASTA with .fai index
- Move-table BAM files from Dorado basecalling (dna_r10.4.1_e8.2_400bps_hac@v4.3.0, --emit-moves, --reference)
- Per-base feature matrices from generate_features.py (21-bp windows centered at labeled CpGs)

## Outputs

- Trained PyTorch BiLSTM checkpoint (model.cfg plus model.epoch10)
- Per-read methylation predictions as tab-separated per-read BED (read_name, chromosome, ref_position, read_position, strand, methylation_score, read_phase, ref_cpg) with methylation_score in [0,1]
- Per-read validation accuracy on the 50% held-out validation split

## How to apply

Download the DeepMod2 sample training data (mod/unmod POD5 plus coordinate labels in 'contig position strand label' format) from the v0.3.1 release and the GRCh38 FASTA with .fai index. Basecall the POD5 slices with Dorado (dna_r10.4.1_e8.2_400bps_hac@v4.3.0) using --emit-moves and --reference to produce move-table BAMs, then run generate_features.py with --pos_list label files, --window 10 (21-bp window), --seq_type dna, and --include_ref, excluding feature matrices where read-reference identity in the window falls below 0.75. Re-implement the BiLSTM in PyTorch as two bidirectional LSTM layers (32 hidden nodes per the paper) with a 32-node fully connected classifier and one-hot sequence embedding including the reference sequence, training with batch_size 128, 10 epochs, lr 0.01, L2 regularization 0.01, seed 0, and a random 50% validation split. Rationale: the move table approximately aligns ionic-current signal to basecalls so 19 signal summary statistics per base in the 21-bp CpG-centered window can be extracted, and the BiLSTM maps this feature matrix to a sigmoid methylation probability per CpG on the read.

## Related tools

- **DeepMod2** (Provides sample training data (v0.3.1 release), generate_features.py for 21-bp CpG-window feature extraction, and the reference BiLSTM architecture being re-implemented) — https://github.com/WGLab/DeepMod2
- **Dorado** (Basecalls modified/unmodified POD5 slices with --emit-moves and --reference to produce move-table BAMs aligned to GRCh38 for signal-to-base alignment) — https://github.com/nanoporetech/dorado
- **PyTorch** (Framework used to re-implement and train the bidirectional LSTM classifier (2 recurrent layers, 32 hidden nodes, batch_size 128, 10 epochs, lr 0.01, L2 0.01, seed 0))
- **samtools** (Converts move-table BAMs (e.g. via samtools fastq -T mv,ts) for realignment while retaining move tables)

## Examples

```
dorado basecaller dna_r10.4.1_e8.2_400bps_hac@v4.3.0 --emit-moves --reference GRCh38.fa pod5s/ > basecall.bam
```

## Evaluation signals

- Validation accuracy of the re-implemented BiLSTM on the random 50% split should approach the reported 78.46% at epoch 10.
- The per-read output file exists and is a valid tab-separated BED with the DeepMod2 per-read columns (read_name, chromosome, ref_position, read_position, strand, methylation_score, read_phase, ref_cpg).
- All methylation_score values lie in [0,1]; applying the 0.5 probability threshold yields binary methylated/unmethylated per-read calls.
- Training runs reproduce deterministically with fixed seed 0 and stop after the configured 10 epochs.
- Feature matrices used for training all satisfy the read-reference identity ≥0.75 filter in the 21-bp window.

## Limitations

- Move tables provide only approximate signal-to-basecall alignment, which can cause uninformative signal summary statistics in some windows.
- Training labels are restricted to sites with coverage ≥10 and ≥90% (positive) or <10% (negative) methylation per six methods; residual heterogeneity in these labels biases models to overestimate methylation at some CpG sites.
- DeepMod2 only provides models for 5mCpG methylation; no training data exist for other modification types such as 5hmC in GIAB cell lines HG001–HG007.

## Evidence

- [results] A feature matrix is fed to a BiLSTM or Transformer model that makes per-read predictions for each CpG site on the read: "A feature matrix is fed to a BiLSTM or Transformer model that makes per-read predictions for each CpG site on the read"
- [results] get signal summary statistics and alignment information for each base of a read in a 21-bp window centered at the CpG of interest: "get signal summary statistics and alignment information for each base of a read in a 21-bp window centered at the CpG of interest"
- [results] if the read sequence had less than 0.75 percent identity with aligned the reference sequence in the 21-bp window centered at the CpG site, we excluded that feature matrix: "if the read sequence had less than 0.75 percent identity with aligned the reference sequence in the 21-bp window centered at the CpG site, we excluded that feature matrix"
- [results] It requires POD5/FAST5 files to be basecalled using Guppy or Dorado with move table output to allow signal alignment with the basecalls: "It requires POD5/FAST5 files to be basecalled using Guppy or Dorado with move table output to allow signal alignment with the basecalls"
- [results] DeepMod2 adds per-read predictions to the input BAM file as modification tags and also produces a tab separated text file output: "DeepMod2 adds per-read predictions to the input BAM file as modification tags and also produces a tab separated text file output"
- [readme] It uses a BiLSTM model to predict per-read and per-site 5mC methylations for CpG sites and produces a methylation tagged BAM file.: "It uses a BiLSTM model to predict per-read and per-site 5mC methylations for CpG sites and produces a methylation tagged BAM file."
