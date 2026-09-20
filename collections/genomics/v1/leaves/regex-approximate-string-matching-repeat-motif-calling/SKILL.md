---
name: regex-approximate-string-matching-repeat-motif-calling
description: Use when profiling tandem repeats de novo from haplotype-resolved long-read
  alignments (ONT or PacBio BAM), or when re-tuning TRiCoLOR REFER via '--size' to
  detect longer motifs such as mini-satellites beyond the default micro-satellite
  settings.
when_to_use_negative:
- The goal is to call repeats in unassembled genomic regions such as centromeres and
  telomeres, which TRiCoLOR cannot access
- Input is a haploid chromosome (e.g., human Y) or a polyploid genome, since the tool
  assumes diploid haplotype resolution
- 'A validated ground-truth mini-satellite call set is required for benchmarking:
  the tool has been extensively applied only to micro-satellites, and mini-satellite
  parameterizations are unvalidated'
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: TRiCoLOR
  role: End-to-end tandem repeat profiler invoked to identify and genotype TRs de
    novo from haplotype-resolved long-read alignments
  repo: https://github.com/davidebolo1993/TRiCoLOR
- name: TRiCoLOR REFER
  role: Module running the RegEx-based exact and approximate motif matching, configurable
    via '--size' for longer motifs
  repo: https://github.com/davidebolo1993/TRiCoLOR
- name: TRiCoLOR SENSoR
  role: Upstream module producing the candidate repetitive-region BED file consumed
    by the RegEx matching stage
  repo: https://github.com/davidebolo1993/TRiCoLOR
- name: minimap2
  role: Aligns SPOA consensus sequences to the GRCh38 reference before approximate
    string matching
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giaa101
    title: 'TRiCoLOR: tandem repeat profiling using whole-genome long-read sequencing
      data'
  source_content_hash: sha256:14a510b2c481d23110e2aa5480b588b67ba8fe8e1d92faa45cb1198f565f99f8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/davidebolo1993/TRiCoLOR
  tools:
  - TRiCoLOR
  - TRiCoLOR REFER
  - TRiCoLOR SENSoR
  - minimap2
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# regex-approximate-string-matching-repeat-motif-calling

## Summary

Two-stage RegEx-based repeat resolution within TRiCoLOR's REFER module: first identify motifs perfectly repeated (default length ≤6 bp, ≥5 repeats) in haplotype-specific consensus sequences, then match approximate repetitions up to an edit distance ≤1 and resolve overlapping predictions with an N-gram model favoring the most frequent perfect repeat motif.

## When to use

Use when profiling tandem repeats de novo from haplotype-resolved long-read alignments (ONT or PacBio BAM), or when re-tuning TRiCoLOR REFER via '--size' to detect longer motifs such as mini-satellites beyond the default micro-satellite settings.

## When NOT to use

- The goal is to call repeats in unassembled genomic regions such as centromeres and telomeres, which TRiCoLOR cannot access
- Input is a haploid chromosome (e.g., human Y) or a polyploid genome, since the tool assumes diploid haplotype resolution
- A validated ground-truth mini-satellite call set is required for benchmarking: the tool has been extensively applied only to micro-satellites, and mini-satellite parameterizations are unvalidated

## Inputs

- BED file of candidate repetitive regions from TRiCoLOR SENSoR
- Haplotype-resolved BAM files of long-read alignments (ONT or PacBio) to GRCh38
- SPOA-derived haplotype-specific low-error consensus sequences
- Minimap2 alignments of consensus sequences to the GRCh38 reference

## Outputs

- BCF-compliant tandem repeat call set with repeat motif, multiplicity, and haplotype-specific genotypes
- BED files describing the identified TRs
- Haplotype-specific BAM files with aligned consensus sequences

## How to apply

Feed the SENSoR BED file of candidate low-entropy regions (Shannon entropy drop ≤1.23 in 20 bp sliding windows, ≥5 reads, merged within 100 bp) to TRiCoLOR REFER, which first scans SPOA-computed, haplotype-specific low-error consensus sequences for motifs perfectly repeated at least 5 times, overriding the default motif-length (≤6 bp) with '--size' when targeting longer motifs such as mini-satellites. The RegEx approximate string-matching stage then re-screens reference-aligned consensus sequences (aligned with minimap2 to GRCh38) for imperfect repetitions of the identified motifs up to a user-defined edit distance ≤1, accounting for residual consensus errors. Competing overlapping TR predictions are resolved with an N-gram model that favors the most frequently occurring perfect repeat motif. Retain only TRs ≥50 bp that vary between haplotypes or versus the reference, and write the call set in BCF-compliant format with accompanying BED files.

## Related tools

- **TRiCoLOR** (End-to-end tandem repeat profiler invoked to identify and genotype TRs de novo from haplotype-resolved long-read alignments) — https://github.com/davidebolo1993/TRiCoLOR
- **TRiCoLOR REFER** (Module running the RegEx-based exact and approximate motif matching, configurable via '--size' for longer motifs) — https://github.com/davidebolo1993/TRiCoLOR
- **TRiCoLOR SENSoR** (Upstream module producing the candidate repetitive-region BED file consumed by the RegEx matching stage) — https://github.com/davidebolo1993/TRiCoLOR
- **minimap2** (Aligns SPOA consensus sequences to the GRCh38 reference before approximate string matching)

## Examples

```
tricolor REFER --bam sample.hap1.bam --size 20 --out trs.bcf
```

## Evaluation signals

- Pipeline completes without errors and the output BCF and BED files exist and are non-empty (necessary since no ground-truth mini-satellite call set exists for this parameterization)
- Every reported TR is ≥50 bp and varies between haplotypes or versus the reference
- Overlapping predictions are resolved deterministically: each region carries the motif favored by the N-gram model (most frequently occurring perfect repeat)
- Detected approximate repeats should exceed the default perfect-repeat match, consistent with the edit distance ≤1 tolerance for residual consensus errors
- Called motifs conform to the '--size' setting (e.g., length ≥7 bp when overriding the ≤6 bp default for mini-satellites)

## Limitations

- The tool has been extensively applied only to micro-satellites; mini-satellite parameter values ('--size' overrides) are not validated against ground truth
- Repeats in unassembled regions of the genome (e.g., human centromeres and telomeres) cannot be identified
- No support for polyploid species or haploid chromosomes such as the human Y chromosome
- Exact entropy threshold and window size values for de novo identification were empirically estimated rather than validated

## Evidence

- [methods] identifying motifs (motifs of length ≤6, by default) that are perfectly repeated a minimum number of times (5, by default): "identifying motifs (motifs of length ≤6, by default) that are perfectly repeated a minimum number of times (5, by default)"
- [methods] looking for approximate repetitions of the identified motifs to account for remaining consensus errors, i.e., imperfect repeated motifs up to a user-defined edit distance (≤1, by default): "looking for approximate repetitions of the identified motifs to account for remaining consensus errors, i.e., imperfect repeated motifs up to a user-defined edit distance (≤1, by default)"
- [methods] resolving these competing TR predictions using an N-gram model that favors the most frequently occurring perfect repeat motif: "resolving these competing TR predictions using an N-gram model that favors the most frequently occurring perfect repeat motif"
- [methods] TRs (those ≥50 bp, by default) varying between the haplotypes or the reference are eventually stored in BCF-compliant format: "TRs (those ≥50 bp, by default) varying between the haplotypes or the reference are eventually stored in BCF-compliant format"
- [discussion] TRiCoLOR has been extensively applied so far only to micro-satellites: "TRiCoLOR has been extensively applied so far only to micro-satellites"
- [readme] TRiCoLOR is available to be installed from source or as a [Docker container](https://hub.docker.com/r/davidebolo1993/tricolor).: "TRiCoLOR is available to be installed from source or as a [Docker container](https://hub.docker.com/r/davidebolo1993/tricolor)."
