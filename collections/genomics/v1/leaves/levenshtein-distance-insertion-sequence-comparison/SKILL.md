---
name: levenshtein-distance-insertion-sequence-comparison
description: Use when merging multi-sample insertion callsets where callers report
  coordinates only and distinct events at near-identical coordinates must be preserved
  by comparing their inserted sequences rather than coordinates alone.
when_to_use_negative:
- Callsets already contain consensus insertion sequences from all callers, so reassembly
  from supporting reads is unnecessary
- Only deletions or non-insertion structural variant classes are being merged (sequence
  comparison is specific to inserted sequences)
- A coordinate-only merge with SURVIVOR is acceptable because no distinct insertion
  events are expected at near-identical coordinates
edam_operation: http://edamontology.org/operation_0491
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: RetroInspector
  role: Snakemake pipeline implementing the sequence-aware insertion merging with
    Levenshtein distance
  repo: github.com/javiercguard/retroinspector
- name: pysam
  role: Retrieves reads supporting each insertion for reassembly of the inserted sequence
  repo: github.com/pysam-developers/pysam
- name: spoa
  role: Generates partial-order alignment consensus of reassembled inserted sequences
  repo: github.com/rvaser/spoa
- name: cuteSV
  role: Variant caller whose insertion callsets (which may lack consensus sequences)
    are merged
  repo: github.com/tjiangHIT/cuteSV
- name: Sniffles2
  role: Variant caller combined with cuteSV to produce per-sample insertion callsets
  repo: github.com/fritzsedlazeck/Sniffles
- name: SURVIVOR
  role: Coordinate-only baseline merge used for comparison against the sequence-aware
    merge
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-025-98847-7
    title: Advanced analysis of retrotransposon variation in the human genome with
      nanopore sequencing using RetroInspector
  source_content_hash: sha256:cb5fcb55f9de9d637e2fbe1da032c9105cfc073d5d4671f1f8932101bae4123c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/javiercguard/retroinspector
  tools:
  - RetroInspector
  - SURVIVOR
  - Sniffles2
  - cuteSV
  - pysam
  - spoa
  edam_operation: http://edamontology.org/operation_0491
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# levenshtein-distance-insertion-sequence-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A sequence-aware strategy for merging insertion variant calls, in which reassembled insertion sequences (spoa consensus from supporting reads) are compared with Levenshtein distance in addition to coordinate and length criteria, so distinct insertion events at near-identical positions are not collapsed.

## When to use

Use when merging multi-sample insertion callsets where callers report coordinates only and distinct events at near-identical coordinates must be preserved by comparing their inserted sequences rather than coordinates alone.

## When NOT to use

- Callsets already contain consensus insertion sequences from all callers, so reassembly from supporting reads is unnecessary
- Only deletions or non-insertion structural variant classes are being merged (sequence comparison is specific to inserted sequences)
- A coordinate-only merge with SURVIVOR is acceptable because no distinct insertion events are expected at near-identical coordinates

## Inputs

- Per-sample insertion callsets from the cuteSV-Sniffles2 combination (VCF)
- Aligned nanopore reads (BAM) supporting each insertion, accessed with pysam
- Reported insertion lengths from the variant callers
- Reference human genome GRCh38

## Outputs

- Sequence-aware merged insertion callset retro.merged.vcf.gz containing reassembled insertion sequences
- Coordinate-only SURVIVOR baseline callset retro.merged.survivor.vcf.gz
- Comparison table of near-coordinate insertion events preserved by the Levenshtein-based merge but collapsed by SURVIVOR

## How to apply

First reassemble each inserted sequence from its supporting reads with pysam, excluding reads whose insertion length differs from the reported insertion length by more than 15%, and build a consensus with spoa, since not all variant callers (cuteSV, Sniffles2) report a consensus insertion sequence. Merge insertion calls within each sample using a 20 bp coordinate limit (insertionDistanceLimitIntraPatient=20) while retaining length as a comparison criterion. For the multi-sample merge, apply the same 20 bp inter-patient coordinate limit (insertionDistanceLimitInterPatient=20) plus length comparison, and additionally compare insertion sequences by Levenshtein distance, which acts as a gapped, substitution-allowing local pairwise alignment. Run a coordinate-only baseline merge of the same callsets with SURVIVOR to identify near-coordinate events it collapses but the sequence-aware method keeps separate. Write the sequence-aware result as retro.merged.vcf.gz and the SURVIVOR baseline as retro.merged.survivor.vcf.gz, then tabulate the differences.

## Related tools

- **RetroInspector** (Snakemake pipeline implementing the sequence-aware insertion merging with Levenshtein distance) — github.com/javiercguard/retroinspector
- **pysam** (Retrieves reads supporting each insertion for reassembly of the inserted sequence) — github.com/pysam-developers/pysam
- **spoa** (Generates partial-order alignment consensus of reassembled inserted sequences) — github.com/rvaser/spoa
- **cuteSV** (Variant caller whose insertion callsets (which may lack consensus sequences) are merged) — github.com/tjiangHIT/cuteSV
- **Sniffles2** (Variant caller combined with cuteSV to produce per-sample insertion callsets) — github.com/fritzsedlazeck/Sniffles
- **SURVIVOR** (Coordinate-only baseline merge used for comparison against the sequence-aware merge)

## Evaluation signals

- retro.merged.vcf.gz exists and contains the reassembled insertion sequences
- The comparison table shows at least one instance of distinct near-coordinate insertion events kept separate by the Levenshtein-based merge that SURVIVOR collapsed into a single record
- Intra-sample merges respect the 20 bp coordinate limit (insertionDistanceLimitIntraPatient=20) and retain length as a comparison criterion
- Reassembled sequences derive only from supporting reads with insertion length within 15% of the reported insertion length
- SURVIVOR baseline run with survivorInsertionDistanceLimitIntraPatient=10 and survivorInsertionDistanceLimitInterPatient=500 produces retro.merged.survivor.vcf.gz

## Limitations

- HGSVC truth-set data was basecalled with an older basecaller, so even after reassembly insertion sequences may be imperfect, particularly for homopolymers, which can inflate Levenshtein distances
- Not all variant callers report a consensus sequence for insertions, so sequences must be reassembled from supporting reads before sequence comparison is possible
- The 15% length-difference exclusion filter may drop genuinely variable reads that callers report as supporting reads

## Evidence
