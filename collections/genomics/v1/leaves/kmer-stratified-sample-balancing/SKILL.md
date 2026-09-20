---
name: kmer-stratified-sample-balancing
description: Use when constructing a deep-learning training set from Nanopore signal
  features (e.g. fast5s.C.features.tsv) where methylated/unmethylated cytosines are
  heavily skewed across k-mer contexts (CpG, CHG, CHH) and the classifier must not
  confound methylation with sequence context.
when_to_use_negative:
- The input is already a balanced feature table — re-balancing would discard valid
  samples for no benefit.
- The goal is methylation calling on new reads (use `call_mods`/`call_freq`), not
  training-set construction.
- The data are raw FAST5/POD5 signal files that have not yet been resquiggled (Tombo
  1.5.1) and converted into an extracted-features TSV.
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0102
tools:
- name: DeepSignal-plant
  role: Provides the sample-balancing/denoising strategy (Algorithm 1) that balances
    k-mer types and per-k-mer read counts before BRNN/LSTM training
  repo: https://github.com/PengNi/deepsignal-plant
- name: pandas
  role: Loads and splits the extracted-features TSV by methy_label and subsamples
    per-k-mer rows
  repo: https://pandas.pydata.org/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-26278-9
    title: Genome-wide detection of cytosine methylations in plant from Nanopore data
      using deep learning
  source_content_hash: sha256:ec759a83c174968ff26e1ddc72045431e7ee98d0fc2b5a63d867026a7b312044
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PengNi/deepsignal-plant
  tools:
  - DeepSignal-plant
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# k-mer-stratified training-sample balancing

## Summary

Balances DeepSignal-plant's training dataset by first equalizing k-mer types between methylated and unmethylated cytosine samples, then equalizing per-k-mer read counts, so the BiLSTM model learns methylation status from Nanopore signal features rather than k-mer identity.

## When to use

Use when constructing a deep-learning training set from Nanopore signal features (e.g. fast5s.C.features.tsv) where methylated/unmethylated cytosines are heavily skewed across k-mer contexts (CpG, CHG, CHH) and the classifier must not confound methylation with sequence context.

## When NOT to use

- The input is already a balanced feature table — re-balancing would discard valid samples for no benefit.
- The goal is methylation calling on new reads (use `call_mods`/`call_freq`), not training-set construction.
- The data are raw FAST5/POD5 signal files that have not yet been resquiggled (Tombo 1.5.1) and converted into an extracted-features TSV.

## Inputs

- Extracted-features TSV file (e.g. fast5s.C.features.tsv) with columns chrom, pos, strand, pos_in_strand, readname, read_strand, k_mer, signal_means, signal_stds, signal_lens, raw_signals, methy_label
- methy_label column (0 = unmethylated, 1 = methylated) used to split positive/negative sets

## Outputs

- Balanced tab-delimited training dataset preserving the original 11-column schema, with identical k-mer type sets and matched per-k-mer read counts between methylated and unmethylated subsets

## How to apply

Load the extracted-features TSV (fast5s.C.features.tsv, 12 columns: chrom, pos, strand, pos_in_strand, readname, read_strand, k_mer, signal_means, signal_stds, signal_lens, raw_signals, methy_label) with pandas and split rows by methy_label (0/1) into Spos and Sneg. Compute Kcomm (13-mer k-mer types shared by both sets) and Kdiff (k-mer types in only one set); for Kcomm members, randomly subsample the over-represented class so per-k-mer read counts match between positives and negatives. Drop samples whose k-mer type has no counterpart in the opposite class (Kdiff handling) and cap the negative count at |Spos| to enforce class balance. Write the balanced set as a tab-delimited file preserving the original 11-column schema. Rationale: methylated cytosines are rare (especially CHH), and balancing k-mer types plus per-k-mer read counts is part of DeepSignal-plant's sample selection strategy that significantly improves trained-model performance, especially for CHH and CHG.

## Related tools

- **DeepSignal-plant** (Provides the sample-balancing/denoising strategy (Algorithm 1) that balances k-mer types and per-k-mer read counts before BRNN/LSTM training) — https://github.com/PengNi/deepsignal-plant
- **pandas** (Loads and splits the extracted-features TSV by methy_label and subsamples per-k-mer rows) — https://pandas.pydata.org/

## Examples

```
import pandas as pd; df = pd.read_csv('fast5s.C.features.tsv', sep='\t'); spos, sneg = df[df.methy_label == 1], df[df.methy_label == 0]
```

## Evaluation signals

- Output file exists and is parseable as tab-delimited TSV with the original 11-column schema.
- Positive and negative subsets cover identical k-mer type sets after balancing (Kdiff is empty).
- Per-k-mer read counts are equal between methylated and unmethylated subsets for every k-mer in Kcomm.
- Negative sample count does not exceed positive sample count (class balance invariant).
- Total balanced set respects the per-motif cap of at most 20 million samples (half positive, half negative).

## Limitations

- Balancing drops samples whose k-mer type has no counterpart in the opposite class (Kdiff), so some genomic contexts are discarded entirely.
- Downstream model quality is bounded by the bisulfite-derived gold-standard labels used to define methylated/unmethylated samples.
- For CHG/CHH in A. thaliana, high-confidence methylated sites were taken as the union of three replicates (lower stringency), which can inject label noise that balancing cannot remove.

## Evidence

- [discussion] k-mer type and per-read-count balancing drives methylation learning: "we balance the types of k-mer in positive and negative samples, as well as the number of reads covered for each k-mer"
- [intro] motivation: unbalanced data harms CHG/CHH model performance: "we develop a sample selection strategy to balance and denoise training samples, which can significantly improve the performances of the trained models, especially for CHH and CHG"
- [discussion] training-set size cap per motif: "we randomly subsample at most 20 million (half positive and half negative) samples as the training dataset for each motif"
