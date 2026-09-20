---
name: m6a-site-probability-prediction-nanopore-direct-rna
description: Use when you already have RedNano read-level feature TSVs (k_signals_rect,
  qual, mis, ins, dele) and need site-level m6A probabilities from a pre-trained model
  for evaluation or cross-species testing.
when_to_use_negative:
- Input is still read-level feature output (test.features.sort.tsv) that has not been
  aggregated with aggre_features_to_site_level.py — aggregation and coverage filtering
  must come first
- Sites have fewer than 20 supporting signals — the awk $4>=20 filter must be applied
  before prediction, otherwise the site-level features are too sparse for the multiple
  instance learning approach
- You need single-read-level modification calling rather than site-level probability
  prediction — this skill uses the multi instance learning (site-level) pathway
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3524
tools:
- name: RedNano
  role: 'Deep-learning predictor: scripts/predict.py scores site-aggregated features
    with a pre-trained checkpoint (comb_basecall_raw model type); scripts/aggre_features_to_site_level.py
    aggregates read-level features beforehand'
  repo: https://github.com/Derryxu/RedNano
- name: PyTorch
  role: Framework that loads and executes the RedNano residual-network checkpoints
    during inference (v1.10.0 in RedNano.yml)
- name: CUDA
  role: GPU backend used via CUDA_VISIBLE_DEVICES=0 to run predict.py (CUDA 11.1)
- name: Python
  role: Runtime for RedNano scripts and for computing ROC AUC / AUPR from the output
    probabilities (v3.7, scikit-learn 0.19.2)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae375
    title: RNA m6A detection using raw current signals and basecalling errors from
      Nanopore direct RNA sequencing reads
  source_content_hash: sha256:9be30fc92e4cbf3c581ca2e71444b269adfea078211cf5ff28f64e49c635ecb6
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# m6a-site-probability-prediction-nanopore-direct-rna

## Summary

Predicts per-site m6A modification probabilities on Nanopore direct RNA sequencing (DRS) data using RedNano, a deep-learning model that combines raw current signals with basecalling errors. Running the pre-trained checkpoint on a coverage-filtered, site-aggregated feature table yields per-site probabilities usable for ROC AUC / AUPR evaluation, including cross-species validation.

## When to use

Use when you already have RedNano read-level feature TSVs (k_signals_rect, qual, mis, ins, dele) and need site-level m6A probabilities from a pre-trained model for evaluation or cross-species testing.

## When NOT to use

- Input is still read-level feature output (test.features.sort.tsv) that has not been aggregated with aggre_features_to_site_level.py — aggregation and coverage filtering must come first
- Sites have fewer than 20 supporting signals — the awk $4>=20 filter must be applied before prediction, otherwise the site-level features are too sparse for the multiple instance learning approach
- You need single-read-level modification calling rather than site-level probability prediction — this skill uses the multi instance learning (site-level) pathway

## Inputs

- Coverage-filtered site-aggregated feature TSV (test.features.sort.aggre.c20.tsv) with columns chrom, pos, alignstrand, loc_in_ref, readname, strand, k_mer, k_signals_rect, qual, mis, ins, dele plus aggregate columns
- Pre-trained RedNano checkpoint (.pt), e.g. models/mil_allspecies_model_states.pt (DRACH m6A model trained on synthetic, human, and Arabidopsis data) or a single-species checkpoint
- CUDA-enabled GPU runtime (Python 3.7, PyTorch 1.10.0, CUDA 11.1 per RedNano.yml)

## Outputs

- Per-site prediction TSV with columns transcript, pos, strand, prob (modification probability), label
- Downstream ROC AUC and precision-recall AUPR metrics computed from prob vs label (e.g. Supplementary Table S4 cross-species values)

## How to apply

First aggregate read-level features to site level with RedNano's scripts/aggre_features_to_site_level.py and filter sites to coverage >= 20 signals using the awk condition $4>=20, producing a c20 TSV. Then run scripts/predict.py with --input_file set to the c20 TSV, --model pointing to a pre-trained checkpoint (e.g. models/mil_allspecies_model_states.pt or a single-species checkpoint for cross-species tests), and architecture parameters matched to training: --num_iterations 1000, --batch_size 1, --signal_lens 65, --hidden_size 128, --embedding_size 4, --seq_lens 5, --model_type comb_basecall_raw, on a CUDA GPU. The rationale is that RedNano's residual networks exploit both raw signal shapes and basecalling errors, so inference must use the same feature widths and model type used at training. Parse the output TSV's transcript/pos/strand/prob/label columns and compute ROC AUC and precision-recall AUPR from prob against the binary label (e.g. with scikit-learn). Correct application is judged by reproducing reported metrics — e.g. human-trained model on Arabidopsis (AUC 0.8432, AUPR 0.8561) and Arabidopsis-trained on human (AUC 0.8159, AUPR 0.8246).

## Related tools

- **RedNano** (Deep-learning predictor: scripts/predict.py scores site-aggregated features with a pre-trained checkpoint (comb_basecall_raw model type); scripts/aggre_features_to_site_level.py aggregates read-level features beforehand) — https://github.com/Derryxu/RedNano
- **PyTorch** (Framework that loads and executes the RedNano residual-network checkpoints during inference (v1.10.0 in RedNano.yml))
- **CUDA** (GPU backend used via CUDA_VISIBLE_DEVICES=0 to run predict.py (CUDA 11.1))
- **Python** (Runtime for RedNano scripts and for computing ROC AUC / AUPR from the output probabilities (v3.7, scikit-learn 0.19.2))

## Examples

```
CUDA_VISIBLE_DEVICES=0 python scripts/predict.py --input_file test.features.sort.aggre.c20.tsv --model models/mil_allspecies_model_states.pt --num_iterations 1000 --batch_size 1 --signal_lens 65 --hidden_size 128 --embedding_size 4 --seq_lens 5 --model_type comb_basecall_raw --output_file test.features.sort.aggre.c20.results.tsv --nproc 3
```

## Evaluation signals

- Output TSV has exactly the documented schema: transcript, pos, strand, prob, label columns
- prob values are valid modification probabilities (bounded in [0,1]) for every site row
- Every output site row traces back to a c20-filtered input row (coverage >= 20 signals)
- Computed ROC AUC and AUPR reproduce reported values within tolerance: AUC 0.8432 / AUPR 0.8561 for human-trained model on Arabidopsis and AUC 0.8159 / AUPR 0.8246 for Arabidopsis-trained model on human (Fig. 3 / Supplementary Table S4)
- Architecture flags passed to predict.py (signal_lens 65, seq_lens 5, hidden_size 128, embedding_size 4, model_type comb_basecall_raw) match the checkpoint's training configuration

## Limitations

- Coverage threshold: the workflow filters to sites with >= 20 signals ($4>=20); sites below this cutoff are excluded and cannot be scored reliably by the site-level model
- Cross-species performance degrades relative to within-species performance — the human-trained model reaches AUC 0.8432 on Arabidopsis and the Arabidopsis-trained model AUC 0.8159 on human, so checkpoint choice materially affects results
- Checkpoint and inference architecture must match: predict.py requires the same --num_iterations 1000, --signal_lens 65, --hidden_size 128, --embedding_size 4, --seq_lens 5, --model_type comb_basecall_raw used at training

## Evidence

- [readme] RedNano is a deep-learning method that utilizes both raw signals and basecalling errors to detect m6A from Nanopore DRS reads.: "RedNano is a deep-learning method that utilizes both raw signals and basecalling errors to detect m6A from Nanopore DRS reads."
- [methods] aggregate and filter to c20: "python /path/to/RedNano/scripts/aggre_features_to_site_level.py --input test.features.sort.tsv --output test.features.sort.aggre.tsv"
- [methods] coverage >= 20 filter: "awk 'BEGIN{ FS=OFS="\t" } $4>=20 {print $0}' test.features.sort.aggre.tsv > test.features.sort.aggre.c20.tsv"
- [readme] predict.py invocation: "CUDA_VISIBLE_DEVICES=0 python scripts/predict.py --input_file test.features.sort.aggre.c20.tsv --model models/mil_allspecies_model_states.pt --num_iterations 1000 --batch_size 1 --signal_lens 65"
- [readme] output format: "Output: Modified probability for each site are in the following format: | transcript | pos | strand | prob | label |"
- [abstract] cross-species robustness: "Furthermore, RedNano performs better in cross-species validation, demonstrating its robustness."
- [supplementary] cross-species metrics from source card finding: [evidence span withheld — anchor status: not_found]
