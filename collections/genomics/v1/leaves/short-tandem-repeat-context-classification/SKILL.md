---
name: short-tandem-repeat-context-classification
description: Use when you have a set of known truth-set variants (e.g. GIAB/1KGp3)
  that were missed by one or more sequencing technologies and need to quantify the
  fraction lying in homopolymer runs or STRs to explain the misses.
when_to_use_negative:
- Your goal is calling variants de novo from sequencing reads (use Clairvoyante/Clair-style
  callers instead of post-hoc context annotation)
- Variants already carry curated repeat/low-complexity annotations (e.g. from a truth-set
  confidence-region BED) — recounting contexts would duplicate that annotation
- The variant positions and reference FASTA are on mismatched genome builds — coordinate
  offsets would corrupt the extracted flanks
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Clairvoyante
  role: Deep neural network variant caller used to determine which GIAB/1KGp3 truth
    variants are missed by Illumina, PacBio, and ONT data, producing the undetected-variant
    sets whose contexts are classified
  repo: https://github.com/aquaskyline/Clairvoyante
- name: samtools/pysam
  role: faidx-style genome FASTA reader used to extract downstream reference sequence
    flanking each variant position for homopolymer/STR scanning
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-09025-z
    title: A multi-task convolutional deep neural network for variant calling in single
      molecule sequencing
  source_content_hash: sha256:f3e6ab1653ce48efa3e794ddcfe027965888cbcf67e2157edcbe87d3ab7d6285
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aquaskyline/Clairvoyante
  tools:
  - Clairvoyante
  - Illumina
  - Oxford Nanopore
  - PacBio
  - samtools/pysam
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# short-tandem-repeat-context-classification

## Summary

Annotates known germline variant positions by sequence context — homopolymer runs (≥4 identical consecutive bases) and short tandem repeats (≥3 units of a 1–6 bp motif) — to quantify why variants go undetected by combinations of Illumina, PacBio, and Oxford Nanopore callers.

## When to use

Use when you have a set of known truth-set variants (e.g. GIAB/1KGp3) that were missed by one or more sequencing technologies and need to quantify the fraction lying in homopolymer runs or STRs to explain the misses.

## When NOT to use

- Your goal is calling variants de novo from sequencing reads (use Clairvoyante/Clair-style callers instead of post-hoc context annotation)
- Variants already carry curated repeat/low-complexity annotations (e.g. from a truth-set confidence-region BED) — recounting contexts would duplicate that annotation
- The variant positions and reference FASTA are on mismatched genome builds — coordinate offsets would corrupt the extracted flanks

## Inputs

- VCF/TSV-style list of known truth-set variant positions (chrom, pos, ref, alt) flagged as undetected by a given combination of technologies (Illumina, PacBio, ONT)
- Build-matched reference genome FASTA (GRCh37 or GRCh38) with samtools .fai index
- samtools or pysam for faidx-style random FASTA access

## Outputs

- Per-variant context annotation table (e.g. homopolymer_str_classification.csv) with homopolymer-run and STR labels per variant
- Summary of homopolymer and STR fractions per undetected-variant set (e.g. 26.67% / 29.45% for the 12,331 all-technology-missed set)

## How to apply

Start from the undetected-variant sets derived from GIAB/1KGp3 truth variants — e.g. the 12,331 variants missed by Illumina, PacBio, and Oxford Nanopore callers simultaneously, and the 178,331 missed by both PacBio and ONT. For each variant position, extract the downstream reference flank from a build-matched FASTA (GRCh37 or GRCh38) using samtools faidx or a pysam FastaFile reader. Scan the extracted flanks with regex/tandem-repeat logic for homopolymer runs (≥4 identical consecutive bases) and STRs (≥3 repeat units of a 1–6 bp motif). Tabulate per-variant context labels and compute the fraction of variants in each context class; validate against published fractions (26.67% homopolymer / 29.45% STR of 12,331 all-technology-missed variants; 57.67% / 18.54% of 178,331 PacBio+ONT-missed variants), which are consistent with the elevated error rates of single-molecule sequencing in these contexts.

## Related tools

- **Clairvoyante** (Deep neural network variant caller used to determine which GIAB/1KGp3 truth variants are missed by Illumina, PacBio, and ONT data, producing the undetected-variant sets whose contexts are classified) — https://github.com/aquaskyline/Clairvoyante
- **samtools/pysam** (faidx-style genome FASTA reader used to extract downstream reference sequence flanking each variant position for homopolymer/STR scanning)

## Examples

```
import pysam; fa=pysam.FastaFile('GRCh37.fa'); seq=fa.fetch('chr1', pos, pos+50); re.findall(r'(\w)\1{3,}', seq)
```

## Evaluation signals

- Homopolymer fraction for the 12,331 variants missed by all three technologies reproduces ~26.67% and STR fraction ~29.45%
- PacBio+ONT-only missed set (178,331 variants) reproduces ~57.67% homopolymer and ~18.54% STR, showing a distinct context profile from the all-technology set
- Every variant in the output CSV carries exactly one context label (homopolymer, STR, both, or none) and downstream-flank extraction returns non-empty sequence for the build-matched reference
- Per-variant annotations are consistent between the summary fractions and the raw label counts (sum of labels / total variants equals the reported fractions)

## Limitations

- The underlying variant calls come from Clairvoyante-era SMS callers; Clairvoyante's ONT indel F1-score remains around 50%, so indel misses may be caller artifacts rather than true context-driven misses
- Clairvoyante ignores multi-allelic variants and cannot output the exact alternative allele for indels >4 bp, so some undetected variants reflect model scope rather than sequence context
- Only HG001 ONT data were available for this analysis, limiting cross-sample generality of the undetected-variant sets

## Evidence

- [abstract] we present 3,135 variants that are missed using Illumina but supported independently by both PacBio and Oxford Nanopore reads: "we present 3,135 variants that are missed using Illumina but supported independently by both PacBio and Oxford Nanopore reads"
- [discussion] we used as our truth dataset the latest dataset version 3.3.2 for HG001: "we used as our truth dataset the latest dataset version 3.3.2 for HG001"
- [readme] On NA12878, Clairvoyante achieved 99.73%, 97.68% and 95.36% accuracy on known variants, and achieved 98.65%, 92.57%, 87.26% F1 score on the whole genome, in Illumina, PacBio, and Oxford Nanopore data: "On NA12878, Clairvoyante achieved 99.73%, 97.68% and 95.36% accuracy on known variants, and achieved 98.65%, 92.57%, 87.26% F1 score on the whole genome, in Illumina, PacBio, and Oxford Nanopore data"
