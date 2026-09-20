---
name: sam-bam-alignment-parsing
description: Use when the analysis goal requires extracting per-position clipping
  information (CIGAR S/H operations) from MAPQ-filtered short-read (SR) or long-read
  (LR) BAM alignments, e.g. to build clipping/coverage tables before calling assembly
  error breakpoints.
when_to_use_negative:
- Alignments are in minimap2's default PAF format without CIGAR strings (no S/H operations
  are present, so no clipping positions can be extracted — request -a or -c output
  first)
- The BAM is unfiltered raw mapper output; applying this parsing directly would count
  low-MAPQ (<20), unmapped, secondary, QC-failed and duplicate records that CRAQ explicitly
  removes before extraction
- The goal is genome completeness or block-level misassembly estimation (e.g. BUSCO,
  QUAST, LAI) rather than read-clipping-based error localization
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: SAMtools
  role: Filters alignments (-F 1796 -q 20) and provides the SAM/BAM reading backend
    for iterating reads and their CIGAR strings
- name: CRAQ
  role: Consumer of the extracted clipping tables; its pipeline defines the S/H clip
    extraction, position recording, and clipping-ratio thresholds applied to the parsed
    BAM data
  repo: https://github.com/JiaoLaboratory/CRAQ
- name: Perl
  role: Scripting language used in the CRAQ workflow (version >= 5) for processing
    the extracted per-position clipping and coverage records
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-42336-w
    title: Identification of errors in draft genome assemblies at single-nucleotide
      resolution for quality assessment and improvement
  source_content_hash: sha256:6c6712e2a5d284353b967174e9520f798c2f61f2249ea60942cea7b98ae87d53
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JiaoLaboratory/CRAQ
  tools:
  - CRAQ
  - Perl
  - SAMtools
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# sam-bam-alignment-parsing

## Summary

Parse filtered SAM/BAM alignment files with SAMtools/pysam to extract soft-clipped (S) and hard-clipped (H) reads, recording each clip's reference position, strand orientation, and per-position coverage — the first step CRAQ uses to reveal assembly errors from NGS and SMS read-clipping signals.

## When to use

Use when the analysis goal requires extracting per-position clipping information (CIGAR S/H operations) from MAPQ-filtered short-read (SR) or long-read (LR) BAM alignments, e.g. to build clipping/coverage tables before calling assembly error breakpoints.

## When NOT to use

- Alignments are in minimap2's default PAF format without CIGAR strings (no S/H operations are present, so no clipping positions can be extracted — request -a or -c output first)
- The BAM is unfiltered raw mapper output; applying this parsing directly would count low-MAPQ (<20), unmapped, secondary, QC-failed and duplicate records that CRAQ explicitly removes before extraction
- The goal is genome completeness or block-level misassembly estimation (e.g. BUSCO, QUAST, LAI) rather than read-clipping-based error localization

## Inputs

- Filtered NGS short-read alignment BAM (SR_sort.bam) produced by 'samtools view -F 1796 -q 20'
- Filtered SMS long-read alignment BAM (LR_sort.bam) produced by 'samtools view -F 1796 -q 20'
- SAM/BAM CIGAR strings containing S (soft-clip) or H (hard-clip) operations

## Outputs

- Tab-delimited NGS clipping table (SR_clip.coverRate equivalent) with columns: chr, position, strand, number of clipped-reads, total coverage at that position
- Tab-delimited SMS clipping table (LR_clip.coverRate equivalent) with the same five columns

## How to apply

Start from alignments pre-filtered with 'samtools view -F 1796 -q 20' to drop unmapped, secondary, QC-failed, PCR-duplicate and MAPQ<20 records, as CRAQ does for its SR_sort.bam and LR_sort.bam inputs. Iterate each BAM with SAMtools/pysam and select every alignment whose CIGAR string contains an S (soft-clip) or H (hard-clip) operation. For each clipped read, record the mapped reference coordinate of the clip and encode orientation as strand '+' for a left-side clip and '-' for a right-side clip. At each genomic position tally the number of clipped reads and the total mapped-read coverage, then write tab-delimited tables with the five CRAQ columns (chr, position, strand, number of clipped-reads, total coverage at that position) — one equivalent to SR_clip.coverRate for NGS and one to LR_clip.coverRate for SMS alignments. Clip-position extraction is the foundation of CRAQ's downstream clipping-ratio logic, where ratios within 0.4–0.6 are heterozygous and ratios above 0.75 flag mapping breakpoints.

## Related tools

- **SAMtools** (Filters alignments (-F 1796 -q 20) and provides the SAM/BAM reading backend for iterating reads and their CIGAR strings)
- **CRAQ** (Consumer of the extracted clipping tables; its pipeline defines the S/H clip extraction, position recording, and clipping-ratio thresholds applied to the parsed BAM data) — https://github.com/JiaoLaboratory/CRAQ
- **Perl** (Scripting language used in the CRAQ workflow (version >= 5) for processing the extracted per-position clipping and coverage records)

## Examples

```
samtools view -F 1796 -q 20 SR_sort.bam | python extract_clips.py --read-type SR --out SR_clip.coverRate
```

## Evaluation signals

- Both output tables (SR_clip.coverRate and LR_clip.coverRate equivalents) exist and contain exactly the five required columns: chr, position, strand, number of clipped-reads, total coverage
- At every position, the reported clipped-read count never exceeds the total mapped-read coverage
- Every extracted read was emitted by the '-F 1796 -q 20' samtools view filter (MAPQ >= 20, no unmapped/secondary/QC-failed/duplicate records)
- Clip strand encoding is consistent: '+' for left-side clips and '-' for right-side clips of the same alignment
- Spot-check a few clipped alignments in IGV or against their CIGAR strings to confirm recorded positions match the mapped clip coordinate

## Limitations

- Extraction quality depends on upstream mapping choices; CRAQ uses Minimap2 presets ('-ax sr' for short reads, 'map-hifi'/'map-pb'/'map-ont' for long reads) and poor mapping will yield spurious clip positions
- Clipping ratios at error breakpoints can be below 100% in practice due to sequencing errors or inaccurate read mapping, so per-position clip counts are noisy signals rather than definitive calls
- Multi-mapped reads can inflate clip counts at repetitive positions; CRAQ filters multi-mapped reads when identifying CSE breakpoints by default

## Evidence

- [discussion] CIGAR clip coding: "CRAQ first extracts all clipped reads, coded as “S” or “H” in the Compact Idiosyncratic Gapped Alignment Report (CIGAR) string"
- [discussion] samtools filter parameters: "filtered out using the ‘-F 1796 -q 20’ parameters in ‘samtools view’"
- [abstract] clipping signals rationale: "CRAQ integrates the reads-mapping status (reads coverage & clipping signals) of NGS short-reads and SMS long-reads to detect types of assembly errors"
- [readme] CRAQ tool description: "CRAQ (Clipping Reveals Assembly Quality) is a reference-free genome assembly evaluator"
