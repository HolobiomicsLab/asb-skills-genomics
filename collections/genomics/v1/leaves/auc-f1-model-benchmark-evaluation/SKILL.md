---
name: auc-f1-model-benchmark-evaluation
description: Use when comparing trained Z-Calling model configurations (k-mer size
  or full-ZA vs mixed-AZ training data) on filtered PacBio CCS reads, where per-dataset
  AUC and F1 scores are needed to justify a model choice.
when_to_use_negative:
- Input BAMs lack the kinetic tags (fi, ri, fp, rp) or fail the filter_bam.py criteria
  (Median_PW/Median_IPD < 0.3 or > 99th percentile, fn/rn < 3), so no ZP probabilities
  can be scored
- The goal is base-level quality or error assessment (e.g., Merqury QVs, minimap2/Bcftools
  error typing) rather than dZ-DNA vs canonical or A/Z discrimination benchmarking
- Evaluating a model on hybrid A/Z sites without restricting context k-mers to a single
  A/Z target base, which the paper warns causes interference among adjacent A/Zs
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: Z-Calling
  role: MLP/SVM toolkit whose z-calling-base and z-calling-read outputs provide the
    read- and base-level predictions scored by AUC and F1
  repo: https://github.com/xiaochuanle/Z-Calling
- name: PyTorch
  role: CPU-only framework used to train and export the k-mer MLP models as TorchScript
    checkpoints (scripted_m21.pth, scripted_m11.pth)
- name: PacBio CCS
  role: Sequencing modality whose PW/IPD kinetic signals in CCS BAM reads are the
    features being benchmarked
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1038/s42003-026-09849-8
    title: 'Z-Calling: a tool for A/Z (2,6-diaminopurine) base calling and dZ-DNA
      detection using PacBio HiFi reads'
  source_content_hash: sha256:9294bad9f4d3b0e81701de47ab142ad3932e73369e8b2600afb76eb8faf573c6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xiaochuanle/Z-Calling
  repo_url: https://github.com/xiaochuanle/Z-Calling
  tools:
  - PacBio CCS
  - PyTorch
  - Z-Calling
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# auc-f1-model-benchmark-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks Z-Calling MLP base-calling models by aggregating read-level dZ-DNA vs canonical (and single-nucleotide A/Z) predictions and computing AUC and F1 on held-out PacBio HiFi test sets. It is used both to sweep k-mer sizes (selecting k=21) and to compare training strategies (k11-mixed-AZ vs k11-full-ZA).

## When to use

Use when comparing trained Z-Calling model configurations (k-mer size or full-ZA vs mixed-AZ training data) on filtered PacBio CCS reads, where per-dataset AUC and F1 scores are needed to justify a model choice.

## When NOT to use

- Input BAMs lack the kinetic tags (fi, ri, fp, rp) or fail the filter_bam.py criteria (Median_PW/Median_IPD < 0.3 or > 99th percentile, fn/rn < 3), so no ZP probabilities can be scored
- The goal is base-level quality or error assessment (e.g., Merqury QVs, minimap2/Bcftools error typing) rather than dZ-DNA vs canonical or A/Z discrimination benchmarking
- Evaluating a model on hybrid A/Z sites without restricting context k-mers to a single A/Z target base, which the paper warns causes interference among adjacent A/Zs

## Inputs

- Filtered PacBio CCS BAM files with kinetic tags (fi, ri, fp, rp) from test datasets (fruit fly Revio and Sequel II, E. coli Revio, fruit fly HGAI HgaI datasets)
- TorchScript MLP checkpoints: model/k21-full-ZA/scripted_m21.pth (used with -k 21) and model/k11-mixed-AZ/scripted_m11.pth (used with -k 11)
- Training feature matrices of PW/IPD kinetic signals per context k-mer (odd k in 7-25 for the sweep; 11-bp contexts with a single A/Z for the mixed model)
- Pre-trained baseline model model/k11-full-ZA (k11 MLP trained only on dZ-DNA/canonical data)

## Outputs

- Evaluation table (TSV) of AUC and F1 per model and per held-out dataset
- Plot of AUC and CPU training/calling time versus k-mer size (7-25) justifying k=21
- Output BAMs with per-base Z probabilities stored in MM/ML and ZP tags
- Read-level classification scores for dZ-DNA vs canonical on each test set

## How to apply

First filter raw BAMs with filter_bam.py, keeping reads with kinetic tags (fi, ri, fp, rp), discarding reads with Median_PW/Median_IPD < 0.3 or above the 99th percentile, and requiring at least 3 passes on both strands (fn >= 3, rn >= 3). For the k-mer sweep, build PW/IPD kinetic feature matrices at each odd k in the 7-25 range, train a PyTorch (CPU) MLP on E. coli dZ-DNA (full-Z) vs canonical (full-dA) reads, and compute AUC plus CPU training/calling time per k, plotting AUC and time vs k to justify k=21 as the balance of model strength and efficiency. Export the selected checkpoints as TorchScript artifacts (model/k21-full-ZA/scripted_m21.pth for -k 21, model/k11-mixed-AZ/scripted_m11.pth for -k 11), run build/z-calling-base to write per-base Z probabilities into MM/ML and ZP tags, then aggregate read-level predictions on each held-out test set and compute AUC and F1 for dZ-DNA vs canonical classification. For training-strategy comparison, evaluate the mixed model (trained on combined E. coli dZ-DNA/canonical plus fruit fly HgaI data, restricted to context k-mers with a single A/Z target base) against the full-ZA-only baseline on the same HGAI datasets to quantify the improvement.

## Related tools

- **Z-Calling** (MLP/SVM toolkit whose z-calling-base and z-calling-read outputs provide the read- and base-level predictions scored by AUC and F1) — https://github.com/xiaochuanle/Z-Calling
- **PyTorch** (CPU-only framework used to train and export the k-mer MLP models as TorchScript checkpoints (scripted_m21.pth, scripted_m11.pth))
- **PacBio CCS** (Sequencing modality whose PW/IPD kinetic signals in CCS BAM reads are the features being benchmarked)

## Examples

```
build/z-calling-base -k 21 -t 16 <Input_BAM> model/k21-full-ZA/scripted_m21.pth <Output_BAM>
```

## Evaluation signals

- k21-full-ZA model achieves AUCs of 0.9646-0.9796 and F1 scores of 0.9045-0.9384 across fruit fly (Revio and Sequel II) and E. coli (Revio) test sets
- k11-mixed-AZ model achieves HGAI AUCs of 0.942-0.949 and F1 of 0.868-0.878, and retains Revio/Sequel II AUCs of 0.951-0.952 and F1 of 0.864-0.894 on dZ-DNA/canonical data
- The k11-full-ZA baseline (dZ-DNA/canonical training only) scores AUC 0.609-0.622 on HGAI datasets and must be clearly exceeded by the mixed model
- The evaluation table exists as a TSV and computed AUC values fall within the reported 0.94-0.96 range and F1 within 0.85-0.92 for the Z base calling module
- The exported TorchScript artifacts exist at model/k21-full-ZA/scripted_m21.pth and model/k11-mixed-AZ/scripted_m11.pth and are invoked with the matching -k 21 / -k 11 arguments

## Limitations

- The subtle Z-induced kinetic shift (0.5-1.5-fold) overlaps substantially with canonical adenine variance, limiting hybrid sample quantification
- The k11-full-ZA model trained only on full-dZ/canonical data performs poorly on HgaI hybrid data (AUC 0.609-0.622); models must be retrained with mixed data for A/Z coexisting samples
- Reads shorter than the min_len (>= 500) criterion or failing length criteria remain unclassified, so per-dataset coverage of predictions is incomplete

## Evidence

- [abstract] Its A/Z base-calling module demonstrates robust performance, with AUC scores of 0.942–0.952 across diverse DNA sequence contexts: "Its A/Z base-calling module demonstrates robust performance, with AUC scores of 0.942–0.952 across diverse DNA sequence contexts"
- [readme] Across multiple datasets, the Z base calling module achieves **AUCs of 0.94–0.96** and **F1 scores of 0.85–0.92**.: "Across multiple datasets, the Z base calling module achieves **AUCs of 0.94–0.96** and **F1 scores of 0.85–0.92**."
- [methods] Reads with Median_PW / Median_IPD < 0.3 or > 99th percentile are discarded.: "Reads with Median_PW / Median_IPD < 0.3 or > 99th percentile are discarded."
- [discussion] To avoid interference among adjacent A/Zs, only context k-mers containing one A/Z (target base) were analyzed: "To avoid interference among adjacent A/Zs, only context k-mers containing one A/Z (target base) were analyzed"
