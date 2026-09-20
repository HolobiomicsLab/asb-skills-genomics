---
name: cross-tool-error-overlap-analysis
description: Use when two error/variant callers (e.g., CRAQ and SyRI) have been run
  on the same draft assembly and you need to compute what fraction of one tool's breakpoints
  fall inside the other's SVs, and categorize the unique calls.
when_to_use_negative:
- Only one caller has been run — there is no second set of SV/error coordinates to
  intersect, so no overlap analysis is possible
- The two callers were run against different assemblies or incompatible coordinate
  systems (e.g., different references), making breakpoint-in-SV-boundary comparison
  meaningless without liftover
- Genome has ploidy higher than diploid — the article notes CRAQ accuracy 'for species
  with higher ploidy levels may not be as accurate as the benchmarked cases presented
  here'
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3673
tools:
- name: CRAQ
  role: Calls CRE/CSE error breakpoints from clipped-alignment signals in the draft
    assembly; its output coordinates are one side of the overlap analysis
  repo: github.com/JiaoLaboratory/CRAQ
- name: Minimap2
  role: Maps Nanopore long reads ('-ax map-ont') and Illumina short reads ('-ax sr')
    to the assembly to produce the BAMs CRAQ consumes
  repo: github.com/lh3/minimap2
- name: SAMtools
  role: Filters alignments with 'samtools view -F 1796 -q 20' before CRAQ, removing
    low-MAPQ, unmapped, secondary, QC-failed, and duplicate reads
- name: SyRI
  role: Independent SV caller producing the SV boundary set that CRAQ breakpoints
    are intersected against; run with the alternative assembly as reference
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-42336-w
    title: Identification of errors in draft genome assemblies at single-nucleotide
      resolution for quality assessment and improvement
  source_content_hash: sha256:3edc3471b34e65ef98025b9da4fa69bb41c81c46149e300a27b308acf2f18734
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CRAQ
  - Minimap2
  - SAMtools
  - SyRI
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# cross-tool-error-overlap-analysis

## Summary

Quantify the concordance between assembly errors called by CRAQ (CREs/CSEs from clipped-read signals) and structural variants called by an independent tool (e.g., SyRI) on the same assembly, classifying breakpoint overlaps and assigning non-overlapping calls to error categories.

## When to use

Use when two error/variant callers (e.g., CRAQ and SyRI) have been run on the same draft assembly and you need to compute what fraction of one tool's breakpoints fall inside the other's SVs, and categorize the unique calls.

## When NOT to use

- Only one caller has been run — there is no second set of SV/error coordinates to intersect, so no overlap analysis is possible
- The two callers were run against different assemblies or incompatible coordinate systems (e.g., different references), making breakpoint-in-SV-boundary comparison meaningless without liftover
- Genome has ploidy higher than diploid — the article notes CRAQ accuracy 'for species with higher ploidy levels may not be as accurate as the benchmarked cases presented here'

## Inputs

- Filtered BAM files of Nanopore long-read and Illumina short-read alignments to the draft assembly (e.g., Canu S. pennellii assembly)
- CRAQ CRE/CSE breakpoint coordinate lists (output of CRAQ run)
- SyRI output listing SVs (e.g., translocations/inversions) with boundaries, using an alternative assembly (e.g., CaSM) as reference
- Error-category definitions (errors in reference, heterozygous sites, noisy base-error clusters, CRAQ low-confidence regions, shared errors, others)

## Outputs

- Overlap fractions: proportion of CRAQ errors overlapping SyRI SVs and proportion of SyRI SVs overlapping CRAQ errors (e.g., 5736/8029 = 71.4% and 6539/13114 = 49.8% in the S. pennellii case)
- Per-breakpoint overlap classification table (overlapping vs. unique)
- Category-assigned breakdown of non-overlapping calls (Fig. 3a-style)

## How to apply

First run both callers on the same assembly: CRAQ needs Minimap2-filtered BAMs (short reads with '-ax sr', long reads with '-ax map-ont', both filtered with 'samtools view -F 1796 -q 20') and uses default clipping-ratio cutoffs (heterozygous h = 0.4–0.6, breakpoint f = 0.75) with multi-mapped-read filtering for CSEs. Run SyRI with the alternative assembly (e.g., CaSM) as reference and the target assembly (e.g., Canu, >500k contigs) as query, ignoring SNPs, HDRs, INSs, and DELs < 40 bp, and merge overlapped SV regions. Classify each CRAQ CRE/CSE breakpoint as overlapping a SyRI SV if it falls within the SV boundary or the adjacent 50-bp flanking region, then compute overlap fractions in both directions (CRAQ errors overlapping SVs, and SVs overlapping CRAQ errors). For the Fig. 3a-style breakdown, assign CRAQ-unique and SyRI-unique calls to reported categories: errors in the CaSM reference, heterozygous sites, noisy base-error clusters, CRAQ low-confidence regions, errors shared between Canu and CaSM, and others.

## Related tools

- **CRAQ** (Calls CRE/CSE error breakpoints from clipped-alignment signals in the draft assembly; its output coordinates are one side of the overlap analysis) — github.com/JiaoLaboratory/CRAQ
- **Minimap2** (Maps Nanopore long reads ('-ax map-ont') and Illumina short reads ('-ax sr') to the assembly to produce the BAMs CRAQ consumes) — github.com/lh3/minimap2
- **SAMtools** (Filters alignments with 'samtools view -F 1796 -q 20' before CRAQ, removing low-MAPQ, unmapped, secondary, QC-failed, and duplicate reads)
- **SyRI** (Independent SV caller producing the SV boundary set that CRAQ breakpoints are intersected against; run with the alternative assembly as reference)

## Examples

```
minimap2 -ax map-ont canu_spen.fa ont_reads.fastq | samtools view -F 1796 -q 20 -b > ont_filtered.bam; python scripts/overlap_craq_syri.py --craq CREs_CSEs.bed --syri syri.out --flank 50
```

## Evaluation signals

- Reproduce the reported S. pennellii overlap: ~71.4% (5736/8029) of CRAQ errors overlapping 49.8% (6539/13,114) of SyRI SVs
- CRAQ should report 8029 breakpoints in the Canu S. pennellii assembly (7910 CREs, 119 CSEs)
- SAMtools filtering must use '-F 1796 -q 20'; clipping-ratio cutoffs h = 0.4–0.6 and f = 0.75 should be at defaults
- Overlap classification should include the 50-bp flanking region around each SV boundary, not just strict boundary containment
- Every non-overlapping call must be assignable to one of the reported categories (CaSM-reference errors, heterozygous sites, noisy base-error clusters, CRAQ low-confidence regions, shared errors, others)

## Limitations

- Optical mapping-based validation (e.g., Bionano DLS) 'could only infer the approximate genomic locations of misjoins', so third-party validation of overlap calls is coarse
- CRAQ accuracy for polyploid genomes is unestablished — overlap analysis inherits this caveat
- Error-breakpoint clipping ratios can fall below 100% in practice 'due to sequencing errors or inaccurate read mapping', complicating CRE/CSE classification
- The 50-bp adjacent-region overlap window is a user-defined tolerance that affects the computed overlap fractions

## Evidence

- [discussion] Filter both alignment files with samtools view: "filtered out using the ‘-F 1796 -q 20’ parameters in ‘samtools view’35"
- [discussion] Clipping-ratio cutoffs: "a locus is classified as heterozygous when the clipping ratio of NGS/SMS mapping is within a cutoff region h (default = 0.4–0.6)"
- [discussion] SyRI query/reference setup: "The Canu assemblies (>500k) of S. pennellii were input as the query genome and the CaSM assembly was input as the reference"
- [readme] CRAQ overview: "It can report precise locations of small-scale Clip-based Regional Errors (CREs), large-scale Clip-based Structural Errors (CSEs)"
- [abstract] Minimap2 presets: "By default, Minimap2 ‘–ax sr’ and ‘–ax map-hifi’(‘map-hifi’ for PacBio HiFi,‘map-pb’ for PacBio CLR, ‘map-ont’ for ONT library) options were used"
- [discussion] Polyploid caveat: "evaluation of genome assemblies for species with higher ploidy levels may not be as accurate as the benchmarked cases presented here"
