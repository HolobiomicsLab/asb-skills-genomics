---
name: ngram-disambiguation-of-overlapping-repeats
description: Use when multiple overlapping approximate repeat predictions compete
  on the same haplotype-specific consensus sequence and one motif/multiplicity assignment
  must be selected; also reference for RegEx-based TR genotyping design.
when_to_use_negative:
- Input reads are not haplotype-resolved — TRiCoLOR requires haplotype-resolved long-read
  alignments as input
- Target regions are unassembled (e.g., human centromeres and telomeres) — TRiCoLOR
  cannot identify repeats there
- Repeats have motifs longer than 6 bp (mini-satellites) — REFER defaults only cover
  micro-satellites and mini-satellite parameters are not validated
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: TRiCoLOR REFER
  role: Implements the RegEx-based approximate string matching, motif/multiplicity
    resolution, and N-gram disambiguation of competing overlapping TR predictions
  repo: https://github.com/davidebolo1993/TRiCoLOR/tree/master/TRiCoLOR
- name: TRiCoLOR
  role: 'Parent pipeline: runs SENSoR entropy scanning, POA consensus generation,
    and REFER profiling, producing the consensus sequences the N-gram model operates
    on'
  repo: https://github.com/davidebolo1993/TRiCoLOR
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giaa101
    title: 'TRiCoLOR: tandem repeat profiling using whole-genome long-read sequencing
      data'
  source_content_hash: sha256:0f611100356436d778c90445f7b81649fe4b344f57c2c325c1e66fa885e8c479
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - TRiCoLOR
  - TRiCoLOR REFER
  - minimap2
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# N-gram disambiguation of overlapping repeat predictions

## Summary

A step within TRiCoLOR REFER that resolves competing, overlapping approximate tandem-repeat predictions on haplotype-specific consensus sequences using an N-gram model that favors the most frequently occurring perfect repeat motif.

## When to use

Use when multiple overlapping approximate repeat predictions compete on the same haplotype-specific consensus sequence and one motif/multiplicity assignment must be selected; also reference for RegEx-based TR genotyping design.

## When NOT to use

- Input reads are not haplotype-resolved — TRiCoLOR requires haplotype-resolved long-read alignments as input
- Target regions are unassembled (e.g., human centromeres and telomeres) — TRiCoLOR cannot identify repeats there
- Repeats have motifs longer than 6 bp (mini-satellites) — REFER defaults only cover micro-satellites and mini-satellite parameters are not validated

## Inputs

- Haplotype-specific low-error consensus sequences from POA (built from haplotype-resolved ONT or PacBio long-read alignments)
- User-defined parameters: motif length ≤6 bp, minimum 5 perfect repeat units, edit distance ≤1, minimum TR length 50 bp

## Outputs

- Disambiguated tandem repeat calls: one structured record per TR with repeat motif, multiplicity, and start/end coordinates, stored in BCF-compliant format (e.g., tr_motif_calls.tsv)

## How to apply

After computing haplotype-specific low-error consensus sequences via partial order alignment (POA), first scan each consensus with a RegEx to identify motifs of length ≤6 bp that are perfectly repeated at least 5 times (REFER defaults). Then search for approximate repetitions of those motifs, tolerating imperfect repeated motifs up to a user-defined edit distance of ≤1 to absorb residual consensus errors from error-prone ONT or PacBio reads. When multiple overlapping approximate repetitions compete for the same region, apply an N-gram model that favors the most frequently occurring perfect repeat motif to select the winning motif and multiplicity assignment. Retain only TRs ≥50 bp and emit structured records (motif, multiplicity, start/end coordinates) in BCF-compliant format.

## Related tools

- **TRiCoLOR REFER** (Implements the RegEx-based approximate string matching, motif/multiplicity resolution, and N-gram disambiguation of competing overlapping TR predictions) — https://github.com/davidebolo1993/TRiCoLOR/tree/master/TRiCoLOR
- **TRiCoLOR** (Parent pipeline: runs SENSoR entropy scanning, POA consensus generation, and REFER profiling, producing the consensus sequences the N-gram model operates on) — https://github.com/davidebolo1993/TRiCoLOR

## Evaluation signals

- Every emitted record satisfies the ≥50 bp TR length threshold
- Every retained TR carries a motif of length ≤6 bp that was perfectly repeated at least 5 times in the consensus (mirroring REFER defaults)
- Each competing overlapping region resolves to exactly one motif/multiplicity assignment, consistent with the most frequently occurring perfect repeat motif
- Approximate repetitions accepted during matching do not exceed the user-defined edit distance of ≤1
- Output is BCF-compliant with motif, multiplicity, and start/end coordinates per TR, differing between haplotypes or the reference

## Limitations

- No support for polyploid species or haploid chromosomes (e.g., human Y chromosome)
- Repeats in unassembled regions (centromeres, telomeres) cannot be detected
- Edit-distance tolerance of ≤1 absorbs residual consensus errors, but heavier error rates may cause incorrect motif assignment even after N-gram disambiguation
- Parameter values for profiling mini-satellites are not validated; the tool has been extensively applied only to micro-satellites

## Evidence

- [intro] RegEx approximate matching precedes disambiguation: "processed by means of a fast regular expression (RegEx)-based approximate string-matching algorithm to resolve repeat motif and multiplicity of the discovered TRs"
- [methods] N-gram rule for competing predictions: "resolving these competing TR predictions using an N-gram model that favors the most frequently occurring perfect repeat motif"
- [methods] Edit distance tolerance for approximate repetitions: "imperfect repeated motifs up to a user-defined edit distance (≤1, by default)"
- [methods] Perfect-repeat seed identification constraints: "identifying motifs (motifs of length ≤6, by default) that are perfectly repeated a minimum number of times (5, by default)"
