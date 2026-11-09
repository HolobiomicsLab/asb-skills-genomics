---
name: svm-modification-site-scoring
description: Use when ONT direct RNA sequencing reads (base-called FASTQ/FAST5) are
  available and per-site m6A modification calls are needed, restricting candidates
  to RRACH motif adenosines rather than calling per-molecule.
when_to_use_negative:
- Per-read (single-molecule) modification detection is required — EpiNano 'does not
  predict RNA modifications in individual RNA molecules' and only aggregates reads
  per site
- The caller cannot restrict candidates to a known sequence motif (e.g., RRACH) or
  the modification type is not represented by a pre-trained SVM model — EpiNano-SVM
  depends on the base-caller version and available pretrained models
- Distinguishing modification types (m1A vs m6A) is required — the algorithm does
  not distinguish between them
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: EpiNano
  role: Feature-extraction and SVM scoring of per-site base-calling error features
    to classify RRACH sites as m6A-modified or unmodified
  repo: https://github.com/novoalab/EpiNano
- name: Oxford Nanopore Technologies (ONT)
  role: Direct RNA sequencing platform producing the native RNA reads whose base-calling
    errors are the classification features
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-11713-9
    title: Accurate detection of m6A RNA modifications in native RNA sequences
  source_content_hash: sha256:ee7ff49c409f870b11d0aa23b21671fbf72faa50f28fc1486b18b09c035884dd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/novoalab/EpiNano
  tools:
  - EpiNano
  - Oxford Nanopore Technologies (ONT)
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# svm-modification-site-scoring

## Summary

Score RRACH motif adenosine sites in ONT direct RNA base-called reads for m6A using the EpiNano SVM, which classifies sites from per-site base-calling error features (per-base qualities, mismatch and deletion frequencies) learned on synthetic m6A-modified/unmodified training sequences.

## When to use

Use when ONT direct RNA sequencing reads (base-called FASTQ/FAST5) are available and per-site m6A modification calls are needed, restricting candidates to RRACH motif adenosines rather than calling per-molecule.

## When NOT to use

- Per-read (single-molecule) modification detection is required — EpiNano 'does not predict RNA modifications in individual RNA molecules' and only aggregates reads per site
- The caller cannot restrict candidates to a known sequence motif (e.g., RRACH) or the modification type is not represented by a pre-trained SVM model — EpiNano-SVM depends on the base-caller version and available pretrained models
- Distinguishing modification types (m1A vs m6A) is required — the algorithm does not distinguish between them

## Inputs

- Base-called ONT direct RNA sequencing reads (FASTQ/FAST5) from modified and control samples (e.g., S. cerevisiae WT SAy841 and ime4Δ SAy966)
- Candidate motif site list (61,163 RRACH adenosine sites in the yeast transcriptome)
- Reference m6A annotation (m6A-Seq set of 363 known sites) for validation
- Pre-trained EpiNano SVM model (Albacore 2.1.7 or Guppy 3.1.5 base-called)

## Outputs

- Per-site feature table (per base and per k-mer: mismatch, insertion, deletion frequencies, base quality, sumErr)
- SVM modification score and predicted m6A status for each RRACH candidate site
- Accuracy, recovery, and specificity metrics against the reference m6A-Seq site set

## How to apply

Aggregate base-calling error features from all reads mapping to each candidate site — per-base quality, mismatch frequency, deletion frequency (EpiNano-Error/SVM per-site tables, organized per base and per k-mer) — instead of scoring single molecules. Restrict scoring to RRACH motif adenosines (61,163 sites in the yeast workflow, 363 annotated m6A by the m6A-Seq reference). Apply the EpiNano SVM pre-trained on synthetic m6A-modified and unmodified sequences (Albacore 2.1.7 models for v1.0/1.1, Guppy 3.1.5 models for v1.2) to assign each candidate site a modification score and classification. Optionally compute delta features (differences between modified and unmodified samples) which the README notes have improved performance, and use a matched control (WT vs ime4Δ) to filter error-rich background. Validate by comparing classified sites against the m6A-Seq reference set and confirming error signals are absent in the ime4Δ knockout.

## Related tools

- **EpiNano** (Feature-extraction and SVM scoring of per-site base-calling error features to classify RRACH sites as m6A-modified or unmodified) — https://github.com/novoalab/EpiNano
- **Oxford Nanopore Technologies (ONT)** (Direct RNA sequencing platform producing the native RNA reads whose base-calling errors are the classification features)

## Evaluation signals

- In vivo accuracy reaches ~87–87.8% against the m6A-Seq reference set
- Overall recovery of known sites reaches ~32% (117/363 annotated RRACH sites)
- Specificity is ~89% on candidate motif sites
- Error-based signals are typically absent in the ime4Δ strain, which lacks m6A
- In vitro benchmark with synthetic training sequences yields ~90% accuracy before in vivo extension

## Limitations

- Only 32% of known modification sites are recovered; sensitivity is low
- No per-read resolution — predictions aggregate information from all reads mapping to a site
- Cannot distinguish between different RNA modification types (e.g., m1A vs m6A)
- Feature contributions vary across k-mers, so k-mer-specific training is needed to improve accuracy and reduce false positives
- SVM predictions are base-caller dependent (Guppy 3.1.5 for v1.2 pretrained models; Albacore 2.1.7 for earlier versions) and direct RNA base-calling is error-rich, so a matched control (KO/KD) is highly recommended

## Evidence

- [abstract] our algorithm, trained with m 6 A-modified and unmodified synthetic sequences, can predict m 6 A RNA modifications with ~90% accuracy: "our algorithm, trained with m 6 A-modified and unmodified synthetic sequences, can predict m 6 A RNA modifications with ~90% accuracy"
- [discussion] We find that the method accurately detects modifications both in vitro (90% accuracy) and in vivo (87% accuracy), with an overall recovery of 32% of known sites.: "We find that the method accurately detects modifications both in vitro (90% accuracy) and in vivo (87% accuracy), with an overall recovery of 32% of known sites."
- [discussion] our current algorithm does not predict RNA modifications in individual RNA molecules, but rather employs information from all the reads mapping to a specific site: "our current algorithm does not predict RNA modifications in individual RNA molecules, but rather employs information from all the reads mapping to a specific site"
- [readme] EpiNano-SVM can be run using either pre-trained models for a given RNA modification, or by building your own models.: [evidence span withheld — anchor status: not_found]
- [readme] Base-calling algorithm dependent (data must be base-called with Guppy 3.1.5): [evidence span withheld — anchor status: external_doc]
