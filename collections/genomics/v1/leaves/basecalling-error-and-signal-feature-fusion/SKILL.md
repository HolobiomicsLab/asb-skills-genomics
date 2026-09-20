---
name: basecalling-error-and-signal-feature-fusion
description: Use when raw Nanopore DRS FAST5 reads have been basecalled and resquiggled
  (RawGenomeCorrected_001), and per-site m6A modification probabilities must be predicted
  using both raw current signals and basecalling error profiles.
when_to_use_negative:
- Input is already a site-level prediction file or m6A site BED — re-extracting features
  from FAST5 is unnecessary.
- Only single-molecule read-level modification calls are needed without site-level
  aggregation; RedNano's multi-instance aggregation to coverage >= 20 sites is designed
  for site-level prediction.
- Data are not Nanopore DRS reads (e.g. short-read RNA-seq or meRIP-seq), since the
  method requires raw current signals and tombo-resquiggled events.
edam_operation: http://edamontology.org/operation_3902
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3524
tools:
- name: RedNano
  role: Deep-learning framework that processes raw-signal (k_signals_rect) and basecalling-error
    (qual, mis, ins, dele) features through residual networks for m6A detection
  repo: https://github.com/Derryxu/RedNano
- name: ont-tombo
  role: Resquiggles raw signals against the reference transcript fasta to produce
    RawGenomeCorrected_001 signal events used as features
- name: ont-fast5-api
  role: Splits multi-read FAST5 files into single-read FAST5 files prior to basecalling
    and resquiggling
- name: minimap2
  role: Maps basecalled reads (-ax map-ont) to the reference transcriptome for alignment-derived
    error features
- name: samtools
  role: Sorts BAM and filters reads (-h -F 3844) before conversion to TSV via sam2tsv
- name: jvarkit
  role: sam2tsv converts alignment BAM to per-base TSV used to derive basecalling
    error features
  repo: https://github.com/lindenb/jvarkit
- name: PyTorch
  role: Backend for training and inference of the comb_basecall_raw residual network
    model
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae375
    title: RNA m6A detection using raw current signals and basecalling errors from
      Nanopore direct RNA sequencing reads
  source_content_hash: sha256:f45fc72d77580f24b332fb07758524068341459dde4add37179aeda0195fada0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Derryxu/RedNano
  tools:
  - CUDA
  - PyTorch
  - Python
  - RedNano
  - h5py
  - jvarkit
  - minimap2
  - ont-fast5-api
  - ont-tombo
  - samtools
  edam_operation: http://edamontology.org/operation_3902
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8695652173913043
n_claims: 23
---

# basecalling-error-and-signal-feature-fusion

## Summary

A RedNano-based deep learning approach that fuses Nanopore direct RNA sequencing (DRS) raw current signals with basecalling error features (qual, mis, ins, dele) through residual networks to detect m6A sites. Combining both modalities outperforms signal-only or error-only detection, achieving higher AUC and AUPR than methods like m6Anet.

## When to use

Use when raw Nanopore DRS FAST5 reads have been basecalled and resquiggled (RawGenomeCorrected_001), and per-site m6A modification probabilities must be predicted using both raw current signals and basecalling error profiles.

## When NOT to use

- Input is already a site-level prediction file or m6A site BED — re-extracting features from FAST5 is unnecessary.
- Only single-molecule read-level modification calls are needed without site-level aggregation; RedNano's multi-instance aggregation to coverage >= 20 sites is designed for site-level prediction.
- Data are not Nanopore DRS reads (e.g. short-read RNA-seq or meRIP-seq), since the method requires raw current signals and tombo-resquiggled events.

## Inputs

- Nanopore DRS FAST5 raw signal files (multi-read or single-read)
- Reference transcriptome FASTA (e.g. referance.transcript.fa)
- Transcriptome length BED file with header (ID, length columns)
- Basecalled FASTQ from guppy
- SAM/BAM alignment of basecalled reads against the transcriptome
- Labeled site-level feature TSV (mod/unmod rows with 0/1 label column) for training

## Outputs

- Per-read combined feature TSV with columns chrom, pos, alignstrand, loc_in_ref, readname, strand, k_mer, k_signals_rect, qual, mis, ins, dele
- Site-level aggregated and coverage-filtered feature file (test.features.sort.aggre.c20.tsv)
- Trained model checkpoint (e.g. train/ directory, comb_basecall_raw)
- Site-level prediction TSV with transcript, pos, strand, prob, label columns

## How to apply

Preprocess FAST5 reads by splitting multi-read files with ont_fast5_api (multi_to_single_fast5, -t 30), basecalling with guppy v3.1.5 (rna_r9.4.1_70bps_hac.cfg, --fast5_out), and resquiggling with tombo v1.5.1 (--fit-global-scale --include-event-stdev, corrected-group RawGenomeCorrected_001) against a reference transcript fasta. Map basecalled reads to the transcriptome with minimap2 2.17 (-ax map-ont), sort with samtools 1.7, convert BAM to per-read TSV via jvarkit sam2tsv, and split TSV into per-read files. Extract combined raw-signal and basecalling-error features with RedNano scripts/extract_features.py (-k 5, -s 65, -n 30, --corrected_group RawGenomeCorrected_001), producing rows with k_mer, k_signals_rect, qual, mis, ins, dele columns. Sort read-level features, aggregate to site level with scripts/aggre_features_to_site_level.py, and retain sites with coverage >= 20 (awk '$4>=20'). Train the residual-network model with scripts/train.py (--model_type comb_basecall_raw, --signal_lens 65, --hidden_size 128, --epochs 50 --patience 5 early stopping) and predict site-level m6A probabilities with scripts/predict.py (--num_iterations 1000, --batch_size 1, --model_type comb_basecall_raw), which outputs per-site transcript, pos, strand, prob, label rows.

## Related tools

- **RedNano** (Deep-learning framework that processes raw-signal (k_signals_rect) and basecalling-error (qual, mis, ins, dele) features through residual networks for m6A detection) — https://github.com/Derryxu/RedNano
- **ont-tombo** (Resquiggles raw signals against the reference transcript fasta to produce RawGenomeCorrected_001 signal events used as features)
- **ont-fast5-api** (Splits multi-read FAST5 files into single-read FAST5 files prior to basecalling and resquiggling)
- **minimap2** (Maps basecalled reads (-ax map-ont) to the reference transcriptome for alignment-derived error features)
- **samtools** (Sorts BAM and filters reads (-h -F 3844) before conversion to TSV via sam2tsv)
- **jvarkit** (sam2tsv converts alignment BAM to per-base TSV used to derive basecalling error features) — https://github.com/lindenb/jvarkit
- **PyTorch** (Backend for training and inference of the comb_basecall_raw residual network model)

## Examples

```
CUDA_VISIBLE_DEVICES=0 python scripts/train.py --train_option 0 --train_file test/train.txt --valid_file test/valid.txt --epochs 50 --patience 5 --seq_lens 5 --signal_lens 65 --batch_size 4 --hidden_size 128 --dropout_rate 0.5 --clip_grad 0.5 --lr 1e-4 --weight_decay 1e-5 --embedding_size 4 --num_workers 2 --model_type comb_basecall_raw --save_dir train/
```

## Evaluation signals

- Site-level prediction output should contain exactly the columns transcript, pos, strand, prob, label per README.
- Coverage filter check: aggregated site file must only retain rows where column 4 (coverage) >= 20 (test.features.sort.aggre.c20.tsv).
- On the human HEK293T dataset (PRJEB40872) at transcriptome level, expect AUC ~0.8511 (RRACH) / 0.8513 (DRACH), higher than m6Anet's 0.8230 / 0.8187.
- Training should show early stopping within 50 epochs (patience 5) on validation performance with hidden_size 128, dropout 0.5, lr 1e-4.
- Feature file rows must match the expected 12-column schema (chrom..dele) with k_signals_rect length consistent with -s 65 and k_mer consistent with -k 5.

## Limitations

- Tombo resquiggle requires a reference transcript/gene fasta, not a genome file; using a genome reference will break the corrected-group generation.
- Guppy version (v3.1.5) and config (rna_r9.4.1_70bps_hac.cfg) are tied to the R9.4.1 chemistry; other flowcells/configs are not validated in this context.
- Sites with read coverage below 20 are discarded by the aggregation filter, so low-coverage m6A sites cannot be predicted.
- Feature extraction requires paired raw-signal workspace (fast5_guppy) and per-read error TSVs (test/tmp/); missing either blocks comb_basecall_raw training.

## Evidence

- [abstract] RedNano processes the raw-signal feature and basecalling-error feature through residual networks.: "RedNano processes the raw-signal feature and basecalling-error feature through residual networks."
- [readme] Feature extraction output schema from README.: "| chrom | pos  | alignstrand | loc_in_ref | readname | strand | k_mer | k_signals_rect | qual | mis  | ins  | dele |"
- [readme] Coverage filter for site-level aggregation.: "awk 'BEGIN{ FS=OFS="\t" } $4>=20 {print $0}' test.features.sort.aggre.tsv > test.features.sort.aggre.c20.tsv"
- [readme] Resquiggle requirement that reference must be transcript fasta, not genome.: "The tombo (v1.5.1) resquiggle referance.transcript.fa should not be genome file, it should be the referance gene fasta file."
- [readme] Training command showing comb_basecall_raw model type and hyperparameters.: "--model_type comb_basecall_raw --save_dir train/"
- [methods] Finding on human dataset performance vs m6Anet from source card workflow.: [evidence span withheld — anchor status: not_found]
