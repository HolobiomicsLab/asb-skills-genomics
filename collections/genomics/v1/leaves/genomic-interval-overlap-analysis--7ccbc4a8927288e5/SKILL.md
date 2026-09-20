---
name: genomic-interval-overlap-analysis
description: Use when you need to quantify how many nanopore (or Illumina) reads fall
  inside annotated genomic intervals (ChIP-seq peaks, CpG islands) or to compare read-based
  metrics across feature classes and technologies.
when_to_use_negative:
- The question is base modification calling (6mA/mCpG) per read rather than interval-level
  read assignment — use the nanoHiMe 6mA/mCG modules instead
- The input is already a per-peak enrichment matrix or feature count table; no further
  interval overlap is needed
- Peaks are not defined by an external annotation and must be discovered de novo from
  coverage — this skill intersects pre-defined interval sets, it does not call peaks
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: minimap2
  role: Align basecalled nanopore reads to hg19 with the map-ont preset (8 threads)
    before interval intersection
  repo: https://github.com/YinLabTJ/nanoHiMe
- name: samtools
  role: Filter aligned reads to mapped, MAPQ ≥20 (-q 20 -F 4) prior to overlap with
    peak intervals
  repo: https://github.com/YinLabTJ/nanoHiMe
- name: bedtools
  role: Interval-operation dependency used in the nanoHiMe workflow for intersecting
    reads with peak/CGI annotations
- name: Margin
  role: 'Version v2.2: haplotag variants and reads so each overlapping read carries
    a haplotype assignment'
- name: WhatsHap
  role: 'Version v1.4: phase variants used for the haplotype assignment of overlapping
    reads'
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-35650-2
    title: Simultaneous profiling of histone modifications and DNA methylation via
      nanopore sequencing
  source_content_hash: sha256:7f308cdfb38b44ea39e238d12643ec7d320725052a0ebefd689635f974ed0ebb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YinLabTJ/nanoHiMe
  tools:
  - Margin
  - WhatsHap
  - bedtools
  - minimap2
  - nanoHiMe
  - samtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# Genomic interval overlap analysis

## Summary

Intersects haplotagged nanopore reads (nanoHiMe-seq) against curated genomic interval sets — ENCODE HepG2 H3K4me3/H3K27me3 ChIP-seq peaks and CpG island annotations — to compute per-feature-class read assignment rates, restricted to regions with read coverage ≥20.

## When to use

Use when you need to quantify how many nanopore (or Illumina) reads fall inside annotated genomic intervals (ChIP-seq peaks, CpG islands) or to compare read-based metrics across feature classes and technologies.

## When NOT to use

- The question is base modification calling (6mA/mCpG) per read rather than interval-level read assignment — use the nanoHiMe 6mA/mCG modules instead
- The input is already a per-peak enrichment matrix or feature count table; no further interval overlap is needed
- Peaks are not defined by an external annotation and must be discovered de novo from coverage — this skill intersects pre-defined interval sets, it does not call peaks

## Inputs

- Aligned, haplotagged nanopore BAM (nanoHiMe-seq reads on hg19, MAPQ ≥20, mapped only)
- ENCODE HepG2 H3K4me3 ChIP-seq peak BED files (ENCFF223BZS, ENCFF632WJV, ENCFF712HMU)
- ENCODE HepG2 H3K27me3 ChIP-seq peak BED files (ENCFF546TJV, ENCFF950VUB)
- CpG island annotation intervals
- Illumina comparison BAMs (H3K4me3 CUT&Tag, H3K27me3 CUT&Tag, WGBS)

## Outputs

- Per-feature-class, per-technology table of haplotype-assigned read fractions (phasing_rates.tsv)
- Read counts per overlapping interval with haplotype assignment and coverage

## How to apply

First align nanoHiMe-seq nanopore reads to hg19 with minimap2 (map-ont preset, 8 threads), sort, and retain only mapped reads with mapping quality ≥20 using `samtools view -b -q 20 -F 4`. Haplotag and phase variants with Margin v2.2 and WhatsHap v1.4 so each read carries a haplotype assignment. Then intersect haplotagged reads with ENCODE HepG2 H3K4me3 ChIP-seq peaks (ENCFF223BZS, ENCFF632WJV, ENCFF712HMU), H3K27me3 ChIP-seq peaks (ENCFF546TJV, ENCFF950VUB) and CpG island annotations (bedtools is the interval-operation dependency in the nanoHiMe workflow), restricting to regions with read coverage ≥20. Finally, per feature class, compute the fraction of reads assigned to either haplotype and tabulate nanopore versus Illumina rates (e.g. 72.2%/69.6%/74.1% nanopore vs 14.1%/16.6%/5.7% Illumina).

## Related tools

- **minimap2** (Align basecalled nanopore reads to hg19 with the map-ont preset (8 threads) before interval intersection) — https://github.com/YinLabTJ/nanoHiMe
- **samtools** (Filter aligned reads to mapped, MAPQ ≥20 (-q 20 -F 4) prior to overlap with peak intervals) — https://github.com/YinLabTJ/nanoHiMe
- **bedtools** (Interval-operation dependency used in the nanoHiMe workflow for intersecting reads with peak/CGI annotations)
- **Margin** (Version v2.2: haplotag variants and reads so each overlapping read carries a haplotype assignment)
- **WhatsHap** (Version v1.4: phase variants used for the haplotype assignment of overlapping reads)

## Examples

```
samtools view -b -q 20 -F 4 test.sorted.bam > test.sorted.q20.mapped.bam
```

## Evaluation signals

- Nanopore phasing fractions over H3K4me3 peaks, H3K27me3 peaks and CGIs match 72.2%, 69.6% and 74.1% respectively
- Nanopore phasing rates exceed the reported Illumina rates (14.1% H3K4me3 CUT&Tag, 16.6% H3K27me3 CUT&Tag, 5.7% WGBS)
- Only reads with MAPQ ≥20 and not unmapped (-q 20 -F 4) are included in overlap counting
- Overlapped regions carry read coverage ≥20 before rate computation
- Output table has rows per feature class (H3K4me3 peaks, H3K27me3 peaks, CGIs) and per technology (nanopore vs Illumina)

## Limitations

- Allele assignment depends on heterozygous variant density: haplotype phasing of regions fails at loci with low density of heterozygous SNVs and/or INDELs
- Regions below the ≥20x read coverage threshold are excluded, biasing rate estimates toward well-covered loci
- Peak/CGI annotations are cell-type specific (ENCODE HepG2, hg19); reuse requires matching reference genome and cell type

## Evidence

- [methods] mapped, MAPQ≥20 read filter: "samtools view -b -q 20 -F 4 test.sorted.bam > test.sorted.q20.mapped.bam"
- [discussion] coverage threshold for allele-specific analysis: "the regions with reads coverage ≥20 were selected for the following analysis"
- [discussion] haplotagging and phasing tools: "We used Margin version v2.2 and WhatsHap version v1.4 to haplotag and phase the variants"
- [discussion] ENCODE peak accessions used as interval inputs: "accession number ENCFF223BZS, ENCFF632WJV and ENCFF712HMU for H3K4me3 peak information"
