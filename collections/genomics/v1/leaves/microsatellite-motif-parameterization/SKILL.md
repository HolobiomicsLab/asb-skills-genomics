---
name: microsatellite-motif-parameterization
description: 'Use when de novo profiling tandem repeats in error-prone long-read (ONT
  or PacBio) alignments: after consensus generation, to assign each discovered TR
  its motif (≤6 bp), multiplicity (≥5 perfect repeats), and coordinates.'
when_to_use_negative:
- Detecting repeats in unassembled genomic regions such as human centromeres and telomeres
  — TRiCoLOR cannot identify TRs there
- Profiling tandem repeats in polyploid species or haploid chromosomes (e.g., the
  human Y chromosome) — not supported
- Profiling mini-satellites (larger repeat units) — parameter values have only been
  extensively validated for micro-satellites
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: TRiCoLOR REFER
  role: Module that identifies motifs of length ≤6 bp perfectly repeated ≥5 times
    and resolves approximate repetitions up to edit distance ≤1 with an N-gram model
    to parameterize the TR motif and multiplicity
  repo: https://github.com/davidebolo1993/TRiCoLOR/tree/master/TRiCoLOR
- name: TRiCoLOR
  role: 'Overall pipeline: computes haplotype-specific low-error consensus via POA
    and hands them to REFER for motif/multiplicity resolution'
  repo: https://github.com/davidebolo1993/TRiCoLOR
- name: minimap2
  role: Aligns the consensus sequences to the reference genome to obtain TR coordinates
- name: SPOA (POA)
  role: Partial order alignment producing the low-error consensus sequences (~77%
    ONT, ~88% PB error reduction) that REFER scans
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giaa101
    title: 'TRiCoLOR: tandem repeat profiling using whole-genome long-read sequencing
      data'
  source_content_hash: sha256:00214f3fe4a207db07023868dbe982c81c1ac3f54b0d77a6a922ebaa916e17f2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SPOA (POA)
  - TRiCoLOR
  - TRiCoLOR REFER
  - minimap2
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# microsatellite-motif-parameterization

## Summary

Resolves the repeat motif and multiplicity of tandem repeats discovered in haplotype-specific low-error consensus sequences using a fast regular expression (RegEx)-based approximate string-matching algorithm, as implemented in the TRiCoLOR REFER module.

## When to use

Use when de novo profiling tandem repeats in error-prone long-read (ONT or PacBio) alignments: after consensus generation, to assign each discovered TR its motif (≤6 bp), multiplicity (≥5 perfect repeats), and coordinates.

## When NOT to use

- Detecting repeats in unassembled genomic regions such as human centromeres and telomeres — TRiCoLOR cannot identify TRs there
- Profiling tandem repeats in polyploid species or haploid chromosomes (e.g., the human Y chromosome) — not supported
- Profiling mini-satellites (larger repeat units) — parameter values have only been extensively validated for micro-satellites

## Inputs

- Haplotype-specific low-error consensus sequences from long reads (computed via POA from error-prone ONT or PacBio reads)
- Haplotype-resolved long-read alignments (BAM) to the reference genome (e.g., GRCh38) via minimap2
- User-defined edit distance threshold for approximate repeat matching (default ≤1)

## Outputs

- Per-TR structured records with repeat motif (≤6 bp), multiplicity (number of repeat units), and start/end genomic coordinates, stored in BCF-compliant format
- Genotyped TRs ≥50 bp differing between haplotypes or the reference

## How to apply

Take haplotype-specific low-error consensus sequences produced by partial order alignment (POA/SPOA, ~77% ONT and ~88% PB error reduction) and align them to the reference genome with minimap2 to obtain coordinates. Scan each consensus with a RegEx identifying motifs of length ≤6 bp perfectly repeated at least 5 times (REFER defaults). Extend the match to approximate repetitions of the identified motif, tolerating imperfect repeats up to a user-defined edit distance ≤1 to absorb residual consensus errors. When overlapping approximate repetitions compete, resolve them with an N-gram model favoring the most frequently occurring perfect repeat motif, and retain only TRs ≥50 bp, writing motif, multiplicity, and start/end coordinates per record.

## Related tools

- **TRiCoLOR REFER** (Module that identifies motifs of length ≤6 bp perfectly repeated ≥5 times and resolves approximate repetitions up to edit distance ≤1 with an N-gram model to parameterize the TR motif and multiplicity) — https://github.com/davidebolo1993/TRiCoLOR/tree/master/TRiCoLOR
- **TRiCoLOR** (Overall pipeline: computes haplotype-specific low-error consensus via POA and hands them to REFER for motif/multiplicity resolution) — https://github.com/davidebolo1993/TRiCoLOR
- **minimap2** (Aligns the consensus sequences to the reference genome to obtain TR coordinates)
- **SPOA (POA)** (Partial order alignment producing the low-error consensus sequences (~77% ONT, ~88% PB error reduction) that REFER scans)

## Evaluation signals

- Every emitted TR record has a motif of length ≤6 bp perfectly repeated a minimum of 5 times (default REFER parameters)
- Every retained TR is ≥50 bp long (default length threshold)
- Approximate repeat matches respect the user-defined edit distance limit (≤1 by default) to remaining consensus errors
- Competing overlapping TR predictions within a locus resolve to the most frequently occurring perfect repeat motif under the N-gram model
- Genotyped TRs differ between haplotypes or the reference and are written in BCF-compliant format

## Limitations

- Repeats in unassembled regions of the genome (e.g., human centromeres and telomeres) cannot be identified
- No support for polyploid species or haploid chromosomes such as the human Y chromosome
- Tool has been extensively applied only to micro-satellites; mini-satellite profiling parameters are not validated
- Approximate matching tolerates only ≤1 edit distance by default, so higher residual consensus error rates may cause missed or mis-typed repeats

## Evidence

- [intro] RegEx-based approximate string matching resolves motif and multiplicity: "processed by means of a fast regular expression (RegEx)-based approximate string-matching algorithm to resolve repeat motif and multiplicity of the discovered TRs"
- [methods] REFER default parameters for motif length, repeat count, edit distance: "identifying motifs (motifs of length ≤6, by default) that are perfectly repeated a minimum number of times (5, by default)"
- [methods] Approximate repetitions tolerate residual consensus errors up to edit distance 1: "looking for approximate repetitions of the identified motifs to account for remaining consensus errors, i.e., imperfect repeated motifs up to a user-defined edit distance (≤1, by default)"
- [methods] N-gram model resolves competing overlapping TR predictions: "resolving these competing TR predictions using an N-gram model that favors the most frequently occurring perfect repeat motif"
- [methods] POA achieves large error reduction in consensus sequences: "to compute highly accurate consensus sequences with an approximate error reduction of ~77% and ~88% for ONT and PB, respectively"
- [methods] TRiCoLOR profiles TRs in haplotype-resolved BAM files via REFER: "TRiCoLOR can profile TRs in haplotype-resolved BAM files through the REFER (REpeats FindER) module"
- [abstract] TRiCoLOR identifies repeats without prior motif/location knowledge: "The method can identify repetitive regions in sequencing data without a prior knowledge of their motifs or locations and resolve repeat multiplicity and period size in a haplotype-specific manner."
- [discussion] Tool limited to micro-satellites; documentation available: "TRiCoLOR has been extensively applied so far only to micro-satellites"
- [readme] README points to source and documentation for installation and usage: "Please have a look at [TRiCoLOR documentation](https://davidebolo1993.github.io/tricolordoc/) for any installation or usage questions."
