---
name: per-site-base-quality-aggregation
description: Use when you have base-called ONT direct RNA reads (FASTQ/FAST5, Albacore
  or Guppy) aligned to a transcriptome and need per-site error/quality features for
  RNA modification classification with EpiNano-SVM or EpiNano-Error.
when_to_use_negative:
- Input is already a per-site feature/error table — feature extraction would be redundant;
  proceed directly to SVM classification or pairwise error comparison
- Single-read (per-molecule) modification detection is required — EpiNano aggregates
  information from all reads at a site and explicitly lacks per-read resolution
- A specific modification type must be distinguished (e.g., m1A vs m6A) — the error
  features do not discriminate between modification types
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: EpiNano
  role: Extracts per-base and per-kmer base-calling 'error' features (mismatch, insertion,
    deletion, per-base quality) from direct RNA reads; in v1.0/1.1 it also performs
    the sam2tsv-style per-site feature table construction used for m6A SVM classification
  repo: https://github.com/novoalab/EpiNano
- name: samtools/pysam
  role: Load and traverse the sorted, indexed BAM of direct RNA reads while walking
    CIGAR strings to emit per-position read-base records
- name: Oxford Nanopore Technologies (ONT)
  role: Direct RNA sequencing platform whose base-called reads supply the quality
    scores and error profiles aggregated per site
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-11713-9
    title: Accurate detection of m6A RNA modifications in native RNA sequences
  source_content_hash: sha256:f2075190853ff399a8f18066635e7fab9c117539d02cf31b7632e0b71c05ea4a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/novoalab/EpiNano
  tools:
  - EpiNano
  - Oxford Nanopore Technologies (ONT)
  - samtools/pysam
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# per-site-base-quality-aggregation

## Summary

Extracts per-site base-called features (mean per-base quality, mismatch, insertion, and deletion frequencies) from ONT direct RNA sequencing alignments, organized over the 5-mer positions (−2, −1, 0, +1, +2) surrounding each transcriptomic site. These error-rich features are the inputs EpiNano uses to detect m6A as systematic base-calling errors and decreased base-calling qualities.

## When to use

Use when you have base-called ONT direct RNA reads (FASTQ/FAST5, Albacore or Guppy) aligned to a transcriptome and need per-site error/quality features for RNA modification classification with EpiNano-SVM or EpiNano-Error.

## When NOT to use

- Input is already a per-site feature/error table — feature extraction would be redundant; proceed directly to SVM classification or pairwise error comparison
- Single-read (per-molecule) modification detection is required — EpiNano aggregates information from all reads at a site and explicitly lacks per-read resolution
- A specific modification type must be distinguished (e.g., m1A vs m6A) — the error features do not discriminate between modification types

## Inputs

- Sorted, indexed BAM file of ONT direct RNA reads aligned to the reference transcriptome
- Reference transcriptome used for alignment
- Base-called FASTQ/FAST5 files (EpiNano 1.2 SVM models require Guppy 3.1.5; EpiNano 1.0/1.1 models require Albacore 2.1.7)

## Outputs

- epinano_features_per_site.tsv — one row per transcriptomic position with coverage, mean per-base qualities (A, C, G, T), and mismatch/insertion/deletion frequencies
- Optional per-kmer feature table summarizing the same errors across 5-mer positions (−2, −1, 0, +1, +2)

## How to apply

Load the sorted, indexed BAM of direct RNA reads aligned to the reference transcriptome with samtools/pysam, then perform a sam2tsv-style walk of each read's CIGAR string to emit one record per reference position carrying read base identity, base-calling quality, and match/mismatch/insertion/deletion status. Aggregate records across all reads mapping to each site, computing the mean per-base quality for each of the four bases (A, C, G, T) and the mismatch, insertion, and deletion frequencies per position. Restrict the table to positions with sufficient read coverage so per-site error statistics are estimable, retaining per-site depth alongside the error features. Write the result as epinano_features_per_site.tsv (one row per transcriptomic position) for downstream EpiNano SVM classification; the rationale is that m6A manifests as altered per-base qualities and mismatch/deletion frequencies rather than changes in raw current intensity, so no squiggle realignment is needed.

## Related tools

- **EpiNano** (Extracts per-base and per-kmer base-calling 'error' features (mismatch, insertion, deletion, per-base quality) from direct RNA reads; in v1.0/1.1 it also performs the sam2tsv-style per-site feature table construction used for m6A SVM classification) — https://github.com/novoalab/EpiNano
- **samtools/pysam** (Load and traverse the sorted, indexed BAM of direct RNA reads while walking CIGAR strings to emit per-position read-base records)
- **Oxford Nanopore Technologies (ONT)** (Direct RNA sequencing platform whose base-called reads supply the quality scores and error profiles aggregated per site)

## Evaluation signals

- Output table contains per-site mean per-base quality columns (A, C, G, T) and mismatch/insertion/deletion frequency columns for every covered position, plus a coverage/depth column
- One row per transcriptomic position of the 5-mer window (−2, −1, 0, +1, +2), with error statistics estimable only at positions with sufficient read coverage
- Known m6A sites show altered per-base qualities, elevated mismatch and deletion frequencies relative to unmodified sites (e.g., WT vs KO/KD controls)
- Features fed to the EpiNano SVM reproduce the paper's reported ~90% prediction accuracy on synthetic training sequences and 87% accuracy on yeast in vivo data
- Detected 'errors' are largely absent in m6A-lacking controls such as yeast ime4-knockout strains

## Limitations

- EpiNano has no per-read resolution: predictions aggregate information from all reads mapping to a site, so single-read modification detection is not achieved
- The algorithm cannot distinguish between different types of RNA modifications (e.g., m1A vs m6A), and only ~32% of known modification sites are recovered
- The contribution of each feature varies across different k-mers, so k-mer-specific training is needed to improve accuracy and reduce false positives
- SVM predictions are base-caller dependent — pre-trained models are accurate only for data base-called with Albacore 2.1.7 (v1.0/1.1) or Guppy 3.1.5 (v1.2); direct RNA base-calling is error-rich even on unmodified sequences, so a matched control (WT vs KO/KD) is highly recommended

## Evidence

- [other] EpiNano compiles base-called features — base quality, mismatch frequency, and deletion frequency — for each position of the 5-mer (−2, −1, 0, +1, and +2) as the per-site features used for m6A classification.: [evidence span withheld — anchor status: not_found]
- [discussion] These “errors” can be detected in the form of altered per-base qualities, mismatch frequencies and deletion frequencies at the modified site.: "These “errors” can be detected in the form of altered per-base qualities, mismatch frequencies and deletion frequencies at the modified site."
- [readme] It extracts both base-calling 'errors' (mismatch, insertion, seletion, per-base quality) as well as current intensity values: [evidence span withheld — anchor status: external_doc]
- [readme] These features can be organized in per base and per kmer formats: [evidence span withheld — anchor status: external_doc]
- [readme] EpiNano does not have per-read resolution. We are currently working on an improved version of EpiNano to obtain predictions at per-read level.: [evidence span withheld — anchor status: external_doc]
