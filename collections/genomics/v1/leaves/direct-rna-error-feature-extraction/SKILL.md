---
name: direct-rna-error-feature-extraction
description: Use when ONT direct RNA sequencing reads (BAM aligned to a transcriptome,
  or Albacore/Guppy base-called FASTQ/FAST5) must be transformed into per-site error-feature
  tables for RNA modification (e.g. m6A) detection.
when_to_use_negative:
- Input is already an aggregated per-site or per-kmer feature table — proceed directly
  to SVM classification or error-difference scoring instead
- Goal is single-read (per-molecule) modification calling — EpiNano aggregates information
  from all reads mapping to a site and has no per-read resolution
- Analysis requires raw current-intensity squiggle realignment — this skill uses only
  base-called features, without squiggle realignment or manipulation of raw current
  intensity data
tools:
- name: EpiNano
  role: Toolkit that performs the sam2tsv-style CIGAR walk and per-site/per-kmer aggregation
    of base-calling errors (mismatch, insertion, deletion, quality, sumErr) into feature
    tables
  repo: https://github.com/novoalab/EpiNano
- name: Oxford Nanopore Technologies (ONT)
  role: Direct RNA sequencing platform producing the native RNA reads and base-called
    qualities/errors that are the source of the features
- name: samtools/pysam
  role: Loading the sorted, indexed BAM and accessing CIGAR strings, read bases and
    per-base qualities for per-position record emission
provenance:
  source_task_ids:
  - task_006
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-11713-9
    title: Accurate detection of m6A RNA modifications in native RNA sequences
  source_content_hash: sha256:db91601a0f7edc709d15ab1713b8558982e70eab1d08f3cba638822f7d1c9dfb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/novoalab/EpiNano
  tools:
  - EpiNano
  - Oxford Nanopore Technologies (ONT)
  - samtools/pysam
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# direct-rna-error-feature-extraction

## Summary

Converts Oxford Nanopore direct RNA sequencing alignments into per-site and per-kmer base-called error features — per-base quality scores, mismatch frequency, insertion frequency, deletion frequency and sumErr — which serve as the quantitative inputs EpiNano uses to detect m6A RNA modifications, exploiting the observation that modifications manifest as systematic base-calling errors and reduced base qualities.

## When to use

Use when ONT direct RNA sequencing reads (BAM aligned to a transcriptome, or Albacore/Guppy base-called FASTQ/FAST5) must be transformed into per-site error-feature tables for RNA modification (e.g. m6A) detection.

## When NOT to use

- Input is already an aggregated per-site or per-kmer feature table — proceed directly to SVM classification or error-difference scoring instead
- Goal is single-read (per-molecule) modification calling — EpiNano aggregates information from all reads mapping to a site and has no per-read resolution
- Analysis requires raw current-intensity squiggle realignment — this skill uses only base-called features, without squiggle realignment or manipulation of raw current intensity data

## Inputs

- Sorted, indexed BAM of ONT direct RNA reads aligned to a reference transcriptome (e.g. S. cerevisiae wild-type SAy841 and ime4Δ SAy966 direct RNA reads)
- Base-called FASTQ (Albacore or Guppy) and, for EpiNano 1.0, base-called Albacore FAST5 files for current-intensity features
- Reference transcriptome/FASTA used for alignment

## Outputs

- epinano_features_per_site.tsv — one row per transcriptomic position with position, coverage/depth, mean per-base quality for A/C/G/T, and mismatch/insertion/deletion frequencies
- Per-kmer feature table (features aggregated over the 5-mer context −2, −1, 0, +1, +2) with sumErr
- Feature matrix ready for EpiNano SVM scoring or pairwise EpiNano-Error delta-feature computation

## How to apply

Load the sorted, indexed BAM of direct RNA reads aligned to the reference transcriptome with samtools/pysam, and perform a sam2tsv-style walk of each read's CIGAR string, emitting one record per reference position containing the read base identity, base-calling quality, and match/mismatch/insertion/deletion status. For every transcriptomic position, aggregate across all mapping reads: compute the mean per-base quality for each of A/C/G/T, plus mismatch, insertion and deletion frequencies (and sumErr), retaining per-site read depth. Restrict the table to positions with sufficient coverage so error statistics are estimable; optionally organize features in per-kmer (e.g. 5-mer spanning −2 to +2 around the site) format. Write the result as epinano_features_per_site.tsv with one row per position — columns for position, coverage, mean per-base qualities and mismatch/insertion/deletion frequencies — ready for EpiNano SVM classification or pairwise EpiNano-Error comparison (WT vs KO/KD). Note that EpiNano 1.0 additionally extracted current intensity from Albacore FAST5 files, but this feature was dropped from 1.1 onward because it was not used to train the final models.

## Related tools

- **EpiNano** (Toolkit that performs the sam2tsv-style CIGAR walk and per-site/per-kmer aggregation of base-calling errors (mismatch, insertion, deletion, quality, sumErr) into feature tables) — https://github.com/novoalab/EpiNano
- **Oxford Nanopore Technologies (ONT)** (Direct RNA sequencing platform producing the native RNA reads and base-called qualities/errors that are the source of the features)
- **samtools/pysam** (Loading the sorted, indexed BAM and accessing CIGAR strings, read bases and per-base qualities for per-position record emission)

## Evaluation signals

- Output table contains, for every covered transcriptomic position, columns for per-site mean per-base quality (A/C/G/T) and mismatch/insertion/deletion frequencies, alongside per-site read depth
- Low-coverage positions are filtered out so per-site error statistics are estimable at retained sites
- Downstream EpiNano SVM trained on these features reaches the reported ~90% prediction accuracy on synthetic test sequences
- Applying features from yeast data reproduces in vivo performance: ~87.8% accuracy, 32% recovery (117/363) of known RRACH m6A sites at 89% specificity
- Extracted error signals at modified sites are typically absent in the ime4Δ control strain, which lacks m6A

## Limitations

- The algorithm aggregates information from all reads mapping to a site — it does not predict modifications in individual RNA molecules (single-read detection is future work)
- It cannot distinguish between different types of RNA modifications (e.g. m1A vs m6A)
- Direct RNA base-calling is error-rich, so unmodified sequences already show substantial errors; pairing with a control (WT vs KO/KD or demethylase-treated) is strongly recommended
- Feature contribution varies across k-mers, requiring k-mer-specific training to reduce false positives; SVM predictions are base-caller-version dependent (Albacore 2.1.7 or Guppy 3.1.5 models)
- Overall recovery of known sites is only ~32%; current-intensity features were dropped in EpiNano 1.1+, so only base-called error features are available in current versions

## Evidence

- [readme] per k-mer error features: [evidence span withheld — anchor status: external_doc]
- [readme] v1.0 feature set including current intensity: [evidence span withheld — anchor status: external_doc]
- [discussion] modification signal is error-shaped: "These “errors” can be detected in the form of altered per-base qualities, mismatch frequencies and deletion frequencies at the modified site."
- [discussion] no per-read resolution: "our current algorithm does not predict RNA modifications in individual RNA molecules, but rather employs information from all the reads mapping to a specific site"
- [abstract] detection principle: "using direct RNA sequencing, N 6 -methyladenosine (m 6 A) RNA modifications can be detected with high accuracy, in the form of systematic errors and decreased base-calling qualities"
