---
name: per-site-read-feature-aggregation
description: Use when base-called ONT direct RNA reads must be condensed into per-site
  error/quality features over RRACH motif adenosines to score RNA modifications at
  site level, since per-read prediction is not yet supported.
when_to_use_negative:
- When per-read (single-molecule) modification detection is required — the algorithm
  explicitly works at site level and does not predict modifications in individual
  RNA molecules
- When the goal is to distinguish between different modification types (e.g., m1A
  vs m6A) at the same site
- When an aggregated per-site feature table already exists — aggregation would be
  redundant; proceed directly to SVM scoring
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0621
tools:
- name: EpiNano
  role: Feature-extraction scripts computing per-base and per-k-mer base-calling error
    features and aggregating them across all reads mapping to each site; also provides
    the SVM used on the aggregated features
  repo: https://github.com/novoalab/EpiNano
- name: Oxford Nanopore Technologies (ONT)
  role: Source of the direct RNA base-called reads (base-calling errors and quality
    scores) that supply the per-read features being aggregated
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-11713-9
    title: Accurate detection of m6A RNA modifications in native RNA sequences
  source_content_hash: sha256:ab15a7a1ea0c75853902231c5bfd99faec5da107092a44cf545fbfde20c17c51
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/novoalab/EpiNano
  tools:
  - EpiNano
  - Oxford Nanopore Technologies (ONT)
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# per-site-read-feature-aggregation

## Summary

Aggregates base-calling error features (per-base quality, mismatch frequency, deletion frequency, insertion frequency) from all ONT direct RNA reads mapping to a candidate site, producing a per-site (and per-k-mer) feature table used to classify m6A modifications with EpiNano-SVM rather than calling single molecules.

## When to use

Use when base-called ONT direct RNA reads must be condensed into per-site error/quality features over RRACH motif adenosines to score RNA modifications at site level, since per-read prediction is not yet supported.

## When NOT to use

- When per-read (single-molecule) modification detection is required — the algorithm explicitly works at site level and does not predict modifications in individual RNA molecules
- When the goal is to distinguish between different modification types (e.g., m1A vs m6A) at the same site
- When an aggregated per-site feature table already exists — aggregation would be redundant; proceed directly to SVM scoring

## Inputs

- Base-called ONT direct RNA sequencing reads (FASTQ; e.g., S. cerevisiae wild-type SAy841 and ime4Δ SAy966)
- Read-to-reference alignments assigning individual reads to genomic/transcript sites
- Candidate site list restricted to RRACH motif adenosines (61,163 sites in the yeast benchmark, 363 annotated m6A)
- Optional matched control dataset (wild-type vs knockout/knockdown)

## Outputs

- Per-site aggregated error feature table (per-base and per-k-mer: mismatch, insertion, deletion frequency, base quality scores, sumErr)
- Per-site modification scores from EpiNano-SVM classification of RRACH candidate sites

## How to apply

Start from ONT direct RNA reads base-called on the ONT platform and map all reads to the transcriptome/reference. For each candidate site (in the yeast benchmark, the 61,163 RRACH motif adenosine positions, of which 363 are annotated m6A in the m6A-Seq reference), extract per-read base-called features — per-base qualities, mismatch, insertion, and deletion frequencies — and aggregate them across every read mapping to that site, rather than attempting single-molecule calls. Organize the aggregated features in per-base and per-k-mer formats, optionally computing sumErr (a combination of error types and base quality scores). Feed the per-site feature vectors into the EpiNano SVM (EpiNano-SVM) trained on m6A-modified and unmodified synthetic sequences to obtain a modification score per site. Validate that aggregated error signals are reduced or absent in a matched control such as the yeast ime4Δ strain lacking m6A.

## Related tools

- **EpiNano** (Feature-extraction scripts computing per-base and per-k-mer base-calling error features and aggregating them across all reads mapping to each site; also provides the SVM used on the aggregated features) — https://github.com/novoalab/EpiNano
- **Oxford Nanopore Technologies (ONT)** (Source of the direct RNA base-called reads (base-calling errors and quality scores) that supply the per-read features being aggregated)

## Evaluation signals

- Each site's features are computed from information pooled across ALL reads mapping to that site, not from individual reads
- Feature table contains per-base and per-k-mer error columns: mismatch frequency, deletion frequency, insertion frequency, per-base quality scores
- On the yeast in vivo benchmark (61,163 RRACH adenosines, 363 annotated m6A), site classification reaches ~87.8% accuracy with 32% recovery (117/363) of known sites and 89% specificity
- Aggregated error signals are typically absent in the ime4Δ control strain lacking m6A
- Only RRACH-motif adenosine sites carry modification scores

## Limitations

- No per-read resolution: predictions are site-level aggregations; single-read modification detection remains future work
- Only 32% of known m6A sites are recovered (a recall ceiling), even at high specificity
- Direct RNA base-calling is 'error'-rich even on unmodified sequences, so a matched control (WT vs KO/KD) is strongly recommended for confidence
- The contribution of each feature varies across k-mers, so accuracy can be improved with k-mer-specific training; performance also depends on modification stoichiometry

## Evidence

- [discussion] site-level aggregation rationale: "our current algorithm does not predict RNA modifications in individual RNA molecules, but rather employs information from all the reads mapping to a specific site"
- [discussion] features aggregated: "These “errors” can be detected in the form of altered per-base qualities, mismatch frequencies and deletion frequencies at the modified site."
- [readme] no per-read resolution: [evidence span withheld — anchor status: external_doc]
- [readme] extracted feature types: [evidence span withheld — anchor status: external_doc]
- [abstract] in vivo benchmark accuracy: "our method can identify m 6 A RNA modifications in vivo with an accuracy of 87%"
