---
name: classification-metrics-computation
description: Use when you have per-read prediction outputs (.tsv) from a signal-based
  RNA classifier (e.g., RISER run over HeLa fast5-derived signals) and need accuracy/precision/TPR/FPR
  to check reproduction of published accuracy (mRNA ~94%, mtRNA ~99%).
when_to_use_negative:
- The question is about enrichment/depletion read-until performance (rejection efficiency)
  rather than offline classification accuracy — that requires live-run read length
  analysis, not metric computation on .tsv outputs.
- Predictions come from basecalling-plus-mapping sequence-based adaptive sampling
  rather than direct raw-signal classification; the metric pipeline here is specific
  to RISER-style signal classifiers with biotype labels.
- Reads lack high-confidence biotype labels (primary mapping biotype not matching
  the most frequent secondary mapping biotype) and must first be filtered out before
  scoring.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0080
tools:
- name: RISER
  role: Deep learning classifier whose per-read .tsv outputs are scored; run via riser/test.py
    with best_model.pth and config yaml
  repo: https://github.com/comprna/riser
- name: BoostNano
  role: Removes the sequencing adapter and variable-length poly(A) tail from raw signals
    so that classification metrics reflect model performance alone
  repo: https://github.com/haotianteng/BoostNano
- name: Guppy (v4.0.14)
  role: Basecalls HeLa fast5 files (FLO-MIN106 / SQK-RNA002 settings) to obtain labels
    for scoring
- name: minimap2 (v2.17)
  role: Maps basecalled reads to the GENCODE release 34 transcriptome (-ax map-ont
    --secondary=no) to assign biotypes used as ground truth
- name: samtools (v1.10)
  role: Filters alignments with -F 2324 to retain only high-confidence primary mappings
    for biotype labels
- name: PyTorch (v1.9.0)
  role: Framework in which the models were built, trained, and tested
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-48673-8
    title: Biochemical-free enrichment or depletion of RNA classes in real-time during
      direct RNA sequencing with RISER
  source_content_hash: sha256:c27b5d791914001a1d1ea581a01f97090512847db8d2b68958733775143223a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/riser
  tools:
  - BoostNano
  - Guppy (v4.0.14)
  - PyTorch (v1.9.0)
  - RISER
  - minimap2 (v2.17)
  - samtools (v1.10)
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# classification-metrics-computation

## Summary

Compute binary-classification performance metrics (accuracy, precision, true positive rate, false positive rate) from per-read prediction tables produced by a deep learning RNA-class classifier, optionally stratified per biotype, to validate model reproduction against published figures.

## When to use

Use when you have per-read prediction outputs (.tsv) from a signal-based RNA classifier (e.g., RISER run over HeLa fast5-derived signals) and need accuracy/precision/TPR/FPR to check reproduction of published accuracy (mRNA ~94%, mtRNA ~99%).

## When NOT to use

- The question is about enrichment/depletion read-until performance (rejection efficiency) rather than offline classification accuracy — that requires live-run read length analysis, not metric computation on .tsv outputs.
- Predictions come from basecalling-plus-mapping sequence-based adaptive sampling rather than direct raw-signal classification; the metric pipeline here is specific to RISER-style signal classifiers with biotype labels.
- Reads lack high-confidence biotype labels (primary mapping biotype not matching the most frequent secondary mapping biotype) and must first be filtered out before scoring.

## Inputs

- Per-read prediction .tsv files from riser/test.py (mRNA and mtRNA model outputs)
- Trained model checkpoint (best_model.pth) and config yaml
- Trimmed raw nanopore signals from HeLa poly(A)+ DRS fast5 files (adapter and poly(A) tail removed with BoostNano)
- High-confidence biotype labels from GENCODE release 34 (GRCh38.p13) mappings (primary biotype matching most frequent secondary biotype)

## Outputs

- Accuracy, precision, TPR, and FPR per model (mRNA and mtRNA)
- Accuracy per biotype with lncRNA, mRNA, mtRNA reported separately and remaining biotypes aggregated as 'other'
- Reproduction verdict against Fig. 3a-d reference values (mRNA ~94%, mtRNA ~99%)

## How to apply

Run the trained model (e.g., RISER mRNA and mtRNA models via riser/test.py with best_model.pth and the config yaml, PyTorch v1.9.0) over the held-out test signals (607,740 HeLa poly(A)+ DRS reads trimmed of adapter and poly(A) tail with BoostNano). Parse the resulting per-read .tsv files, treat protein_coding as the positive class for the mRNA model, and compute accuracy, precision, TPR and FPR per model. As a secondary analysis, compute accuracy stratified by GENCODE biotype, reporting mRNA, mtRNA, and lncRNA separately and aggregating remaining biotypes as 'other'. Validation is grounded in Fig. 3a-d: accept the reproduction only if mRNA model accuracy is approximately 94% and mtRNA model accuracy approximately 99% on the HeLa test set (precision 0.99, TPR 0.94, FPR 0.05 for mRNA).

## Related tools

- **RISER** (Deep learning classifier whose per-read .tsv outputs are scored; run via riser/test.py with best_model.pth and config yaml) — https://github.com/comprna/riser
- **BoostNano** (Removes the sequencing adapter and variable-length poly(A) tail from raw signals so that classification metrics reflect model performance alone) — https://github.com/haotianteng/BoostNano
- **Guppy (v4.0.14)** (Basecalls HeLa fast5 files (FLO-MIN106 / SQK-RNA002 settings) to obtain labels for scoring)
- **minimap2 (v2.17)** (Maps basecalled reads to the GENCODE release 34 transcriptome (-ax map-ont --secondary=no) to assign biotypes used as ground truth)
- **samtools (v1.10)** (Filters alignments with -F 2324 to retain only high-confidence primary mappings for biotype labels)
- **PyTorch (v1.9.0)** (Framework in which the models were built, trained, and tested)

## Examples

```
python riser/test.py --config config.yaml --model best_model.pth --input hela_trimmed_fast5/ --output per_read_results.tsv
```

## Evaluation signals

- mRNA model accuracy on the HeLa test set is approximately 94% (matching Fig. 3a-d)
- mRNA model precision ≈ 0.99, TPR ≈ 0.94, FPR ≈ 0.05
- mtRNA model accuracy is approximately 99% on the same HeLa test set
- Number of scored reads equals 607,740 HeLa test signals after filtering to high-confidence biotype labels
- Per-biotype accuracy table contains separate entries for mRNA, mtRNA, lncRNA, and an aggregated 'other' bin

## Limitations

- High-confidence labels require discarding reads whose primary mapping biotype does not match the most frequent secondary mapping biotype; inconsistent biotype calls are excluded from scoring.
- Accurate lncRNA identification from the 3' end signal remains challenging, and cell type-specific expression of lncRNAs makes it harder for predictive models to generalize, so per-biotype accuracies may degrade for low-abundance classes.
- Metric values are specific to the HeLa poly(A)+ DRS dataset (SQK-RNA002, FLO-MIN106, GENCODE release 34); they are not guaranteed for other kits (e.g., RNA004), cell types, or reference transcriptome releases.

## Evidence

- [methods] Per-read .tsv metric computation from RISER test output: "All models were built, trained, and tested using PyTorch (v1.9.0)36 with a single NVIDIA Tesla V100 graphics processing unit (GPU)"
- [methods] Adapter/poly(A) trimming isolates model performance: "the sequencing adapter and poly(A) tail were removed from the raw nanopore signals using BoostNano32"
- [methods] High-confidence biotype filtering for labels: "unmapped reads were removed using samtools34 (v1.10) with option -F 2324"
- [readme] RISER classifies RNA directly from raw nanopore signals without basecalling or reference: "RISER accurately classifies RNA molecules live during sequencing, directly from the first ~70-280nt-worth of raw nanopore signals with a convolutional neural network"
