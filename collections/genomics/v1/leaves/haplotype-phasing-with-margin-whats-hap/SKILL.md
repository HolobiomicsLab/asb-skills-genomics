---
name: haplotype-phasing-with-margin-whats-hap
description: Use when long nanopore reads must be assigned to haplotypes at heterozygous
  SNVs/INDELs to compute per-allele phasing rates or allele-specific methylation/enrichment
  over peak regions or CpG islands.
when_to_use_negative:
- Loci with a low density of heterozygous SNVs and/or INDELs, where phasing is unreliable
- Illumina short-read data where much lower phasing rates are expected (14.1% H3K4me3
  CUT&Tag, 16.6% H3K27me3 CUT&Tag, 5.7% WGBS) rather than nanopore reads
- Analysis of aggregated methylation levels rather than allele-resolved (haplotype-assigned)
  read-level calls
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: minimap2
  role: Aligns basecalled nanopore reads to hg19 with the map-ont preset (v2.17, default
    settings) before filtering and haplotagging
- name: samtools
  role: Sorts, indexes and filters the BAM to mapped reads with mapping quality ≥20
    (-q 20 -F 4)
- name: PEPPER-Margin-DeepVariant
  role: Calls heterozygous SNVs and INDELs from the filtered BAM that seed haplotagging
    and phasing
- name: Margin
  role: Haplotags the nanopore reads against the heterozygous variants (version v2.2)
- name: WhatsHap
  role: Phases the heterozygous variants (version v1.4)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-35650-2
    title: Simultaneous profiling of histone modifications and DNA methylation via
      nanopore sequencing
  source_content_hash: sha256:643aec6f1ca33a433d113fdf5e9a5d8fa7c8672b1e6e2876cf8d32d0e5c2280d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Margin
  - PEPPER-Margin-DeepVariant
  - WhatsHap
  - minimap2
  - nanoHiMe
  - samtools
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# haplotype-phasing-with-margin-whats hap

## Summary

Assigns nanoHiMe-seq nanopore reads to their parental alleles by haplotagging with Margin v2.2 and phasing variants with WhatsHap v1.4, enabling allele-specific epigenetic (6mA/mCpG) analysis on HepG2 H3K4me3/H3K27me3 peaks and CpG islands.

## When to use

Use when long nanopore reads must be assigned to haplotypes at heterozygous SNVs/INDELs to compute per-allele phasing rates or allele-specific methylation/enrichment over peak regions or CpG islands.

## When NOT to use

- Loci with a low density of heterozygous SNVs and/or INDELs, where phasing is unreliable
- Illumina short-read data where much lower phasing rates are expected (14.1% H3K4me3 CUT&Tag, 16.6% H3K27me3 CUT&Tag, 5.7% WGBS) rather than nanopore reads
- Analysis of aggregated methylation levels rather than allele-resolved (haplotype-assigned) read-level calls

## Inputs

- Basecalled nanopore FASTQ (with indexed FAST5 signal data)
- hg19 reference FASTA (samtools-faidxed, upper-cased)
- Aligned BAM filtered for mapped reads with MAPQ ≥20 (samtools view -b -q 20 -F 4)
- Heterozygous variant calls (PEPPER-Margin-DeepVariant; HepG2 dataset from Zhou et al.)
- ENCODE HepG2 H3K4me3 and H3K27me3 ChIP-seq peak BED files and CpG island annotations

## Outputs

- Haplotagged BAM with reads assigned to haplotypes (HP tags)
- Phased VCF of heterozygous SNVs/INDELs
- phasing_rates.tsv: fraction of reads assigned to either haplotype per feature class (H3K4me3 peaks, H3K27me3 peaks, CGIs) per technology (nanopore vs Illumina)

## How to apply

First align basecalled nanopore reads to hg19 with minimap2 (map-ont preset, 8 threads), sort, and keep only mapped reads with MAPQ ≥20 via `samtools view -b -q 20 -F 4`. Call SNVs/INDELs from the filtered BAM with PEPPER-Margin-DeepVariant, using the HepG2 heterozygous variant dataset from Zhou et al. as reference calls. Haplotag reads with Margin version v2.2 and phase variants with WhatsHap version v1.4, then intersect haplotagged reads with ENCODE HepG2 H3K4me3 peaks (ENCFF223BZS, ENCFF632WJV, ENCFF712HMU), H3K27me3 peaks (ENCFF546TJV, ENCFF950VUB) and CpG island annotations, restricting to regions with read coverage ≥20. Finally compute, per feature class and technology, the fraction of reads assigned to either haplotype and write the results to phasing_rates.tsv.

## Related tools

- **minimap2** (Aligns basecalled nanopore reads to hg19 with the map-ont preset (v2.17, default settings) before filtering and haplotagging)
- **samtools** (Sorts, indexes and filters the BAM to mapped reads with mapping quality ≥20 (-q 20 -F 4))
- **PEPPER-Margin-DeepVariant** (Calls heterozygous SNVs and INDELs from the filtered BAM that seed haplotagging and phasing)
- **Margin** (Haplotags the nanopore reads against the heterozygous variants (version v2.2))
- **WhatsHap** (Phases the heterozygous variants (version v1.4))

## Examples

```
samtools view -b -q 20 -F 4 test.sorted.bam > test.sorted.q20.mapped.bam
```

## Evaluation signals

- Nanopore phasing rates reproduce the reported values: 72.2% of reads over H3K4me3 peaks, 69.6% over H3K27me3 peaks and 74.1% over CpG islands
- Nanopore phasing rates exceed the reported Illumina rates (14.1% H3K4me3 CUT&Tag, 16.6% H3K27me3 CUT&Tag, 5.7% WGBS)
- Filtered BAM contains only mapped reads with MAPQ ≥20 (samtools view -b -q 20 -F 4)
- Allele-specific analysis is restricted to regions with read coverage ≥20
- Haplotagged BAM carries HP tags consistent with the phased VCF from Margin v2.2 / WhatsHap v1.4

## Limitations

- Haplotype phasing of 6mA-containing regions and CpG methylation depends on the density of heterozygous SNVs/INDELs, and degrades at loci with low heterozygous variant density
- Phasing quality is tied to the accuracy of the input variant calls (here the HepG2 dataset from Zhou et al.)
- Region-level rates require sufficient depth; only regions with read coverage ≥20 are retained, so low-coverage peaks are excluded

## Evidence

- [discussion] We used Margin version v2.2 and WhatsHap version v1.4 to haplotag and phase the variants: "We used Margin version v2.2 and WhatsHap version v1.4 to haplotag and phase the variants"
- [discussion] the regions with reads coverage ≥20 were selected for the following analysis: "the regions with reads coverage ≥20 were selected for the following analysis"
- [discussion] haplotype phasing ... depends on the density of heterozygous SNVs/INDELs: "especially at the genomic loci with a low density of heterozygous SNVs and/or INDELs"
- [readme] samtools MAPQ filter command: "samtools view -b -q 20 -F 4 test.sorted.bam > test.sorted.q20.mapped.bam"
