---
name: structural-variant-callset-merging
description: Use when merging insertion variant calls from per-sample callsets (e.g.,
  from cuteSV and Sniffles2 VCFs) where coordinate-only merging would collapse distinct
  near-coordinate events that carry different inserted sequences.
when_to_use_negative:
- When a standardized, coordinate-only SV merge is sufficient and sequence identity
  between insertions is not of interest — SURVIVOR with its own distance limits is
  the baseline for this case
- When variants are not insertions (e.g., deletions or inversions), since the Levenshtein
  comparison relies on reassembled inserted sequences
- When supporting reads for an insertion are so variable that the 15% length filter
  excludes most reads, leaving no reliable consensus for sequence comparison
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: RetroInspector
  role: Snakemake pipeline that implements the custom sequence-aware intra- and inter-sample
    insertion merging with Levenshtein distance
  repo: https://github.com/javiercguard/retroinspector
- name: cuteSV
  role: One of the two variant callers whose insertion callsets are merged
  repo: https://github.com/tjiangHIT/cuteSV
- name: Sniffles2
  role: Second variant caller; its insertion calls are combined with cuteSV's before
    merging
  repo: https://github.com/fritzsedlazeck/Sniffles
- name: pysam
  role: Retrieves reads that support each insertion for sequence reassembly
  repo: https://github.com/pysam-developers/pysam
- name: spoa
  role: Generates the consensus sequence of the reassembled inserted region from supporting
    reads
  repo: https://github.com/rvaser/spoa
- name: SURVIVOR
  role: Baseline coordinate-only merge of the same callsets for comparison against
    the sequence-aware merge
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-025-98847-7
    title: Advanced analysis of retrotransposon variation in the human genome with
      nanopore sequencing using RetroInspector
  source_content_hash: sha256:c0ba95b30c2e5a74eefaed209696d0786255d263422c95dfec7e7a9f2153c7b7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/javiercguard/retroinspector
  repo_url: https://github.com/javiercguard/retroinspector
  tools:
  - RetroInspector
  - SURVIVOR
  - Sniffles2
  - cuteSV
  - pysam
  - spoa
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# structural-variant-callset-merging

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A sequence-aware method for merging insertion calls from multiple structural variant callsets, used in RetroInspector to combine cuteSV and Sniffles2 insertion records within and across samples using coordinates, insertion length, and Levenshtein distance on reassembled inserted sequences.

## When to use

Use when merging insertion variant calls from per-sample callsets (e.g., from cuteSV and Sniffles2 VCFs) where coordinate-only merging would collapse distinct near-coordinate events that carry different inserted sequences.

## When NOT to use

- When a standardized, coordinate-only SV merge is sufficient and sequence identity between insertions is not of interest — SURVIVOR with its own distance limits is the baseline for this case
- When variants are not insertions (e.g., deletions or inversions), since the Levenshtein comparison relies on reassembled inserted sequences
- When supporting reads for an insertion are so variable that the 15% length filter excludes most reads, leaving no reliable consensus for sequence comparison

## Inputs

- Per-sample insertion callsets from cuteSV and Sniffles2 (VCF)
- Aligned nanopore reads (BAM/CRAM) supporting each insertion, accessed via pysam
- Reference human genome GRCh38 (hg38 FASTA) for read alignment context

## Outputs

- Sequence-aware merged multi-sample insertion callset (retro.merged.vcf.gz) with reassembled inserted sequences
- Coordinate-only baseline merge (retro.merged.survivor.vcf.gz) from SURVIVOR
- Comparison table of insertion events at near-identical coordinates collapsed by SURVIVOR but kept separate by the Levenshtein-based merge

## How to apply

First obtain per-sample insertion callsets from the cuteSV-Sniffles2 combination, reassembling each inserted sequence from its supporting reads with pysam and spoa; exclude supporting reads whose insertion length differs by more than 15% of the reported insertion length, then derive a spoa consensus. Merge insertions within a sample using a coordinate distance limit of 20 bp (insertionDistanceLimitIntraPatient=20), retaining length as a comparison criterion. For cross-sample merging into one multi-sample callset, keep the 20 bp inter-patient coordinate limit (insertionDistanceLimitInterPatient=20) and additionally compare insertion sequences with the Levenshtein distance, which acts as a gapped, substitution-allowing local pairwise alignment. As a baseline, run a coordinate-only merge of the same callsets with SURVIVOR (survivorInsertionDistanceLimitIntraPatient=10, survivorInsertionDistanceLimitInterPatient=500) and diff the two callsets to identify near-coordinate events SURVIVOR collapses but the sequence-aware merge preserves.

## Related tools

- **RetroInspector** (Snakemake pipeline that implements the custom sequence-aware intra- and inter-sample insertion merging with Levenshtein distance) — https://github.com/javiercguard/retroinspector
- **cuteSV** (One of the two variant callers whose insertion callsets are merged) — https://github.com/tjiangHIT/cuteSV
- **Sniffles2** (Second variant caller; its insertion calls are combined with cuteSV's before merging) — https://github.com/fritzsedlazeck/Sniffles
- **pysam** (Retrieves reads that support each insertion for sequence reassembly) — https://github.com/pysam-developers/pysam
- **spoa** (Generates the consensus sequence of the reassembled inserted region from supporting reads) — https://github.com/rvaser/spoa
- **SURVIVOR** (Baseline coordinate-only merge of the same callsets for comparison against the sequence-aware merge)

## Examples

```
snakemake --use-conda -c 8 --config referenceGenome=/home/yourname/hg38.fa outputPath=/home/yourname/experiment_results fastq_directory=/home/yourname/data/samples
```

## Evaluation signals

- retro.merged.vcf.gz exists and contains the reassembled insertion sequences for merged records
- The comparison table contains at least one instance of distinct insertion events at near-identical coordinates that SURVIVOR collapsed into a single record but the Levenshtein-based merge kept separate
- Intra-sample merging respects insertionDistanceLimitIntraPatient=20 and inter-sample merging respects insertionDistanceLimitInterPatient=20
- Supporting reads excluded during reassembly all had insertion lengths differing by more than 15% of the reported insertion length
- The SURVIVOR baseline (retro.merged.survivor.vcf.gz) uses survivorInsertionDistanceLimitIntraPatient=10 and survivorInsertionDistanceLimitInterPatient=500

## Limitations

- Not all variant callers report a consensus sequence for insertions, so the inserted sequence must be reassembled from supporting reads; with older, imperfectly basecalled data, 'even after reassembly, sequences may be imperfect, particularly for homopolymers'
- The reassembly filter (15% length difference) can remove legitimately shorter supporting reads, reducing consensus quality
- Levenshtein comparison assumes the reassembled consensus is an adequate local alignment proxy; no substitute for full pairwise alignment

## Evidence
