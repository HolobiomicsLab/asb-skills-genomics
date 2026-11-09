---
name: ont-read-alignment-parameterization
description: Use when nanopore long reads have been base-converted (C-to-T and G-to-A
  fastqs) and must be mapped to an enzymatically converted reference so methylation
  can later be resolved per read.
when_to_use_negative:
- Reads are native (unconverted) nanopore signal-level data intended for direct methylation
  calling with nanopolish, not enzymatic-conversion reads
- Reads have not yet been base-converted — aligning *_CT.fq.gz/*_GA.fq.gz against
  a non-converted reference genome would produce meaningless mappings
- The goal is short-read methylation data (e.g. EMseq paired-end short reads), where
  ONT 'map-ont' presets and long-read alignment are inappropriate
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: minimap2
  role: Aligns converted reads to the converted reference in ONT long-read mode with
    '-ax map-ont', --split-prefix, and --eqx output
  repo: https://github.com/yos-sk/nanoEM
- name: samtools
  role: Converts minimap2 SAM output to BAM (samtools view -b), coordinate-sorts it
    (samtools sort -@ 8), and indexes it (samtools index)
  repo: https://github.com/yos-sk/nanoEM
- name: convert_ref.py
  role: Produces output.fa, the converted reference genome the alignments target
  repo: https://github.com/yos-sk/nanoEM
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkab397
    title: Long-read whole-genome methylation patterning using enzymatic base conversion
      and nanopore sequencing
  source_content_hash: sha256:a204c2259316ea52fe08bc1f88cc890e7c35a692428b850f3164145acba3837c
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
  - convert_ref.py
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# ont-read-alignment-parameterization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aligns base-converted nanoEM nanopore reads (C-to-T and G-to-A) to a correspondingly converted reference genome using minimap2 in ONT long-read mode, producing coordinate-sorted BAMs that a best-alignment step can consume for downstream methylation calling.

## When to use

Use when nanopore long reads have been base-converted (C-to-T and G-to-A fastqs) and must be mapped to an enzymatically converted reference so methylation can later be resolved per read.

## When NOT to use

- Reads are native (unconverted) nanopore signal-level data intended for direct methylation calling with nanopolish, not enzymatic-conversion reads
- Reads have not yet been base-converted — aligning *_CT.fq.gz/*_GA.fq.gz against a non-converted reference genome would produce meaningless mappings
- The goal is short-read methylation data (e.g. EMseq paired-end short reads), where ONT 'map-ont' presets and long-read alignment are inappropriate

## Inputs

- C-to-T converted FASTQ (*_CT.fq.gz) from convert_reads.py
- G-to-A converted FASTQ (*_GA.fq.gz) from convert_reads.py
- Converted reference genome FASTA (output.fa) from convert_ref.py

## Outputs

- 1.sorted.bam: coordinate-sorted BAM of C-to-T converted reads aligned to output.fa, with .bai index
- 2.sorted.bam: coordinate-sorted BAM of G-to-A converted reads aligned to output.fa, with .bai index

## How to apply

First convert the reference genome (convert_ref.py) to output.fa, since the converted reads must not be aligned against the native genome. Run minimap2 twice in ONT long-read mode: once with '-ax map-ont --eqx' and '--split-prefix temp_sam1' for the C-to-T converted fastq (*_CT.fq.gz), and once with '--split-prefix temp_sam2' for the G-to-A converted fastq (*_GA.fq.gz), using 8 threads in both cases. Pipe each SAM output through 'samtools view -b | samtools sort -@ 8' to produce 1.sorted.bam and 2.sorted.bam, then index each with samtools index. The --eqx option is required so the best_align.py stage can compare the two alignments per read and choose the best one; --split-prefix handles split alignments, and it can be omitted for minimap2 versions cloned after April 7, 2023.

## Related tools

- **minimap2** (Aligns converted reads to the converted reference in ONT long-read mode with '-ax map-ont', --split-prefix, and --eqx output) — https://github.com/yos-sk/nanoEM
- **samtools** (Converts minimap2 SAM output to BAM (samtools view -b), coordinate-sorts it (samtools sort -@ 8), and indexes it (samtools index)) — https://github.com/yos-sk/nanoEM
- **convert_ref.py** (Produces output.fa, the converted reference genome the alignments target) — https://github.com/yos-sk/nanoEM

## Examples

```
minimap2 -t 8 --split-prefix temp_sam1 -ax map-ont output.fa *_CT.fq.gz --eqx | samtools view -b | samtools sort -@ 8 -o 1.sorted.bam; samtools index 1.sorted.bam
```

## Evaluation signals

- Both 1.sorted.bam and 2.sorted.bam exist as coordinate-sorted BAM files with usable .bai indexes after samtools index
- minimap2 was run with '-ax map-ont' and '--eqx'; the --eqx cigar format is needed so best_align.py can consume --bam1 and --bam2 in the next stage
- Both alignments were produced against the same converted reference (output.fa), not the native reference genome
- SAM output was piped through samtools view -b and samtools sort -@ 8, so BAM headers show SO:coordinate
- Alignment mapping rates for the two conversion strands (C-to-T vs G-to-A) are comparable; a gross imbalance suggests reads or reference were mis-converted

## Limitations

- Older minimap2 builds require the '--split-prefix' option; only minimap2 obtained by 'git clone' after April 7, 2023 allows omitting it (and is then faster)
- The pipeline depends on converted reads being aligned only to the converted reference; aligning converted reads to a native genome will fail
- No software version numbers are stated in the methods; the source card reports minimap2 version 2.16-r922 but the README gives no versions

## Evidence
