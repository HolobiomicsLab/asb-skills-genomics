---
name: coverage-based-error-breakpoint-detection
description: Use when raw NGS short reads and SMS long reads have been mapped back
  to a draft assembly (Minimap2) and clipped positions must be classified into error
  breakpoints (CRE/CSE) versus heterozygous variants (CRH/CSH) using clipping-ratio
  and coverage cutoffs.
when_to_use_negative:
- The assembly is from a polyploid species — the article states evaluation of genomes
  with higher ploidy levels may not be as accurate as the benchmarked cases.
- The goal is overall completeness/contiguity scoring rather than pinpointing clipping-based
  error breakpoints — BUSCO, QUAST, and LAI are the article's complementary methods
  for those goals.
- Only one read type (NGS or SMS) is available — CRE/CSE classification depends on
  integrating NGS and SMS mapping coverage and clipping signals.
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: CRAQ
  role: Computes effective clipping ratios and read-mapping coverage from filtered
    BAM files and classifies breakpoint candidates into CRE/CSE versus CRH/CSH, emitting
    BED coordinate files
  repo: https://github.com/JiaoLaboratory/CRAQ
- name: Minimap2
  role: Maps NGS short reads ('-ax sr') and SMS long reads ('-ax map-hifi/map-pb/map-ont')
    back to the draft assembly to produce the alignment BAMs that clipping statistics
    are derived from
  repo: https://github.com/lh3/minimap2
- name: SAMtools
  role: Filters alignments by MAPQ and flags ('samtools view -F 1796 -q 20') before
    clipped-read extraction and provides SAM/BAM access
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-42336-w
    title: Identification of errors in draft genome assemblies at single-nucleotide
      resolution for quality assessment and improvement
  source_content_hash: sha256:c549c0990d3f7100c87b98d1fad79dd091caad7f0abfa408b37e65c126f61650
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JiaoLaboratory/CRAQ
  tools:
  - CRAQ
  - Minimap2
  - Perl
  - SAMtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# coverage-based-error-breakpoint-detection

## Summary

CRAQ's two-stage clipping-ratio classification: candidate clipped positions from NGS/SMS mappings are first labeled as heterozygous loci (CRH/CSH) or mapping breakpoints using clipping-ratio cutoffs h (0.4–0.6) and f (0.75), then errors are assigned as CREs or CSEs from read-mapping coverage status, pinpointing assembly errors at single-nucleotide resolution.

## When to use

Use when raw NGS short reads and SMS long reads have been mapped back to a draft assembly (Minimap2) and clipped positions must be classified into error breakpoints (CRE/CSE) versus heterozygous variants (CRH/CSH) using clipping-ratio and coverage cutoffs.

## When NOT to use

- The assembly is from a polyploid species — the article states evaluation of genomes with higher ploidy levels may not be as accurate as the benchmarked cases.
- The goal is overall completeness/contiguity scoring rather than pinpointing clipping-based error breakpoints — BUSCO, QUAST, and LAI are the article's complementary methods for those goals.
- Only one read type (NGS or SMS) is available — CRE/CSE classification depends on integrating NGS and SMS mapping coverage and clipping signals.

## Inputs

- Filtered BAM files of NGS short-read and SMS long-read alignments to the draft assembly (Minimap2 '-ax sr' and '-ax map-hifi/map-pb/map-ont', filtered with 'samtools view -F 1796 -q 20')
- Clipping-position tables in SR_clip.coverRate / LR_clip.coverRate format (chr, position, strand, number of clipped reads, total coverage)
- User-defined parameters: h (default 0.4–0.6), f (default 0.75), m (2), M (5× average coverage), '-d' coverage-discrepancy cutoff, --gapmodel

## Outputs

- Four BED files of exact error/heterozygous breakpoints: CRE, CSE, CRH, CSH
- ambiguous.RE.RH and ambiguous.SE.SH coordinate files for ambiguous heterozygous-or-error regions
- Per-breakpoint coverage and clipped-read counts for manual inspection (e.g., in IGV)

## How to apply

Filter Minimap2 alignments with 'samtools view -F 1796 -q 20' (removing MAPQ<20, unmapped, secondary, QC-failed, duplicate reads), then extract all clipped reads coded as 'S' or 'H' in the CIGAR string and build clipping-position tables (SR_clip.coverRate / LR_clip.coverRate: chr, position, strand, clipped-read count, total coverage). At each position compute the clipping ratio (clipped reads / total coverage) and classify loci with ratios in h (default 0.4–0.6) as heterozygous and those exceeding f (default 0.75) as mapping breakpoints, excluding candidates with coverage below m=2 or above M=5× average coverage and SMS candidates whose clipped length is <0.1× total read length. Assign breakpoints lying within gaps to CRE or CSE according to the --gapmodel setting, then apply the coverage-spanning rules: call a CRE when the SMS long read spans the NGS breakpoint but shows a user '-d' coverage discrepancy across the 200-bp flanking regions (measured with a 20-bp sliding window), and call a CSE when the 100-bp region flanking an SMS breakpoint contains an NGS breakpoint or has no NGS coverage (multi-mapped reads are filtered out for CSE by default). Regions with clipping ratios between the upper heterozygous bound (0.6) and f (0.75) are reported separately as ambiguous heterozygous-or-error regions. The rationale is that true errors drive near-complete clipping (ratio approaching 100%, though it can be lower in practice due to sequencing errors or inaccurate mapping), whereas haplotype differences yield intermediate ratios, so the ratio plus coverage discrepancy separates errors from heterozygosity.

## Related tools

- **CRAQ** (Computes effective clipping ratios and read-mapping coverage from filtered BAM files and classifies breakpoint candidates into CRE/CSE versus CRH/CSH, emitting BED coordinate files) — https://github.com/JiaoLaboratory/CRAQ
- **Minimap2** (Maps NGS short reads ('-ax sr') and SMS long reads ('-ax map-hifi/map-pb/map-ont') back to the draft assembly to produce the alignment BAMs that clipping statistics are derived from) — https://github.com/lh3/minimap2
- **SAMtools** (Filters alignments by MAPQ and flags ('samtools view -F 1796 -q 20') before clipped-read extraction and provides SAM/BAM access)

## Examples

```
samtools view -F 1796 -q 20 minimap2_mapped.bam > filtered.bam
```

## Evaluation signals

- Every heterozygous call has a clipping ratio within h (0.4–0.6) and every breakpoint call exceeds f (0.75); ratios between 0.6 and 0.75 appear only in the ambiguous.RE.RH / ambiguous.SE.SH files
- Output check: exactly four BED files (CRE, CSE, CRH, CSH) plus the two ambiguous coordinate files exist and are BED-formatted
- Candidates violating coverage bounds (below m=2 or above M=5× average coverage) or the SMS clipped-length filter (<0.1× total read length) are absent from all outputs
- Each reported CSE breakpoint's 100-bp flanking region contains an NGS breakpoint or shows no NGS coverage; each CRE shows '-d' coverage discrepancy across the 200-bp flanking regions
- Every reported error breakpoint and heterozygous locus carries a manually inspectable coverage/clipped-read count (verifiable in IGV)

## Limitations

- Regions with clipping ratios between the upper level of h (0.6) and f (0.75) are reported only as ambiguous heterozygous-or-error regions without definitive classification
- The CRE coverage-discrepancy cutoff is user-tunable via '-d' with no stated universal value, and the ratio for assembly errors can fall below 100% in practice due to sequencing errors or inaccurate read mapping
- Accuracy for polyploid genomes is not established; multi-mapped reads are filtered out by default when identifying CSE breakpoints

## Evidence

- [discussion] CRAQ distinguishes error breakpoints and heterozygous loci based on read-mapping coverage data and effective clipping ratios.: "CRAQ distinguishes error breakpoints and heterozygous loci based on read-mapping coverage data and effective clipping ratios."
- [discussion] heterozygous cutoff h: "a locus is classified as heterozygous when the clipping ratio of NGS/SMS mapping is within a cutoff region h (default = 0.4–0.6)"
- [discussion] breakpoint cutoff f: "A region is classified as a mapping breakpoint when the ratio exceeds a stringent cutoff value f (default = 0.75)"
- [discussion] ambiguous regions: "If the assembly regions exhibited coverage over the upper level of h and below the f value, CRAQ reported these regions as ambiguous heterozygous or error region"
- [discussion] coverage filters m and M: "excludes candidates with extremely low coverage (m, default = 2) and high coverage (M, default = 5 * average coverage)"
- [discussion] -d parameter: "The cutoff for coverage differences is set with the ‘-d’ parameter, which compares the discrepancy in coverage of SMS reads"
- [readme] CRH/CSH outputs: "It can output coordinates of regional heterozygous variants (CRHs) and coordinates of structural heterozygous variants (CSHs) based on the ratio of clipped"
