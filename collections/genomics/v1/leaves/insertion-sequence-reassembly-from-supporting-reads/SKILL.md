---
name: insertion-sequence-reassembly-from-supporting-reads
description: Use when per-sample insertion callsets from long-read callers (e.g.,
  cuteSV-Sniffles2 VCFs) lack a consensus inserted sequence, and downstream merging,
  genotyping, or annotation requires the actual inserted bases.
when_to_use_negative:
- When the insertion callset already contains reliable consensus inserted sequences
  from the caller, reassembly adds little and the 15% length filter could discard
  legitimate support
- When only coordinate-level merging is required and inserted sequence content is
  irrelevant (a coordinate-only SURVIVOR merge suffices)
- When input reads are short reads (e.g., MELT-style Illumina analysis) rather than
  the nanopore long reads RetroInspector expects
edam_operation: http://edamontology.org/operation_0310
tools:
- name: RetroInspector
  role: Snakemake pipeline that orchestrates retrieval of supporting reads, reassembly
    of the affected region, and downstream merging of insertion calls
  repo: https://github.com/javiercguard/retroinspector
- name: pysam
  role: Retrieves reads supporting each insertion call from the alignment file for
    reassembly
  repo: https://github.com/pysam-developers/pysam
- name: spoa
  role: Partial-order alignment consensus generation from the retained supporting
    reads to produce the inserted sequence
  repo: https://github.com/rvaser/spoa
- name: minimap2
  role: Produces the nanopore read alignments against GRCh38 from which supporting
    reads are drawn
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-025-98847-7
    title: Advanced analysis of retrotransposon variation in the human genome with
      nanopore sequencing using RetroInspector
  source_content_hash: sha256:a670d866d2f7cf50bca5f6dc6d99dff13bbcd372642beb81ba8e7580387b02e8
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
  - Sniffles2
  - cuteSV
  - minimap2
  - pysam
  - spoa
  edam_operation: http://edamontology.org/operation_0310
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# insertion-sequence-reassembly-from-supporting-reads

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reassembles the inserted nucleotide sequence of TE insertion calls from the nanopore reads supporting each call, since variant callers like cuteSV and Sniffles2 do not reliably report a consensus inserted sequence. The reassembled sequences enable sequence-aware merging of insertion calls and characterization of retrotransposition hallmarks.

## When to use

Use when per-sample insertion callsets from long-read callers (e.g., cuteSV-Sniffles2 VCFs) lack a consensus inserted sequence, and downstream merging, genotyping, or annotation requires the actual inserted bases.

## When NOT to use

- When the insertion callset already contains reliable consensus inserted sequences from the caller, reassembly adds little and the 15% length filter could discard legitimate support
- When only coordinate-level merging is required and inserted sequence content is irrelevant (a coordinate-only SURVIVOR merge suffices)
- When input reads are short reads (e.g., MELT-style Illumina analysis) rather than the nanopore long reads RetroInspector expects

## Inputs

- Sorted, indexed minimap2 BAM/CRAM alignments of nanopore reads against GRCh38
- Per-sample insertion callsets (VCF) from cuteSV and Sniffles2 with supporting read information
- RetroInspector template configuration file (e.g., 15% length-difference filter, merge distance parameters)

## Outputs

- Reassembled insertion sequences (spoa consensus per insertion call) embedded in the merged insertion callset retro.merged.vcf.gz
- Sequence-aware multi-sample insertion callset (retro.merged.vcf.gz) and coordinate-only baseline retro.merged.survivor.vcf.gz for comparison

## How to apply

For each insertion call, retrieve the reads that support the insertion from the sorted, indexed minimap2 BAM against GRCh38 using pysam, and use spoa to generate a consensus of the affected region. Exclude supporting reads whose reported insertion length differs from the caller-reported insertion length by more than 15%, since callers report variable reads as supporting reads; this mirrors RetroInspector's filter for greatly differing insertion lengths. Feed the resulting reassembled sequences into the downstream insertion merging step, where they are compared (Levenshtein distance) alongside the 20 bp coordinate limits (insertionDistanceLimitIntraPatient=20, insertionDistanceLimitInterPatient=20) and length comparison. The rationale is that coordinates alone collapse distinct near-coordinate insertion events, whereas reassembled sequences discriminate them; reassembled sequences also support checks for polyA tails and the L1 endonuclease motif, and SVA F1 identification via the MAST2 thumbprint.

## Related tools

- **RetroInspector** (Snakemake pipeline that orchestrates retrieval of supporting reads, reassembly of the affected region, and downstream merging of insertion calls) — https://github.com/javiercguard/retroinspector
- **pysam** (Retrieves reads supporting each insertion call from the alignment file for reassembly) — https://github.com/pysam-developers/pysam
- **spoa** (Partial-order alignment consensus generation from the retained supporting reads to produce the inserted sequence) — https://github.com/rvaser/spoa
- **minimap2** (Produces the nanopore read alignments against GRCh38 from which supporting reads are drawn) — https://github.com/lh3/minimap2

## Examples

```
snakemake --use-conda -c 16 --config referenceGenome="/home/yourname/hg38.fa" outputPath="/home/yourname/experiment_results" fastq_directory="/home/yourname/data/samples"
```

## Evaluation signals

- The merged VCF (retro.merged.vcf.gz) exists and contains reassembled insertion sequences, not just coordinates
- Reads used for reassembly deviate from the reported insertion length by no more than 15%
- A comparison against the SURVIVOR coordinate-only merge (retro.merged.survivor.vcf.gz) shows at least one pair of near-coordinate insertions (<20 bp apart) kept separate by the sequence-aware Levenshtein merge but collapsed by SURVIVOR
- Reassembled sequences pass post-hoc checks such as polyA tails at the inserted sequence terminus and the L1 endonuclease target motif
- Reassembled sequences of validated insertions match Sanger sequencing where available (the pipeline achieved 75% experimental validation, 15 of 20 insertions)

## Limitations

- Reassembly excludes reads whose insertion is considerably shorter (length difference >15% of the reported insertion length), so very divergent support reads are discarded and may bias the consensus
- Publicly available HGSVC data was basecalled with an older basecaller, so even after reassembly, sequences may be imperfect, particularly for homopolymers
- Insertions below the minimum supporting read threshold (default 3 reads) lack enough support for reliable consensus generation

## Evidence
