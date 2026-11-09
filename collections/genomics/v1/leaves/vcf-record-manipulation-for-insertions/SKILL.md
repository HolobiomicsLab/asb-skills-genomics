---
name: vcf-record-manipulation-for-insertions
description: Reference for merging insertion VCF callsets when callers (cuteSV, Sniffles2)
  do not report consensus inserted sequences, or when coordinate-only merging (e.g.,
  SURVIVOR) collapses distinct insertion events at near-identical positions.
when_to_use_negative:
- Input callsets contain only deletions or other SV classes — this method targets
  insertion records where inserted sequence matters for identity
- A coordinate-only merge at a wide threshold is sufficient and distinguishing inserted
  sequences is unnecessary, e.g., when only approximate event counts are needed
- Input reads are short reads; reassembly from supporting reads with spoa assumes
  long nanopore reads spanning the insertion
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: RetroInspector
  role: Snakemake pipeline implementing the sequence-aware insertion merging and consensus
    reassembly
  repo: github.com/javiercguard/retroinspector
- name: pysam
  role: Retrieves reads supporting each insertion from the alignment for sequence
    reassembly
  repo: github.com/pysam-developers/pysam
- name: spoa
  role: Partial-order alignment consensus generation from reassembled supporting reads
  repo: github.com/rvaser/spoa
- name: SURVIVOR
  role: Baseline coordinate-only merge of the same insertion callsets for comparison
- name: cuteSV
  role: One of the two variant callers producing per-sample insertion callsets
  repo: github.com/tjiangHIT/cuteSV
- name: Sniffles2
  role: Second variant caller producing per-sample insertion callsets
  repo: github.com/fritzsedlazeck/Sniffles
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-025-98847-7
    title: Advanced analysis of retrotransposon variation in the human genome with
      nanopore sequencing using RetroInspector
  source_content_hash: sha256:acf33ac07686afd7e31df52e4eb4c6c6ce1eae8e52c157693861e8c1ef4937e4
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# vcf-record-manipulation-for-insertions

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Sequence-aware merging of insertion VCF records from long-read variant callers: RetroInspector reassembles inserted sequences from supporting reads (pysam + spoa consensus) and merges insertion calls using coordinates, length, and Levenshtein sequence distance instead of coordinates alone.

## When to use

Reference for merging insertion VCF callsets when callers (cuteSV, Sniffles2) do not report consensus inserted sequences, or when coordinate-only merging (e.g., SURVIVOR) collapses distinct insertion events at near-identical positions.

## When NOT to use

- Input callsets contain only deletions or other SV classes — this method targets insertion records where inserted sequence matters for identity
- A coordinate-only merge at a wide threshold is sufficient and distinguishing inserted sequences is unnecessary, e.g., when only approximate event counts are needed
- Input reads are short reads; reassembly from supporting reads with spoa assumes long nanopore reads spanning the insertion

## Inputs

- Per-sample insertion callsets from cuteSV and Sniffles2 (VCF)
- Aligned nanopore reads (BAM) for retrieving insertion-supporting reads via pysam
- SURVIVOR merge parameters (intra-patient 10 bp, inter-patient 500 bp) for the coordinate-only baseline

## Outputs

- retro.merged.vcf.gz — sequence-aware merged multi-sample insertion callset containing reassembled insertion sequences
- retro.merged.survivor.vcf.gz — baseline coordinate-only SURVIVOR merge
- Comparison table of near-coordinate insertion events preserved by Levenshtein-based merging but collapsed by SURVIVOR

## How to apply

Obtain per-sample insertion callsets from the cuteSV-Sniffles2 combination and, since callers may not report the inserted sequence, retrieve supporting reads with pysam and reassemble the affected region into a consensus with spoa; during reassembly, exclude reads whose insertion length differs from the reported insertion length by more than 15%. Merge insertions within each sample using a 20 bp coordinate distance limit (insertionDistanceLimitIntraPatient=20) while retaining length as a comparison criterion. Merge across samples with the same 20 bp inter-patient limit (insertionDistanceLimitInterPatient=20) plus Levenshtein distance comparison of the reassembled sequences, which acts as a gapped, substitution-allowing local pairwise alignment. Run SURVIVOR coordinate-only merging as a baseline (10 bp intra-patient, 500 bp inter-patient) to tabulate events that SURVIVOR collapses but the sequence-aware method keeps separate, and write the result to retro.merged.vcf.gz.

## Related tools

- **RetroInspector** (Snakemake pipeline implementing the sequence-aware insertion merging and consensus reassembly) — github.com/javiercguard/retroinspector
- **pysam** (Retrieves reads supporting each insertion from the alignment for sequence reassembly) — github.com/pysam-developers/pysam
- **spoa** (Partial-order alignment consensus generation from reassembled supporting reads) — github.com/rvaser/spoa
- **SURVIVOR** (Baseline coordinate-only merge of the same insertion callsets for comparison)
- **cuteSV** (One of the two variant callers producing per-sample insertion callsets) — github.com/tjiangHIT/cuteSV
- **Sniffles2** (Second variant caller producing per-sample insertion callsets) — github.com/fritzsedlazeck/Sniffles

## Examples

```
snakemake --use-conda -np --config referenceGenome=/home/yourname/hg38.fa outputPath=/home/yourname/experiment_results fastq_directory=/home/yourname/data/samples
```

## Evaluation signals

- retro.merged.vcf.gz exists and its records contain the reassembled insertion sequences, not only coordinates
- The comparison table shows at least one instance of distinct near-coordinate insertion events kept separate by the Levenshtein-based merge that the SURVIVOR coordinate-only merge collapsed into a single record
- Reassembled sequence lengths are consistent with the reported insertion lengths (reads deviating by more than 15% were excluded from reassembly)
- Merged records respect the 20 bp intra- and inter-patient coordinate distance limits (distinct from SURVIVOR's 10/500 bp settings)

## Limitations

- HGSVC truth-set data was basecalled with an older basecaller, so even after reassembly sequences may be imperfect, particularly for homopolymers
- The 15% length-difference filter during reassembly can discard genuinely variable supporting reads reported by callers
- The inter-patient sequence comparison depends on Levenshtein distance, which may misjudge very long or low-quality insertions

## Evidence
