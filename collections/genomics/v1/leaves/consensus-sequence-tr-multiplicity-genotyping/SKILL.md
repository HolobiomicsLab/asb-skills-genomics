---
name: consensus-sequence-tr-multiplicity-genotyping
description: Use when profiling de novo tandem repeats from haplotype-resolved ONT
  or PacBio long-read alignments and the repeat motif and multiplicity must be resolved
  per haplotype from low-error consensus sequences.
when_to_use_negative:
- Repeats in unassembled genome regions (e.g., human centromeres and telomeres) —
  TRiCoLOR cannot identify repeats there
- Polyploid species or haploid chromosomes such as the human Y chromosome — the diploid,
  haplotype-resolved model does not support them
- Mini-satellite profiling — TRiCoLOR has been extensively applied only to micro-satellites
  and mini-satellite parameters are not validated
edam_operation: http://edamontology.org/operation_3199
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3170
tools:
- name: TRiCoLOR REFER
  role: RegEx-based approximate string-matching module that resolves repeat motif
    and multiplicity from haplotype-specific consensus sequences
  repo: https://github.com/davidebolo1993/TRiCoLOR
- name: SPOA
  role: Partial order alignment engine that computes the haplotype-specific low-error
    consensus sequences (~77%/~88% error reduction for ONT/PB) fed to the RegEx matcher
- name: minimap2
  role: Aligns ONT/PB reads to the reference genome (e.g., GRCh38), supplying the
    haplotype-resolved alignments and coordinates underlying the TR records
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giaa101
    title: 'TRiCoLOR: tandem repeat profiling using whole-genome long-read sequencing
      data'
  source_content_hash: sha256:dd5fcf13de5c282423c2715b117b7fdfee137421e7f78a75412893b6c959a656
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/davidebolo1993/TRiCoLOR
  tools:
  - SPOA
  - TRiCoLOR
  - TRiCoLOR REFER
  - minimap2
  edam_operation: http://edamontology.org/operation_3199
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# consensus-sequence-tr-multiplicity-genotyping

## Summary

Resolves the repeat motif and multiplicity of discovered tandem repeats from haplotype-specific low-error consensus sequences using TRiCoLOR REFER's fast RegEx-based approximate string-matching algorithm, with default parameters of motifs of length ≤6 bp perfectly repeated ≥5 times, edit distance ≤1, and TRs ≥50 bp stored in BCF-compliant format.

## When to use

Use when profiling de novo tandem repeats from haplotype-resolved ONT or PacBio long-read alignments and the repeat motif and multiplicity must be resolved per haplotype from low-error consensus sequences.

## When NOT to use

- Repeats in unassembled genome regions (e.g., human centromeres and telomeres) — TRiCoLOR cannot identify repeats there
- Polyploid species or haploid chromosomes such as the human Y chromosome — the diploid, haplotype-resolved model does not support them
- Mini-satellite profiling — TRiCoLOR has been extensively applied only to micro-satellites and mini-satellite parameters are not validated

## Inputs

- Haplotype-specific low-error consensus sequences computed via partial order alignment (POA/SPOA) from error-prone long reads
- Haplotype-resolved long-read alignments (BAM, e.g., ONT reads aligned to GRCh38 with minimap2) providing reference coordinates
- User-defined parameters: motif length ≤6, minimum 5 perfect repeat units, edit distance ≤1, minimum TR length ≥50 bp

## Outputs

- Per-TR records with resolved repeat motif, multiplicity, and haplotype-specific start/end coordinates for TRs ≥50 bp, stored in BCF-compliant format

## How to apply

Start from haplotype-specific low-error consensus sequences produced by partial order alignment (SPOA), which reduces ONT and PB consensus error by ~77% and ~88%, respectively. Run a RegEx scan identifying motifs of length ≤6 bp (by default) that are perfectly repeated a minimum of 5 times (by default), then search for approximate repetitions of the identified motifs up to a user-defined edit distance ≤1 to absorb remaining consensus errors. When multiple overlapping approximate repetitions compete, resolve them with an N-gram model favoring the most frequently occurring perfect repeat motif. Retain only TRs ≥50 bp (by default) that vary between haplotypes or the reference, and emit one record per TR carrying motif, multiplicity, and coordinates in BCF-compliant format. Treat motif length, minimum perfect repeats, edit distance, and TR length as the tunable parameters mirroring the REFER defaults of 6 bp, 5, 1, and 50 bp.

## Related tools

- **TRiCoLOR REFER** (RegEx-based approximate string-matching module that resolves repeat motif and multiplicity from haplotype-specific consensus sequences) — https://github.com/davidebolo1993/TRiCoLOR
- **SPOA** (Partial order alignment engine that computes the haplotype-specific low-error consensus sequences (~77%/~88% error reduction for ONT/PB) fed to the RegEx matcher)
- **minimap2** (Aligns ONT/PB reads to the reference genome (e.g., GRCh38), supplying the haplotype-resolved alignments and coordinates underlying the TR records)

## Examples

```
docker pull davidebolo1993/tricolor
```

## Evaluation signals

- Every emitted TR record is ≥50 bp, mirroring the default REFER length threshold
- Each resolved motif is ≤6 bp and was perfectly repeated ≥5 times before approximate matching was applied
- Approximate repetitions respect the user-defined edit distance (≤1 by default); calls exceeding it indicate misconfiguration or excessive residual consensus error
- Competing overlapping TR predictions are resolved deterministically by the N-gram model favoring the most frequently occurring perfect repeat motif
- Output is BCF-compliant and carries motif, multiplicity, and haplotype-specific coordinates for every retained TR

## Limitations

- TRiCoLOR cannot identify repeats in unassembled regions of the genome (e.g., human centromeres and telomeres)
- No support for polyploid species or haploid chromosomes (e.g., the human Y chromosome)
- Extensively applied only to micro-satellites; parameter values for profiling mini-satellites are not validated
- Accuracy depends on prior POA error reduction (~77% for ONT, ~88% for PB); residual consensus errors beyond the edit-distance tolerance can break motif matching

## Evidence

- [intro] The RegEx-based approximate string-matching algorithm resolves repeat motif and multiplicity of discovered TRs: "processed by means of a fast regular expression (RegEx)-based approximate string-matching algorithm to resolve repeat motif and multiplicity of the discovered TRs"
- [methods] Default REFER scan: motifs of length ≤6 perfectly repeated ≥5 times: "identifying motifs (motifs of length ≤6, by default) that are perfectly repeated a minimum number of times (5, by default)"
- [methods] Approximate repetitions tolerate imperfect motifs up to edit distance ≤1 to account for residual consensus errors: "looking for approximate repetitions of the identified motifs to account for remaining consensus errors, i.e., imperfect repeated motifs up to a user-defined edit distance (≤1, by default)"
- [methods] Competing overlapping TR predictions are resolved by an N-gram model favoring the most frequent perfect repeat motif: "resolving these competing TR predictions using an N-gram model that favors the most frequently occurring perfect repeat motif"
- [methods] TRs ≥50 bp varying between haplotypes or reference are stored in BCF-compliant format: "TRs (those ≥50 bp, by default) varying between the haplotypes or the reference are eventually stored in BCF-compliant format"
- [methods] POA consensus achieves ~77% (ONT) and ~88% (PB) error reduction before genotyping: "to compute highly accurate consensus sequences with an approximate error reduction of ~77% and ~88% for ONT and PB, respectively"
- [discussion] Repeats in unassembled regions such as centromeres and telomeres cannot be detected: "TRiCoLOR cannot identify repeats in unassembled regions of the genome (e.g., human centromeres and telomeres)"
- [discussion] No support for polyploid species or haploid chromosomes such as the human Y chromosome: "future work includes extending its feature set to polyploid species and haploid chromosomes (human Y chromosome)"
- [readme] TRiCoLOR is the tandem repeats profiler for long-read sequencing data available as source and Docker container: "TRiCoLOR: tandem repeats profiler for long-read sequencing data"
