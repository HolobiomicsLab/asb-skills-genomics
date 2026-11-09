---
name: nanopore-direct-rna-m6a-model-training
description: Use when you have RedNano-extracted read-level features (k_signals_rect,
  qual, mis, ins, dele columns) from modified and unmodified Nanopore DRS samples
  and need to train a species- or dataset-specific m6A detection model instead of
  using a pre-trained one.
when_to_use_negative:
- You only need off-the-shelf predictions and have no labeled modified/unmodified
  training data — use the pre-trained mil_allspecies_model_states.pt with predict.py
  instead
- Input is already a site-level aggregated feature table without read-level labels
  — training requires read-level features labeled 0/1 per read
- Data were not basecalled with the compatible Guppy version and resquiggled with
  tombo (RawGenomeCorrected_001), since feature extraction depends on that corrected-group
  layout
edam_operation: http://edamontology.org/operation_0432
edam_topics:
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_3168
tools:
- name: RedNano
  role: Provides extract_features.py, aggre_features_to_site_level.py, train.py, and
    predict.py for feature extraction, aggregation, model training, and inference
  repo: https://github.com/Derryxu/RedNano
- name: PyTorch 1.10.0
  role: Deep-learning framework in which the comb_basecall_raw residual-network model
    is trained
- name: CUDA 11.1
  role: GPU backend used to accelerate training (CUDA_VISIBLE_DEVICES=0)
- name: ont-tombo 1.5.1
  role: Resquiggles raw signals to produce the RawGenomeCorrected_001 corrected group
    consumed during feature extraction
- name: Guppy v3.1.5
  role: Basecalls FAST5 reads (rna_r9.4.1_70bps_hac.cfg) to supply basecalling-error
    features
- name: Python 3.7
  role: Runtime environment for the RedNano training scripts
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae375
    title: RNA m6A detection using raw current signals and basecalling errors from
      Nanopore direct RNA sequencing reads
  source_content_hash: sha256:a3669d2c56c63f2d58c1e5f2f23d87c5bfff0bbdd3f36f866212abf3e9de7bd7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Derryxu/RedNano
  tools:
  - CUDA
  - CUDA 11.1
  - Guppy v3.1.5
  - PyTorch 1.10.0
  - Python
  - Python 3.7
  - RedNano
  - ont-tombo 1.5.1
  edam_operation: http://edamontology.org/operation_0432
  edam_topics:
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# nanopore-direct-rna-m6a-model-training

## Summary

Train a RedNano comb_basecall_raw deep-learning model that combines raw Nanopore current signals and basecalling errors to detect m6A in RRACH motifs from direct RNA sequencing (DRS) reads, using a multi-instance learning pipeline with labeled read-level features aggregated to site level.

## When to use

Use when you have RedNano-extracted read-level features (k_signals_rect, qual, mis, ins, dele columns) from modified and unmodified Nanopore DRS samples and need to train a species- or dataset-specific m6A detection model instead of using a pre-trained one.

## When NOT to use

- You only need off-the-shelf predictions and have no labeled modified/unmodified training data — use the pre-trained mil_allspecies_model_states.pt with predict.py instead
- Input is already a site-level aggregated feature table without read-level labels — training requires read-level features labeled 0/1 per read
- Data were not basecalled with the compatible Guppy version and resquiggled with tombo (RawGenomeCorrected_001), since feature extraction depends on that corrected-group layout

## Inputs

- Single-read FAST5 files resquiggled with tombo (corrected_group RawGenomeCorrected_001)
- Feature TSV files from extract_features.py (columns: chrom, pos, alignstrand, loc_in_ref, readname, strand, k_mer, k_signals_rect, qual, mis, ins, dele)
- Labeled modified and unmodified read-level feature files (label column appended via awk)
- Train and valid TSV splits of site-level aggregated features
- Reference transcriptome length file with header (e.g. TAIR10/Araport11 .bed with ID and length columns)

## Outputs

- Trained comb_basecall_raw model checkpoint (e.g. train/*.pt model_states file)
- Sorted and aggregated site-level feature files (test.features.sort.tsv, test.features.sort.aggre.tsv, .c20.tsv)
- Per-site m6A modification probability TSV with transcript/pos/strand/prob/label schema

## How to apply

Extract combined raw-signal and basecalling-error features with RedNano's extract_features.py (k=5, s=65, corrected_group RawGenomeCorrected_001) against a transcriptome length file such as the TAIR10/Araport11 bed. Label modified and unmodified feature files with binary 1/0 labels via awk (e.g. "awk -v OFS='\t' '{print $0, 1}' mod_features.txt > mod_features_labeled.txt"), split into train/valid sets, then sort read-level features and aggregate to site level with aggre_features_to_site_level.py, retaining only sites with coverage >= 20 reads ($4>=20). Train with train.py using the comb_basecall_raw model type and the published hyperparameters: epochs 50, patience 5, batch_size 4, hidden_size 128, dropout_rate 0.5, clip_grad 0.5, lr 1e-4, weight_decay 1e-5, embedding_size 4, seq_lens 5, signal_lens 65, num_workers 2, saving checkpoints to train/. The rationale is that RedNano processes both the raw-signal feature and basecalling-error feature through residual networks, and multi-instance aggregation to site level lets read-level labels supervise site-level m6A prediction.

## Related tools

- **RedNano** (Provides extract_features.py, aggre_features_to_site_level.py, train.py, and predict.py for feature extraction, aggregation, model training, and inference) — https://github.com/Derryxu/RedNano
- **PyTorch 1.10.0** (Deep-learning framework in which the comb_basecall_raw residual-network model is trained)
- **CUDA 11.1** (GPU backend used to accelerate training (CUDA_VISIBLE_DEVICES=0))
- **ont-tombo 1.5.1** (Resquiggles raw signals to produce the RawGenomeCorrected_001 corrected group consumed during feature extraction)
- **Guppy v3.1.5** (Basecalls FAST5 reads (rna_r9.4.1_70bps_hac.cfg) to supply basecalling-error features)
- **Python 3.7** (Runtime environment for the RedNano training scripts)

## Examples

```
CUDA_VISIBLE_DEVICES=0 python scripts/train.py --train_option 0 --train_file test/train.txt --valid_file test/valid.txt --epochs 50 --patience 5 --seq_lens 5 --signal_lens 65 --batch_size 4 --hidden_size 128 --dropout_rate 0.5 --clip_grad 0.5 --lr 1e-4 --weight_decay 1e-5 --embedding_size 4 --num_workers 2 --model_type comb_basecall_raw --save_dir train/
```

## Evaluation signals

- Trained checkpoint file and results TSV exist with the transcript/pos/strand/prob/label schema
- Held-out Arabidopsis test-set evaluation reaches AUC ~0.9405, AUPR ~0.9474, accuracy ~0.8629 against the 3106 Parker et al. 2020 RRACH ground-truth sites (vs DENA's 0.8763/0.8941/0.6382)
- Site aggregation retains only sites with coverage >= 20 reads ($4>=20 in the awk filter)
- Feature file rows contain all 12 expected columns (chrom through dele) and label column appended as 0 or 1
- Training respects patience 5 (early stopping) and produces checkpoints in train/ with the specified signal_lens 65, seq_lens 5, embedding_size 4, hidden_size 128

## Limitations

- Performance depends on the exact basecaller and resquiggling pipeline (Guppy v3.1.5, tombo v1.5.1, RawGenomeCorrected_001); deviating versions may shift feature distributions
- Sites with fewer than 20 covering reads are filtered out, so low-coverage transcripts cannot be called
- Published validation is on RRACH-motif m6A in Arabidopsis (TAIR10/Araport11), synthesized and human DRS data; transfer to other species/motifs requires cross-species validation (demonstrated only on Populus trichocarpa)

## Evidence

- [intro] RedNano is a deep-learning method that utilizes both raw signals and basecalling errors to detect m6A from Nanopore DRS reads.: "RedNano is a deep-learning method that utilizes both raw signals and basecalling errors to detect m6A from Nanopore DRS reads."
- [readme] Train command with hyperparameters: "CUDA_VISIBLE_DEVICES=0 python scripts/train.py --train_option 0 --train_file test/train.txt --valid_file test/valid.txt --epochs 50 --patience 5 --seq_lens 5 --signal_lens 65 --batch_size 4"
- [readme] Labeling with awk: "awk -v OFS="\t" '{print $0, 1}' mod_features.txt > mod_features_labeled.txt"
- [methods] Aggregation and coverage filter: "awk 'BEGIN{ FS=OFS="\t" } $4>=20 {print $0}' test.features.sort.aggre.tsv > test.features.sort.aggre.c20.tsv"
- [abstract] Residual networks rationale: "RedNano processes the raw-signal feature and basecalling-error feature through residual networks."
