---
name: nanopore-read-variant-calling
description: Use when nanopore reads (e.g. nanoHiMe-seq FASTQ/BAM aligned to hg19)
  must be assigned to parental alleles — e.g. to compute per-allele phasing rates
  over H3K4me3/H3K27me3 peaks or CpG islands — and heterozygous variants are available
  or need calling.
when_to_use_negative:
- The locus has a low density of heterozygous SNVs/INDELs — phasing of 6mA-containing
  regions and CpG methylation depends on heterozygous variant density and will be
  unreliable
- The analysis does not require allele resolution (e.g. bulk methylation calling or
  6mA region detection, which nanoHiMe performs on unphased eventalign input)
- Input reads are short Illumina reads, whose reported phasing rates (14.1% H3K4me3
  CUT&Tag, 16.6% H3K27me3 CUT&Tag, 5.7% WGBS) are far below nanopore and are meant
  only as comparisons
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: minimap2
  role: Aligns basecalled nanopore reads to hg19 with the map-ont preset before filtering
- name: samtools
  role: Filters the alignment to mapped reads with mapping quality ≥20 via 'samtools
    view -b -q 20 -F 4'
- name: PEPPER-Margin-DeepVariant
  role: Calls SNVs and INDELs from the filtered nanopore BAM for haplotagging
- name: Margin
  role: Haplotags heterozygous variants and reads (version v2.2)
- name: WhatsHap
  role: Phases the haplotagged variants (version v1.4)
- name: nanoHiMe
  role: Upstream HMM caller whose 6mA/mCpG calls on haplotagged reads enable allele-specific
    epigenetic analysis
  repo: github.com/YinLabTJ/nanoHiMe
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-35650-2
    title: Simultaneous profiling of histone modifications and DNA methylation via
      nanopore sequencing
  source_content_hash: sha256:3617ed6fa5f9b42386bae39fa89defca7678ae3e142928b7a1bbd90347815415
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# nanopore-read-variant-calling

## Summary

Calls and phases heterozygous variants directly from nanopore long reads (SNVs and INDELs via PEPPER-Margin-DeepVariant, haplotagging with Margin v2.2, phasing with WhatsHap v1.4) so that individual nanoHiMe-seq reads can be assigned to parental haplotypes for allele-specific epigenetic analysis.

## When to use

Use when nanopore reads (e.g. nanoHiMe-seq FASTQ/BAM aligned to hg19) must be assigned to parental alleles — e.g. to compute per-allele phasing rates over H3K4me3/H3K27me3 peaks or CpG islands — and heterozygous variants are available or need calling.

## When NOT to use

- The locus has a low density of heterozygous SNVs/INDELs — phasing of 6mA-containing regions and CpG methylation depends on heterozygous variant density and will be unreliable
- The analysis does not require allele resolution (e.g. bulk methylation calling or 6mA region detection, which nanoHiMe performs on unphased eventalign input)
- Input reads are short Illumina reads, whose reported phasing rates (14.1% H3K4me3 CUT&Tag, 16.6% H3K27me3 CUT&Tag, 5.7% WGBS) are far below nanopore and are meant only as comparisons

## Inputs

- Basecalled nanopore FASTQ plus FAST5 signal files (e.g. nanoHiMe-seq HepG2 reads)
- BAM of reads aligned to hg19, filtered to mapped, MAPQ ≥20
- Heterozygous SNV/INDEL variant set (e.g. HepG2 variants from Zhou et al.) or PEPPER-Margin-DeepVariant calls from the filtered BAM
- BED of target regions (ENCODE H3K4me3/H3K27me3 ChIP-seq peaks, CpG island annotations)

## Outputs

- Haplotagged BAM with reads assigned to haplotype 1 or haplotype 2
- Phased variant VCF (WhatsHap v1.4)
- Per-feature-class phasing-rate table (e.g. phasing_rates.tsv) with the fraction of haplotype-assigned reads for nanopore and Illumina technologies

## How to apply

Align basecalled nanopore reads to hg19 with minimap2 (map-ont preset, 8 threads) and retain only mapped reads with mapping quality ≥20 using 'samtools view -b -q 20 -F 4'. Call SNVs and INDELs from the filtered BAM with PEPPER-Margin-DeepVariant, using an established heterozygous variant set (e.g. the HepG2 dataset from Zhou et al.) as reference calls. Haplotag the variants and reads with Margin v2.2 and phase them with WhatsHap v1.4. Intersect haplotagged reads with the features of interest (ENCODE HepG2 H3K4me3 ChIP-seq peaks ENCFF223BZS/ENCFF632WJV/ENCFF712HMU, H3K27me3 peaks ENCFF546TJV/ENCFF950VUB, or CpG island annotations), restricting to regions with read coverage ≥20, then compute the fraction of reads assigned to either haplotype per feature class and technology.

## Related tools

- **minimap2** (Aligns basecalled nanopore reads to hg19 with the map-ont preset before filtering)
- **samtools** (Filters the alignment to mapped reads with mapping quality ≥20 via 'samtools view -b -q 20 -F 4')
- **PEPPER-Margin-DeepVariant** (Calls SNVs and INDELs from the filtered nanopore BAM for haplotagging)
- **Margin** (Haplotags heterozygous variants and reads (version v2.2))
- **WhatsHap** (Phases the haplotagged variants (version v1.4))
- **nanoHiMe** (Upstream HMM caller whose 6mA/mCpG calls on haplotagged reads enable allele-specific epigenetic analysis) — github.com/YinLabTJ/nanoHiMe

## Examples

```
minimap2 -ax map-ont -t 8 hg19.fa test.fastq | samtools sort -o test.sorted.bam && samtools view -b -q 20 -F 4 test.sorted.bam > test.sorted.q20.mapped.bam
```

## Evaluation signals

- Nanopore phasing rates should match the reported values: 72.2% over H3K4me3 peaks, 69.6% over H3K27me3 peaks, and 74.1% over CpG islands
- Nanopore phasing rates should exceed the reported Illumina rates (14.1% H3K4me3 CUT&Tag, 16.6% H3K27me3 CUT&Tag, 5.7% WGBS)
- All reads entering variant calling must be mapped with MAPQ ≥20 (samtools -q 20 -F 4 filter applied)
- Allele-specific intersections should be restricted to regions with read coverage ≥20
- Haplotag/phasing tool versions must match Margin v2.2 and WhatsHap v1.4 as reported

## Limitations

- Haplotype phasing of 6mA-containing regions and CpG methylation depends on the density of heterozygous SNVs/INDELs at the locus and degrades at low-variant loci
- Only regions with read coverage ≥20 are selected for allele-specific analysis, excluding sparse features
- Reference variant calls rely on an external heterozygous dataset (Zhou et al. for HepG2); without such a set, de novo PEPPER-Margin-DeepVariant calls are required

## Evidence

- [discussion] Margin and WhatsHap haplotagging and phasing: "We used Margin version v2.2 and WhatsHap version v1.4 to haplotag and phase the variants"
- [discussion] Coverage ≥20 filter for allele-specific analysis: "the regions with reads coverage ≥20 were selected for the following analysis"
- [discussion] HepG2 reference variant dataset: "or the dataset (HepG2 cell) obtained by Zhou et al.18"
- [readme] MAPQ ≥20 mapped-read filter: "samtools view -b -q 20 -F 4 test.sorted.bam > test.sorted.q20.mapped.bam"
- [results] Nanopore phasing-rate result over features: [evidence span withheld — anchor status: not_found]
