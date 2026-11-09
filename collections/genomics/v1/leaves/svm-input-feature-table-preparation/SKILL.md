---
name: svm-input-feature-table-preparation
description: Use when ONT direct RNA reads are aligned to a reference transcriptome
  (sorted, indexed BAM) and per-site base-calling 'error' features must be aggregated
  to feed EpiNano-SVM m6A classification.
when_to_use_negative:
- The per-site/per-kmer feature table already exists — proceed directly to SVM classification
  rather than re-extracting features.
- Per-read (single-molecule) modification calls are required — EpiNano has no per-read
  resolution; it aggregates information from all reads mapping to a site.
- A squiggle/raw current-intensity approach is intended — this skill deliberately
  uses only base-called features, without squiggle realignment or manipulation of
  raw current intensity data.
edam_topics:
- http://edamontology.org/topic_0099
tools:
- name: EpiNano
  role: Feature-extraction toolkit that converts aligned direct RNA reads into per-base/per-kmer
    base-calling error and quality features for m6A SVM classification
  repo: https://github.com/novoalab/EpiNano
- name: Oxford Nanopore Technologies (ONT)
  role: Direct RNA sequencing platform whose base-called reads supply the quality,
    mismatch, insertion, and deletion signals
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-11713-9
    title: Accurate detection of m6A RNA modifications in native RNA sequences
  source_content_hash: sha256:c507f80f124d1c59f98fe9fb33a0ea986f835ca15bff9913373ffa98b79e6ce8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/novoalab/EpiNano
  tools:
  - EpiNano
  - Oxford Nanopore Technologies (ONT)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# svm-input-feature-table-preparation

## Summary

Extracts per-site base-called features — mean per-base quality (A/C/G/T), mismatch, insertion, and deletion frequencies over the 5-mer positions (−2, −1, 0, +1, +2) — from ONT direct RNA BAM alignments, producing the feature table that EpiNano's SVM consumes for m6A classification.

## When to use

Use when ONT direct RNA reads are aligned to a reference transcriptome (sorted, indexed BAM) and per-site base-calling 'error' features must be aggregated to feed EpiNano-SVM m6A classification.

## When NOT to use

- The per-site/per-kmer feature table already exists — proceed directly to SVM classification rather than re-extracting features.
- Per-read (single-molecule) modification calls are required — EpiNano has no per-read resolution; it aggregates information from all reads mapping to a site.
- A squiggle/raw current-intensity approach is intended — this skill deliberately uses only base-called features, without squiggle realignment or manipulation of raw current intensity data.

## Inputs

- Sorted, indexed BAM of ONT direct RNA reads aligned to the reference transcriptome
- Base-called ONT direct RNA reads (FASTQ from Albacore 2.1.7 or Guppy 3.1.5 for compatible pre-trained SVM models)
- Reference transcriptome used for alignment

## Outputs

- epinano_features_per_site.tsv: one row per transcriptomic position with coverage depth, mean per-base quality for A/C/G/T, and mismatch/insertion/deletion frequencies
- Per-kmer (5-mer, positions −2..+2) feature vectors formatted as EpiNano-SVM input

## How to apply

Load the sorted, indexed direct RNA BAM with samtools/pysam and perform a sam2tsv-style walk of each read's CIGAR string, emitting one record per reference position with read base identity, base-calling quality, and match/mismatch/insertion/deletion status. Aggregate records across all reads mapping to each site, computing mean per-base quality for each of the four bases (A, C, G, T) plus mismatch, insertion, and deletion frequencies, keeping the per-site depth. Restrict to positions with sufficient read coverage so error statistics are estimable, and arrange features over the 5-mer context (−2, −1, 0, +1, +2) surrounding the candidate site. Write the table (e.g. epinano_features_per_site.tsv) with one row per transcriptomic position, ready for EpiNano SVM classification. Rationale: m6A manifests as altered per-base qualities and mismatch/deletion frequencies in base-called output, so these features carry the classification signal without needing raw current-intensity data.

## Related tools

- **EpiNano** (Feature-extraction toolkit that converts aligned direct RNA reads into per-base/per-kmer base-calling error and quality features for m6A SVM classification) — https://github.com/novoalab/EpiNano
- **Oxford Nanopore Technologies (ONT)** (Direct RNA sequencing platform whose base-called reads supply the quality, mismatch, insertion, and deletion signals)

## Evaluation signals

- Output table contains per-site mean per-base quality (A, C, G, T) and mismatch/insertion/deletion frequency columns for every covered position.
- Features are organized in per-base and per-kmer formats covering the 5-mer context (−2, −1, 0, +1, +2) around each candidate site.
- Positions retained all pass the read-coverage threshold, and per-site depth is retained alongside the error features.
- Downstream SVM trained on these features attains ~90% accuracy on synthetic m6A/unmodified sequences and ~87% in vivo (yeast).
- Comparison against ime4-knockout (m6A-lacking) yeast shows the detected 'errors' are not observed in the knockout strain.

## Limitations

- SVM predictions are only accurate if data are base-called with a matching base-caller version (Albacore 2.1.7 for EpiNano 1.0/1.1 models; Guppy 3.1.5 for EpiNano 1.2 pre-trained models).
- Direct RNA base-calling is error-rich even in unmodified sequences, so a matched control (WT vs KO/KD) is highly recommended for confident calls.
- Feature contributions vary across different k-mers, so k-mer-specific training is needed to improve accuracy and reduce false positives.
- The algorithm predicts m6A per site, not per individual RNA molecule, and does not distinguish between modification types (e.g., m1A vs m6A); only ~32% of known sites are recovered.
- EpiNano 1.1+ does not extract current intensity in the feature table because this feature was not used to train the final models.

## Evidence

- [methods] EpiNano compiles base-called features — base quality, mismatch frequency, and deletion frequency — for each position of the 5-mer: [evidence span withheld — anchor status: not_found]
- [discussion] These “errors” can be detected in the form of altered per-base qualities: "These “errors” can be detected in the form of altered per-base qualities, mismatch frequencies and deletion frequencies at the modified site."
- [readme] Regardless of the basecallers used, EpiNano can be used as a toolkit to extract per k-mer base-calling 'errors': [evidence span withheld — anchor status: external_doc]
- [readme] the SVM predictions will be only accurate if your data has been base-called using Albacore 2.1.7.: [evidence span withheld — anchor status: external_doc]
- [readme] EpiNano does not have per-read resolution.: [evidence span withheld — anchor status: external_doc]
