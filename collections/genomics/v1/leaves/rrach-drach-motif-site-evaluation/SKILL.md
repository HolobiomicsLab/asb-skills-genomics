---
name: rrach-drach-motif-site-evaluation
description: Use when transcriptome-level m6A site predictions (transcript/pos/strand/prob/label
  TSVs) must be scored for AUC, AUPR, and accuracy on RRACH-motif sites against a
  published ground-truth site set, or compared to a baseline method such as DENA.
when_to_use_negative:
- Input is raw multi-read FAST5 or unaggregated read-level features — site-level prediction
  requires prior sorting and aggre_features_to_site_level.py aggregation with coverage
  >= 20
- Evaluating non-RRACH motifs or the DRACH pre-trained model outputs against a ground
  truth restricted to RRACH sites — motif contexts must match the ground-truth definition
- Data types other than Nanopore direct RNA sequencing (DRS) with resquiggled RawGenomeCorrected_001
  signals
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: RedNano
  role: Provides predict.py and the comb_basecall_raw model that emit per-site m6A
    probabilities for RRACH-site evaluation; aggre_features_to_site_level.py supplies
    the coverage-filtered site-level inputs
  repo: https://github.com/Derryxu/RedNano
- name: PyTorch
  role: Deep-learning framework used to run the trained model checkpoint during prediction
    of site-level modification probabilities
- name: CUDA
  role: GPU runtime enabling CUDA_VISIBLE_DEVICES=0 execution of predict.py
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae375
    title: RNA m6A detection using raw current signals and basecalling errors from
      Nanopore direct RNA sequencing reads
  source_content_hash: sha256:296b1afb1fa6fa9ea70beb9400706e8642f09bd86180e6c82a0dc2aa0fac6e14
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Derryxu/RedNano
  tools:
  - CUDA
  - PyTorch
  - Python
  - RedNano
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# rrach-drach-motif-site-evaluation

## Summary

Evaluate Nanopore DRS m6A site predictions within RRACH (and related DRACH-style) motifs at the transcriptome level using AUC, AUPR, and accuracy against a curated ground-truth site set, as RedNano does against Parker et al. 2020 sites and the DENA baseline on Arabidopsis.

## When to use

Use when transcriptome-level m6A site predictions (transcript/pos/strand/prob/label TSVs) must be scored for AUC, AUPR, and accuracy on RRACH-motif sites against a published ground-truth site set, or compared to a baseline method such as DENA.

## When NOT to use

- Input is raw multi-read FAST5 or unaggregated read-level features — site-level prediction requires prior sorting and aggre_features_to_site_level.py aggregation with coverage >= 20
- Evaluating non-RRACH motifs or the DRACH pre-trained model outputs against a ground truth restricted to RRACH sites — motif contexts must match the ground-truth definition
- Data types other than Nanopore direct RNA sequencing (DRS) with resquiggled RawGenomeCorrected_001 signals

## Inputs

- Aggregated site-level feature TSV (RedNano aggre file) with coverage >= 20 reads per site
- Trained RedNano comb_basecall_raw checkpoint (e.g. models/mil_allspecies_model_states.pt)
- Ground-truth m6A site list (3106 Parker et al. 2020 Arabidopsis sites, RRACH motif)
- Transcriptome length file with header (TAIR10/Araport11 .bed-style ID/length file)

## Outputs

- Per-site modified-probability TSV with schema transcript/pos/strand/prob/label
- Transcriptome-level RRACH-site metrics: AUC, AUPR, accuracy versus baseline (Fig. 2B / Supplementary Table S4)

## How to apply

Train the RedNano comb_basecall_raw model (epochs 50, patience 5, batch_size 4, hidden_size 128, dropout_rate 0.5, clip_grad 0.5, lr 1e-4, weight_decay 1e-5, embedding_size 4, seq_lens 5, signal_lens 65) on labeled read-level features, then aggregate read-level features to site level with aggre_features_to_site_level.py after sorting by transcript/position/read name. Restrict evaluation to sites with coverage >= 20 reads via the awk '$4>=20' filter, since multi-instance learning operates on site-level bags rather than single reads. Run predict.py with --num_iterations 1000 --batch_size 1 --signal_lens 65 --hidden_size 128 --embedding_size 4 --seq_lens 5 --model_type comb_basecall_raw to emit per-site modified probabilities in the transcript/pos/strand/prob/label schema. Score these probabilities against the 3106 Parker et al. 2020 m6A ground-truth sites restricted to the RRACH motif, computing AUC, AUPR, and accuracy, and compare directly with the DENA baseline (target: AUC 0.9405, AUPR 0.9474, accuracy 0.8629 on Arabidopsis, Fig. 2B / Supplementary Table S4).

## Related tools

- **RedNano** (Provides predict.py and the comb_basecall_raw model that emit per-site m6A probabilities for RRACH-site evaluation; aggre_features_to_site_level.py supplies the coverage-filtered site-level inputs) — https://github.com/Derryxu/RedNano
- **PyTorch** (Deep-learning framework used to run the trained model checkpoint during prediction of site-level modification probabilities)
- **CUDA** (GPU runtime enabling CUDA_VISIBLE_DEVICES=0 execution of predict.py)

## Examples

```
CUDA_VISIBLE_DEVICES=0 python scripts/predict.py --input_file test.features.sort.aggre.c20.tsv --model models/mil_allspecies_model_states.pt --num_iterations 1000 --batch_size 1 --signal_lens 65 --hidden_size 128 --embedding_size 4 --seq_lens 5 --model_type comb_basecall_raw --output_file test.features.sort.aggre.c20.results.tsv --nproc 3
```

## Evaluation signals

- Reproduced Arabidopsis evaluation reaches AUC ~0.9405, AUPR ~0.9474, accuracy ~0.8629 (exceeding DENA's 0.8763 / 0.8941 / 0.6382)
- Results TSV exists with the transcript/pos/strand/prob/label schema matching the RedNano output format
- Trained comb_basecall_raw checkpoint exists in the --save_dir (train/) with patience-5 early stopping over 50 epochs
- Every evaluated site passes the coverage >= 20 filter ($4>=20) applied to the aggregated feature file
- Predictions are restricted to RRACH-motif A sites and scored against the 3106 Parker et al. 2020 ground-truth sites

## Limitations

- Site-level inference requires coverage >= 20 reads per site; lower-coverage sites are excluded from evaluation
- Evaluation replicates the article's Arabidopsis benchmark against Parker et al. 2020 sites — different ground-truth sets or species will not reproduce AUC 0.9405 / AUPR 0.9474 / accuracy 0.8629
- The comb_basecall_raw model parameters (seq_lens 5, signal_lens 65, hidden_size 128) are specific to 5-mer RRACH windows with 65 raw signals; other motif/k-mer settings need retraining

## Evidence

- [intro] RedNano is a deep-learning method that utilizes both raw signals and basecalling errors to detect m6A from Nanopore DRS reads.: "RedNano is a deep-learning method that utilizes both raw signals and basecalling errors to detect m6A from Nanopore DRS reads."
- [abstract] The results demonstrate that RedNano surpasses existing methods by achieving higher area under the ROC curve (AUC) and area under the precision-recall curve (AUPRs) in all three datasets.: "The results demonstrate that RedNano surpasses existing methods by achieving higher area under the ROC curve (AUC) and area under the precision-recall curve (AUPRs) in all three datasets."
- [readme] awk coverage filter: "awk 'BEGIN{ FS=OFS="\t" } $4>=20 {print $0}' test.features.sort.aggre.tsv > test.features.sort.aggre.c20.tsv"
- [readme] Output: Modified probability for each site are in the following format:: "Output: Modified probability for each site are in the following format:"
- [methods] python aggre_features_to_site_level.py: "python /path/to/RedNano/scripts/aggre_features_to_site_level.py --input test.features.sort.tsv --output test.features.sort.aggre.tsv"
