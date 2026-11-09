---
name: regex-based-tandem-repeat-motif-discovery
description: 'Use when genotyping tandem repeats de novo from error-prone long-read
  (ONT/PacBio) alignments: once haplotype-specific POA consensus sequences exist and
  candidate low-entropy regions must be decomposed into motif + multiplicity.'
when_to_use_negative:
- Target repeats in unassembled regions of the genome (e.g., human centromeres and
  telomeres) — TRiCoLOR cannot identify repeats there
- Profiling mini-satellites or polyploid/haploid (e.g., human Y chromosome) data —
  parameters validated only for micro-satellites and diploid genomes
- Short reads or non-haplotype-resolved alignments — the method requires haplotype-resolved
  long-read alignments as input
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: TRiCoLOR
  role: End-to-end tandem repeat profiler whose REFER module performs the RegEx-based
    approximate string-matching motif/multiplicity resolution
  repo: https://github.com/davidebolo1993/TRiCoLOR/tree/master/TRiCoLOR
- name: TRiCoLOR REFER
  role: Repeats FindER module that runs the RegEx scan for perfectly repeated motifs
    (≤6 bp, ≥5 repeats), approximate matching (edit distance ≤1), and N-gram disambiguation
  repo: https://github.com/davidebolo1993/TRiCoLOR/tree/master/TRiCoLOR
- name: minimap2
  role: Aligns haplotype-specific consensus sequences to the reference genome to obtain
    TR coordinates
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giaa101
    title: 'TRiCoLOR: tandem repeat profiling using whole-genome long-read sequencing
      data'
  source_content_hash: sha256:465a4e0c61d26cb63d11a24f3300a617eb91f3b82dd51d4947734433272c2771
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - TRiCoLOR
  - TRiCoLOR REFER
  - minimap2
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# regex-based-tandem-repeat-motif-discovery

## Summary

Resolves the repeat motif and multiplicity of tandem repeats from haplotype-specific low-error consensus sequences using a fast regular expression (RegEx)-based approximate string-matching algorithm, as implemented in the TRiCoLOR REFER module.

## When to use

Use when genotyping tandem repeats de novo from error-prone long-read (ONT/PacBio) alignments: once haplotype-specific POA consensus sequences exist and candidate low-entropy regions must be decomposed into motif + multiplicity.

## When NOT to use

- Target repeats in unassembled regions of the genome (e.g., human centromeres and telomeres) — TRiCoLOR cannot identify repeats there
- Profiling mini-satellites or polyploid/haploid (e.g., human Y chromosome) data — parameters validated only for micro-satellites and diploid genomes
- Short reads or non-haplotype-resolved alignments — the method requires haplotype-resolved long-read alignments as input

## Inputs

- Haplotype-specific low-error consensus sequences computed via POA (SPOA) from error-prone ONT or PacBio long reads
- Haplotype-resolved long-read alignments (BAM) aligned to the reference genome with minimap2
- User-defined parameters: motif length (≤6 bp default), minimum perfect repeats (5 default), edit distance (≤1 default), minimum TR length (≥50 bp default)

## Outputs

- Per-TR records with resolved repeat motif, multiplicity, and start/end coordinates (BCF-compliant format per the article; motif calls table)
- Genotyped TRs ≥50 bp varying between haplotypes or the reference

## How to apply

Take the haplotype-specific low-error consensus sequences computed with partial order alignment (SPOA, ~77% error reduction for ONT and ~88% for PB) and align them to the reference genome with minimap2 to obtain coordinates. Run a RegEx scan identifying motifs of length ≤6 bp that are perfectly repeated a minimum of 5 times (default REFER parameters). Extend matching to approximate repetitions of the identified motifs, tolerating imperfect repeated motifs up to a user-defined edit distance ≤1 to account for residual consensus errors. Where multiple overlapping approximate repetitions compete, resolve them with an N-gram model favoring the most frequently occurring perfect repeat motif, then retain TRs of length ≥50 bp that vary between haplotypes or the reference.

## Related tools

- **TRiCoLOR** (End-to-end tandem repeat profiler whose REFER module performs the RegEx-based approximate string-matching motif/multiplicity resolution) — https://github.com/davidebolo1993/TRiCoLOR/tree/master/TRiCoLOR
- **TRiCoLOR REFER** (Repeats FindER module that runs the RegEx scan for perfectly repeated motifs (≤6 bp, ≥5 repeats), approximate matching (edit distance ≤1), and N-gram disambiguation) — https://github.com/davidebolo1993/TRiCoLOR/tree/master/TRiCoLOR
- **minimap2** (Aligns haplotype-specific consensus sequences to the reference genome to obtain TR coordinates)

## Evaluation signals

- Every emitted TR record has motif length ≤6 bp with ≥5 perfect repeat units (mirroring default REFER parameters), checked programmatically on the output table
- Every retained TR satisfies the ≥50 bp length threshold and shows variation between haplotypes or versus the reference
- Approximate repetitions match within the user-defined edit distance ≤1 of the identified motif
- Competing overlapping TR predictions are resolved by the N-gram model toward the most frequently occurring perfect repeat motif
- Downstream: high validation rates on real human WGS data and improved sensitivity/specificity versus alternative tools on synthetic data

## Limitations

- TRiCoLOR cannot identify repeats in unassembled regions of the genome (e.g., human centromeres and telomeres)
- No support yet for polyploid species or haploid chromosomes (e.g., human Y chromosome)
- Parameter values for profiling mini-satellites are not validated; the tool has been extensively applied so far only to micro-satellites
- Residual consensus errors after POA must still be tolerated via the edit-distance-≤1 approximate matching

## Evidence

- [intro] RegEx approximate string matching resolves motif/multiplicity: [evidence span withheld — anchor status: not_found]
- [methods] Perfect motif repetition criteria: "identifying motifs (motifs of length ≤6, by default) that are perfectly repeated a minimum number of times (5, by default)"
- [methods] Approximate repetitions tolerance: "looking for approximate repetitions of the identified motifs to account for remaining consensus errors, i.e., imperfect repeated motifs up to a user-defined edit distance (≤1, by default)"
- [methods] N-gram disambiguation: "resolving these competing TR predictions using an N-gram model that favors the most frequently occurring perfect repeat motif"
- [methods] TR length retention filter: "TRs (those ≥50 bp, by default) varying between the haplotypes or the reference are eventually stored in BCF-compliant format"
- [methods] Consensus error reduction rationale: "to compute highly accurate consensus sequences with an approximate error reduction of ~77% and ~88% for ONT and PB, respectively"
- [readme] README citation pointer: "TRiCoLOR: tandem repeat profiling using whole-genome long-read sequencing data"
