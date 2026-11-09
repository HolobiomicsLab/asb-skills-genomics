---
name: roc-prc-benchmark-comparison
description: Use when comparing modification-detection methods on the same read-level
  feature set and you need quantitative read-level ACC/SEN/SPE/AUC/AUPR metrics written
  as a benchmark table.
when_to_use_negative:
- Features have already been aggregated to site level with aggre_features_to_site_level.py
  and coverage-filtered ($4>=20) — the task then calls for site-level, not read-level,
  benchmarking
- Input is raw FAST5/FASTQ data that has not yet gone through guppy basecalling, tombo
  resquiggle, minimap2 mapping, and feature extraction
- The evaluation target is a species or motif outside the article's scope (e.g., non-DRACH/RRACH
  contexts) where no trained model or labeled comparison data exist
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_3170
tools:
- name: RedNano
  role: Deep-learning m6A detector (comb_basecall_raw model) trained and evaluated
    on read-level raw-signal and basecalling-error features
  repo: https://github.com/Derryxu/RedNano
- name: nanom6A
  role: Competing m6A detection method trained and tested on the same RRACH site features
    for head-to-head comparison
- name: PyTorch
  role: Framework used to train and run RedNano's residual-network models (version
    1.10.0)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae375
    title: RNA m6A detection using raw current signals and basecalling errors from
      Nanopore direct RNA sequencing reads
  source_content_hash: sha256:ac658863781d4856885187294ab1a80b8512fb2c2a35ff8f5e3e9a97a9a7eb55
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Derryxu/RedNano
  tools:
  - PyTorch
  - Python
  - RedNano
  - nanom6A
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# roc-prc-benchmark-comparison

## Summary

Benchmark a deep-learning m6A detector (RedNano) against a competing method (nanom6A) at the read level by computing ACC, SEN, SPE, AUC, and AUPR on held-out test features from the synthesized RNA dataset (PRJNA511582), focusing on RRACH (and DRACH) motif sites.

## When to use

Use when comparing modification-detection methods on the same read-level feature set and you need quantitative read-level ACC/SEN/SPE/AUC/AUPR metrics written as a benchmark table.

## When NOT to use

- Features have already been aggregated to site level with aggre_features_to_site_level.py and coverage-filtered ($4>=20) — the task then calls for site-level, not read-level, benchmarking
- Input is raw FAST5/FASTQ data that has not yet gone through guppy basecalling, tombo resquiggle, minimap2 mapping, and feature extraction
- The evaluation target is a species or motif outside the article's scope (e.g., non-DRACH/RRACH contexts) where no trained model or labeled comparison data exist

## Inputs

- Read-level feature TSV from RedNano's extract_features.py (columns: chrom, pos, alignstrand, loc_in_ref, readname, strand, k_mer, k_signals_rect, qual, mis, ins, dele)
- Binary labels (1=modified, 0=unmodified) appended per feature row (mod_features_labeled.txt)
- train.txt / valid.txt splits formatted for RedNano's train.py
- Trained RedNano checkpoint (comb_basecall_raw model states)
- nanom6A predictions on the same RRACH site features of PRJNA511582

## Outputs

- Per-read modification probabilities from predict.py (transcript, pos, strand, prob, label)
- TSV benchmark table of read-level ACC, SEN, SPE, AUC, and AUPR for RedNano vs nanom6A

## How to apply

First train RedNano's comb_basecall_raw model (which combines the raw-signal and basecalling-error branches) with scripts/train.py using epochs 50, patience 5, seq_lens 5, signal_lens 65, batch_size 4, hidden_size 128, dropout_rate 0.5, clip_grad 0.5, lr 1e-4, weight_decay 1e-5, embedding_size 4 on labeled train/valid splits produced by appending 0/1 labels with awk. Then run scripts/predict.py on the held-out test features with the trained checkpoint (num_iterations 1000, batch_size 1, signal_lens 65, hidden_size 128, embedding_size 4, seq_lens 5) to get per-read modification probabilities. Train and test nanom6A on the identical RRACH-site features from PRJNA511582 so both methods are evaluated on the same samples. Finally compute read-level ACC, SEN, SPE, AUC, and AUPR for both methods and write the benchmark table to a TSV. The rationale is that RedNano's dual-branch residual networks over raw signals plus basecalling errors should outperform nanom6A on all criteria for RRACH sites.

## Related tools

- **RedNano** (Deep-learning m6A detector (comb_basecall_raw model) trained and evaluated on read-level raw-signal and basecalling-error features) — https://github.com/Derryxu/RedNano
- **nanom6A** (Competing m6A detection method trained and tested on the same RRACH site features for head-to-head comparison)
- **PyTorch** (Framework used to train and run RedNano's residual-network models (version 1.10.0))

## Examples

```
CUDA_VISIBLE_DEVICES=0 python scripts/predict.py --input_file test.features.sort.aggre.c20.tsv --model models/mil_allspecies_model_states.pt --num_iterations 1000 --batch_size 1 --signal_lens 65 --hidden_size 128 --embedding_size 4 --seq_lens 5 --model_type comb_basecall_raw --output_file results.tsv --nproc 3
```

## Evaluation signals

- RedNano must achieve read-level accuracy > 0.95 and AUC > 0.99 on RRACH sites of the synthesized dataset
- On DRACH sites the expected performance is accuracy > 0.94 and AUC > 0.98 (Fig. 2A)
- RedNano should attain higher AUC and AUPR than nanom6A across the evaluated criteria, matching the paper's claim of surpassing existing methods on synthesized, Arabidopsis, and human DRS data
- Predict output must follow the schema transcript, pos, strand, prob, label with probabilities in [0, 1]
- Both methods must be evaluated on the identical held-out RRACH feature rows from PRJNA511582 so metrics are directly comparable

## Limitations

- Reported performance is specific to the synthesized RNA dataset (PRJNA511582) and motif contexts (RRACH/DRACH); cross-species and independent Populus trichocarpa results are cited separately and may differ
- Comparison depends on nanom6A being trained and tested on exactly the same feature set; any mismatch in features or labels invalidates the benchmark
- RedNano's pretrained mil_allspecies_model_states.pt targets RNA DRACH m6A across synthetic, human, and Arabidopsis data; use-case-specific models require retraining

## Evidence

- [other] RedNano outperforms nanom6A on all evaluation criteria for RRACH sites, achieving >0.95 accuracy and >0.99 AUC: [evidence span withheld — anchor status: not_found]
- [abstract] RedNano surpasses existing methods by achieving higher AUC and AUPRs in all three datasets: "The results demonstrate that RedNano surpasses existing methods by achieving higher area under the ROC curve (AUC) and area under the precision-recall curve (AUPRs) in all three datasets."
- [readme] train.py comb_basecall_raw training parameters: "python scripts/train.py --train_option 0 --train_file test/train.txt --valid_file test/valid.txt --epochs 50 --patience 5 --seq_lens 5 --signal_lens 65 --batch_size 4 --hidden_size 128"
- [readme] predict.py invocation with pretrained model: "CUDA_VISIBLE_DEVICES=0 python scripts/predict.py --input_file test.features.sort.aggre.c20.tsv --model models/mil_allspecies_model_states.pt --num_iterations 1000 --batch_size 1 --signal_lens 65"
- [readme] labeling features with awk: "awk -v OFS="\t" '{print $0, 1}' mod_features.txt > mod_features_labeled.txt"
