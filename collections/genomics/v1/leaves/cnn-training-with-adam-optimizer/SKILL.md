---
name: cnn-training-with-adam-optimizer
description: Use when a pipeline needs to refine predicted splice-site positions from
  long-read (Iso-seq) alignments, or when reproducing TAGET's CNN splice-site correction
  trained on Ensembl-annotated exon/intron boundaries.
when_to_use_negative:
- Input reads are not Iso-seq/full-length transcript data — TAGET's alignment and
  CNN correction are designed for PacBio Iso-seq transcripts, not short-read RNA-seq
  alone
- Splice sites already match Ensembl annotation (FSM/ISM transcripts) — the CNN correction
  targets mispredicted novel junctions, not canonically supported ones
- Reference genome or annotation files (hg38.fa / hg38.ensembl.gtf pickle) are unavailable
  — the training windows cannot be extracted
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: TAGET
  role: Toolkit whose TransAnnot alignment step produces the inconsistent long/short-read
    mappings that the CNN locally corrects at splice sites
  repo: https://github.com/XiDsLab/TAGET
- name: Python
  role: Environment for building and training the CNN and encoding the splice-site
    windows
- name: numpy
  role: One-hot encoding of A/C/G/T nucleotides into fixed-length input tensors
- name: pandas
  role: Handling of site tables during training data preparation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-41649-0
    title: 'TAGET: a toolkit for analyzing full-length transcripts from long-read
      sequencing'
  source_content_hash: sha256:3d6193c9852c35ced9881ce79817886ec9c4768a67bf19d7228f34823a74488c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiDsLab/TAGET
  tools:
  - Python
  - TAGET
  - numpy
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# cnn-training-with-adam-optimizer

## Summary

Trains a three-block convolutional neural network on one-hot encoded 21-bp splice-site windows from hg38 to locally correct misaligned splice junctions during TAGET's transcript alignment of PacBio Iso-seq reads, reaching ~98% validation accuracy (Supplementary Fig. 12).

## When to use

Use when a pipeline needs to refine predicted splice-site positions from long-read (Iso-seq) alignments, or when reproducing TAGET's CNN splice-site correction trained on Ensembl-annotated exon/intron boundaries.

## When NOT to use

- Input reads are not Iso-seq/full-length transcript data — TAGET's alignment and CNN correction are designed for PacBio Iso-seq transcripts, not short-read RNA-seq alone
- Splice sites already match Ensembl annotation (FSM/ISM transcripts) — the CNN correction targets mispredicted novel junctions, not canonically supported ones
- Reference genome or annotation files (hg38.fa / hg38.ensembl.gtf pickle) are unavailable — the training windows cannot be extracted

## Inputs

- hg38 reference genome FASTA (hg38.fa)
- Ensembl gene annotation in GTF format (hg38.ensembl.gtf) converted to a pickle via gtf_db_make.py
- One-hot encoded 21-bp splice-site sequence windows (90% train / 10% validation split)
- Long-read/short-read mapping candidates with inconsistent splice-junction placements from TAGET's TransAnnot step

## Outputs

- Trained three-block CNN model (3x3 conv kernels, 50 channels, ReLU pooling, batch normalization per block)
- Validation accuracy record (expected ~0.98, must fall in 0.9-1.0)
- Locally adjusted splice-junction positions for mapped subsequences

## How to apply

Extract 21-bp genomic sequence windows centered on Ensembl-annotated exon/intron splice sites from hg38.fa using the hg38.ensembl.gtf annotation (converted via gtf_db_make.py into a pickle), and split sites 90% training / 10% validation. One-hot encode the A/C/G/T nucleotides of each window into fixed-length input tensors with Python, NumPy and pandas. Build the three-block CNN where each block applies 3x3 convolutional kernels with 50 channels followed by ReLU pooling and batch normalization. Train for 1000 epochs with the Adam optimizer at learning rate 0.01 using categorical cross-entropy loss, then evaluate on the 10% validation split and record the validation accuracy (~98%). The rationale is that after TAGET assembles mappings for subsequences with inconsistent long-read and short-read alignments, this classifier locally adjusts junction positions the mapping step misplaces.

## Related tools

- **TAGET** (Toolkit whose TransAnnot alignment step produces the inconsistent long/short-read mappings that the CNN locally corrects at splice sites) — https://github.com/XiDsLab/TAGET
- **Python** (Environment for building and training the CNN and encoding the splice-site windows)
- **numpy** (One-hot encoding of A/C/G/T nucleotides into fixed-length input tensors)
- **pandas** (Handling of site tables during training data preparation)

## Examples

```
python gtf_db_make.py hg38.ensembl.gtf hg38.ensembl.v20200306.1.pickle
```

## Evaluation signals

- Output file cnn_splice_site_validation_accuracy. exists and the recorded validation accuracy falls in the range 0.9-1.0
- Recorded accuracy is consistent with the ~98% reported in Supplementary Fig. 12
- The 90/10 train/validation partition of Ensembl splice-site windows was applied as specified
- Model architecture matches three blocks of 3x3 convolutional kernels with 50 channels, ReLU pooling, and batch normalization
- Training ran for 1000 epochs with Adam at learning rate 0.01 and categorical cross-entropy loss

## Limitations

- The 98% accuracy claim is documented only in a supplementary figure not present in the main text
- The model is trained on Ensembl-annotated hg38 splice sites, so performance on other genomes, annotations, or non-human Iso-seq data is unverified
- TAGET's alignment is designed for PacBio Iso-seq (HiFi) data; CCS-read filtering in Iso-seq preprocessing can discard about 80% of CCS reads, reducing effective coverage

## Evidence

- [supplementary] Train the network for 1000 epochs using the Adam optimizer with learning rate 0.01 and categorical cross-entropy loss on the training split.: [evidence span withheld — anchor status: not_found]
- [discussion] Overall, our results strongly validated the performance of TAGET on Iso-seq data.: "Overall, our results strongly validated the performance of TAGET on Iso-seq data."
- [readme] python gtf_db_make.py hg38.ensembl.gtf hg38.ensembl.v20200306.1.pickle: "python gtf_db_make.py hg38.ensembl.gtf hg38.ensembl.v20200306.1.pickle"
