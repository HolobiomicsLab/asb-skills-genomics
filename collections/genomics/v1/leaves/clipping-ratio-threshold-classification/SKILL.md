---
name: clipping-ratio-threshold-classification
description: Use when NGS and SMS reads mapped to a draft assembly (filtered BAMs)
  yield clipped-position tables (SR_clip.coverRate / LR_clip.coverRate) whose candidates
  must be sorted into CRE, CSE, CRH, or CSH classes by clipping ratio and coverage.
when_to_use_negative:
- Genome assemblies of species with higher ploidy levels (e.g. polyploids), for which
  the article states evaluation 'may not be as accurate as the benchmarked cases presented
  here'
- Candidates whose clipping ratio lies between the upper heterozygous bound (0.6)
  and the breakpoint cutoff (0.75) — CRAQ reports these as ambiguous rather than definitive
  CRE/CSE or CRH/CSH calls
- CSE identification in repetitive regions where multi-mapped reads dominate, unless
  multi-mapped reads are filtered as CRAQ does by default
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: CRAQ
  role: Computes effective clipping ratios from mapped-read CIGAR 'S'/'H' signals,
    applies the h (0.4-0.6) and f (0.75) cutoffs, and emits CRE/CSE/CRH/CSH coordinate
    files
  repo: https://github.com/JiaoLaboratory/CRAQ
- name: SAMtools
  role: Filters alignments with '-F 1796 -q 20' before clipping ratios are computed
    so low-MAPQ and duplicate reads do not distort the ratio
- name: Minimap2
  role: Maps NGS short reads (-ax sr) and SMS long reads (-ax map-hifi/map-pb/map-ont)
    to the draft assembly to generate the clipping evidence used for classification
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-42336-w
    title: Identification of errors in draft genome assemblies at single-nucleotide
      resolution for quality assessment and improvement
  source_content_hash: sha256:0a459c5742e63d46694575e40d58e8a76a84fd581576e0c96dba944ce20bde51
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# clipping-ratio-threshold-classification

## Summary

CRAQ's two-stage classification that labels clipped-position candidates as heterozygous loci (CRH/CSH) or mapping breakpoints using clipping-ratio cutoffs (h = 0.4–0.6, f = 0.75), then assigns errors to CREs or CSEs from NGS/SMS read-mapping status.

## When to use

Use when NGS and SMS reads mapped to a draft assembly (filtered BAMs) yield clipped-position tables (SR_clip.coverRate / LR_clip.coverRate) whose candidates must be sorted into CRE, CSE, CRH, or CSH classes by clipping ratio and coverage.

## When NOT to use

- Genome assemblies of species with higher ploidy levels (e.g. polyploids), for which the article states evaluation 'may not be as accurate as the benchmarked cases presented here'
- Candidates whose clipping ratio lies between the upper heterozygous bound (0.6) and the breakpoint cutoff (0.75) — CRAQ reports these as ambiguous rather than definitive CRE/CSE or CRH/CSH calls
- CSE identification in repetitive regions where multi-mapped reads dominate, unless multi-mapped reads are filtered as CRAQ does by default

## Inputs

- Filtered NGS BAM of short reads mapped to the assembly (samtools view -F 1796 -q 20: MAPQ >= 20, unmapped/secondary/QC-failed/duplicate removed)
- Filtered SMS BAM of long reads mapped with Minimap2 (-ax map-hifi/map-pb/map-ont)
- Clipping-position tables SR_clip.coverRate and LR_clip.coverRate (columns: chr, position, strand, number of clipped reads, total coverage)
- Average sequencing coverage of the mapping and assembly gap coordinates (for --gapmodel)

## Outputs

- Four BED files of exact breakpoints: CRE, CSE, CRH, and CSH coordinates
- ambiguous.RE.RH and ambiguous.SE.SH coordinate files for regions with clipping ratios between 0.6 and 0.75
- Per-breakpoint coverage and clipped-read counts attached to every reported error breakpoint and heterozygous locus for IGV inspection

## How to apply

Compute the clipping ratio (clipped reads / total coverage) at each position in the SR_clip.coverRate and LR_clip.coverRate tables and call a locus heterozygous when the ratio falls within h (default 0.4–0.6) and a mapping breakpoint when it exceeds f (default 0.75). Exclude candidates with extremely low coverage (m, default = 2) or high coverage (M, default = 5x average coverage), and drop SMS candidates whose clipped length is < 0.1 of total read length. Classify breakpoints passing these filters as CREs or CSEs from read-mapping status: a CRE is called when the SMS long read spans an NGS breakpoint but shows coverage discrepancy across the 200-bp flanking regions (20-bp sliding window, user-tunable '-d' threshold), while a CSE is called when the 100-bp region flanking an SMS breakpoint contains an NGS breakpoint or has no NGS coverage; breakpoints inside gaps follow the --gapmodel setting (gap[N] as CRE when 1, CSE when 2). Filter out multi-mapped reads when identifying CSE breakpoints (default). Loci with ratios above the upper h bound (0.6) but below f (0.75) are reported as ambiguous heterozygous-or-error regions rather than force-classified, because true assembly-error ratios can fall below 100% due to sequencing errors or inaccurate read mapping.

## Related tools

- **CRAQ** (Computes effective clipping ratios from mapped-read CIGAR 'S'/'H' signals, applies the h (0.4-0.6) and f (0.75) cutoffs, and emits CRE/CSE/CRH/CSH coordinate files) — https://github.com/JiaoLaboratory/CRAQ
- **SAMtools** (Filters alignments with '-F 1796 -q 20' before clipping ratios are computed so low-MAPQ and duplicate reads do not distort the ratio)
- **Minimap2** (Maps NGS short reads (-ax sr) and SMS long reads (-ax map-hifi/map-pb/map-ont) to the draft assembly to generate the clipping evidence used for classification) — https://github.com/lh3/minimap2

## Examples

```
samtools view -F 1796 -q 20 aln.bam > filtered.bam
```

## Evaluation signals

- Every locus classified heterozygous has a clipping ratio within h (0.4-0.6) and every breakpoint exceeds f (0.75); loci in 0.6-0.75 appear only in the ambiguous.RE.RH / ambiguous.SE.SH files
- No reported candidate survives with total coverage below m = 2 or above M = 5x average coverage, and all retained SMS candidates have clipped length >= 0.1 of the total read length
- All four output BED files (CRE, CSE, CRH, CSH) exist, are BED-formatted, and every reported error breakpoint carries an inspectable coverage / clipped-read count
- CRE calls coincide with SMS long reads spanning the NGS breakpoint showing coverage discrepancy across the 200-bp flanking regions (20-bp sliding window), and CSE calls have an NGS breakpoint or zero NGS coverage within 100 bp of the SMS breakpoint
- Multi-mapped reads are absent from CSE breakpoint identification (default filtering) and gap-region assignments agree with the chosen --gapmodel setting

## Limitations

- If assembly regions exhibit coverage over the upper level of h and below the f value, CRAQ reports them only as ambiguous heterozygous-or-error regions without definitive classification
- The ratio for assembly errors can be lower than 100% in practice due to sequencing errors or inaccurate read mapping, complicating strict threshold separation
- The CRE coverage-discrepancy cutoff is the user-tunable '-d' parameter with no stated universal value, and CSE/CRE assignment in gaps depends on the --gapmodel setting
- Accuracy for polyploid genomes has not been established

## Evidence

- [discussion] Heterozygous classification when clipping ratio is within h: "a locus is classified as heterozygous when the clipping ratio of NGS/SMS mapping is within a cutoff region h (default = 0.4–0.6)"
- [discussion] Breakpoint cutoff f: "A region is classified as a mapping breakpoint when the ratio exceeds a stringent cutoff value f (default = 0.75)"
- [discussion] Coverage coverage filters m and M: "excludes candidates with extremely low coverage (m, default = 2) and high coverage (M, default = 5 * average coverage)"
- [discussion] Ambiguous regions between h and f: "If the assembly regions exhibited coverage over the upper level of h and below the f value, CRAQ reported these regions as ambiguous heterozygous or error region"
- [discussion] CRE/CSE error classes: "CRAQ distinguishes error breakpoints and heterozygous loci based on read-mapping coverage data and effective clipping ratios."
- [readme] CRH/CSH output coordinates: "It can output coordinates of regional heterozygous variants (CRHs) and coordinates of structural heterozygous variants (CSHs) based on the ratio of clipped"
- [discussion] Multi-mapped read filtering for CSE: "such multi-mapped reads were filtered out when CRAQ was applied to identify CSE breakpoints by default"
