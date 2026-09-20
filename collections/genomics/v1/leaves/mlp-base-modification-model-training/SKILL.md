---
name: mlp-base-modification-model-training
description: Use when you have filtered PacBio CCS BAMs with kinetic tags (fi, ri,
  fp, rp) and need to train a supervised k-mer MLP to classify modified-vs-canonical
  reads or call Z/A at single-nucleotide resolution.
when_to_use_negative:
- Input BAMs lack kinetic tags (fi, ri, fp, rp) or fail the Median_PW/Median_IPD and
  >=3-pass quality filters, so no reliable training features exist
- The goal is detecting modification types whose kinetic signatures are not represented
  in training data (e.g. hybrid reads with diluted signatures for the SVM read classifier,
  which is optimized for fully substituted dZ-genomes)
- No labeled ground-truth modified/canonical training reads are available — supervised
  MLP training is the wrong choice
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Z-Calling
  role: Toolkit providing filter_bam.py for read filtering, PyTorch MLP training for
    k-mer kinetic-feature classification, and TorchScript model export used with z-calling-base
    (-k 21 or -k 11)
  repo: https://github.com/xiaochuanle/Z-Calling
- name: PacBio Circular Consensus Sequencing (CCS)
  role: Source of HiFi reads with preserved PW/IPD kinetic signals (generated with
    --keep-kinetics) used as MLP training features
- name: PyTorch
  role: CPU-based framework for training the MLP classifiers and exporting scripted
    TorchScript checkpoints
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1038/s42003-026-09849-8
    title: 'Z-Calling: a tool for A/Z (2,6-diaminopurine) base calling and dZ-DNA
      detection using PacBio HiFi reads'
  source_content_hash: sha256:9d7b77b3d12420bc68f1d7804ec1b30ed8ffad4ef0efcd10d4aa77f44b3064ae
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
  - PacBio Circular Consensus Sequencing (CCS)
  - PyTorch
  - Z-Calling
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# mlp-base-modification-model-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train PyTorch multi-layer perceptron (MLP) classifiers on PacBio CCS kinetic features (pulse width and inter-pulse duration over k-mer contexts) to discriminate modified (dZ/Z) from canonical (dA/A) bases and reads, exporting TorchScript checkpoints for downstream base calling with Z-Calling.

## When to use

Use when you have filtered PacBio CCS BAMs with kinetic tags (fi, ri, fp, rp) and need to train a supervised k-mer MLP to classify modified-vs-canonical reads or call Z/A at single-nucleotide resolution.

## When NOT to use

- Input BAMs lack kinetic tags (fi, ri, fp, rp) or fail the Median_PW/Median_IPD and >=3-pass quality filters, so no reliable training features exist
- The goal is detecting modification types whose kinetic signatures are not represented in training data (e.g. hybrid reads with diluted signatures for the SVM read classifier, which is optimized for fully substituted dZ-genomes)
- No labeled ground-truth modified/canonical training reads are available — supervised MLP training is the wrong choice

## Inputs

- Filtered PacBio CCS BAM files with kinetic tags (fi, ri, fp, rp), Median_PW/Median_IPD between 0.3 and the 99th percentile, fn >= 3 and rn >= 3
- Labeled training reads: dZ-DNA (e.g. PCR-substituted E. coli) versus canonical (full-dA) controls, plus optional hybrid HgaI A/Z-coexisting data (fruit fly HGAI)
- k-mer context windows containing a single A/Z target base

## Outputs

- TorchScript model checkpoints (e.g. model/k21-full-ZA/scripted_m21.pth, model/k11-mixed-AZ/scripted_m11.pth)
- AUC and F1 evaluation table (TSV) per test dataset and k-mer size
- AUC and CPU time versus k plot justifying the chosen k-mer size

## How to apply

First filter raw CCS BAMs with filter_bam.py, keeping reads with kinetic tags (fi, ri, fp, rp), discarding reads with Median_PW/Median_IPD < 0.3 or above the 99th percentile, and requiring at least 3 passes on both forward (fn >= 3) and reverse (rn >= 3) strands. Build k-mer feature matrices from the PW/IPD signals for each odd k-mer length (e.g. 7-25) and train a PyTorch MLP (CPU) on labeled dZ-DNA versus canonical training reads; restrict k-mers containing a single A/Z target base to avoid interference among adjacent A/Zs, and use datasets with at least 60,000 reads. Select the k-mer size by plotting AUC and CPU training/calling time versus k — the article selected k=21 for full-substitution dZ/canonical read classification and k=11 for the mixed A/Z single-nucleotide model trained on combined E. coli dZ/canonical and fruit fly HgaI data. Export each trained model as a TorchScript checkpoint (e.g. model/k21-full-ZA/scripted_m21.pth used with -k 21, model/k11-mixed-AZ/scripted_m11.pth with -k 11) and evaluate held-out test BAMs by computing read-level AUC and F1 scores.

## Related tools

- **Z-Calling** (Toolkit providing filter_bam.py for read filtering, PyTorch MLP training for k-mer kinetic-feature classification, and TorchScript model export used with z-calling-base (-k 21 or -k 11)) — https://github.com/xiaochuanle/Z-Calling
- **PacBio Circular Consensus Sequencing (CCS)** (Source of HiFi reads with preserved PW/IPD kinetic signals (generated with --keep-kinetics) used as MLP training features)
- **PyTorch** (CPU-based framework for training the MLP classifiers and exporting scripted TorchScript checkpoints)

## Examples

```
python py filter_bam.py -b <RawBAM> -o <FilteredBAM>; build/z-calling-base -k 21 -t 16 <Input_BAM> model/k21-full-ZA/scripted_m21.pth <Output_BAM>
```

## Evaluation signals

- Held-out dZ-DNA/canonical read classification AUC falls in the reported 0.94-0.96 range (k21-full-ZA: 0.9646-0.9796) with F1 scores within 0.85-0.92 (0.9045-0.9384)
- k11-mixed-AZ model achieves HGAI AUC 0.942-0.949 and F1 0.868-0.878, while retaining Revio/Sequel II dZ/canonical AUC 0.951-0.952 and F1 0.864-0.894
- Mixed-strategy model clearly exceeds the k11-full-ZA baseline trained only on dZ/canonical data (HGAI AUC 0.609-0.622, F1 0.569-0.586)
- Evaluation table exists as a TSV and AUC/CPU-time versus k supports the selected k-mer size (k=21 for full-ZA, k=11 for mixed-AZ)
- Training datasets meet the quality threshold of at least 60,000 reads with forward/reverse pass numbers >= 3

## Limitations

- The SVM read classifier is optimized for fully substituted dZ-genomes and misses hybrid reads with diluted kinetic signatures
- Subtle Z-induced kinetic shifts (0.5-1.5-fold) overlap substantially with canonical Adenine variance due to inherent PacBio kinetic measurement noise, limiting hybrid sample quantification
- Detailed command lines and custom scripts are deferred to Supplementary Note 1 rather than the main text
- Lack of 'ground truth' datasets containing multiple verified modifications at defined loci hinders extension to a unified multi-modification detection tool

## Evidence

- [results] filter_bam.py kinetic tags: [evidence span withheld — anchor status: not_found]
- [results] k-mer selection: [evidence span withheld — anchor status: not_found]
- [results] mixed model performance: [evidence span withheld — anchor status: not_found]
- [readme] readme models: [evidence span withheld — anchor status: not_found]
