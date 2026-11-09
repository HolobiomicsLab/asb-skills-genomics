---
name: read-level-rna-modification-classification
description: Use when per-read (single-molecule) m6A or m5C modification probabilities
  are needed from Oxford Nanopore direct RNA data, e.g. to reproduce CHEUI-solo Model
  1 accuracy/precision/recall/AUC on IVT test sets.
when_to_use_negative:
- Data were produced with the RNA004 chemistry or you need pseudouridine predictions
  — use SWARM, which CHEUI's authors state improves accuracy and adds a pseudouridine
  model
- The goal is transcript-site stoichiometry or differential methylation between two
  conditions — that requires CHEUI Model 2 or CHEUI-diff, not read-level Model 1 alone
- Input is not nanopore direct RNA sequencing signal (e.g. bulk RNA-seq or antibody-based
  m6A/miCLIP data) — CHEUI operates on ionic current signals
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_3512
tools:
- name: CHEUI
  role: Preprocessing scripts and trained Model 1 CNNs producing per-read m6A/m5C
    posterior probabilities
  repo: https://github.com/comprna/CHEUI-public
- name: Nanopolish
  role: eventalign resquiggling of signals to transcript references with rescaled
    events, supplying the expected 5-mer means used as input features
- name: Guppy
  role: Basecalling of FAST5 files (template_rna_r9.4.1_70bps model, v4.0.11+)
- name: Minimap2
  role: Alignment of basecalled reads to the reference transcriptome before event
    alignment
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-47953-7
    title: Prediction of m6A and m5C at single-molecule resolution reveals a transcriptome-wide
      co-occurrence of RNA modifications
  source_content_hash: sha256:581f5bf2fe3b2223969afd22a7d8dc0e2cda30c5b418612f816649835e0448f7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/CHEUI-public
  tools:
  - CHEUI
  - Guppy
  - Minimap2
  - Nanopolish
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# read-level-rna-modification-classification

## Summary

Classify individual nanopore direct RNA reads as m6A- or m5C-modified by feeding resquiggled ionic-current signals through CHEUI-solo Model 1 CNNs (Keras/TensorFlow), yielding a posterior modification probability per read for benchmarking model accuracy and recall.

## When to use

Use when per-read (single-molecule) m6A or m5C modification probabilities are needed from Oxford Nanopore direct RNA data, e.g. to reproduce CHEUI-solo Model 1 accuracy/precision/recall/AUC on IVT test sets.

## When NOT to use

- Data were produced with the RNA004 chemistry or you need pseudouridine predictions — use SWARM, which CHEUI's authors state improves accuracy and adds a pseudouridine model
- The goal is transcript-site stoichiometry or differential methylation between two conditions — that requires CHEUI Model 2 or CHEUI-diff, not read-level Model 1 alone
- Input is not nanopore direct RNA sequencing signal (e.g. bulk RNA-seq or antibody-based m6A/miCLIP data) — CHEUI operates on ionic current signals

## Inputs

- Oxford Nanopore FAST5 files from nanopore direct RNA sequencing (RNA002, R9.4.1 chemistry)
- Reference transcriptome FASTA (e.g. GRCh38/Gencode v38 or Jenjaroenpun et al. IVT transcript references)
- Nanopolish eventalign output with per-read samples and signal indices
- CHEUI k-mer expected-signal model file (kmer_models/model_kmer.csv)
- Trained CHEUI Model 1 .h5 weights: CHEUI_m6A_model1.h5, CHEUI_m5C_model1.h5

## Outputs

- Per-read posterior modification probabilities (m6A and m5C) keyed by read ID and 9-mer site
- Accuracy, precision, recall, and AUC per modification per test set (e.g. cheui_model1_ivt_test_metrics.csv)

## How to apply

Basecall FAST5 files with Guppy (>=4.0.11, template_rna_r9.4.1_70bps model), align to the reference transcriptome with minimap2 keeping primary positive-strand alignments (samtools -F 2324, or -F 232457 best-match filtering), and resquiggle signals with nanopolish eventalign using --scale-events --signal-index --samples --print-read-names. Run the CHEUI preprocessing script (CHEUI_preprocess_m6A.py or CHEUI_preprocess_m5C.py with the model_kmer.csv expected-signal file) to build per-read inputs of 100 median-vectorized observed signal values plus 100 absolute distances from the Nanopolish 5-mer model, centered on 9-mers containing exactly one A (m6A) or one C (m5C), with signals flipped per 5-mer to 5'–3' orientation. Apply the trained CHEUI Model 1 .h5 CNNs (CHEUI_m6A_model1.h5 / CHEUI_m5C_model1.h5) to obtain a posterior modification probability per read, then compute accuracy as (TN+TP)/(TN+TP+FN+FP), precision as TP/(TP+FP), recall as TP/(TP+FN), and AUC from the per-read probability distributions. Validate against the paper's reported benchmarks: m6A accuracy 0.835 (IVT test 1) / 0.777 (IVT test 2), m5C accuracy 0.793 / 0.741, m6A AUC 0.910 / 0.856, m5C AUC 0.879 / 0.827.

## Related tools

- **CHEUI** (Preprocessing scripts and trained Model 1 CNNs producing per-read m6A/m5C posterior probabilities) — https://github.com/comprna/CHEUI-public
- **Nanopolish** (eventalign resquiggling of signals to transcript references with rescaled events, supplying the expected 5-mer means used as input features)
- **Guppy** (Basecalling of FAST5 files (template_rna_r9.4.1_70bps model, v4.0.11+))
- **Minimap2** (Alignment of basecalled reads to the reference transcriptome before event alignment)

## Examples

```
../scripts/CHEUI_preprocess_m6A.py -i nanopolish_out.txt -m ../kmer_models/model_kmer.csv -o cheui_out -n 8
```

## Evaluation signals

- Reproduced m6A accuracy on IVT test 1 and test 2 is within a few percent of 0.835 and 0.777 respectively
- Reproduced m5C accuracy falls within a few percent of 0.793 (test 1) and 0.741 (test 2)
- AUC values match reported figures within a few percent: m6A 0.910/0.856 and m5C 0.879/0.827 (test 1/test 2)
- Per-read inputs are 100 observed signal values + 100 Nanopolish 5-mer model distances per 9-mer, restricted to 9-mers with a single center A (m6A) or C (m5C)
- All Model 1 outputs are probabilities bounded in [0, 1] and include read IDs

## Limitations

- Very strict transcriptome-wide cut-offs used to keep expected FDR low reduce CHEUI sensitivity; many orthogonally supported sites have high probabilities below the thresholds
- Nanopore signal variability limits detection where the difference between modified and non-modified reads is comparable to variation among non-modified reads
- Signal interference at distances <5 nt is exacerbated in densely modified molecules such as rRNA
- Modified sites detected only from nanopore reads require independent validation experiments

## Evidence

- [methods] CHEUI-solo Model 1 was trained using read signals generated from in-vitro transcript (IVT) data: "CHEUI-solo Model 1 was trained using read signals generated from in-vitro transcript (IVT) data26,27"
- [methods] eventalign command: "eventalign was then used to align the read signals to the matched transcript references"
- [intro] two-stage neural network: "CHEUI is based on a two-stage neural network and was trained using read signals generated from in-vitro transcripts (IVTs)"
- [abstract] accuracy claims: "CHEUI processes observed and expected nanopore direct RNA sequencing signals to achieve high single-molecule, transcript-site, and stoichiometry accuracies"
- [readme] README model description: [evidence span withheld — anchor status: external_doc]
- [readme] README eventalign flags: [evidence span withheld — anchor status: external_doc]
