---
name: small-indel-size-thresholding
description: Use when a NanoSV VCF from long-read (MinION nanopore) data contains
  indel calls and you must exclude sub-40 bp indels that are not genuine split-alignment
  structural variants.
when_to_use_negative:
- When calling short indels (<40 bp) themselves — this skill removes them by design,
  since they are not typically split-aligned; use a dedicated small-variant caller
  instead.
- When the input is not a long-read (MinION/PacBio) NanoSV-style call set — split-alignment-based
  size thresholds are not calibrated for short-read indel callers such as those applied
  to Illumina data.
- When a downstream analysis specifically targets small indel events; this filter
  intentionally discards them as out of SV scope.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_0625
tools:
- name: NanoSV
  role: Produces the initial long-read SV call set (split- and gapped-read clustering)
    whose indel calls are subjected to the 40 bp size-threshold filter
  repo: mroosmalen/nanosv
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-017-01343-4
    title: Mapping and phasing of structural variation in patient genomes using nanopore
      sequencing
  source_content_hash: sha256:c213c7c3a66765280d307172d6d9c5220be33a469e4d5bb2714a5197806c194f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mroosmalen/nanosv
  tools:
  - NanoSV
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# small-indel-size-thresholding

## Summary

A post-calling filter that removes indel calls smaller than a 40 base pair threshold from a NanoSV structural-variant call set, because indels below this size typically do not produce split-read alignments and therefore fall outside NanoSV's SV detection regime. Applied after repeat-region filtering, it reduced the NA12878 chromosome 1 call set from 657 to 654 SVs.

## When to use

Use when a NanoSV VCF from long-read (MinION nanopore) data contains indel calls and you must exclude sub-40 bp indels that are not genuine split-alignment structural variants.

## When NOT to use

- When calling short indels (<40 bp) themselves — this skill removes them by design, since they are not typically split-aligned; use a dedicated small-variant caller instead.
- When the input is not a long-read (MinION/PacBio) NanoSV-style call set — split-alignment-based size thresholds are not calibrated for short-read indel callers such as those applied to Illumina data.
- When a downstream analysis specifically targets small indel events; this filter intentionally discards them as out of SV scope.

## Inputs

- NanoSV VCF of SV calls from long-read nanopore data (e.g. 3957 initial chr1 NA12878 calls)
- Homopolymer and tandem-repeat (STR) region annotations for breakpoint overlap filtering (BED-like regions)

## Outputs

- Intermediate repeat-filtered VCF with 657 SV calls
- Final filtered VCF (e.g. nanosv_na12878_filtered.vcf) with 654 SV calls after removing indels <40 bp

## How to apply

Load the NanoSV VCF (e.g. the NA12878 chromosome 1 call set with 3957 initial structural-variant calls) and parse each variant's coordinates, type, and size. First discard calls whose breakpoints fall within homopolymer or short-tandem-repeat region annotations, retaining the intermediate call set (expected 657 calls). From that repeat-filtered call set, exclude indel calls smaller than the 40 bp SV size threshold, yielding the final call set (expected 654 calls). The rationale is that indels below 40 bp do not typically result in split alignments, so they are not reliably detectable or classifiable by NanoSV's split- and gapped-read clustering algorithm. Write the final set to a VCF (e.g. nanosv_na12878_filtered.vcf) and verify row counts at both filter stages.

## Related tools

- **NanoSV** (Produces the initial long-read SV call set (split- and gapped-read clustering) whose indel calls are subjected to the 40 bp size-threshold filter) — mroosmalen/nanosv

## Evaluation signals

- The repeat-filter stage retains exactly 657 calls from the initial 3957, matching the reported intermediate count.
- The indel-size filter yields a final call set of exactly 654 calls, matching the paper's reported cascade.
- No variant in the final VCF is an indel with size <40 bp.
- No retained call has a breakpoint falling within a homopolymer or short-tandem-repeat region annotation.
- The final output parses as a valid VCF with consistent coordinate, type, and size fields per record.

## Limitations

- The 40 bp cutoff excludes sub-40 bp indels from the SV set entirely; these are out of scope rather than invalidated.
- Extremely stringent post-calling filtering lowers estimated sensitivity (~75%), so some true SVs are lost by the conservative cascade.
- Medium sequencing coverage and intrinsic nanopore sequencing biases limit overall detection, particularly in complex repeat regions and segmental duplications.

## Evidence

- [readme] NanoSV is a software package that can be used to identify structural genomic variations in long-read sequencing data: "NanoSV is a software package that can be used to identify structural genomic variations in long-read sequencing data"
- [readme] split- and gapped-aligned reads clustering: "The core algorithm of NanoSV identifies split- and gapped-aligned reads and clusters the reads according to the orientations"
- [discussion] stringent filtering and ~75% sensitivity: "we employ extremely stringent filtering that results in a good estimated sensitivity (~75%)"
- [discussion] conservative calling and post-calling filtering: "this is partly due to our conservative SV calling and post-calling filtering steps"
