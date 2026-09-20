---
name: nucleotide-variation-polymorphism-table-annotation
description: 'Use when refining SNP candidates from nanopore MinION read alignments:
  after the transition/transversion weighted-probability filter has produced candidates
  (alternate-nucleotide coverage >= 20% of reads and >= 30% of maximum contig coverage),
  to attach a per-candidate alignment-reliability score.'
when_to_use_negative:
- Input is raw FASTQ/FASTA MinION reads that have not yet been aligned with LAST and
  passed the SNP candidate coverage filters — run the earlier NanoPipe stages first
- The analysis goal is consensus calling or IUPAC ambiguity assignment (20% count-difference
  rule), not per-SNP alignment-reliability scoring
- Data are metagenomic; the article notes metagenomics analysis is not yet supported
  by NanoPipe
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3577
tools:
- name: NanoPipe
  role: Pipeline stage that computes the p-error per SNP candidate from LAST alignment
    scores and attaches it to the polymorphisms table
  repo: https://github.com/IOB-Muenster/nanopipe2
- name: LAST
  role: Sequence aligner (version 946) whose per-base reliability scores (ASCII quality
    symbols) supply the error probabilities aggregated into the p-error
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giy169
    title: NanoPipe—a web server for nanopore MinION sequencing data analysis
  source_content_hash: sha256:4c14daf953684dd4dfd15b10d9e1803108d96bffac13b7a44aea347c3b13611e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IOB-Muenster/nanopipe2
  tools:
  - LAST
  - NanoPipe
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# nucleotide-variation-polymorphism-table-annotation

## Summary

Annotate NanoPipe's weighted SNP candidate table with a p-error column that quantifies local alignment quality around each polymorphism candidate, computed from LAST per-base quality symbols in a ±10 nt window around the SNP. Higher p-error (max 1) flags candidates whose support may stem from local sequencing errors.

## When to use

Use when refining SNP candidates from nanopore MinION read alignments: after the transition/transversion weighted-probability filter has produced candidates (alternate-nucleotide coverage >= 20% of reads and >= 30% of maximum contig coverage), to attach a per-candidate alignment-reliability score to the polymorphisms table.

## When NOT to use

- Input is raw FASTQ/FASTA MinION reads that have not yet been aligned with LAST and passed the SNP candidate coverage filters — run the earlier NanoPipe stages first
- The analysis goal is consensus calling or IUPAC ambiguity assignment (20% count-difference rule), not per-SNP alignment-reliability scoring
- Data are metagenomic; the article notes metagenomics analysis is not yet supported by NanoPipe

## Inputs

- Weighted SNP candidate table (calc.nuccounts.1.poly precursor) from the transition/transversion weighted-probability filter
- LAST v946 alignment output containing per-base reliability scores (ASCII quality symbols) for MinION reads mapped against the target

## Outputs

- Refined polymorphisms table (calc.nuccounts.1.poly) with a p-error column, one value in [0,1] per SNP candidate row

## How to apply

Start from the weighted SNP candidate table produced after the transition/transversion filter (transitions weighted 2, transversions 1; alternate-nucleotide coverage >= 20% and >= 30% of maximum contig coverage). Extract LAST reliability scores (ASCII quality symbols) for every aligned base pair from the LAST v946 alignment output. For each SNP candidate, collect aligned bases within up to 10 nucleotides before and after the SNP position (excluding the SNP itself) across all reads mapped to that region, giving i = N(nucleotides around SNP) x N(reads mapped to the region). Compute p-error = (1/i) * sum of 10^(-(ASCII_value - 33)/10) over the collected bases, then attach it as a quality column to the polymorphisms table and write the refined table to calc.nuccounts.1.poly. The rationale is that a higher p-error indicates lower local alignment quality, so candidates in noisy regions deserve lower trust.

## Related tools

- **NanoPipe** (Pipeline stage that computes the p-error per SNP candidate from LAST alignment scores and attaches it to the polymorphisms table) — https://github.com/IOB-Muenster/nanopipe2
- **LAST** (Sequence aligner (version 946) whose per-base reliability scores (ASCII quality symbols) supply the error probabilities aggregated into the p-error)

## Examples

```
bash NANOPIPE/check.sh  # verify NanoPipe/LAST installation, then run the NanoPipe pipeline so calc.nuccounts.1.poly is written with the p-error quality column
```

## Evaluation signals

- calc.nuccounts.1.poly exists and contains a p-error entry for every SNP candidate row
- Every p-error value lies in [0, 1] (each per-base term 10^(-(ASCII-33)/10) is at most 1 since ASCII_value >= 33)
- Candidates in high-quality alignment regions show p-error near 0, while candidates flanked by low LAST quality symbols show p-error closer to 1
- The number of table rows is unchanged relative to the pre-filter weighted SNP candidate table (annotation only, no candidate removal)
- Downstream polymorphisms reporting still reproduces the expected mutations, e.g. in the Plasmodium falciparum 3D7 validation case where NanoPipe mapped 99.9% of reads and detected the expected mutations

## Limitations

- The p-error window excludes the SNP base itself (up to 10 nt on each side), so a candidate's own quality symbol does not contribute to its score
- Current SNP detection statistics are not yet sophisticated; the authors state more sophisticated statistics for SNP detection and evaluation are planned future work
- The p-error reflects local alignment quality only and is itself an estimate of sequencing-error influence, not a definitive candidate verdict

## Evidence

- [methods] an assessment of the alignment quality around a potential SNP by estimating a p-error: "an assessment of the alignment quality around a potential SNP by estimating a p-error"
- [methods] transitions/transversion weighting precedes p-error: "transversions are weighted by the factor of 1 and transitions by the factor of 2"
- [methods] mapping with LAST version 946: "mapping of sequencing reads to the target using version 946 of the LAST sequence aligner"
- [discussion] planned improvement of SNP statistics: "The next step for the NanoPipe project will include more sophisticated statistics for SNP detection and evaluation."
- [readme] README repo reference: "NanoPipe analyzes reads generated by the Oxford Nanopore sequencing devices. As a result, it provides alignments to any target of interest, alignment statistics and information about polymorphisms."
