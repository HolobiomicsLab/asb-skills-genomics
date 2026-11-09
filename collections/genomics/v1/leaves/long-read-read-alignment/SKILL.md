---
name: long-read-read-alignment
description: Use when you have C-to-T (*_CT.fq.gz) and G-to-A (*_GA.fq.gz) converted
  nanopore FASTQ reads plus a converted reference (output.fa from convert_ref.py)
  and need read-to-genome alignments before -alignment selection and methylation calling.
when_to_use_negative:
- Reads are native (unconverted) nanopore signal-called data for direct methylation
  calling — that is the nanopolish WGS route, not the nanoEM conversion pipeline
- Input is short-read (Illumina) methylation data such as EMseq — map-ont is the wrong
  preset and long-read alignment assumptions do not apply
- The reference genome has not been base-converted with convert_ref.py; aligning converted
  reads to an unconverted reference destroys methylation-informative mismatches
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0102
tools:
- name: minimap2
  role: aligns converted CT/GA FASTQ reads to the converted reference using the map-ont
    ONT preset with --eqx and --split-prefix
  repo: https://github.com/yos-sk/nanoEM
- name: samtools
  role: converts minimap2 SAM output to BAM (view -b), coordinate-sorts (sort -@ 8),
    and indexes the resulting BAM files
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkab397
    title: Long-read whole-genome methylation patterning using enzymatic base conversion
      and nanopore sequencing
  source_content_hash: sha256:e4a73959547241566ff8ad1f49066e8a764bb08a040e921270d95035f83a37f4
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
  - samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# long-read-read-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Align base-converted (C-to-T and G-to-A) nanoEM nanopore reads against the correspondingly base-converted reference genome using minimap2 in ONT long-read mode, producing coordinate-sorted BAMs whose per-read best alignment is later resolved for methylation calling.

## When to use

Use when you have C-to-T (*_CT.fq.gz) and G-to-A (*_GA.fq.gz) converted nanopore FASTQ reads plus a converted reference (output.fa from convert_ref.py) and need read-to-genome alignments before best-alignment selection and methylation calling.

## When NOT to use

- Reads are native (unconverted) nanopore signal-called data for direct methylation calling — that is the nanopolish WGS route, not the nanoEM conversion pipeline
- Input is short-read (Illumina) methylation data such as EMseq — map-ont is the wrong preset and long-read alignment assumptions do not apply
- The reference genome has not been base-converted with convert_ref.py; aligning converted reads to an unconverted reference destroys methylation-informative mismatches

## Inputs

- C-to-T converted nanopore reads (*_CT.fq.gz) from convert_reads.py
- G-to-A converted nanopore reads (*_GA.fq.gz) from convert_reads.py
- Base-converted reference genome FASTA (output.fa) from convert_ref.py

## Outputs

- 1.sorted.bam + 1.sorted.bam.bai (coordinate-sorted CT-conversion alignments)
- 2.sorted.bam + 2.sorted.bam.bai (coordinate-sorted GA-conversion alignments)
- Intermediate minimap2 SAM output (temp_sam1 / temp_sam2 split prefixes)

## How to apply

First convert the reference (convert_ref.py ref > output.fa) and the reads (convert_reads.py) so unmethylated cytosines appear as T (and G-to-A on the opposite strand). Align each converted FASTQ against the converted reference using minimap2 2.16-r922 with '-ax map-ont --eqx', 8 threads, and '--split-prefix temp_sam1' (or temp_sam2 for the GA file); the map-ont preset suits error-prone ONT reads and --eqx emits =/X CIGAR strings needed downstream. Pipe each SAM through 'samtools view -b | samtools sort -@ 8 -o 1.sorted.bam' (or 2.sorted.bam) and index with samtools index, so both CT and GA alignments are coordinate-sorted BAMs with .bai indexes. Note that recent minimap2 builds (git clone after April 7, 2023) allow omitting --split-prefix for faster execution.

## Related tools

- **minimap2** (aligns converted CT/GA FASTQ reads to the converted reference using the map-ont ONT preset with --eqx and --split-prefix) — https://github.com/yos-sk/nanoEM
- **samtools** (converts minimap2 SAM output to BAM (view -b), coordinate-sorts (sort -@ 8), and indexes the resulting BAM files)

## Examples

```
minimap2 -t 8 --split-prefix temp_sam1 -ax map-ont output.fa *_CT.fq.gz --eqx | samtools view -b | samtools sort -@ 8 -o 1.sorted.bam && samtools index 1.sorted.bam
```

## Evaluation signals

- Both 1.sorted.bam and 2.sorted.bam exist and are coordinate-sorted with usable .bai indexes (samtools index succeeds)
- Alignment CIGAR strings use the =/X EQX representation, confirming --eqx was active for downstream best_align.py consumption
- Both BAMs are accepted as --bam1 and --bam2 inputs by best_align.py in the next pipeline stage without errors
- Alignments come from minimap2 run in map-ont (ONT) mode, e.g. verifiable from the @PG header line of the BAM
- Two parallel alignments exist — one for *_CT.fq.gz and one for *_GA.fq.gz — both mapped against the same converted reference output.fa

## Limitations

- minimap2 version matters: '--split-prefix' is only optional (and faster to omit) for minimap2 obtained via git clone after April 7, 2023
- Converted reads lose methylation-bearing C/G bases, so alignment discrimination relies on the remaining converted bases from generally error-prone ONT data; the pipeline must later choose the best alignment between CT and GA maps

## Evidence
