---
name: in-silico-c-to-t-conversion
description: 'Use when building the first step of a nanoEM-style methylation pipeline:
  before aligning C-to-T or G-to-A converted reads, you need a reference FASTA in
  which every C is replaced by T and every G by A.'
when_to_use_negative:
- Reads are unconverted native DNA being aligned for direct methylation calling (e.g.
  nanopolish call-methylation) — use the original reference instead
- Only the reads need conversion (the *_CT.fq.gz / *_GA.fq.gz step) — that is convert_reads.py,
  not the reference-conversion step
- A converted reference for the same genome build already exists and its sequence
  lengths match the original reference
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3295
tools:
- name: pysam
  role: Used in convert_ref.py to load the reference FASTA and perform the per-base
    C→T / G→A substitution while preserving sequence names
  repo: https://github.com/yos-sk/nanoEM
- name: Python3
  role: Runtime for the convert_ref.py script in the nanoEM pipeline
  repo: https://github.com/yos-sk/nanoEM
- name: nanopore sequencing
  role: The downstream platform whose enzymatically C-to-T converted reads the converted
    reference is built to align
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkab397
    title: Long-read whole-genome methylation patterning using enzymatic base conversion
      and nanopore sequencing
  source_content_hash: sha256:90c8eec34568a5f0760d98adc1d498bfd32b59b1c5c09ba8a3cb10329b3021cd
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/yos-sk/nanoEM
  repo_url: https://github.com/yos-sk/nanoEM
  tools:
  - Python3
  - nanopore sequencing
  - pysam
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# in-silico-c-to-t-conversion

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs a C-to-T / G-to-A converted copy of a reference genome (GRCh38.p12 in the nanoEM pipeline) so that enzymatically base-converted nanopore long reads carrying methylation-sensitive C-to-T (or G-to-A) conversions can be aligned to a genome mirroring their converted state.

## When to use

Use when building the first step of a nanoEM-style methylation pipeline: before aligning C-to-T or G-to-A converted reads, you need a reference FASTA in which every C is replaced by T and every G by A.

## When NOT to use

- Reads are unconverted native DNA being aligned for direct methylation calling (e.g. nanopolish call-methylation) — use the original reference instead
- Only the reads need conversion (the *_CT.fq.gz / *_GA.fq.gz step) — that is convert_reads.py, not the reference-conversion step
- A converted reference for the same genome build already exists and its sequence lengths match the original reference

## Inputs

- Reference genome FASTA (e.g. GRCh38.p12), passed as the 'ref' argument
- pysam Python environment (Python3 + pysam as listed in the nanoEM requirements)

## Outputs

- output.fa — C-to-T / G-to-A converted reference genome FASTA in the original sequence-name and coordinate space

## How to apply

Load the reference genome FASTA (e.g. the GRCh38.p12 FASTA substituted for the 'ref' placeholder) with pysam, then for every reference sequence replace each C with T and each G with A so the reference mirrors the enzymatic base conversions applied to the nanoEM reads on both strands. Preserve original sequence names and coordinate space, because the converted reference must remain index-compatible with the unconverted genome for downstream CpG-site coordinates used by sambamba mpileup with cpg_sites.bed. Write converted sequences to stdout and redirect to output.fa, which is then used as the minimap2 'map-ont' reference for both the *_CT.fq.gz and *_GA.fq.gz read sets. Validate that output.fa exists, is readable by pysam as FASTA, and that per-sequence lengths match the original GRCh38.p12 reference.

## Related tools

- **pysam** (Used in convert_ref.py to load the reference FASTA and perform the per-base C→T / G→A substitution while preserving sequence names) — https://github.com/yos-sk/nanoEM
- **Python3** (Runtime for the convert_ref.py script in the nanoEM pipeline) — https://github.com/yos-sk/nanoEM
- **nanopore sequencing** (The downstream platform whose enzymatically C-to-T converted reads the converted reference is built to align)

## Examples

```
python src/convert_ref.py ref > output.fa
```

## Evaluation signals

- output.fa exists and is a valid FASTA readable by pysam
- Per-sequence lengths of output.fa exactly match the original GRCh38.p12 reference (coordinate space preserved)
- No residual C or G bases remain in the converted reference sequences
- Sequence names in output.fa are identical to the unconverted reference so CpG-site coordinates (cpg_sites.bed, mpileup -f ref) remain index-compatible
- Downstream minimap2 'map-ont' alignment of *_CT.fq.gz and *_GA.fq.gz against output.fa produces usable sorted/indexed BAMs (1.sorted.bam, 2.sorted.bam)

## Limitations

- No software version numbers are stated for the pipeline tools in the article or README
- The README only gives the placeholder 'ref'; you must substitute your own reference genome path, and converted-reference accuracy depends on matching the genome build used for CpG coordinates
- nanoEM (and thus this converted-reference alignment step) cannot distinguish 5-methylcytosine from 5-hydroxymethylcytosine, unlike direct nanopore WGS calling

## Evidence
