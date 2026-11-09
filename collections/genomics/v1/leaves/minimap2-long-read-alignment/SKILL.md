---
name: minimap2-long-read-alignment
description: Use when ONT FASTQ tumor/control reads (e.g. COLO829, H2009, HCC1954
  pairs) must be aligned to GRCh38.d1.vd1 before running 'nanomonsv parse' and 'nanomonsv
  get'.
when_to_use_negative:
- The input is short-read sequencing data — 'map-ont' is specific to noisy ONT reads
  and Illumina data requires a different aligner/preset
- Illumina or other short-read data, where the 'map-ont' preset and long-read handling
  are inappropriate.
- The goal is germline SV calling alone with a general-purpose caller (e.g. Sniffles2)
  rather than tumor/control paired somatic SV detection with nanomonsv.
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: minimap2
  role: Primary long-read aligner using the '-ax map-ont' preset for ONT reads
  repo: https://github.com/lh3/minimap2
- name: samtools
  role: Sorts and indexes the minimap2 SAM output into BAM/BAI ready for 'nanomonsv
    parse'
- name: nanomonsv
  role: Downstream consumer of the indexed tumor/control BAMs for somatic SV detection
  repo: https://github.com/friend1ws/nanomonsv
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkad526
    title: Precise characterization of somatic complex structural variations from
      tumor/control paired long-read sequencing data with nanomonsv
  source_content_hash: sha256:6455317040022f225c953655d89bcc78503c714c0ee16976caaa6c134fe9e56f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - htslib
  - minimap2
  - nanomonsv
  - racon
  - samtools
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# minimap2-long-read-alignment

## Summary

Align ONT (and PacBio) long reads from tumor/control pairs to a reference genome with minimap2 using the '-ax map-ont' preset, producing sorted, indexed BAMs that serve as the required input for nanomonsv somatic SV detection.

## When to use

Use when ONT FASTQ tumor/control reads (e.g. COLO829, H2009, HCC1954 pairs) must be aligned to GRCh38.d1.vd1 before running 'nanomonsv parse' and 'nanomonsv get'.

## When NOT to use

- The input is short-read sequencing data — 'map-ont' is specific to noisy ONT reads and Illumina data requires a different aligner/preset
- Illumina or other short-read data, where the 'map-ont' preset and long-read handling are inappropriate.
- The goal is germline SV calling alone with a general-purpose caller (e.g. Sniffles2) rather than tumor/control paired somatic SV detection with nanomonsv.

## Inputs

- ONT FASTQ reads for tumor and matched-control samples (e.g. from BioProject PRJDB10898)
- GRCh38.d1.vd1 reference genome FASTA

## Outputs

- Per-sample alignment BAM files
- Sorted and indexed BAMs (with .bai) consumable by 'nanomonsv parse'

## How to apply

Align each tumor and matched-control FASTQ from the ONT pairs to the GRCh38.d1.vd1 reference with minimap2 using the '-ax map-ont' preset; the reproduction workflow specifies '-ax map-ont -t 8 -p 0.1'. Pipe the SAM output into samtools to sort and index the BAM, since the subsequent 'nanomonsv parse' step requires an indexed tumor BAM and matched-control BAM. Keep tumor and control alignments separate (one BAM per sample) because somatic SV calling jointly compares supporting reads between the pair. The '-p 0.1' secondary-alignment setting is deliberately permissive, and 'map-ont' is the ONT-specific preset; PacBio data would use a different preset. Validate alignments by checking BAM index presence and read depth before proceeding to nanomonsv.

## Related tools

- **minimap2** (Primary long-read aligner using the '-ax map-ont' preset for ONT reads) — https://github.com/lh3/minimap2
- **samtools** (Sorts and indexes the minimap2 SAM output into BAM/BAI ready for 'nanomonsv parse')
- **nanomonsv** (Downstream consumer of the indexed tumor/control BAMs for somatic SV detection) — https://github.com/friend1ws/nanomonsv

## Examples

```
minimap2 -ax map-ont -t 8 -p 0.1 GRCh38.d1.vd1.fa tumor.fastq | samtools sort -o tumor.sorted.bam && samtools index tumor.sorted.bam
```

## Evaluation signals

- Each sample produces a sorted, indexed BAM (.bam plus .bai) without errors from samtools index
- Tumor and matched-control BAMs are generated separately for each pair (COLO829, H2009, HCC1954)
- Downstream 'nanomonsv parse' runs without complaints about missing indices, yielding the expected deletion/insertion/rearrangement/bp_info sorted bed.gz/bedpe.gz files
- Alignment coverage and depth are consistent with the expected ONT library for each cell line before SV calling

## Limitations

- The '-p 0.1' setting allows many secondary alignments; runs without sufficient threads (-t) will be slow on large ONT FASTQs.
- Reference-genome-based alignment may not be reliable in long repeat expansion regions, which the article notes can hinder the reference genome framework.
- Noisy long-read alignments limit breakpoint resolution evaluation; alignment quality affects downstream SV precision.

## Evidence

- [results] Align the ONT FASTQ reads for each tumor/control pair to the GRCh38.d1.vd1 reference genome with minimap2 using the '-ax map-ont -t 8 -p 0.1' preset, then sort and index the BAMs with samtools.: [evidence span withheld — anchor status: not_found]
- [results] Run 'nanomonsv parse' on the indexed tumor BAM and matched-control BAM of each pair: [evidence span withheld — anchor status: not_found]
- [intro] we introduced our approach, nanomonsv ... identify somatic SVs with single-nucleotide resolution jointly using both tumor and control long-read sequencing data: "identify somatic SVs with single-nucleotide resolution jointly using both tumor and control long-read sequencing data"
- [results] Run 'nanomonsv parse' separately on each tumor BAM and each matched-control BAM to extract supporting reads for putative SVs: [evidence span withheld — anchor status: not_found]
