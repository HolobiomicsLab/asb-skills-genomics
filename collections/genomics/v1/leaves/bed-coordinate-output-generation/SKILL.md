---
name: bed-coordinate-output-generation
description: Use when CRAQ clipping-ratio classification has identified candidate
  breakpoints and you must export exact CRE/CSE/CRH/CSH coordinates as BED files with
  per-locus coverage and clipped-read counts.
when_to_use_negative:
- The draft assembly has no raw NGS/SMS reads re-mapped to it — there is no clipping
  signal to classify, so no coordinate files can be produced.
- The goal is completeness or assembly-contiguity assessment (e.g., BUSCO, QUAST,
  LAI) rather than pinpointing single-nucleotide-resolution error and heterozygous
  breakpoints.
- The assembly is from a polyploid genome, where CRAQ's classification accuracy is
  explicitly not established.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: CRAQ
  role: Classifies clipping-ratio candidates into CRE/CSE errors and CRH/CSH heterozygous
    loci and writes their exact breakpoints to the four BED files plus ambiguous coordinate
    files
  repo: https://github.com/JiaoLaboratory/CRAQ.git
- name: SAMtools
  role: Filters Minimap2 BAM files ('-F 1796 -q 20') before clipped-read extraction
    and coverage/clipping-ratio computation that feeds the BED output
- name: Minimap2
  role: Maps raw NGS ('-ax sr') and SMS ('-ax map-hifi/map-pb/map-ont') reads back
    to the draft assembly, generating the alignments whose clipping signals (CIGAR
    S/H) drive classification
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-42336-w
    title: Identification of errors in draft genome assemblies at single-nucleotide
      resolution for quality assessment and improvement
  source_content_hash: sha256:7ecbeefb706c3369a2af7f7999365bb52f65f47b0b144286ac4cbee314ce51a2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JiaoLaboratory/CRAQ.git
  tools:
  - CRAQ
  - Minimap2
  - Perl
  - SAMtools
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# bed-coordinate-output-generation

## Summary

Emits exact genomic coordinates of clipping-based assembly errors (CREs, CSEs) and heterozygous loci (CRHs, CSHs) detected by CRAQ as BED-formatted files, alongside ambiguous heterozygous-or-error regions, so breakpoints can be visualized, inspected, and used for assembly correction.

## When to use

Use when CRAQ clipping-ratio classification has identified candidate breakpoints and you must export exact CRE/CSE/CRH/CSH coordinates as BED files with per-locus coverage and clipped-read counts.

## When NOT to use

- The draft assembly has no raw NGS/SMS reads re-mapped to it — there is no clipping signal to classify, so no coordinate files can be produced.
- The goal is completeness or assembly-contiguity assessment (e.g., BUSCO, QUAST, LAI) rather than pinpointing single-nucleotide-resolution error and heterozygous breakpoints.
- The assembly is from a polyploid genome, where CRAQ's classification accuracy is explicitly not established.

## Inputs

- BAM files of Minimap2-mapped NGS short reads and SMS long reads filtered with 'samtools view -F 1796 -q 20' (removing MAPQ<20, unmapped, secondary, QC-failed, PCR-duplicate alignments)
- Filtered clipping-position tables in SR_clip.coverRate / LR_clip.coverRate format (chr, position, strand, number of clipped-reads, total coverage)
- CRAQ parameters: heterozygous cutoff h (0.4-0.6), breakpoint cutoff f (0.75), coverage bounds m=2 and M=5x average coverage, SMS clipped-length threshold (0.1x read length), coverage-discrepancy '-d', --gapmodel (1=CRE, 2=CSE)

## Outputs

- CRE.bed — BED coordinates of Clip-based Regional Errors
- CSE.bed — BED coordinates of Clip-based Structural Errors
- CRH.bed — BED coordinates of regional heterozygous variants
- CSH.bed — BED coordinates of structural heterozygous variants
- ambiguous.RE.RH and ambiguous.SE.SH coordinate files for loci with clipping ratios between 0.6 and 0.75
- Per-locus coverage and clipped-read counts attached to every reported error breakpoint and heterozygous locus

## How to apply

After filtering BAMs with 'samtools view -F 1796 -q 20' and extracting clipped alignments (CIGAR S/H), CRAQ computes the clipping ratio (clipped reads / total coverage) at each position from the SR_clip.coverRate and LR_clip.coverRate tables (chr, position, strand, clipped-reads, total coverage). Candidates are classified as heterozygous when the ratio falls within h (default 0.4-0.6) and as mapping breakpoints when it exceeds f (default 0.75), after excluding candidates with coverage below m=2 or above M=5x average coverage and SMS clips shorter than 0.1x read length. Error breakpoints are then assigned to CRE or CSE via the --gapmodel setting (gap[N] as CRE when 1, CSE when 2), the SMS coverage-discrepancy '-d' threshold across 200-bp flanking regions (20-bp sliding window), and the CSE rule that a 100-bp region flanking an SMS breakpoint contains an NGS breakpoint or lacks NGS coverage. CRAQ finally writes the exact breakpoints to four BED files (CRE, CSE, CRH, CSH) plus the ambiguous.RE.RH and ambiguous.SE.SH coordinate files, attaching an inspectable coverage/clipped-read count to every reported error breakpoint and heterozygous locus.

## Related tools

- **CRAQ** (Classifies clipping-ratio candidates into CRE/CSE errors and CRH/CSH heterozygous loci and writes their exact breakpoints to the four BED files plus ambiguous coordinate files) — https://github.com/JiaoLaboratory/CRAQ.git
- **SAMtools** (Filters Minimap2 BAM files ('-F 1796 -q 20') before clipped-read extraction and coverage/clipping-ratio computation that feeds the BED output)
- **Minimap2** (Maps raw NGS ('-ax sr') and SMS ('-ax map-hifi/map-pb/map-ont') reads back to the draft assembly, generating the alignments whose clipping signals (CIGAR S/H) drive classification) — https://github.com/lh3/minimap2

## Examples

```
samtools view -F 1796 -q 20 assembly.aln.bam > assembly.filtered.bam  # feed filtered BAMs to CRAQ to emit CRE/CSE/CRH/CSH BED files
```

## Evaluation signals

- All four expected BED files (CRE, CSE, CRH, CSH) exist and are validly BED-formatted coordinate tracks.
- The ambiguous.RE.RH and ambiguous.SE.SH files exist and contain the loci with clipping ratios between the upper heterozygous bound (0.6) and the breakpoint cutoff (0.75).
- Every reported error breakpoint and heterozygous locus carries a manually inspectable coverage/clipped-read count.
- Reported heterozygous loci have clipping ratios within 0.4-0.6 and mapping breakpoints exceed 0.75.
- No reported candidate has total coverage below m=2 or above M=5x the average coverage, and no SMS candidate has clipped length below 0.1x its total read length.

## Limitations

- Regions with clipping coverage over the upper level of h but below f are reported as ambiguous heterozygous-or-error regions without a definitive classification.
- The clipping ratio for assembly errors can fall below 100% in practice due to sequencing errors or inaccurate read mapping, complicating clean classification.
- CRE/CSE classification depends on the user-tunable '-d' coverage-discrepancy parameter, for which no universal value is stated.
- Accuracy of CRAQ's classification for polyploid genomes may not match the benchmarked diploid cases.

## Evidence

- [abstract] It can output coordinates of regional heterozygous variants (CRHs) and coordinates of structural heterozygous variants (CSHs) based on the ratio of clipped alignments and mapping coverage: "It can output coordinates of regional heterozygous variants (CRHs) and coordinates of structural heterozygous variants (CSHs) based on the ratio of clipped"
- [discussion] a locus is classified as heterozygous when the clipping ratio of NGS/SMS mapping is within a cutoff region h (default = 0.4–0.6): "a locus is classified as heterozygous when the clipping ratio of NGS/SMS mapping is within a cutoff region h (default = 0.4–0.6)"
- [discussion] If the assembly regions exhibited coverage over the upper level of h and below the f value, CRAQ reported these regions as ambiguous heterozygous or error region: "If the assembly regions exhibited coverage over the upper level of h and below the f value, CRAQ reported these regions as ambiguous heterozygous or error region"
- [readme] It can report precise locations of small-scale Clip-based Regional Errors (CREs), large-scale Clip-based Structural Errors (CSEs): "It can report precise locations of small-scale Clip-based Regional Errors (CREs), large-scale Clip-based Structural Errors (CSEs)"
