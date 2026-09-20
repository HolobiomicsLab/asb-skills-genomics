---
name: long-read-alignment-of-base-converted-nanopore-reads
description: Use when aligning enzymatically base-converted nanopore long reads (nanoEM/EMseq
  FASTQ, e.g. from DRA011237) where C-to-T/G-to-A conversion must be mirrored in the
  reference to recover high mapping rates over default minimap2.
when_to_use_negative:
- Reads are native (non-base-converted) nanopore DNA intended for direct methylation
  calling — use nanopolish-style signal-level calling instead
- The goal is methylation frequency calling per CpG (call_methylation.py / sambamba
  mpileup) rather than read alignment rate benchmarking
- Input is short-read (e.g. Illumina) methyl-seq data — minimap2 map-ont and the long-read
  best-alignment selection are inappropriate
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_3168
tools:
- name: minimap2
  role: Maps C-to-T and G-to-A converted FASTQ to the converted reference using -ax
    map-ont --eqx (with -t 8 and --split-prefix)
  repo: https://github.com/yos-sk/nanoEM
- name: samtools
  role: Converts minimap2 SAM output to sorted, indexed BAMs and later sorts/indexes
    best_align.py SAM outputs
- name: sambamba
  role: Listed as a nanoEM pipeline requirement (used downstream for mpileup at CpG
    sites)
- name: pysam
  role: Python dependency used by the nanoEM conversion and best-alignment scripts
  repo: https://github.com/yos-sk/nanoEM
- name: Python3
  role: Runs convert_ref.py, convert_reads.py and best_align.py for reference/read
    conversion and best-alignment selection
  repo: https://github.com/yos-sk/nanoEM
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkab397
    title: Long-read whole-genome methylation patterning using enzymatic base conversion
      and nanopore sequencing
  source_content_hash: sha256:997709c6a82003066e501fb11ecb079bb9d7594e2dcece588b9e3db4e440a060
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
  - minimap2
  - pysam
  - sambamba
  - samtools
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# long-read-alignment-of-base-converted-nanopore-reads

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps in silico C-to-T and G-to-A converted nanopore reads from the nanoEM pipeline onto a converted reference genome with minimap2 (map-ont, --eqx), then selects strand-specific best alignments per read with best_align.py to maximize mapping rate.

## When to use

Use when aligning enzymatically base-converted nanopore long reads (nanoEM/EMseq FASTQ, e.g. from DRA011237) where C-to-T/G-to-A conversion must be mirrored in the reference to recover high mapping rates over default minimap2.

## When NOT to use

- Reads are native (non-base-converted) nanopore DNA intended for direct methylation calling — use nanopolish-style signal-level calling instead
- The goal is methylation frequency calling per CpG (call_methylation.py / sambamba mpileup) rather than read alignment rate benchmarking
- Input is short-read (e.g. Illumina) methyl-seq data — minimap2 map-ont and the long-read best-alignment selection are inappropriate

## Inputs

- nanoEM nanopore FASTQ reads (e.g. MB231 and BT474 runs from DDBJ DRA accession DRA011237)
- Reference genome FASTA (e.g. human hg38)
- nanoEM pipeline scripts from github.com/yos-sk/nanoEM (convert_ref.py, convert_reads.py, best_align.py)
- minimap2 and samtools

## Outputs

- output.fa: C-to-T converted reference genome FASTA
- *_CT.fq.gz and *_GA.fq.gz: C-to-T and G-to-A converted FASTQ files
- 1.sorted.bam and 2.sorted.bam: sorted, indexed BAMs of CT and GA mappings
- output_CT.sam and output_GA.sam (and their sorted/indexed BAMs): best strand-specific alignments per read
- Per-run aligned-fraction comparison table (mapping_statistics.tsv) contrasting pipeline vs default minimap2

## How to apply

First in silico convert the reference genome with `python src/convert_ref.py ref > output.fa`. Convert each FASTQ into C-to-T and G-to-A variants with convert_reads.py, producing *_CT.fq.gz and *_GA.fq.gz. Map each converted FASTQ to output.fa with minimap2 using `-t 8 --split-prefix temp_sam1 -ax map-ont output.fa *_CT.fq.gz --eqx`, then convert to sorted/indexed BAM (1.sorted.bam, 2.sorted.bam) via samtools. Run a default minimap2 mapping of unconverted reads in parallel as a baseline. Finally, run `python src/best_align.py --bam1 1.sorted.bam --bam2 2.sorted.bam --fastq fastq` to pick the best strand-specific alignment per read into output_CT.sam and output_GA.sam, then sort and index them. The rationale is that converting both reads and reference restores sequence fidelity lost to the conversion, lifting alignment to ~90%/89% (MB231/BT474) versus 81-84% for default minimap2.

## Related tools

- **minimap2** (Maps C-to-T and G-to-A converted FASTQ to the converted reference using -ax map-ont --eqx (with -t 8 and --split-prefix)) — https://github.com/yos-sk/nanoEM
- **samtools** (Converts minimap2 SAM output to sorted, indexed BAMs and later sorts/indexes best_align.py SAM outputs)
- **sambamba** (Listed as a nanoEM pipeline requirement (used downstream for mpileup at CpG sites))
- **pysam** (Python dependency used by the nanoEM conversion and best-alignment scripts) — https://github.com/yos-sk/nanoEM
- **Python3** (Runs convert_ref.py, convert_reads.py and best_align.py for reference/read conversion and best-alignment selection) — https://github.com/yos-sk/nanoEM

## Examples

```
minimap2 -t 8 --split-prefix temp_sam1 -ax map-ont output.fa *_CT.fq.gz --eqx | samtools view -b | samtools sort -@ 8 -o 1.sorted.bam
```

## Evaluation signals

- Alignment fraction for the conversion pipeline approaches the reported 90% (MB231) and 89% (BT474)
- Default minimap2 baseline on unconverted reads falls in the reported 81-84% range, confirming a 6-8 percentage-point improvement
- output.fa contains only C-to-T substituted reference bases and read conversion yields paired *_CT.fq.gz and *_GA.fq.gz files
- 1.sorted.bam and 2.sorted.bam are indexed and output_CT.sam/output_GA.sam contain exactly one best alignment per input read
- Using minimap2 --eqx output means mismatch positions (=/X CIGAR operators) are explicitly encoded, enabling downstream methylation-aware processing

## Limitations

- No software version numbers are stated for minimap2, samtools, sambamba, pysam or the pipeline scripts, hindering exact reproducibility
- The --split-prefix option is only unnecessary for minimap2 downloaded by 'git clone' after April 7, 2023
- nanoEM cannot distinguish 5-methylcytosine from 5-hydroxymethylcytosine on the same molecule, unlike direct nanopore WGS calling
- Reported rates are specific to the MB231 and BT474 breast cancer cell line runs in DRA011237 and may vary for other samples

## Evidence
