---
name: kmer-size-hyperparameter-sweep
description: Use when configuring a kinetic-signal k-mer classifier (e.g. Z-Calling
  full-substitution model) and you must justify which k-mer size trades model strength
  against compute cost before freezing a production artifact.
when_to_use_negative:
- You are tuning the single-nucleotide A/Z model — that uses the k11-mixed-AZ artifact
  (model/k11-mixed-AZ/scripted_m11.pth, run with -k 11), not the k21 full-substitution
  sweep.
- Input reads lack PacBio kinetic tags (fi, ri, fp, rp) or have Median_PW/Median_IPD
  outside the accepted range; the sweep cannot be trained without valid kinetics.
- The task is per-base Z probability extraction from an already-chosen model — use
  z-calling-base/z-bam2txt directly rather than re-sweeping k.
edam_operation: http://edamontology.org/operation_2424
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_3524
tools:
- name: Z-Calling
  role: Provides filter_bam.py for BAM filtering and build/z-calling-base for evaluating
    the selected k-mer model; ships the pre-trained model/k21-full-ZA/scripted_m21.pth
    artifact run with -k 21
  repo: https://github.com/xiaochuanle/Z-Calling
- name: PyTorch
  role: Trains the per-k MLP classifiers on PW/IPD k-mer feature matrices and exports
    the selected model as TorchScript
- name: PacBio CCS
  role: Source sequencing data; CCS BAMs generated with --keep-kinetics preserve the
    PW and IPD signals used as k-mer features
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-026-09849-8
    title: 'Z-Calling: a tool for A/Z (2,6-diaminopurine) base calling and dZ-DNA
      detection using PacBio HiFi reads'
  source_content_hash: sha256:5bfce12a1303893d50f5e1fc13b322e94165e35199a3d033e494ffb500aeacb2
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
  edam_operation: http://edamontology.org/operation_2424
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# kmer-size-hyperparameter-sweep

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Sweeps odd k-mer sizes (7–25 bp) for PacBio PW/IPD kinetic feature matrices and trains a PyTorch MLP per k to pick the k-mer size that balances classification AUC against CPU training/calling cost, resulting in k=21 for the full-substitution dZ-DNA/canonical (full-Z vs full-dA) model.

## When to use

Use when configuring a kinetic-signal k-mer classifier (e.g. Z-Calling full-substitution model) and you must justify which k-mer size trades model strength against compute cost before freezing a production artifact.

## When NOT to use

- You are tuning the single-nucleotide A/Z model — that uses the k11-mixed-AZ artifact (model/k11-mixed-AZ/scripted_m11.pth, run with -k 11), not the k21 full-substitution sweep.
- Input reads lack PacBio kinetic tags (fi, ri, fp, rp) or have Median_PW/Median_IPD outside the accepted range; the sweep cannot be trained without valid kinetics.
- The task is per-base Z probability extraction from an already-chosen model — use z-calling-base/z-bam2txt directly rather than re-sweeping k.

## Inputs

- Filtered PacBio HiFi/CCS BAM files with kinetic tags (fi, ri, fp, rp), Median_PW/Median_IPD within [0.3, 99th percentile], fn >= 3 and rn >= 3
- Training BAM: E. coli dZ-DNA (full-Z) vs canonical (full-dA) reads
- Held-out test BAMs: fruit fly (Revio and Sequel II) and E. coli (Revio)

## Outputs

- Per-k evaluation table (TSV) of AUC and CPU training/calling time for odd k in 7–25
- AUC and CPU-time-vs-k plot justifying k=21
- TorchScript model artifact model/k21-full-ZA/scripted_m21.pth
- Read-level dZ-DNA vs canonical classification metrics (AUC 0.9646–0.9796, F1 0.9045–0.9384) on held-out test sets

## How to apply

First filter input BAMs with filter_bam.py, keeping reads that carry kinetic tags (fi, ri, fp, rp), requiring at least 3 passes on both strands (fn >= 3, rn >= 3), and discarding reads with Median_PW/Median_IPD < 0.3 or above the 99th percentile. Then build k-mer feature matrices from PW/IPD signals at each odd k in the 7–25 range and train a PyTorch MLP classifier on the E. coli dZ-DNA (full-Z) versus canonical (full-dA) training reads for each k. Compute AUC plus CPU training/calling time per k and plot AUC and CPU time versus k; k=21 was selected as the knee point balancing model strength and efficiency. Export the winning model as the TorchScript artifact model/k21-full-ZA/scripted_m21.pth for use with build/z-calling-base at -k 21. Finally validate on held-out fruit fly (Revio and Sequel II) and E. coli (Revio) test BAMs, confirming AUCs of 0.9646–0.9796 and F1 scores of 0.9045–0.9384.

## Related tools

- **Z-Calling** (Provides filter_bam.py for BAM filtering and build/z-calling-base for evaluating the selected k-mer model; ships the pre-trained model/k21-full-ZA/scripted_m21.pth artifact run with -k 21) — https://github.com/xiaochuanle/Z-Calling
- **PyTorch** (Trains the per-k MLP classifiers on PW/IPD k-mer feature matrices and exports the selected model as TorchScript)
- **PacBio CCS** (Source sequencing data; CCS BAMs generated with --keep-kinetics preserve the PW and IPD signals used as k-mer features)

## Examples

```
build/z-calling-base -k 21 -t 16 FilteredMappingBAM.bam model/k21-full-ZA/scripted_m21.pth Output_BAM
```

## Evaluation signals

- The evaluation table exists as a TSV with AUC and CPU time for each odd k-mer size in the 7–25 range
- Computed test AUCs fall within the reported 0.94–0.96 range and F1 scores within 0.85–0.92 for the Z base calling module (card-specific: AUC 0.9646–0.9796, F1 0.9045–0.9384)
- The selected artifact model/k21-full-ZA/scripted_m21.pth runs successfully with build/z-calling-base -k 21 and writes Z probabilities into MM/ML (and ZP) tags of the output BAM
- The AUC-vs-k and CPU-time-vs-k plots support k=21 as the chosen balance point; adjacent k values should not dominate on both metrics simultaneously
- Torch import check in the Conda environment prints 2.2.2+cpu, confirming the CPU-only PyTorch stack used for training

## Limitations

- The sweep is CPU-bound and memory-hungry: base calling peaks around 31 GB RAM, so >= 32 GB is recommended; no GPU acceleration is used
- The resulting k21 full-substitution model is optimized for fully substituted dZ-genomes and can miss hybrid reads with diluted kinetic signatures (the read-level SVM enforces high confidence to minimize false positives)
- Z-induced kinetic shifts are subtle (0.5–1.5-fold PW change) and overlap with canonical Adenine variance, which caps achievable AUC for hybrid sample quantification

## Evidence

- [other] A k-mer size of 21 bp was selected to balance model strength and efficiency: [evidence span withheld — anchor status: not_found]
- [other] AUC and F1 ranges on held-out sets: [evidence span withheld — anchor status: not_found]
