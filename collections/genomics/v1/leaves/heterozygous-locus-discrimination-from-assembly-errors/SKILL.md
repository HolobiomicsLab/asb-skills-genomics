---
name: heterozygous-locus-discrimination-from-assembly-errors
description: Use when clipped alignments (CIGAR S/H) pile up at loci during read mapping
  to a draft assembly and you must decide whether each locus is a heterozygous variant
  or a genuine assembly-error breakpoint before computing AQI scores.
when_to_use_negative:
- The input assembly is from a polyploid species — the article states evaluation for
  higher ploidy levels 'may not be as accurate as the benchmarked cases presented
  here'
- Only one read type (NGS short reads or SMS long reads) is available — the CRE/CSE
  vs CRH/CSH distinction relies on integrating NGS and SMS mapping status, so single-library
  data cannot support the classification
- The goal is completeness estimation (e.g., BUSCO) rather than pinpointing error
  breakpoints — this skill targets single-nucleotide-resolution localization of regional/structural
  errors, not aggregate completeness metrics
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: CRAQ
  role: Computes effective clipping ratios from filtered NGS/SMS alignments and classifies
    loci as CRE/CSE errors or CRH/CSH heterozygous variants using the h and f cutoffs
  repo: https://github.com/JiaoLaboratory/CRAQ
- name: Minimap2
  role: Maps raw NGS short reads ('-ax sr') and SMS long reads ('map-hifi'/'map-pb'/'map-ont')
    back to the draft assembly to generate the alignments from which clipping signals
    are extracted
  repo: https://github.com/lh3/minimap2
- name: SAMtools
  role: Filters alignments with 'samtools view -F 1796 -q 20' (MAPQ >=20, dropping
    unmapped/secondary/QC-failed/PCR-duplicate records) before clipped-read extraction
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-42336-w
    title: Identification of errors in draft genome assemblies at single-nucleotide
      resolution for quality assessment and improvement
  source_content_hash: sha256:916fe929653e258f84ed0993a38b4e1683ec40389ff431b369b97df6ed37c627
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JiaoLaboratory/CRAQ
  tools:
  - CRAQ
  - Minimap2
  - Perl
  - SAMtools
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# heterozygous-locus-discrimination-from-assembly-errors

## Summary

Classifies candidate clipped positions found by mapping NGS and SMS reads back to a draft assembly into heterozygous loci (CRH/CSH) versus assembly-error breakpoints (CRE/CSE), using CRAQ's clipping-ratio cutoffs so true misassemblies are not confounded with haplotype variation.

## When to use

Use when clipped alignments (CIGAR S/H) pile up at loci during read mapping to a draft assembly and you must decide whether each locus is a heterozygous variant or a genuine assembly-error breakpoint before computing AQI scores.

## When NOT to use

- The input assembly is from a polyploid species — the article states evaluation for higher ploidy levels 'may not be as accurate as the benchmarked cases presented here'
- Only one read type (NGS short reads or SMS long reads) is available — the CRE/CSE vs CRH/CSH distinction relies on integrating NGS and SMS mapping status, so single-library data cannot support the classification
- The goal is completeness estimation (e.g., BUSCO) rather than pinpointing error breakpoints — this skill targets single-nucleotide-resolution localization of regional/structural errors, not aggregate completeness metrics

## Inputs

- BAM files of NGS short-read and SMS long-read alignments to the draft assembly, mapped with Minimap2 ('-ax sr' for short reads; 'map-hifi'/'map-pb'/'map-ont' for long reads)
- Filtered clipping-position tables in SR_clip.coverRate / LR_clip.coverRate format (chr, position, strand, number of clipped reads, total coverage)
- User-defined parameters: h (heterozygous clipping-ratio region, default 0.4-0.6), f (breakpoint ratio cutoff, default 0.75), m/M coverage bounds (2 / 5x average), '-d' coverage-discrepancy threshold, --gapmodel setting

## Outputs

- Four BED files of exact breakpoint coordinates: CRE, CSE, CRH, and CSH
- Ambiguous heterozygous-or-error coordinate files: ambiguous.RE.RH and ambiguous.SE.SH
- Per-locus coverage and clipped-read counts attached to every reported error breakpoint and heterozygous locus

## How to apply

First filter mapped alignments with 'samtools view -F 1796 -q 20' (removing MAPQ<20, unmapped, secondary, QC-failed, PCR-duplicate reads), then extract clipped reads coded as 'S' or 'H' in the CIGAR string and compile clipping-position tables (SR_clip.coverRate / LR_clip.coverRate: chr, position, strand, clipped-reads, total coverage). At each position compute the clipping ratio (clipped reads / total coverage) and classify loci within the cutoff region h (default 0.4-0.6) as heterozygous and those exceeding the stringent cutoff f (default 0.75) as mapping breakpoints; regions falling between the upper h bound (0.6) and f are reported separately as ambiguous heterozygous-or-error. Before classification, exclude candidates with coverage below m=2 or above M=5x average coverage, and drop SMS candidates whose clipped length is <0.1x total read length. Error-classified breakpoints are then split into CREs vs CSEs using read-mapping status: a CRE when the SMS long read spans the NGS breakpoint but shows a user-set '-d' coverage discrepancy across 200-bp flanking regions (20-bp sliding window), and a CSE when the 100-bp region flanking the SMS breakpoint contains an NGS breakpoint, a gap (gap[N] handled via --gapmodel: 1=CRE, 2=CSE), or has no NGS coverage. Multi-mapped reads are filtered by default when identifying CSE breakpoints.

## Related tools

- **CRAQ** (Computes effective clipping ratios from filtered NGS/SMS alignments and classifies loci as CRE/CSE errors or CRH/CSH heterozygous variants using the h and f cutoffs) — https://github.com/JiaoLaboratory/CRAQ
- **Minimap2** (Maps raw NGS short reads ('-ax sr') and SMS long reads ('map-hifi'/'map-pb'/'map-ont') back to the draft assembly to generate the alignments from which clipping signals are extracted) — https://github.com/lh3/minimap2
- **SAMtools** (Filters alignments with 'samtools view -F 1796 -q 20' (MAPQ >=20, dropping unmapped/secondary/QC-failed/PCR-duplicate records) before clipped-read extraction)

## Examples

```
minimap2 -ax sr draft_assembly.fa ngs_reads.fq.gz | samtools view -F 1796 -q 20 -b -o ngs_filtered.bam
```

## Evaluation signals

- Every locus with clipping ratio within 0.4-0.6 is labeled CRH/CSH (heterozygous) and every locus with ratio >0.75 is labeled CRE/CSE (error breakpoint), matching the h and f defaults
- Regions with ratio between 0.6 and 0.75 appear only in ambiguous.RE.RH / ambiguous.SE.SH, never in the definitive CRE/CSE/CRH/CSH BED files
- No reported candidate has total coverage below m=2 or above M=5x average coverage, and SMS candidates all have clipped length >= 0.1x total read length
- All six output coordinate files (CRE, CSE, CRH, CSH, ambiguous.RE.RH, ambiguous.SE.SH) exist, are BED-formatted, and each reported breakpoint carries inspectable coverage/clipped-read counts
- Each CSE call has an NGS breakpoint or absent NGS coverage within the 100-bp region flanking the SMS breakpoint; each CRE call shows a '-d'-threshold coverage discrepancy across the 200-bp NGS-flanking regions

## Limitations

- Regions with clipping ratio between the upper h bound (0.6) and f (0.75) are reported only as ambiguous heterozygous-or-error without definitive classification
- The ratio for assembly errors can be lower than 100% in practice due to sequencing errors or inaccurate read mapping, complicating strict cutoff application
- The CRE/CSE split depends on a user-tunable coverage-discrepancy parameter ('-d') with no stated universal value; accuracy for polyploid genomes is not established
- Multi-mapped reads are filtered out by default when identifying CSE breakpoints, which may exclude repetitive-region evidence

## Evidence

- [abstract] abstract: "CRAQ distinguishes assembly errors from heterozygous sites or structural differences between haplotypes"
- [discussion] discussion: "a locus is classified as heterozygous when the clipping ratio of NGS/SMS mapping is within a cutoff region h (default = 0.4–0.6)"
- [readme] readme: "CRAQ also considered haplotype features which is important for identifying true misassembly."
