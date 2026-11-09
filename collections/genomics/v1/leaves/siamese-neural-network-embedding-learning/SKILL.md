---
name: siamese-neural-network-embedding-learning
description: Use when performing metagenomic binning with SemiBin2 and a trained contig
  embedding model is needed from sequence features, without external taxonomic annotation
  (no mmseqs2/GTDB cannot-links).
when_to_use_negative:
- When external taxonomic annotations (e.g. mmseqs2/GTDB cannot-links) are desired
  and semi-supervised training is preferred — use generate_cannot_links + train_semi
  instead.
- When the goal is only abundance estimation or contig splitting (e.g. strobealign
  --aemb, split_contigs) and no embedding model training is required.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: SemiBin2
  role: Provides the train_self subcommand that trains the deep siamese neural network
    with self-supervised contrastive learning on contig features
  repo: https://github.com/BigDataBiology/SemiBin/
- name: SemiBin
  role: Command-line tool for metagenomic binning with deep learning (siamese neural
    networks); package installation provides the SemiBin2 command
  repo: https://github.com/BigDataBiology/SemiBin/
- name: mmseqs2
  role: Used only in the semi-supervised alternative (generate_cannot_links) to produce
    cannot-link files from GTDB annotations; not needed for self-supervised train_self
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad209
    title: 'SemiBin2: self-supervised contrastive learning leads to better MAGs for
      short- and long-read sequencing'
  source_content_hash: sha256:8745213defe741148b7036e7552dbcb501ae2c17ed27756148a8fb10307a9c60
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BigDataBiology/SemiBin
  tools:
  - SemiBin
  - SemiBin2
  - mmseqs2
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# siamese-neural-network-embedding-learning

## Summary

Trains a deep siamese neural network with a self-supervised contrastive objective to learn contig feature embeddings for metagenomic binning, as implemented in SemiBin2's train_self subcommand. It derives must-link and cannot-link constraints from the data itself rather than from taxonomic annotations.

## When to use

Use when performing metagenomic binning with SemiBin2 and a trained contig embedding model is needed from sequence features, without external taxonomic annotation (no mmseqs2/GTDB cannot-links).

## When NOT to use

- When external taxonomic annotations (e.g. mmseqs2/GTDB cannot-links) are desired and semi-supervised training is preferred — use generate_cannot_links + train_semi instead.
- When the goal is only abundance estimation or contig splitting (e.g. strobealign --aemb, split_contigs) and no embedding model training is required.

## Inputs

- Precomputed sequence feature files data.csv and data_split.csv (from generate_sequence_features_single/multi)
- Input contig FASTA file (assembled metagenomic contigs)
- Optional: multiple samples' data.csv, data_split.csv, and FASTA files in matching order for --train-from-many

## Outputs

- Trained model checkpoint model.pt written to the output directory, loadable via bin_short/bin_long --model
- SemiBinRun.log training log

## How to apply

First generate sequence features with generate_sequence_features_single/multi, producing data.csv and data_split.csv, together with the input contig FASTA file. Run the SemiBin2 train_self subcommand, which is the default training mode since SemiBin 1.3 and requires no external taxonomic annotation. Train the siamese network with the self-supervised contrastive objective using the documented defaults --epochs 15 and --batch-size 2048, optionally fixing --random-seed for reproducibility and choosing --engine (auto/gpu/cpu). To reuse one model across samples, pass several samples in matching order to data, data_split, and fasta files with --train-from-many. The learned embedding is then consumed downstream by the DBSCAN-based clustering (bin_short/bin_long --model) to produce MAGs.

## Related tools

- **SemiBin2** (Provides the train_self subcommand that trains the deep siamese neural network with self-supervised contrastive learning on contig features) — https://github.com/BigDataBiology/SemiBin/
- **SemiBin** (Command-line tool for metagenomic binning with deep learning (siamese neural networks); package installation provides the SemiBin2 command) — https://github.com/BigDataBiology/SemiBin/
- **mmseqs2** (Used only in the semi-supervised alternative (generate_cannot_links) to produce cannot-link files from GTDB annotations; not needed for self-supervised train_self)

## Examples

```
SemiBin train_self -i contigs.fasta --features data.csv:data_split.csv -o output --epochs 15 --batch-size 2048
```

## Evaluation signals

- model.pt checkpoint is written to the output directory and is loadable downstream (e.g. via bin_short/bin_long --model).
- SemiBinRun.log reports no fatal errors after training completes.
- Training used the documented defaults: --epochs 15 and --batch-size 2048 (unless explicitly overridden).
- No external taxonomic annotation files (mmseqs2/GTDB cannot-links) were required or consumed in the self-supervised mode.
- Downstream clustering on the learned embedding produces bins, since self-supervised learning achieves better results than semi-supervised learning in simulated and real datasets.

## Limitations

- SemiBin operations are probabilistic: running the pipeline twice can return different results unless --random-seed is fixed.
- The self-supervised mode replaced the semi-supervised taxonomic-annotation-based training; if taxonomic signal is specifically desired, the train_semi workflow with GTDB-based cannot-links must be used instead.
- Semi-supervised training (train_semi) is an extra step compared to the self-supervised mode, requiring generate_cannot_links and reference genomes (GTDB).

## Evidence

- [abstract] Self-supervised training method: "We propose SemiBin2, which uses self-supervised learning to learn feature embeddings from the contigs."
- [abstract] Self-supervised vs semi-supervised: "self-supervised learning achieves better results than the semi-supervised learning used in SemiBin1"
- [intro] Siamese network purpose: "SemiBin is a command tool for metagenomic binning with deep learning, handles both short and long reads."
- [methods] train_self subcommand: "`train_semi` (replacing `train_self` from the self-supervised mode)"
- [methods] Cannot-links only for semi-supervised: "`generate_cannot_links` (this is the extra step, compared to the self-supervised mode"
- [readme] Training CLI defaults: [evidence span withheld — anchor status: external_doc]
