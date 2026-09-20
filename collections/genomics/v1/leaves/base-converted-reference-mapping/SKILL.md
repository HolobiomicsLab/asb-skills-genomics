---
name: base-converted-reference-mapping
description: Use when nanopore long reads have been enzymatically base-converted (unmethylated
  C→T) and must be mapped to a matching converted reference before methylation calling;
  triggered by *_CT.fq.gz and *_GA.fq.gz inputs plus a converted output.fa reference.
when_to_use_negative:
- Reads and reference are native (non-converted) nanopore WGS — for that, methylation
  is called directly from signal with nanopolish instead of conversion-aware mapping
- Input FASTQ files are not base-converted nanoEM data (e.g., short-read bisulfite/EMseq
  data), since 'map-ont' preset and the CT/GA dual-reference scheme are designed for
  long-read converted reads
- Reference genome has not been converted to output.fa — aligning converted reads
  to an unconverted reference breaks the methylation-informative mismatch pattern
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3169
tools:
- name: minimap2
  role: Aligns C-to-T and G-to-A converted long reads to the converted reference using
    the '-ax map-ont' preset with '--eqx' and '--split-prefix'
  repo: https://github.com/yos-sk/nanoEM
- name: samtools
  role: Converts minimap2 SAM output to BAM ('samtools view -b'), coordinate-sorts
    with '-@ 8', and indexes 1.sorted.bam / 2.sorted.bam
  repo: https://github.com/yos-sk/nanoEM
- name: nanopore sequencing
  role: Sequencing modality that generated the error-prone long reads; the 'map-ont'
    preset is tuned for ONT read characteristics
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkab397
    title: Long-read whole-genome methylation patterning using enzymatic base conversion
      and nanopore sequencing
  source_content_hash: sha256:e724ee0b0dbc5e7136db731ede7d406ed73fb741f9c346e8d29728dd04705115
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
  - minimap2
  - nanopore sequencing
  - samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# base-converted-reference-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aligns base-converted nanoEM reads (C-to-T and G-to-A converted FASTQ) against an enzymatically converted reference genome using minimap2 in ONT long-read mode, producing the coordinate-sorted BAM pairs that downstream best-of-two-alignment methylation calling consumes.

## When to use

Use when nanopore long reads have been enzymatically base-converted (unmethylated C→T) and must be mapped to a matching converted reference before methylation calling; triggered by *_CT.fq.gz and *_GA.fq.gz inputs plus a converted output.fa reference.

## When NOT to use

- Reads and reference are native (non-converted) nanopore WGS — for that, methylation is called directly from signal with nanopolish instead of conversion-aware mapping
- Input FASTQ files are not base-converted nanoEM data (e.g., short-read bisulfite/EMseq data), since 'map-ont' preset and the CT/GA dual-reference scheme are designed for long-read converted reads
- Reference genome has not been converted to output.fa — aligning converted reads to an unconverted reference breaks the methylation-informative mismatch pattern

## Inputs

- *_CT.fq.gz — C-to-T converted fastq from src/convert_reads.py
- *_GA.fq.gz — G-to-A converted fastq from src/convert_reads.py
- output.fa — converted reference genome produced by 'python src/convert_ref.py ref > output.fa'

## Outputs

- 1.sorted.bam (.bai indexed) — coordinate-sorted alignments of C-to-T converted reads
- 2.sorted.bam (.bai indexed) — coordinate-sorted alignments of G-to-A converted reads
- temp_sam1 / temp_sam2 split-index prefix files from minimap2

## How to apply

First convert both the reference genome (convert_ref.py) and the nanoEM reads (convert_reads.py) so that unmethylated cytosines are changed consistently on both strands. Run minimap2 in ONT long-read mode ('-ax map-ont') with 8 threads, '--eqx' output for precise alignment notation, and '--split-prefix' to avoid RAM issues with long reference coordinates: once for the C-to-T converted FASTQ (*_CT.fq.gz) with prefix temp_sam1, once for the G-to-A converted FASTQ (*_GA.fq.gz) with prefix temp_sam2, in both cases against the converted output.fa. Pipe each minimap2 SAM stream through 'samtools view -b | samtools sort -@ 8' to generate coordinate-sorted 1.sorted.bam and 2.sorted.bam, then index both with 'samtools index'. The rationale is that reads and reference must carry the same conversion so that true unmethylated positions match while retained methylated cytosines appear as mismatches; --split-prefix can be dropped (faster) only with minimap2 cloned from GitHub after April 7, 2023. The resulting BAMs are consumed as --bam1/--bam2 by src/best_align.py, which selects the better of the CT and GA alignments per read.

## Related tools

- **minimap2** (Aligns C-to-T and G-to-A converted long reads to the converted reference using the '-ax map-ont' preset with '--eqx' and '--split-prefix') — https://github.com/yos-sk/nanoEM
- **samtools** (Converts minimap2 SAM output to BAM ('samtools view -b'), coordinate-sorts with '-@ 8', and indexes 1.sorted.bam / 2.sorted.bam) — https://github.com/yos-sk/nanoEM
- **nanopore sequencing** (Sequencing modality that generated the error-prone long reads; the 'map-ont' preset is tuned for ONT read characteristics)

## Examples

```
minimap2 -t 8 --split-prefix temp_sam1 -ax map-ont output.fa *_CT.fq.gz --eqx | samtools view -b | samtools sort -@ 8 -o 1.sorted.bam; samtools index 1.sorted.bam
```

## Evaluation signals

- Both 1.sorted.bam and 2.sorted.bam exist as valid, coordinate-sorted BAM files with usable .bai indexes
- BAM header @HD SO:coordinate tag present after 'samtools sort -@ 8' on each branch
- Downstream step runs without error: 'python src/best_align.py --bam1 1.sorted.bam --bam2 2.sorted.bam --fastq fastq' accepts both BAMs as --bam1/--bam2
- Alignments contain --eqx-style (=/X) match-mismatch operations, confirming the --eqx flag was passed
- The same converted reference (output.fa) was used for both CT and GA mapping runs — no mixing with an unconverted 'ref'

## Limitations

- The '--split-prefix' option is required unless using a minimap2 obtained by 'git clone' after April 7, 2023 (with which the step is faster without it)
- Converted reads are mostly C–T substituted and come from generally error-prone nanopore data, so alignments must be interpreted as conversion-aware; dual CT/GA mapping is needed to decide the best strand alignment
- The article text does not state minimap2 version numbers in the methods README; reproducibility requires pinning the tool independently

## Evidence

- [methods] minimap2 -t 8 --split-prefix temp_sam1 -ax map-ont output.fa  *_CT.fq.gz --eqx: "minimap2 -t 8 --split-prefix temp_sam1 -ax map-ont output.fa  *_CT.fq.gz --eqx"
- [readme] You can perform this step without the '--split-prefix' option if you are using minimap2 downloaded by 'git clone' after April 7, 2023.: "You can perform this step without the '--split-prefix' option if you are using minimap2 downloaded by 'git clone' after April 7, 2023. It's faster."
- [intro] a bioinformatics pipeline was required and, thus, was developed, starting from long-read mapping of the mostly C–T converted bases from generally error-prone data: "To analyze the nanoEM data, a bioinformatics pipeline was required and, thus, was developed, starting from long-read mapping of the mostly C–T converted bases from generally error-prone data."
- [readme] *_CT.fq.gz: C-to-T converted fastq: "OUTPUT:
- *_CT.fq.gz: C-to-T converted fastq
- *_GA.fq.gz: G-to-A converted fastq"
