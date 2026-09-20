---
name: class-imbalance-subsampling-for-dl-training
description: Use when constructing a training set from Nanopore signal features where
  methylated cytosines (especially CpG, CHG, CHH contexts) are far outnumbered by
  unmethylated ones and per-k-mer coverage is skewed between classes.
when_to_use_negative:
- The two classes already share the same k-mer type sets with equal per-k-mer counts
  — re-subsampling would discard usable training data
- The goal is methylation calling or per-site frequency estimation on new reads (use
  deepsignal_plant call_mods / call_freq instead), not training-set construction
- Samples are labeled by something other than methylation status (methy_label 0/1),
  e.g. already denoised model outputs, where subsampling on k-mer identity is inappropriate
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: DeepSignal-plant
  role: tool whose training pipeline this balancing algorithm is part of; balanced
    samples feed the BiLSTM/BRNN training for 5mC detection
  repo: github.com/PengNi/deepsignal-plant
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-26278-9
    title: Genome-wide detection of cytosine methylations in plant from Nanopore data
      using deep learning
  source_content_hash: sha256:81dbaf16d00a0cd4a87e0b1f1661c77a5fd3c3bff7729ce30bea27972168ec30
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepSignal-plant
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# class-imbalance-subsampling-for-dl-training

## Summary

Balances a DeepSignal-plant training set of Nanopore signal features by first matching k-mer (13-mer) types between methylated and unmethylated samples, then equalizing per-k-mer read counts across the two classes, so the BRNN learns methylation status rather than k-mer identity.

## When to use

Use when constructing a training set from Nanopore signal features where methylated cytosines (especially CpG, CHG, CHH contexts) are far outnumbered by unmethylated ones and per-k-mer coverage is skewed between classes.

## When NOT to use

- The two classes already share the same k-mer type sets with equal per-k-mer counts — re-subsampling would discard usable training data
- The goal is methylation calling or per-site frequency estimation on new reads (use deepsignal_plant call_mods / call_freq instead), not training-set construction
- Samples are labeled by something other than methylation status (methy_label 0/1), e.g. already denoised model outputs, where subsampling on k-mer identity is inappropriate

## Inputs

- extracted-features TSV (e.g. fast5s.C.features.tsv) with methy_label column and 13-mer k_mer column, produced after Tombo resquiggle and median-shift/MAD signal normalization

## Outputs

- balanced, tab-delimited training dataset preserving the original 11-column schema with matched k-mer type sets and equal per-k-mer read counts between methylated (methy_label=1) and unmethylated (methy_label=0) subsets

## How to apply

Load the extracted-features TSV (e.g. fast5s.C.features.tsv with 11 columns: chrom, pos, strand, pos_in_strand, readname, read_strand, k_mer, signal_means, signal_stds, signal_lens, raw_signals, methy_label) into pandas and split rows by methy_label (0/1) into Spos and Sneg. Compute Kcomm, the k-mer types of the 13-mer k_mer column shared by both sets, and Kdiff, the k-mer types present in only one set. For k-mers in Kcomm, randomly subsample the over-represented class so per-k-mer read counts match between positives and negatives; drop samples whose k-mer type has no counterpart in the opposite class (Kdiff handling) and cap the negative count at |Spos| to enforce class balance. The rationale is that the paper's unbalanced plant data (few fully methylated CHH/CHG cytosines) would otherwise bias the BiLSTM toward k-mer identity rather than methylation signal. Optionally bound the final set per motif (the paper subsamples at most 20 million samples, half positive and half negative) before downstream denoising.

## Related tools

- **DeepSignal-plant** (tool whose training pipeline this balancing algorithm is part of; balanced samples feed the BiLSTM/BRNN training for 5mC detection) — github.com/PengNi/deepsignal-plant

## Examples

```
import pandas as pd; df = pd.read_csv('fast5s.C.features.tsv', sep='\t'); Spos = df[df.methy_label == 1]; Sneg = df[df.methy_label == 0]
```

## Evaluation signals

- Output file exists and parses as tab-delimited TSV retaining the 11-column schema (chrom, pos, strand, pos_in_strand, readname, read_strand, k_mer, signal_means, signal_stds, signal_lens, raw_signals, methy_label)
- Positive and negative subsets cover identical k-mer type sets after balancing (Kdiff removed)
- Per-k-mer read counts are equal between methylated and unmethylated subsets for every shared k-mer
- Negative sample count does not exceed positive sample count (|Sneg| <= |Spos|)
- Per motif, the final training set is bounded by the 20-million cap with half positive and half negative samples

## Limitations

- High-confidence methylated sites are scarce, especially for CHH, so balancing drops samples whose k-mer types lack a counterpart in the opposite class, reducing training data
- Training labels derive from bisulfite sequencing gold standards, so balanced-sample quality is bounded by bisulfite quality and label-noise (e.g. replicate-union CHG/CHH high-confidence methylated sites in A. thaliana)
- Balancing is performed before/with the iterative denoising algorithm (I=10, R=3, E=3); subsampling alone does not remove mislabeled positives with mean predicted methylation probability < 0.5

## Evidence

- [discussion] we balance the types of k-mer in positive and negative samples, as well as the number of reads covered for each k-mer: "we balance the types of k-mer in positive and negative samples, as well as the number of reads covered for each k-mer"
- [intro] Therefore, we develop a sample selection strategy to balance and denoise training samples, which can significantly improve the performances of the trained models, especially for CHH and CHG: "Therefore, we develop a sample selection strategy to balance and denoise training samples, which can significantly improve the performances of the trained models, especially for CHH and CHG"
- [discussion] we randomly subsample at most 20 million (half positive and half negative) samples as the training dataset for each motif: "we randomly subsample at most 20 million (half positive and half negative) samples as the training dataset for each motif"
