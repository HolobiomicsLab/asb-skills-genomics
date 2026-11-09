---
name: m6a-deep-learning-model-training
description: Use when you have RedNano-extracted comb_basecall_raw features (k-mer,
  65-signal window, qual/mis/ins/dele) from modified and unmodified Nanopore DRS FAST5s
  and need a trained m6A site classifier.
when_to_use_negative:
- Input is already a trained checkpoint and you only need site-level m6A probabilities
  — use scripts/predict.py with a pre-trained model such as mil_allspecies_model_states.pt
  instead
- You lack modified/unmodified labeled training data and only have unannotated DRS
  reads — supervised training requires binary labels appended to feature rows
- You need the cross-dataset generalization benchmark rather than a custom model —
  RedNano ships a pre-trained all-species DRACH model for prediction-only use
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: RedNano
  role: Provides extract_features.py, aggre_features_to_site_level.py, and scripts/train.py
    used to build and train the comb_basecall_raw m6A model
  repo: https://github.com/Derryxu/RedNano
- name: PyTorch 1.10.0
  role: Deep-learning framework in which the RedNano residual-network model is defined
    and trained
- name: ont-tombo 1.5.1
  role: Resquiggles raw signals (--fit-global-scale --include-event-stdev) into RawGenomeCorrected_001
    groups that supply the raw-signal features for training
- name: guppy v3.1.5
  role: Basecalls FAST5 reads with rna_r9.4.1_70bps_hac.cfg --fast5_out to produce
    the basecalling-error features and resquiggle inputs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae375
    title: RNA m6A detection using raw current signals and basecalling errors from
      Nanopore direct RNA sequencing reads
  source_content_hash: sha256:236ce57b247b8147c435da100234cb2fcecc5280e303adc4e09f2177a42e9be3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Derryxu/RedNano
  tools:
  - CUDA
  - PyTorch 1.10.0
  - Python
  - RedNano
  - guppy v3.1.5
  - h5py
  - jvarkit
  - minimap2
  - ont-tombo
  - ont-tombo 1.5.1
  - samtools
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5789473684210527
n_claims: 19
---

# m6a-deep-learning-model-training

## Summary

Trains RedNano, a deep-learning model for m6A detection from Nanopore DRS reads, on labeled read/site-level features that combine raw current signals with basecalling errors, so practitioners can build species- or condition-specific m6A predictors on private data.

## When to use

Use when you have RedNano-extracted comb_basecall_raw features (k-mer, 65-signal window, qual/mis/ins/dele) from modified and unmodified Nanopore DRS FAST5s and need a trained m6A site classifier.

## When NOT to use

- Input is already a trained checkpoint and you only need site-level m6A probabilities — use scripts/predict.py with a pre-trained model such as mil_allspecies_model_states.pt instead
- You lack modified/unmodified labeled training data and only have unannotated DRS reads — supervised training requires binary labels appended to feature rows
- You need the cross-dataset generalization benchmark rather than a custom model — RedNano ships a pre-trained all-species DRACH model for prediction-only use

## Inputs

- Labeled RedNano feature TSV (train.txt / valid.txt) with columns chrom, pos, alignstrand, loc_in_ref, readname, strand, k_mer, k_signals_rect, qual, mis, ins, dele, plus appended binary label
- Raw Nanopore DRS FAST5 files resquiggled by tombo (RawGenomeCorrected_001) with guppy v3.1.5 basecalls
- Reference transcriptome FASTA and transcriptome length .bed file with header

## Outputs

- Trained model checkpoint saved under --save_dir (e.g. train/) suitable for scripts/predict.py
- Per-site m6A modified probabilities after running scripts/predict.py (transcript, pos, strand, prob, label)

## How to apply

First extract combined raw-signal and basecalling-error features with RedNano scripts/extract_features.py (-k 5, -s 65, --corrected_group RawGenomeCorrected_001), then aggregate read-level features to site level with scripts/aggre_features_to_site_level.py and keep only sites with column-4 coverage >= 20. Append binary labels (0/1) to the modified and unmodified feature rows with awk (e.g. awk -v OFS="\t" '{print $0, 1}' mod_features.txt > mod_features_labeled.txt) and split into train and valid files. Train with scripts/train.py using --epochs 50 --patience 5 --seq_lens 5 --signal_lens 65 --batch_size 4 --hidden_size 128 --dropout_rate 0.5 --clip_grad 0.5 --lr 1e-4 --weight_decay 1e-5 --embedding_size 4 --num_workers 2 --model_type comb_basecall_raw, allowing early stopping on validation performance to halt training. The residual-network architecture processes the raw-signal feature and basecalling-error feature in parallel, which is the rationale for the comb_basecall_raw model type.

## Related tools

- **RedNano** (Provides extract_features.py, aggre_features_to_site_level.py, and scripts/train.py used to build and train the comb_basecall_raw m6A model) — https://github.com/Derryxu/RedNano
- **PyTorch 1.10.0** (Deep-learning framework in which the RedNano residual-network model is defined and trained)
- **ont-tombo 1.5.1** (Resquiggles raw signals (--fit-global-scale --include-event-stdev) into RawGenomeCorrected_001 groups that supply the raw-signal features for training)
- **guppy v3.1.5** (Basecalls FAST5 reads with rna_r9.4.1_70bps_hac.cfg --fast5_out to produce the basecalling-error features and resquiggle inputs)

## Examples

```
CUDA_VISIBLE_DEVICES=0 python scripts/train.py --train_option 0 --train_file test/train.txt --valid_file test/valid.txt --epochs 50 --patience 5 --seq_lens 5 --signal_lens 65 --batch_size 4 --hidden_size 128 --dropout_rate 0.5 --clip_grad 0.5 --lr 1e-4 --weight_decay 1e-5 --embedding_size 4 --num_workers 2 --model_type comb_basecall_raw --save_dir train/
```

## Evaluation signals

- Training loss decreases over up to 50 epochs and early stopping with patience 5 triggers on validation performance, confirming the checkpoint saved is the best-validation model
- Held-out site-level evaluation (coverage >= 20 filter applied) yields AUC ~0.85 on human HEK293T (PRJEB40872): 0.8511 for RRACH and 0.8513 for DRACH, exceeding m6Anet baselines (0.8230 / 0.8187)
- Output feature TSV matches the RedNano schema (chrom, pos, alignstrand, loc_in_ref, readname, strand, k_mer, k_signals_rect, qual, mis, ins, dele) with a single 0/1 label column appended
- Predicted results TSV contains per-site probabilities with transcript, pos, strand, prob, label columns for every retained site

## Limitations

- Training requires labeled modified and unmodified DRS data with sufficient depth — sites with column-4 coverage below 20 are filtered out and cannot be used or evaluated
- Fixed sequence/signal encodings (k 5-mer, signal_lens 65, seq_lens 5) mean retraining is needed if motifs or signal windows differ; tombo resquiggle reference must be a transcript fasta, not a genome file
- Performance claims were established against specific competitors (e.g. m6Anet) on synthesized, Arabidopsis, and human datasets; results on other species or chemistries are not guaranteed

## Evidence

- [abstract] RedNano is a deep-learning method that utilizes both raw signals and basecalling errors to detect m6A from Nanopore DRS reads: "RedNano is a deep-learning method that utilizes both raw signals and basecalling errors to detect m6A from Nanopore DRS reads."
- [abstract] residual networks: "RedNano processes the raw-signal feature and basecalling-error feature through residual networks."
- [readme] training command: "CUDA_VISIBLE_DEVICES=0 python scripts/train.py --train_option 0 --train_file test/train.txt --valid_file test/valid.txt --epochs 50 --patience 5"
- [readme] labeling step: "awk -v OFS="\t" '{print $0, 1}' mod_features.txt > mod_features_labeled.txt"
- [methods] coverage filter: "awk 'BEGIN{ FS=OFS="\t" } $4>=20 {print $0}' test.features.sort.aggre.tsv > test.features.sort.aggre.c20.tsv"
- [results] human benchmark: [evidence span withheld — anchor status: not_found]
