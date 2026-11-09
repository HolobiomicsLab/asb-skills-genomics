---
name: modified-dna-source-detection-benchmarking
description: Use when benchmarking whether modified-DNA detection (dZ-DNA) can identify
  the taxonomic source organism in artificial metagenomes mixing dZ-positive and dZ-negative
  PacBio CCS reads.
when_to_use_negative:
- When the goal is single-nucleotide A/Z base discrimination within hybrid sequences
  — use the k11-mixed-AZ model instead of the read-level k21 pipeline
- When samples contain partially substituted (hybrid) dZ-DNA with diluted kinetic
  signatures rather than fully substituted dZ-genomes — the SVM read classifier is
  optimized for fully substituted dZ-genomes and misses hybrid reads
- When dZ-positive reads are extremely rare (e.g. only ~10 yeast hybrid reads) — metagenome
  7 with only 10 yeast hybrid reads was the sole false negative
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Z-Calling
  role: Machine-learning toolkit providing filter_bam.py, z-calling-base (k21 MLP,
    ZP tags) and z-calling-read predict (SVM k21ReadClassifier) for dZ-DNA read detection
  repo: https://github.com/xiaochuanle/Z-Calling
- name: minimap2
  role: Maps filtered reads (samtools fasta output) to the multi-organism reference
    with -x map-hifi; alignments with dv:f < 0.05 supply the read-to-contig source
    mapping
- name: pbmm2
  role: Aligns kinetic-filtered CCS reads to the multi-organism reference (--preset
    CCS --sort) before base and read classification
- name: PacBio Circular Consensus Sequencing (CCS)
  role: Data source generating BAM files with --keep-kinetics to preserve pulse width
    (PW) and inter-pulse duration (IPD) kinetic signals used by the classifiers
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-026-09849-8
    title: 'Z-Calling: a tool for A/Z (2,6-diaminopurine) base calling and dZ-DNA
      detection using PacBio HiFi reads'
  source_content_hash: sha256:e69a4b049f640642e054d147da06b8eec9ca1bab85d4d2cc03dcd126dc3c6c4e
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
  - Z-Calling
  - minimap2
  - pbmm2
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# modified-dna-source-detection-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark workflow for detecting and taxonomically attributing dZ-DNA (2,6-diaminopurine) sources in mixed metagenome samples using Z-Calling's read-level SVM classifier on PacBio HiFi kinetics, combined with minimap2 mapping and per-organism LR+ analysis.

## When to use

Use when benchmarking whether modified-DNA detection (dZ-DNA) can identify the taxonomic source organism in artificial metagenomes mixing dZ-positive and dZ-negative PacBio CCS reads.

## When NOT to use

- When the goal is single-nucleotide A/Z base discrimination within hybrid sequences — use the k11-mixed-AZ model instead of the read-level k21 pipeline
- When samples contain partially substituted (hybrid) dZ-DNA with diluted kinetic signatures rather than fully substituted dZ-genomes — the SVM read classifier is optimized for fully substituted dZ-genomes and misses hybrid reads
- When dZ-positive reads are extremely rare (e.g. only ~10 yeast hybrid reads) — metagenome 7 with only 10 yeast hybrid reads was the sole false negative

## Inputs

- Raw PacBio CCS BAM files with kinetic tags (fi, ri, fp, rp, fn, rn) for artificial metagenome mixtures of dZ-positive and dZ-negative reads
- Multi-organism reference FASTA and contig-to-species table (RefContig_Species.tsv)
- Pre-trained Z-Calling models: model/k21-full-ZA/scripted_m21.pth and model/svm/k21ReadClassifier

## Outputs

- Per-read classification TSV from z-calling-read predict (read name; '+' = dZ-DNA, '-' = canonical DNA)
- read_RefContig.tsv mapping read names to reference contigs (primary alignments with dv:f < 0.05)
- SVM.LR.tsv with per-organism LR+ scores indicating which organisms contributed dZ-DNA reads

## How to apply

Construct artificial metagenomes by mixing dZ-positive (e.g. yeast hybrid DNA with pRS426-ApPurZ-ApdATPase/pRS425-ApDUF550) and dZ-negative reads at known proportions. Filter each BAM with py/filter_bam.py, keeping reads with fi/ri/fp/rp kinetic tags present, Median_PW/Median_IPD ratio >= 0.3 and <= 99th percentile, and fn >= 3, rn >= 3; align with pbmm2 --preset CCS --sort. Run build/z-calling-base with the k21-full-ZA model (-k 21) to annotate ZP tags, then build/z-calling-read predict with model/svm/k21ReadClassifier (min_len 500, intervals 100) to produce the per-read '+'/'-' classification TSV. Convert the filtered BAM to FASTA with samtools fasta, map with minimap2 -t 8 -x map-hifi, keep primary read-to-contig pairs with dv:f < 0.05, and run py/SVM_LR_analysis.py with --SVM, --read_ref and --ref_species to compute per-organism LR+ (SVM.LR.tsv). Success is judged against the known mixture composition: metagenomes with >= 25 dZ-positive reads should be called positive with zero false positives.

## Related tools

- **Z-Calling** (Machine-learning toolkit providing filter_bam.py, z-calling-base (k21 MLP, ZP tags) and z-calling-read predict (SVM k21ReadClassifier) for dZ-DNA read detection) — https://github.com/xiaochuanle/Z-Calling
- **minimap2** (Maps filtered reads (samtools fasta output) to the multi-organism reference with -x map-hifi; alignments with dv:f < 0.05 supply the read-to-contig source mapping)
- **pbmm2** (Aligns kinetic-filtered CCS reads to the multi-organism reference (--preset CCS --sort) before base and read classification)
- **PacBio Circular Consensus Sequencing (CCS)** (Data source generating BAM files with --keep-kinetics to preserve pulse width (PW) and inter-pulse duration (IPD) kinetic signals used by the classifiers)

## Examples

```
python py/SVM_LR_analysis.py --SVM z-calling-read-Result.tsv --read_ref read_RefContig.tsv --ref_species RefContig_Species.tsv --output SVM.LR.tsv
```

## Evaluation signals

- All artificial metagenomes containing >= 25 dZ-positive (yeast hybrid) reads must be classified as dZ-DNA-positive
- Known false negative: metagenome 7 with only 10 yeast hybrid reads — consistent with zero false positive discovery in benchmarking
- Per-read TSV classifications must contain only '+', '-' or unclassified entries, with '+' denoting dZ-DNA
- read_RefContig.tsv should retain only primary pairs with dv:f < 0.05 (high-fidelity canonical alignments)
- LR+ output SVM.LR.tsv should attribute dZ-positive calls to the organism(s) known to be dZ-positive in the mixture and none of the canonical controls

## Limitations

- The SVM read classifier is optimized for fully substituted dZ-genomes and misses hybrid reads with diluted kinetic signatures
- Subtle Z-induced kinetic shift (0.5–1.5-fold) overlaps substantially with canonical Adenine variance, limiting hybrid sample quantification
- Detection requires a minimum number of dZ-positive reads: metagenomes with at least 25 yeast hybrid reads were detected, but 10 reads were insufficient (sole false negative)

## Evidence

- [results] Across 12 artificial metagenomes mixing dZ-positive and dZ-negative reads, Z-Calling's only false negative was metagenome 7 (which contained only 10 yeast hybrid reads): [evidence span withheld — anchor status: not_found]
