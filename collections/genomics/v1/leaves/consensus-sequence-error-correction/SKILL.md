---
name: consensus-sequence-error-correction
description: Use when error-prone haplotype-resolved ONT or PacBio long reads span
  candidate repetitive regions (e.g., SENSoR BED output) and low-error consensus sequences
  are needed before motif detection.
when_to_use_negative:
- Repeats lie in unassembled genomic regions such as human centromeres or telomeres,
  where spanning reads cannot be fetched
- Input is short-read or high-accuracy data where POA consensus adds little value
  and the method's error-reduction rationale (~77% ONT, ~88% PB on error-prone reads)
  does not apply
- Samples are polyploid or haploid (e.g., the human Y chromosome), which TRiCoLOR
  does not support
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: SPOA
  role: partial order alignment tool used to compute haplotype-specific low-error
    consensus sequences from spanning reads
- name: minimap2
  role: realigns the generated consensus sequences to the reference genome for repeat
    detection
- name: TRiCoLOR REFER
  role: module that orchestrates read fetching, trimming, consensus building, realignment,
    and motif detection, storing TRs in BCF format
  repo: https://github.com/davidebolo1993/TRiCoLOR
- name: TRiCoLOR SENSoR
  role: upstream module producing the BED of candidate low-entropy repetitive regions
    that feed consensus building
  repo: https://github.com/davidebolo1993/TRiCoLOR
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giaa101
    title: 'TRiCoLOR: tandem repeat profiling using whole-genome long-read sequencing
      data'
  source_content_hash: sha256:9977735140adf29d4e4d5a5a52d6b6324998eb5b28595581b9adbaba51c115b6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/davidebolo1993/TRiCoLOR
  tools:
  - NCRF
  - SPOA
  - TRiCoLOR
  - TRiCoLOR REFER
  - TRiCoLOR SENSoR
  - minimap2
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# consensus-sequence-error-correction

## Summary

Compute haplotype-specific low-error consensus sequences from error-prone ONT or PacBio long reads spanning candidate tandem repeats, using partial order alignment (SPOA), so that repeat motifs and multiplicity can be resolved downstream by approximate string matching.

## When to use

Use when error-prone haplotype-resolved ONT or PacBio long reads span candidate repetitive regions (e.g., SENSoR BED output) and low-error consensus sequences are needed before motif detection.

## When NOT to use

- Repeats lie in unassembled genomic regions such as human centromeres or telomeres, where spanning reads cannot be fetched
- Input is short-read or high-accuracy data where POA consensus adds little value and the method's error-reduction rationale (~77% ONT, ~88% PB on error-prone reads) does not apply
- Samples are polyploid or haploid (e.g., the human Y chromosome), which TRiCoLOR does not support

## Inputs

- haplotype-resolved long-read BAM files (ONT or PacBio, from minimap2 alignments)
- BED file of candidate low-entropy repetitive regions from TRiCoLOR SENSoR
- sequencing reads spanning the candidate repeat regions

## Outputs

- haplotype-specific low-error consensus sequences per candidate repeat
- TR calls with resolved motif and repeat multiplicity stored in BCF-compliant format

## How to apply

Within the TRiCoLOR REFER module, fetch reads spanning each low-entropy candidate region identified by SENSoR (Shannon entropy drop <= 1.23, >= 5 supporting reads), trim them, and build haplotype-specific consensus sequences with SPOA partial order alignment, which reduces raw consensus error by approximately 77% for ONT and 88% for PB reads. Realign the consensus sequences to the reference with minimap2, then run the RegEx-based approximate string-matching step to detect motifs of length <= 6 bp that are perfectly repeated >= 5 times, allowing imperfect repeats up to an edit distance of 1 to absorb remaining consensus errors. Resolve competing overlapping TR predictions with the N-gram model favoring the most frequently occurring perfect repeat motif, and retain only TRs >= 50 bp, storing calls in BCF format.

## Related tools

- **SPOA** (partial order alignment tool used to compute haplotype-specific low-error consensus sequences from spanning reads)
- **minimap2** (realigns the generated consensus sequences to the reference genome for repeat detection)
- **TRiCoLOR REFER** (module that orchestrates read fetching, trimming, consensus building, realignment, and motif detection, storing TRs in BCF format) — https://github.com/davidebolo1993/TRiCoLOR
- **TRiCoLOR SENSoR** (upstream module producing the BED of candidate low-entropy repetitive regions that feed consensus building) — https://github.com/davidebolo1993/TRiCoLOR

## Evaluation signals

- Consensus error reduction should approximate ~77% for ONT and ~88% for PB input reads
- Detected motifs must have length <= 6 bp and be perfectly repeated >= 5 times, with imperfect repeats allowed only up to edit distance <= 1
- Reported TRs must be >= 50 bp and stored in valid BCF-compliant format
- Downstream genotyping fidelity: F1 close to 1 at 1-motif discrepancy threshold on simulated truth sets
- Realignments of consensus sequences with minimap2 must map unambiguously for TR detection to proceed

## Limitations

- Cannot profile repeats in unassembled regions of the genome (e.g., human centromeres and telomeres)
- No support for polyploid species or haploid chromosomes (e.g., the human Y chromosome)
- Profiling of mini-satellites (motifs > 6 bp) is not validated; the tool has been extensively applied only to micro-satellites

## Evidence

- [intro] TRiCoLOR exploits partial order alignment (POA) to compute haplotype-specific low-error consensus sequences: "TRiCoLOR exploits partial order alignment (POA) [14] to compute haplotype-specific low-error consensus sequences"
- [methods] SPOA consensus error reduction rates for ONT and PB: "to compute highly accurate consensus sequences with an approximate error reduction of ~77% and ~88% for ONT and PB, respectively"
- [methods] REFER module profiles TRs in haplotype-resolved BAM files: "TRiCoLOR can profile TRs in haplotype-resolved BAM files through the REFER (REpeats FindER) module"
- [methods] Minimum TR length and BCF output criteria: "TRs (those ≥50 bp, by default) varying between the haplotypes or the reference are eventually stored in BCF-compliant format"
