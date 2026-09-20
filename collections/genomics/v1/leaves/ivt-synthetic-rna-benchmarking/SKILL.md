---
name: ivt-synthetic-rna-benchmarking
description: Use when validating a nanopore direct-RNA modification detection model
  against synthetic IVT controls with known ground truth, e.g., to measure sensor
  generalization (IVT set 1) and k-mer generalization (IVT set 2).
when_to_use_negative:
- The goal is differential methylation between two biological conditions — use CHEUI-diff
  rather than IVT benchmarking, which requires known modified/unmodified ground truth.
- Data are cellular transcriptome reads without modified/unmodified controls — use
  CHEUI-solo site-level Model 2 with FDR estimation by shuffling read signals across
  transcriptomic sites.
- Reads were sequenced with the RNA004 chemistry — the CHEUI README notes SWARM (github.com/comprna/SWARM)
  is the recommended successor for RNA002 accuracy improvements and RNA004 predictions.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CHEUI
  role: Two-stage deep-learning model whose CHEUI-solo Model 1 CNNs produce per-read
    m6A/m5C modification probabilities being benchmarked on IVT sets 1 and 2
  repo: https://github.com/comprna/CHEUI
- name: Nanopolish
  role: eventalign resquiggles read signals to the transcript reference with events
    rescaled, producing the observed-signal/expected-signal pairs used as model inputs
  repo: https://nanopolish.readthedocs.io/en/latest/
- name: minimap2
  role: Aligns basecalled reads to the reference transcriptome; primary positive-strand
    alignments are selected for downstream signal alignment
- name: Guppy
  role: Basecaller (4.0.11+, template_rna_r9.4.1_70bps model) converting FAST5 raw
    signals to FASTQ prior to alignment
  repo: https://community.nanoporetech.com/downloads/guppy/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-47953-7
    title: Prediction of m6A and m5C at single-molecule resolution reveals a transcriptome-wide
      co-occurrence of RNA modifications
  source_content_hash: sha256:a351e24cb8eb23796e0f5ab69d5591545e6eb75ee6b6ce0a78dd1a6b9af04dfe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/CHEUI
  tools:
  - CHEUI
  - Guppy
  - Nanopolish
  - minimap2
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# ivt-synthetic-rna-benchmarking

## Summary

Benchmark nanopore RNA-modification callers (e.g., CHEUI-solo Model 1) using in-vitro transcribed (IVT) synthetic RNA standards with known m6A/m5C status, computing read-level accuracy, precision, recall, and AUC on both seen and unseen sequence contexts.

## When to use

Use when validating a nanopore direct-RNA modification detection model against synthetic IVT controls with known ground truth, e.g., to measure sensor generalization (IVT set 1) and k-mer generalization (IVT set 2).

## When NOT to use

- The goal is differential methylation between two biological conditions — use CHEUI-diff rather than IVT benchmarking, which requires known modified/unmodified ground truth.
- Data are cellular transcriptome reads without modified/unmodified controls — use CHEUI-solo site-level Model 2 with FDR estimation by shuffling read signals across transcriptomic sites.
- Reads were sequenced with the RNA004 chemistry — the CHEUI README notes SWARM (github.com/comprna/SWARM) is the recommended successor for RNA002 accuracy improvements and RNA004 predictions.

## Inputs

- IVT FAST5 raw signal files from synthetic RNA standards (Jenjaroenpun et al. IVT reads, one sequencing experiment per IVT)
- Basecalled FASTQ and transcriptome reference FASTA (e.g., GRCh38/Gencode v38 context)
- Nanopolish eventalign output with --scale-events --signal-index --samples --print-read-names
- Nanopolish 5-mer k-mer model file (CHEUI/kmer_models/model_kmer.csv)
- Trained CHEUI-solo Model 1 .h5 CNN weights (CHEUI_m6A_model1.h5, CHEUI_m5C_model1.h5)

## Outputs

- Per-read m6A and m5C posterior modification probabilities for IVT test set 1 and test set 2
- Metrics table (accuracy, precision, recall, AUC) per modification and test set, e.g., cheui_model1_ivt_test_metrics.csv

## How to apply

Basecall FAST5 files with Guppy (template_rna_r9.4.1_70bps model), align reads to the transcriptome reference with minimap2 keeping primary positive-strand alignments, and resquiggle signals with nanopolish eventalign (--scale-events --signal-index --samples --print-read-names). Restrict evaluation to 9-mers containing exactly one A (m6A) or one C (m5C) at the center, then build inputs of 100 median-vectorized observed signal values plus 100 absolute distances from the Nanopolish 5-mer model per 9-mer. Apply the trained CHEUI-solo Model 1 CNNs (Keras/TensorFlow, CHEUI_m6A_model1.h5 / CHEUI_m5C_model1.h5) to both the IVT test 1 set (9-mer contexts seen during training, sensor generalization) and IVT test 2 set (contexts not seen, k-mer generalization), using Jenjaroenpun et al. IVT reads (one sequencing experiment per IVT). Compute per-read posterior modification probabilities, then derive accuracy (TN+TP)/(TN+TP+FN+FP), precision TP/(TP+FP), recall TP/(TP+FN), and AUC from the probability distributions; write the metrics table (e.g., cheui_model1_ivt_test_metrics.csv). Validation requires reproducing the reported values: m6A accuracy 0.835/0.777 and AUC 0.910/0.856, m5C accuracy 0.793/0.741 and AUC 0.879/0.827 for test 1/test 2.

## Related tools

- **CHEUI** (Two-stage deep-learning model whose CHEUI-solo Model 1 CNNs produce per-read m6A/m5C modification probabilities being benchmarked on IVT sets 1 and 2) — https://github.com/comprna/CHEUI
- **Nanopolish** (eventalign resquiggles read signals to the transcript reference with events rescaled, producing the observed-signal/expected-signal pairs used as model inputs) — https://nanopolish.readthedocs.io/en/latest/
- **minimap2** (Aligns basecalled reads to the reference transcriptome; primary positive-strand alignments are selected for downstream signal alignment)
- **Guppy** (Basecaller (4.0.11+, template_rna_r9.4.1_70bps model) converting FAST5 raw signals to FASTQ prior to alignment) — https://community.nanoporetech.com/downloads/guppy/

## Examples

```
minimap2 -ax map-ont -k14 <transcriptome fasta> <read fastq> | samtools view -F 2324 -b | samtools sort > <sorted-bam-file> && nanopolish eventalign --reads <read fastq> --bam <sorted-bam-file> --genome <transcriptome fasta> --scale-events --signal-index --samples --print-read-names > nanopolish_out.txt
```

## Evaluation signals

- Reproduced m6A accuracy for IVT test 1 and test 2 within a few percent of 0.835 and 0.777 respectively; m5C accuracy within a few percent of 0.793 and 0.741
- Reproduced m6A AUC within a few percent of 0.910 (test 1) and 0.856 (test 2); m5C AUC within a few percent of 0.879 and 0.827
- A drop in metrics from IVT set 1 (sensor generalization, contexts seen in training) to IVT set 2 (k-mer generalization, contexts not seen) is expected and confirms the split was implemented correctly
- Per-read posterior probabilities fall in [0, 1] and every evaluated 9-mer contains exactly one A (m6A model) or one C (m5C model) at its center
- Results table is written with rows for both modifications and both test sets, matching the cheui_model1_ivt_test_metrics.csv schema

## Limitations

- Very strict cut-offs are used on cell transcriptome sites to maintain low expected FDR, which reduces CHEUI sensitivity; many orthogonally supported sites had high probabilities just below thresholds
- Nanopore signal variability limits single-read discrimination, as differences between modified and non-modified reads are often comparable to variation among non-modified reads
- Benchmarking is tied to the R9.4.1 RNA002 chemistry and Guppy 4.0.11+ basecalling; RNA004 chemistry requires the SWARM successor instead of CHEUI

## Evidence

- [intro] CHEUI is based on a two-stage neural network and was trained using read signals generated from in-vitro transcripts (IVTs): "CHEUI is based on a two-stage neural network and was trained using read signals generated from in-vitro transcripts (IVTs)"
- [methods] IVT reads source: "we used the reads from Jenjaroenpun, P. et al.26 corresponding to one sequencing experiment per IVT"
- [methods] Read filtering step: "Reads were then filtered to select the best match for each read using samtools -F 232457."
- [abstract] CHEUI benchmarking rationale: "CHEUI processes observed and expected nanopore direct RNA sequencing signals to achieve high single-molecule, transcript-site, and stoichiometry accuracies"
- [readme] Nanopolish eventalign command: [evidence span withheld — anchor status: not_found]
- [readme] Basecalling requirement: [evidence span withheld — anchor status: external_doc]
