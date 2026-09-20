---
name: bisulfite-high-confidence-site-curation
description: 'Use when training or evaluating a nanopore CpG methylation model and
  you need trusted single-molecule labels: derive positive (fully methylated) and
  negative (fully unmethylated) high-confidence CpG site lists from BS-seq data at
  >=20x coverage to feed DeepBAM''s extract_hc_sites mode.'
when_to_use_negative:
- The goal is modification calling on already-trained models — use DeepBAM's extract_and_call_mods
  workflow instead of re-curating training sites
- No bisulfite-seq or fully methylated/unmethylated treated-DNA controls exist for
  the sample, so trustworthy positive/negative labels cannot be derived
- Targeting non-CpG methylation contexts — the curation and DeepBAM extraction are
  specified for CpG (motif_type CG) only
edam_topics:
- http://edamontology.org/topic_0160
tools:
- name: DeepBAM
  role: Provides the extract_hc_sites mode that consumes the curated high-confidence
    bisulfite site lists and extracts per-site signal features from POD5/BAM inputs
    for model training
  repo: https://github.com/huicongyao/DeepBAM
- name: DeepBAM
  role: Alternative/primary source repository for the DeepBAM tool used in this curation-and-extraction
    workflow
  repo: https://github.com/xiaochuanle/DeepBAM
- name: cnpy
  role: C++ library used by DeepBAM to write the extracted .npz feature files
  repo: https://github.com/rogersce/cnpy
- name: pod5-file-format
  role: C++ library for reading the POD5 nanopore signal files during high-confidence
    site feature extraction
  repo: https://github.com/nanoporetech/pod5-file-format
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae413
    title: 'DeepBAM: a high-accuracy single-molecule CpG methylation detection tool
      for Oxford nanopore sequencing'
  source_content_hash: sha256:5ab540315d45f2b99d56791be7f3539487f6e808a6fe78938274486f9422ae38
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/xiaochuanle/DeepBAM
  repo_url: https://github.com/huicongyao/DeepBAM
  tools:
  - DeepBAM
  - cnpy
  - pod5-file-format
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# bisulfite-high-confidence-site-curation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Curates high-confidence CpG sites from bisulfite-seq (BS-seq) treated DNAs — fully methylated positives and fully unmethylated negatives at sufficient read coverage — to serve as ground-truth labels for DeepBAM's extract_hc_sites feature extraction and Bi-LSTM methylation-model training on nanopore R10.4 data.

## When to use

Use when training or evaluating a nanopore CpG methylation model and you need trusted single-molecule labels: derive positive (fully methylated) and negative (fully unmethylated) high-confidence CpG site lists from BS-seq data at >=20x coverage to feed DeepBAM's extract_hc_sites mode.

## When NOT to use

- The goal is modification calling on already-trained models — use DeepBAM's extract_and_call_mods workflow instead of re-curating training sites
- No bisulfite-seq or fully methylated/unmethylated treated-DNA controls exist for the sample, so trustworthy positive/negative labels cannot be derived
- Targeting non-CpG methylation contexts — the curation and DeepBAM extraction are specified for CpG (motif_type CG) only

## Inputs

- BS-seq derived high-confidence CpG site lists from fully methylated treated DNA (positives)
- BS-seq derived high-confidence CpG site lists from fully unmethylated treated DNA (negatives)
- Nanopore R10.4 POD5 signal files (e.g., HG002, Beta vulgaris)
- Sorted BAM alignments against matching reference genomes
- Reference genome FASTA files
- Coverage threshold parameter (>=20x) and motif_type CG

## Outputs

- Curated positive/negative high-confidence CpG site lists at >=20x coverage used by extract_hc_sites
- Per-site signal feature files stored as ${pod5filename}.npz containing site information and data arrays per k-mer context for model training

## How to apply

Start from BS-seq data on the same biological material as the nanopore reads (e.g., HG002 human and Beta vulgaris plant datasets) and select only CpG sites (motif_type CG) with coverage >=20x, partitioning them into fully methylated (positive) and fully unmethylated (negative) classes based on the treated-DNA controls, since fully-methylated and fully-unmethylated treated DNAs expose the largest performance variances in methylation callers. Pass these site lists, together with the sorted BAM alignments, reference genomes, and POD5 signal files, into DeepBAM's extract_hc_sites mode so it stores per-site signal features as ${pod5filename}.npz files containing site information and data arrays per k-mer context (e.g., kmer_size 13, 21, 51, 101). These labeled features then train the Bi-LSTM model via py/train_lstm.py under 10-fold cross-validation, so label purity at this curation stage directly bounds achievable AUC (~0.9793 with 101-mer, ~97.80% mean AUROC overall).

## Related tools

- **DeepBAM** (Provides the extract_hc_sites mode that consumes the curated high-confidence bisulfite site lists and extracts per-site signal features from POD5/BAM inputs for model training) — https://github.com/huicongyao/DeepBAM
- **DeepBAM** (Alternative/primary source repository for the DeepBAM tool used in this curation-and-extraction workflow) — https://github.com/xiaochuanle/DeepBAM
- **cnpy** (C++ library used by DeepBAM to write the extracted .npz feature files) — https://github.com/rogersce/cnpy
- **pod5-file-format** (C++ library for reading the POD5 nanopore signal files during high-confidence site feature extraction) — https://github.com/nanoporetech/pod5-file-format

## Examples

```
git clone https://github.com/huicongyao/DeepBAM.git  # then run DeepBAM extract_hc_sites on POD5/BAM with BS-seq HC CpG lists at >=20x coverage, motif_type CG
```

## Evaluation signals

- Every curated site is a CpG (motif_type CG) drawn from the BS-seq high-confidence lists at >=20x coverage
- Positives come only from fully methylated treated DNA and negatives only from fully unmethylated treated DNA; mixed-methylation sites are excluded from training labels
- extract_hc_sites produces one ${pod5filename}.npz per POD5 input containing site information and data arrays per k-mer context
- Downstream Bi-LSTM training on these labels reaches test AUCs consistent with the paper's ablation (0.9560/0.9647/0.9750/0.9793 for 13/21/51/101-mers) and ~97.80% mean AUROC
- Validation confirms each site list and resulting npz covers the expected k-mer lengths before model export to traced_script_module

## Limitations

- Label quality is bounded by the bisulfite controls: only fully methylated and fully unmethylated treated DNAs provide unambiguous positives/negatives, so partially methylated native contexts are not directly labeled
- Coverage cutoff of >=20x discards lower-coverage CpGs, reducing the training site universe
- The paper notes ongoing optimization of DeepBAM is necessary, implying curated-label performance is not final; quantitative speed/memory comparisons were deferred to supplementary material not present in the text

## Evidence

- [methods] extract features for model training with high confident bisulfite data: "extract features for model training with high confident bisulfite data"
- [discussion] using treated DNAs (fully-methylated and fully-unmethylated) revealed significant performance variances: "using treated DNAs (fully-methylated and fully-unmethylated) revealed significant performance variances"
- [intro] DeepBAM exhibits improved (up to 7.36% AUC enhancement) and more stable (≥0.956 AUCs) single-molecule: "DeepBAM exhibits improved (up to 7.36% AUC enhancement) and more stable (≥0.956 AUCs) single-molecule"
- [methods] git clone https://github.com/huicongyao/DeepBAM.git: "git clone https://github.com/huicongyao/DeepBAM.git"
