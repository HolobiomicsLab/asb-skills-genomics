---
name: reference-genome-sequence-extraction
description: Use when you have a coordinate-indexed variant list (e.g. CHROM/POS from
  a GIAB/1KGp3 truth VCF) and need the surrounding reference bases from GRCh37 or
  GRCh38 to classify sequence context such as homopolymer runs (>=4 identical bases)
  or STRs (>=3 units of a 1-6 bp motif).
when_to_use_negative:
- The input is already an annotated sequence-context table and the goal is downstream
  statistics rather than sequence retrieval
- The reference genome build does not match the truth-variant coordinate build — extracted
  flanks would come from the wrong alleles/coordinates and inflate context mislabeling
- You need caller-level error modeling (e.g. Clairvoyante tensor extraction from BAMs)
  rather than static reference-sequence context classification
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Clairvoyante
  role: Source of the undetected-variant sets (variants its SMS/Illumina calling missed)
    whose flanking contexts are then extracted and classified
  repo: https://github.com/aquaskyline/Clairvoyante
- name: Illumina
  role: One of the three technologies whose simultaneously-missed variants (12,331)
    define a target undetected-variant set
- name: PacBio
  role: One of the three technologies whose simultaneously-missed variants (12,331)
    and the PacBio+ONT-missed set (178,331) define target undetected-variant sets
- name: Oxford Nanopore
  role: One of the three technologies whose simultaneously-missed variants (12,331)
    and the PacBio+ONT-missed set (178,331) define target undetected-variant sets
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-09025-z
    title: A multi-task convolutional deep neural network for variant calling in single
      molecule sequencing
  source_content_hash: sha256:42afddd9591d5454d84ab611107fce91d1bc0538dd9414d7327c43092679bc12
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aquaskyline/Clairvoyante
  tools:
  - Clairvoyante
  - Illumina
  - Oxford Nanopore
  - PacBio
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# reference-genome-sequence-extraction

## Summary

Extracts downstream reference sequence flanking each variant coordinate from a FASTA-indexed reference genome (GRCh37/GRCh38) so that sequence context — homopolymer runs and short tandem repeats — can be classified for undetected variant sets. This is the sequence-retrieval step that underpins context-based explanations of why SMS variant callers miss known truth variants.

## When to use

Use when you have a coordinate-indexed variant list (e.g. CHROM/POS from a GIAB/1KGp3 truth VCF) and need the surrounding reference bases from GRCh37 or GRCh38 to classify sequence context such as homopolymer runs (>=4 identical bases) or STRs (>=3 units of a 1-6 bp motif).

## When NOT to use

- The input is already an annotated sequence-context table and the goal is downstream statistics rather than sequence retrieval
- The reference genome build does not match the truth-variant coordinate build — extracted flanks would come from the wrong alleles/coordinates and inflate context mislabeling
- You need caller-level error modeling (e.g. Clairvoyante tensor extraction from BAMs) rather than static reference-sequence context classification

## Inputs

- Variant position list (e.g. CHROM/POS pairs from a VCF-derived undetected-variant set, such as 12,331 variants missed by Illumina+PacBio+ONT or 178,331 missed by PacBio+ONT)
- Build-matched reference genome FASTA (GRCh37 or GRCh38) with a samtools faidx .fai index

## Outputs

- Per-variant annotated table (e.g. homopolymer_str_classification.csv) with context labels per variant position
- Fraction summary: homopolymer and STR percentages per undetected-variant set

## How to apply

First confirm the reference build matches the truth set build (GRCh37 or GRCh38; the Clairvoyante study trained/evaluated against GIAB v3.3.2 HG001 truth variants on both builds, with 3,209,315 SNPs/225,097 insertions/245,552 deletions on GRCh37 and 3,042,789 SNPs/241,176 insertions/247,178 deletions on GRCh38). Index the reference FASTA with samtools faidx or open it via pysam.FastaFile, then for each variant position fetch the downstream flanking sequence (query by chromosome and start/end coordinates). Scan each extracted downstream flank for homopolymer runs (>=4 identical consecutive bases) and short tandem repeats (>=3 repeat units of a 1-6 bp motif) using regex/tandem-repeat scanning logic, tabulate per-variant context labels, and compute the fractions. As a sanity anchor, the reported analysis found 26.67% homopolymer and 29.45% STR among the 12,331 known variants undetected by all three technologies, and 57.67% homopolymer / 18.54% STR among the 178,331 variants missed by both PacBio and ONT.

## Related tools

- **Clairvoyante** (Source of the undetected-variant sets (variants its SMS/Illumina calling missed) whose flanking contexts are then extracted and classified) — https://github.com/aquaskyline/Clairvoyante
- **Illumina** (One of the three technologies whose simultaneously-missed variants (12,331) define a target undetected-variant set)
- **PacBio** (One of the three technologies whose simultaneously-missed variants (12,331) and the PacBio+ONT-missed set (178,331) define target undetected-variant sets)
- **Oxford Nanopore** (One of the three technologies whose simultaneously-missed variants (12,331) and the PacBio+ONT-missed set (178,331) define target undetected-variant sets)

## Examples

```
samtools faidx GRCh37.fa; python -c "import pysam; f=pysam.FastaFile('GRCh37.fa'); seq=f.fetch(chrom, pos, pos+50)"
```

## Evaluation signals

- Every variant position yields a non-empty flank string of the requested length from the correct chromosome; empty retrievals indicate coordinate/build mismatch
- Per-variant context labels are binary/multi-label consistent with the scan thresholds (>=4 identical consecutive bases for homopolymer; >=3 repeat units of a 1-6 bp motif for STR)
- Computed homopolymer/STR fractions for the 12,331-variant set reproduce the reported 26.67% / 29.45%; the 178,331-variant set reproduces 57.67% / 18.54%
- Reference build of the FASTA matches the truth-set build (GRCh37 vs GRCh38 allele conventions, decoy handling: reads were aligned to GRCh37 decoy version 5 in the study)
- Output CSV contains one row per input variant with position and context annotation columns, plus a summary block of the two fraction values per undetected-variant set

## Limitations

- Clairvoyante's ONT indel F1-score remains around 50%, so undetected-variant sets built from ONT caller output contain substantial residual caller error that context extraction cannot resolve
- Clairvoyante ignores variants with two or more alternative alleles and multi-allelic SNPs are excluded from training, so context classification is only as complete as the undetected-variant input set
- Downstream-flank scanning alone explains context but not causality; elevated SMS error rates in homopolymers/STRs are inferred, and base quality of reads was not considered in the Clairvoyante implementation

## Evidence

- [results] homopolymer and STR fractions for undetected-variant sets: [evidence span withheld — anchor status: not_found]
- [other] workflow flank extraction method: [evidence span withheld — anchor status: not_found]
- [other] scan thresholds: [evidence span withheld — anchor status: not_found]
- [discussion] reference builds used: "3,209,315 SNPs, 225,097 insertions, and 245,552 deletions for GRCh37"
- [discussion] read alignment to decoy reference: "Reads were extracted from the downloaded alignments and aligned again to GRCh37 decoy version 5 using NGMLR27 version 0.2.3"
