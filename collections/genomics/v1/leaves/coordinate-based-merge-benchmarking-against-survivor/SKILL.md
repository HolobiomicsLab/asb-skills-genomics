---
name: coordinate-based-merge-benchmarking-against-survivor
description: Reference for validating whether a sequence-aware variant merge preserves
  distinct insertion calls at near-identical coordinates that a coordinate-only merger
  (e.g., SURVIVOR) collapses into one record.
when_to_use_negative:
- The input callsets contain no reassembled insertion sequences (callers/reporters
  without consensus sequences) and reassembly from supporting reads is not possible
  — merging must remain coordinate-only.
- The goal is merging non-insertion SV classes (deletions, inversions, duplications),
  for which the Levenshtein sequence comparison step is not applicable.
- A single already-merged multi-sample callset is provided and no coordinate-only
  baseline comparison is required.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: RetroInspector
  role: Runs the sequence-aware intra-/inter-sample insertion merge (20 bp coordinate
    limits, length and Levenshtein comparison) and produces retro.merged.vcf.gz
  repo: https://github.com/javiercguard/retroinspector
- name: SURVIVOR
  role: Coordinate-only baseline merge of the same insertion callsets to benchmark
    against
- name: cuteSV
  role: One of the two variant callers whose insertion callsets are merged and benchmarked
  repo: https://github.com/tjiangHIT/cuteSV
- name: Sniffles2
  role: Second variant caller whose insertion callsets feed the merge and benchmark
  repo: https://github.com/fritzsedlazeck/Sniffles
- name: pysam
  role: Retrieves supporting reads for each insertion to reassemble the inserted sequence
  repo: https://github.com/pysam-developers/pysam
- name: spoa
  role: Partial-order alignment consensus generation for the reassembled insertion
    sequences
  repo: https://github.com/rvaser/spoa
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-025-98847-7
    title: Advanced analysis of retrotransposon variation in the human genome with
      nanopore sequencing using RetroInspector
  source_content_hash: sha256:8d8b477e5786cfb5acac2b2a381fa05c16e3254bea5ff89929da1059f1384cae
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
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# Coordinate-based merge benchmarking against SURVIVOR

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks RetroInspector's sequence-aware insertion merging (coordinates + length + Levenshtein distance on reassembled insert sequences) against SURVIVOR's coordinate-only merge to reveal near-coordinate insertion events that coordinate-only merging wrongly collapses.

## When to use

Reference for validating whether a sequence-aware variant merge preserves distinct insertion calls at near-identical coordinates that a coordinate-only merger (e.g., SURVIVOR) collapses into one record.

## When NOT to use

- The input callsets contain no reassembled insertion sequences (callers/reporters without consensus sequences) and reassembly from supporting reads is not possible — merging must remain coordinate-only.
- The goal is merging non-insertion SV classes (deletions, inversions, duplications), for which the Levenshtein sequence comparison step is not applicable.
- A single already-merged multi-sample callset is provided and no coordinate-only baseline comparison is required.

## Inputs

- Per-sample insertion callsets from cuteSV and Sniffles2 (VCF)
- Aligned nanopore reads (BAM) for reassembly of inserted sequences from supporting reads
- SURVIVOR merge configuration (survivorInsertionDistanceLimitIntraPatient=10, survivorInsertionDistanceLimitInterPatient=500)

## Outputs

- retro.merged.vcf.gz — sequence-aware merged insertion callset containing reassembled insertion sequences
- retro.merged.survivor.vcf.gz — baseline coordinate-only SURVIVOR merge
- Comparison table of near-coordinate insertion events collapsed by SURVIVOR but kept separate by the Levenshtein-based merge

## How to apply

Start from per-sample insertion callsets produced by the cuteSV-Sniffles2 combination, reassembling each inserted sequence from supporting reads with pysam and spoa, discarding reads whose insertion length differs by more than 15% of the reported insertion length. Run RetroInspector's custom intra-sample merge with insertionDistanceLimitIntraPatient=20 (plus length comparison), then the inter-sample merge with insertionDistanceLimitInterPatient=20, length comparison, and Levenshtein distance on the reassembled sequences as a gapped, substitution-allowing local pairwise alignment. In parallel, run SURVIVOR as the coordinate-only baseline with survivorInsertionDistanceLimitIntraPatient=10 and survivorInsertionDistanceLimitInterPatient=500 to produce retro.merged.survivor.vcf.gz. Diff the two callsets to tabulate insertion events at near-identical coordinates that SURVIVOR collapses but the Levenshtein-based merge keeps separate, and write the sequence-aware result as retro.merged.vcf.gz. The rationale is that not all variant callers report a consensus insertion sequence, so reassembly from supporting reads is required to make merging sequence-aware rather than coordinate-only.

## Related tools

- **RetroInspector** (Runs the sequence-aware intra-/inter-sample insertion merge (20 bp coordinate limits, length and Levenshtein comparison) and produces retro.merged.vcf.gz) — https://github.com/javiercguard/retroinspector
- **SURVIVOR** (Coordinate-only baseline merge of the same insertion callsets to benchmark against)
- **cuteSV** (One of the two variant callers whose insertion callsets are merged and benchmarked) — https://github.com/tjiangHIT/cuteSV
- **Sniffles2** (Second variant caller whose insertion callsets feed the merge and benchmark) — https://github.com/fritzsedlazeck/Sniffles
- **pysam** (Retrieves supporting reads for each insertion to reassemble the inserted sequence) — https://github.com/pysam-developers/pysam
- **spoa** (Partial-order alignment consensus generation for the reassembled insertion sequences) — https://github.com/rvaser/spoa

## Examples

```
snakemake --use-conda -c 8 --config referenceGenome=/home/yourname/hg38.fa outputPath=/home/yourname/experiment_results fastq_directory=/home/yourname/data/samples
```

## Evaluation signals

- retro.merged.vcf.gz exists and contains the reassembled insertion sequences, not only coordinates
- The comparison table shows at least one instance of distinct near-coordinate insertion events preserved by the Levenshtein-based merge that the SURVIVOR coordinate-only merge collapsed into a single record
- Intra-sample merges respect the 20 bp coordinate limit (insertionDistanceLimitIntraPatient=20) while the SURVIVOR baseline uses 10 bp intra-patient and 500 bp inter-patient limits
- Supporting reads excluded from reassembly are exactly those with insertion length differing by more than 15% of the reported insertion length
- SURVIVOR baseline output retro.merged.survivor.vcf.gz is generated from the identical input callsets used for the sequence-aware merge

## Limitations

- Not all variant callers report a consensus sequence for insertions, so sequence-aware merging requires prior reassembly from supporting reads
- Reassembly excludes supporting reads whose insertion length differs by more than 15% of the reported insertion length; highly variable reads can distort the consensus
- Publicly available HGSVC data was basecalled with an older basecaller, so even after reassembly, sequences may be imperfect, particularly for homopolymers

## Evidence

- [other] RetroInspector's insertion merging is sequence-aware: the reassembled inserted sequences (reconstructed from supporting reads, excluding reads with insertion length differing by more than 15%) are employed in the variant merging step alongside coordinates.: [evidence span withheld — anchor status: not_found]
- [results] Reassembly excludes supporting reads with insertion length difference >15% of reported insertion length: "The reassembly process excludes reads with a considerably shorter insertion (length difference greater than 15% of the reported insertion length)"
- [other] Inserted sequences reassembled from supporting reads using pysam with spoa consensus generation: [evidence span withheld — anchor status: not_found]
- [results] RetroInspector retrieves supporting reads and reassembles the affected region to obtain a new sequence: "our tool retrieves reads that support the insertion, and reassembles the affected region in order to obtain a new sequence."
- [discussion] HGSVC truth-set basecalled with older basecaller; reassembled sequences may be imperfect for homopolymers: "basecalled with an older basecaller, and even after reassembly, sequences may be imperfect, particularly for homopolymers"
