---
name: imbalanced-sample-rebalancing
description: Use when building 5mC training sets from Nanopore signal features where
  methylated cytosines are rare and label-imbalanced — especially CHH/CHG contexts
  — and random subsampling degrades model correlation with bisulfite.
when_to_use_negative:
- Input is already a balanced, denoised training file with >=99% retention from a
  previous run — re-running the iterative loop would only re-predict and churn samples.
- Your goal is per-read/per-site modification calling on new reads (use deepsignal_plant
  call_mods / call_freq), not training-set construction.
- Working with non-plant Nanopore data where cytosine methylation is abundant (e.g.
  CpG-rich mammalian data with other callers like Megalodon); the extreme CHH imbalance
  this skill targets is plant-specific.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0102
tools:
- name: DeepSignal-plant
  role: Implements the balancing (Algorithm 1) and iterative denoising (Algorithm
    2) sample-selection strategy; its extract step produces the 13-mer/13x16 feature
    files, and train consumes the rebalanced output.
  repo: https://github.com/PengNi/deepsignal-plant
- name: PyTorch
  role: Backend for the BRNN-with-LSTM model (PyTorch >=1.2.0, <=1.11.0) whose cross-prediction
    scores drive removal of false methylated samples during denoising iterations.
  repo: https://pytorch.org/
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-26278-9
    title: Genome-wide detection of cytosine methylations in plant from Nanopore data
      using deep learning
  source_content_hash: sha256:8f927bcfca4f4ba11afe91d93057503cb47210b730985869ca6fcd8905d11db6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PengNi/deepsignal-plant
  tools:
  - DeepSignal-plant
  - Guppy
  - PyTorch
  - Tombo
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# imbalanced-sample-rebalancing

## Summary

A sample-selection strategy for DeepSignal-plant that balances positive (methylated) and negative (unmethylated) k-mer training samples and read coverage per k-mer (Algorithm 1), and iteratively removes false methylated samples (Algorithm 2, I=10, R=3, E=3) so the BiLSTM 5mC model trains on clean, balanced data.

## When to use

Use when building 5mC training sets from Nanopore signal features where methylated cytosines are rare and label-imbalanced — especially CHH/CHG contexts — and random subsampling degrades model correlation with bisulfite.

## When NOT to use

- Input is already a balanced, denoised training file with >=99% retention from a previous run — re-running the iterative loop would only re-predict and churn samples.
- Your goal is per-read/per-site modification calling on new reads (use deepsignal_plant call_mods / call_freq), not training-set construction.
- Working with non-plant Nanopore data where cytosine methylation is abundant (e.g. CpG-rich mammalian data with other callers like Megalodon); the extreme CHH imbalance this skill targets is plant-specific.

## Inputs

- Balanced/refined training sample TSV with 13-mer sequence and 13x16 signal features per targeted cytosine (from deepsignal_plant extract)
- Bisulfite-derived gold-standard labels (methylated: >=5 reads and >=0.9 methylation frequency; unmethylated: >=5 reads and zero methylation frequency)
- Trained/initial DeepSignal-plant BiLSTM model for cross-prediction scoring

## Outputs

- Denoised, k-mer-balanced training sample file (half positive, half negative, capped at 20 million samples per motif) usable by deepsignal_plant train
- Removed-sample log of positive samples with mean predicted methylation probability < 0.5

## How to apply

Start from 13-mer sequence plus 13x16 normalized signal features per targeted cytosine produced by deepsignal_plant extract. First, balance the types of k-mer in positive and negative samples as well as the number of reads covered for each k-mer (Algorithm 1), and randomly subsample at most 20 million samples (half positive, half negative) per motif. Then iteratively denoise: split the current set into two folds and, for R=3 rounds per iteration, train the BiLSTM (Adam, cross-entropy loss) on one fold and cross-predict the held-out fold; remove positive samples whose mean predicted methylation probability is below 0.5 (mean(score(s)) < 0.5). After each removal round, re-balance positive and negative samples per k-mer motif and re-cap read coverage per k-mer. Terminate when retention reaches at least 99% or after I=10 iterations (defaults I=10, R=3, E=3), yielding a balanced denoised file for deepsignal_plant train. The rationale is that denoising plus balancing raises per-site Pearson correlation with bisulfite from 0.8464 to 0.9122 (CHG) and 0.3696 to 0.7840 (CHH).

## Related tools

- **DeepSignal-plant** (Implements the balancing (Algorithm 1) and iterative denoising (Algorithm 2) sample-selection strategy; its extract step produces the 13-mer/13x16 feature files, and train consumes the rebalanced output.) — https://github.com/PengNi/deepsignal-plant
- **PyTorch** (Backend for the BRNN-with-LSTM model (PyTorch >=1.2.0, <=1.11.0) whose cross-prediction scores drive removal of false methylated samples during denoising iterations.) — https://pytorch.org/

## Examples

```
Spos = [s for s in Spos if mean(score(s)) >= 0.5]  # Algorithm 2 removal; then re-balance per k-mer and cap at 20M (half positive, half negative) per motif
```

## Evaluation signals

- Output training set contains at most 20 million samples per motif, split exactly half positive and half negative.
- After balancing, k-mer types and per-k-mer read coverage are matched between positive and negative samples.
- Each denoising iteration removes positive samples only where mean predicted methylation probability < 0.5; retention per iteration should reach at least 99% or the loop stops at I=10.
- Model trained on the rebalanced/denoised set must reproduce the reported Pearson-correlation gains vs. random selection: CHG 0.8464 -> 0.9122, CHH 0.3696 -> 0.7840 (Fig. 1d).
- Labels trace to high-confidence bisulfite criteria: >=5 reads with >=0.9 methylation frequency (methylated) or >=5 reads with zero methylation frequency (unmethylated).

## Limitations

- Gold-standard labels come from bisulfite sequencing, so model performance is bounded by bisulfite quality; incomplete conversion and repetitive-region dropout propagate into the balanced training set.
- For CHG and CHH in A. thaliana, high-confidence methylated sites are the union across three bisulfite replicates (lower stringency), introducing potential label noise that balancing cannot remove.
- The 20-million-sample cap and iterative loop bounds (I=10, R=3, E=3) are paper defaults; hyperparameter tuning details are deferred to Supplementary Note 3 and not present in the main text.

## Evidence

- [discussion] k-mer balancing: "we balance the types of k-mer in positive and negative samples, as well as the number of reads covered for each k-mer"
- [discussion] 20M subsample cap: "we randomly subsample at most 20 million (half positive and half negative) samples as the training dataset for each motif"
- [discussion] denoising removal rule: "if mean(score(s)) < 0.5 thenSpos.remove(s)"
- [discussion] denoising defaults: "In the denoising algorithm, we set I = 10, R = 3, E = 3 as default"
- [readme] tool scope: "deepsignal-plant applies BiLSTM to detect methylation from Nanopore reads. It is built on **Python3** and **PyTorch**."
