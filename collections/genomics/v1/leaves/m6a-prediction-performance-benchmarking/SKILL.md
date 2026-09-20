---
name: m6a-prediction-performance-benchmarking
description: Use when comparing m6A detection methods at the transcriptome level on
  Nanopore DRS data, or when validating that a re-trained RedNano model reproduces
  published metrics such as AUC 0.9405 vs DENA 0.8763 on Arabidopsis RRACH sites.
when_to_use_negative:
- Input is already an aggregated site-level feature table (c20) and only scoring/prediction
  is needed — use the prediction workflow with a pre-trained model instead of re-benchmarking
- Data are not Nanopore direct RNA sequencing reads with raw-signal FAST5 files (e.g.
  short-read m6A-seq such as miCLIP) — RedNano's raw-signal/basecalling-error features
  do not apply
- Candidate sites are in DRACH rather than RRACH motif context, or the goal is cross-species
  generalization with the pre-trained mil_allspecies model rather than a species-specific
  retrained benchmark
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0032
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_0622
tools:
- name: RedNano
  role: Deep-learning framework providing extract_features.py, aggre_features_to_site_level.py,
    train.py, and predict.py for feature extraction, site aggregation, comb_basecall_raw
    training, and per-site m6A probability prediction
  repo: https://github.com/Derryxu/RedNano
- name: PyTorch
  role: Backend for training and running the comb_basecall_raw multi-instance learning
    model
- name: CUDA
  role: GPU runtime for model training and inference
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae375
    title: RNA m6A detection using raw current signals and basecalling errors from
      Nanopore direct RNA sequencing reads
  source_content_hash: sha256:f48ac29f176f0741dbc165fc960072dd82115c9b8d3325a543144d3f18b86985
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Derryxu/RedNano
  tools:
  - CUDA
  - PyTorch
  - Python
  - RedNano
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0032
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# m6a-prediction-performance-benchmarking

## Summary

Reproduce and benchmark transcriptome-level m6A site detection by training RedNano's comb_basecall_raw multi-instance-learning model on Nanopore DRS features and comparing AUC, AUPR, and accuracy against a baseline such as DENA on RRACH sites.

## When to use

Use when comparing m6A detection methods at the transcriptome level on Nanopore DRS data, or when validating that a re-trained RedNano model reproduces published metrics such as AUC 0.9405 vs DENA 0.8763 on Arabidopsis RRACH sites.

## When NOT to use

- Input is already an aggregated site-level feature table (c20) and only scoring/prediction is needed — use the prediction workflow with a pre-trained model instead of re-benchmarking
- Data are not Nanopore direct RNA sequencing reads with raw-signal FAST5 files (e.g. short-read m6A-seq such as miCLIP) — RedNano's raw-signal/basecalling-error features do not apply
- Candidate sites are in DRACH rather than RRACH motif context, or the goal is cross-species generalization with the pre-trained mil_allspecies model rather than a species-specific retrained benchmark

## Inputs

- Single-read FAST5 files with tombo resquiggle'd signals (RawGenomeCorrected_001 corrected group)
- Guppy (v3.1.5) basecalled FASTQ reads
- Reference transcriptome FASTA and transcriptome length BED file (e.g. TAIR10/Araport11)
- Read-level feature TSV (chrom, pos, alignstrand, loc_in_ref, readname, strand, k_mer, k_signals_rect, qual, mis, ins, dele)
- Parker et al. 2020 m6A ground-truth site list (3106 RRACH sites)

## Outputs

- Trained RedNano comb_basecall_raw checkpoint in train/
- Per-site probability TSV with schema transcript/pos/strand/prob/label
- Benchmark metrics table: AUC, AUPR, accuracy for RedNano vs DENA (cf. Fig. 2B, Supplementary Table S4)

## How to apply

Extract combined raw-signal and basecalling-error features from resquiggle'd FAST5 files with RedNano's extract_features.py using -k 5, -s 65, and --corrected_group RawGenomeCorrected_001 against the TAIR10/Araport11 transcriptome length BED file. Label modified and unmodified read-level feature files with binary labels via awk (e.g. "awk -v OFS='\t' '{print $0, 1}' mod_features.txt"), sort read-level features, aggregate to site level with aggre_features_to_site_level.py, and keep only sites with coverage >= 20 reads. Train the comb_basecall_raw model with train.py (epochs 50, patience 5, batch_size 4, hidden_size 128, dropout_rate 0.5, clip_grad 0.5, lr 1e-4, weight_decay 1e-5, embedding_size 4, seq_lens 5, signal_lens 65). Predict site-level modification probabilities on the held-out test set with predict.py (num_iterations 1000, batch_size 1), then evaluate transcriptome-level RRACH-site predictions against the 3106 Parker et al. 2020 ground-truth m6A sites, computing AUC, AUPR, and accuracy for comparison with DENA.

## Related tools

- **RedNano** (Deep-learning framework providing extract_features.py, aggre_features_to_site_level.py, train.py, and predict.py for feature extraction, site aggregation, comb_basecall_raw training, and per-site m6A probability prediction) — https://github.com/Derryxu/RedNano
- **PyTorch** (Backend for training and running the comb_basecall_raw multi-instance learning model)
- **CUDA** (GPU runtime for model training and inference)

## Examples

```
CUDA_VISIBLE_DEVICES=0 python scripts/predict.py --input_file test.features.sort.aggre.c20.tsv --model models/mil_allspecies_model_states.pt --num_iterations 1000 --batch_size 1 --signal_lens 65 --hidden_size 128 --embedding_size 4 --seq_lens 5 --model_type comb_basecall_raw --output_file test.features.sort.aggre.c20.results.tsv --nproc 3
```

## Evaluation signals

- Reproduced Arabidopsis transcriptome-level metrics reach AUC ~0.9405, AUPR ~0.9474, accuracy ~0.8629 (vs DENA AUC 0.8763, AUPR 0.8941, accuracy 0.6382) as in Fig. 2B / Supplementary Table S4
- Results TSV exists with the transcript/pos/strand/prob/label schema and one row per RRACH site with coverage >= 20 reads
- Trained checkpoint file exists in the train/ save_dir and can be loaded with predict.py via --model
- Aggregated feature file retains only instances whose read-coverage column ($4) is >= 20 after the awk c20 filter
- Probabilities are bounded in [0,1] and held-out test sites are scored against the 3106-site Parker et al. 2020 ground truth

## Limitations

- Benchmark is specific to the RRACH motif and transcriptome-level site evaluation; DRACH-context or genomic-level benchmarks require different models/pre-trained weights
- Results depend on tombo resquiggle quality and Guppy v3.1.5 basecalling; the tombo resquiggle reference must be the transcript (gene) FASTA, not the genome
- Site-level evaluation discards sites with fewer than 20 covering reads, so low-coverage transcriptome regions cannot be benchmarked

## Evidence

- [results] RedNano outperforms DENA for RRACH m6A detection with AUC 0.9405 vs 0.8763, AUPR 0.9474 vs 0.8941, and accuracy 0.8629 vs 0.6382: [evidence span withheld — anchor status: not_found]
- [readme] keep only sites with coverage >= 20 reads: "awk 'BEGIN{ FS=OFS="\t" } $4>=20 {print $0}' test.features.sort.aggre.tsv > test.features.sort.aggre.c20.tsv"
- [readme] train comb_basecall_raw with specified hyperparameters: "CUDA_VISIBLE_DEVICES=0 python scripts/train.py --train_option 0 --train_file test/train.txt --valid_file test/valid.txt --epochs 50 --patience 5 --seq_lens 5 --signal_lens 65 --batch_size 4"
- [readme] output schema for per-site modified probabilities: "Output: Modified probability for each site are in the following format: | transcript | pos | strand | prob | label |"
- [abstract] higher AUC and AUPRs in all three datasets: "The results demonstrate that RedNano surpasses existing methods by achieving higher area under the ROC curve (AUC) and area under the precision-recall curve (AUPRs) in all three datasets."
