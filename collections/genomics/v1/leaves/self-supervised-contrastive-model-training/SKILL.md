---
name: self-supervised-contrastive-model-training
description: Use when metagenomic contig sequence features (data.csv/data_split.csv
  from generate_sequence_features_single/multi) are available and a binning model
  must be learned without external taxonomic annotation (no mmseqs2/GTDB cannot-links).
when_to_use_negative:
- Taxonomic annotations and GTDB/mmseqs2 cannot-link files are already available and
  semi-supervised training is preferred — then use generate_cannot_links + train_semi
  instead
- No sequence features (data.csv/data_split.csv) have been generated and you only
  need an off-the-shelf pretrained model — use bin_short/bin_long with --model or
  the pretrained models rather than training
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3474
tools:
- name: SemiBin
  role: Command-line tool providing the train_self subcommand and the self-supervised
    contrastive siamese-network training pipeline
  repo: https://github.com/BigDataBiology/SemiBin/
- name: SemiBin2
  role: Implementation of the self-supervised learning approach that learns feature
    embeddings from contigs (command available since SemiBin 2.2)
  repo: https://github.com/BigDataBiology/SemiBin
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad209
    title: 'SemiBin2: self-supervised contrastive learning leads to better MAGs for
      short- and long-read sequencing'
  source_content_hash: sha256:ea188fbad3ff31be599fb798c9aefa25dacff5d55ce5ab2456d9c19d4e64535f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BigDataBiology/SemiBin
  tools:
  - SemiBin
  - SemiBin2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# self-supervised-contrastive-model-training

## Summary

Trains SemiBin2's deep siamese neural network via a self-supervised contrastive objective to learn a contig feature embedding from must-link and cannot-link constraints, replacing the taxonomic-annotation-dependent semi-supervised training of SemiBin1.

## When to use

Use when metagenomic contig sequence features (data.csv/data_split.csv from generate_sequence_features_single/multi) are available and a binning model must be learned without external taxonomic annotation (no mmseqs2/GTDB cannot-links).

## When NOT to use

- Taxonomic annotations and GTDB/mmseqs2 cannot-link files are already available and semi-supervised training is preferred — then use generate_cannot_links + train_semi instead
- No sequence features (data.csv/data_split.csv) have been generated and you only need an off-the-shelf pretrained model — use bin_short/bin_long with --model or the pretrained models rather than training

## Inputs

- Precomputed sequence feature table data.csv (from generate_sequence_features_single/multi)
- Split-feature table data_split.csv (must-link pairs from split_contigs)
- Input contigs FASTA file
- Optional: multiple samples' data/data_split/fasta files (same order) for --train-from-many

## Outputs

- Trained model checkpoint model.pt in the output directory, loadable downstream via bin_short/bin_long --model
- SemiBinRun.log training log

## How to apply

Load the precomputed sequence features data.csv and data_split.csv together with the input contig FASTA, then run the SemiBin2 train_self subcommand, which is the default training mode since SemiBin 1.3. Train the siamese network with the self-supervised contrastive objective using the documented defaults --epochs 15 and --batch-size 2048, optionally fixing --random-seed for reproducibility and setting --engine (auto/gpu/cpu) for device selection. To build a single reusable model across samples, pass several samples in the same order for data, data_split, and fasta files with --train-from-many. The rationale is that self-supervised contrastive learning from must-link (contig halves from split_contigs) and cannot-link constraints yields better contig embeddings than learning from taxonomic annotations, and it removes the extra generate_cannot_links step that the semi-supervised mode requires.

## Related tools

- **SemiBin** (Command-line tool providing the train_self subcommand and the self-supervised contrastive siamese-network training pipeline) — https://github.com/BigDataBiology/SemiBin/
- **SemiBin2** (Implementation of the self-supervised learning approach that learns feature embeddings from contigs (command available since SemiBin 2.2)) — https://github.com/BigDataBiology/SemiBin

## Examples

```
SemiBin2 train_self -i contigs.fasta --data data.csv --data-split data_split.csv -o output --epochs 15 --batch-size 2048 --engine auto
```

## Evaluation signals

- The trained checkpoint model.pt is written to the output directory and is loadable downstream (e.g. via bin_short/bin_long --model)
- SemiBinRun.log reports no fatal errors during training
- Training completed for the documented --epochs 15 with --batch-size 2048 (or explicitly overridden values) without aborting
- When --train-from-many is used, the data, data_split, and fasta files were supplied in the same sample order and a single reusable single-sample model was produced

## Limitations

- Self-supervised mode skips the generate_cannot_links step; the alternative semi-supervised mode (train_semi) requires mmseqs2/GTDB taxonomic annotation and is only needed when that extra constraint source is desired
- Many SemiBin operations are probabilistic — running training twice can return different results unless --random-seed is fixed
- Device selection depends on --engine (auto/gpu/cpu); GPU unavailability falls back per engine setting

## Evidence

- [abstract] We propose SemiBin2, which uses self-supervised learning to learn feature embeddings from the contigs.: "We propose SemiBin2, which uses self-supervised learning to learn feature embeddings from the contigs."
- [methods] train_self default training mode: "`train_semi` (replacing `train_self` from the self-supervised mode)"
- [methods] semi-supervised cannot-link generation contrast: "Run the contig annotations using mmseqs with GTDB and generate `cannot-link` file used in the semi-supervised deep learning model training."
- [methods] SemiBin2 command availability: [evidence span withheld — anchor status: not_found]
