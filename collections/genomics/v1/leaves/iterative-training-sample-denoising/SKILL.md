---
name: iterative-training-sample-denoising
description: Use when constructing a Nanopore 5mC training set from signal features
  where methylation labels derived from bisulfite contain false positives (partially
  methylated sites), which depresses model performance for CHG/CHH.
when_to_use_negative:
- Training labels are already high-confidence and balanced, e.g. CpG sites with clean
  >=0.9 vs 0 methylation frequency separation — the article reports the largest gains
  are for CHH/CHG, not CpG.
- The input is a feature-extraction problem (raw FAST5 signals not yet converted to
  13-mer + 13x16 features); use deepsignal_plant extract first.
- Evaluating methylation on held-out reads (use deepsignal_plant call_mods / call_freq
  instead of the training-set denoising loop).
edam_operation: http://edamontology.org/operation_3459
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_3474
tools:
- name: DeepSignal-plant
  role: Implements the iterative denoising algorithm (Algorithm 2) and the BiLSTM
    model used for two-fold cross prediction of training samples
  repo: https://github.com/PengNi/deepsignal-plant
- name: PyTorch
  role: Framework for training the BRNN-with-LSTM model (Adam optimizer, cross-entropy
    loss) used inside each denoising iteration
provenance:
  source_task_ids:
  - task_002
  - task_005
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-26278-9
    title: Genome-wide detection of cytosine methylations in plant from Nanopore data
      using deep learning
  source_content_hash: sha256:86a34bd79d99c77eaad3dcead137d90eabe2999c5b93fc39958b2cea4f1e6b61
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PengNi/deepsignal-plant
  tools:
  - DeepSignal-plant
  - Guppy
  - PyTorch
  - Tombo
  edam_operation: http://edamontology.org/operation_3459
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# iterative-training-sample-denoising

## Summary

DeepSignal-plant's iterative denoising procedure (Algorithm 2, I=10, R=3, E=3) removes false methylated training samples by repeated two-fold cross prediction with the BiLSTM model, materially improving 5mC detection especially for CHG and CHH contexts.

## When to use

Use when constructing a Nanopore 5mC training set from signal features where methylation labels derived from bisulfite contain false positives (partially methylated sites), which depresses model performance for CHG/CHH.

## When NOT to use

- Training labels are already high-confidence and balanced, e.g. CpG sites with clean >=0.9 vs 0 methylation frequency separation — the article reports the largest gains are for CHH/CHG, not CpG.
- The input is a feature-extraction problem (raw FAST5 signals not yet converted to 13-mer + 13x16 features); use deepsignal_plant extract first.
- Evaluating methylation on held-out reads (use deepsignal_plant call_mods / call_freq instead of the training-set denoising loop).

## Inputs

- Balanced training sample TSV with 13-mer k_mer and 13x16 normalized signal features plus methy_label per cytosine (output of deepsignal_plant extract and Balance_Negative_Samples)
- Trained BiLSTM (BRNN with LSTM units) model checkpoint initialized for cross prediction

## Outputs

- Denoised, k-mer-balanced positive/negative training sample file (>=99% sample retention or after 10 iterations) usable by deepsignal_plant train
- Per-iteration removal statistics for methylated samples predicted as unmethylated (mean predicted probability < 0.5)

## How to apply

Load the balanced training sample file (13-mer sequence plus 13x16 normalized signal features per cytosine, produced by deepsignal_plant extract), then split the current set into two independent folds. Per iteration, train the BRNN-with-LSTM model on each fold (Adam optimizer, cross-entropy loss) and cross-predict the held-out fold, repeating the two-fold cross prediction for R=3 rounds. Remove positive (methylated) samples whose mean predicted methylation probability mean(score(s)) < 0.5, re-run the k-mer balancing (Algorithm 1: balance k-mer types and per-k-mer read counts between methylated and unmethylated sets, cap negative count at |Spos|), and re-train. Iterate until fewer than 1% of methylated samples are predicted unmethylated (>=99% retention) or the maximum of I=10 iterations is reached, then write the retained balanced positive/negative denoised training file for deepsignal_plant train. Rationale: methylation labels from bisulfite (methylated = >=5 reads and >=0.9 methylation frequency) include noisy positives; cross-prediction by the model itself identifies samples whose signal contradicts the label.

## Related tools

- **DeepSignal-plant** (Implements the iterative denoising algorithm (Algorithm 2) and the BiLSTM model used for two-fold cross prediction of training samples) — https://github.com/PengNi/deepsignal-plant
- **PyTorch** (Framework for training the BRNN-with-LSTM model (Adam optimizer, cross-entropy loss) used inside each denoising iteration)

## Examples

```
python denoise_samples.py --input fast5s.C.features.balanced.tsv --output fast5s.C.features.denosied.tsv --iterations 10 --rounds 3 --epochs 3 --prob_threshold 0.5
```

## Evaluation signals

- Iterative loop terminates with >=99% sample retention (fewer than 1% of methylated samples predicted as unmethylated) or after I=10 iterations.
- Removed samples are exclusively positive (methylated) ones with mean predicted methylation probability mean(score(s)) < 0.5.
- After each removal round, positive and negative subsets cover identical k-mer type sets with equal per-k-mer read counts and negative count <= positive count (Algorithm 1 invariant).
- Model trained on the denoised set reproduces the per-site Pearson correlation gains: 0.8464 -> 0.9122 for CHG and 0.3696 -> 0.7840 for CHH against bisulfite replicates (Fig. 1d).
- Output file is parseable TSV preserving the 11-column schema (chrom, pos, strand, pos_in_strand, readname, read_strand, k_mer, signal_means, signal_stds, signal_lens, raw_signals, methy_label).

## Limitations

- Gold-standard labels depend on bisulfite sequencing, so model performance is bounded by bisulfite quality; CHG/CHH methylated labels use a union of high-confidence sites from three replicates, a potential label-noise source.
- Hyperparameter tuning details are deferred to supplementary materials (Supplementary Note 3), limiting full reproducibility.
- Training set is capped at ~20 million samples per motif (half positive, half negative); large datasets require subsampling before denoising.

## Evidence

- [discussion] if mean(score(s)) < 0.5 thenSpos.remove(s): "if mean(score(s)) < 0.5 thenSpos.remove(s)"
- [discussion] denoising defaults: "In the denoising algorithm, we set I = 10, R = 3, E = 3 as default"
- [results] two-fold cross prediction removal mechanism: [evidence span withheld — anchor status: not_found]
- [results] correlation gains from balancing and denoising: [evidence span withheld — anchor status: not_found]
- [discussion] training sample cap: "we randomly subsample at most 20 million (half positive and half negative) samples as the training dataset for each motif"
- [intro] sample selection strategy objective: "Therefore, we develop a sample selection strategy to balance and denoise training samples, which can significantly improve the performances of the trained models, especially for CHH and CHG"
