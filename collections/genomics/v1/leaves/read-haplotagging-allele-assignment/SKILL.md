---
name: read-haplotagging-allele-assignment
description: Use when nanopore long reads (e.g. nanoHiMe-seq from HepG2) must be assigned
  to paternal/maternal haplotypes at feature regions such as H3K4me3 peaks, H3K27me3
  peaks or CpG islands for allele-specific epigenetics.
when_to_use_negative:
- Loci with a low density of heterozygous SNVs/INDELs, where phasing depends on variant
  density and haplotype assignment is unreliable
- Regions with read coverage <20, which the article excludes from allele-specific
  analysis
- Short-read data where read length does not span multiple heterozygous sites — Illumina
  phasing rates (5.7–16.6%) are far below nanopore rates and this skill's advantage
  does not apply
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: minimap2
  role: Align nanopore reads to hg19 with the map-ont preset (8 threads) before haplotagging
- name: samtools
  role: Filter to mapped reads with mapping quality ≥20 (view -b -q 20 -F 4) and index
    BAMs
- name: PEPPER-Margin-DeepVariant
  role: Call SNVs and INDELs from the filtered BAM that define heterozygous sites
    for phasing
- name: Margin
  role: Haplotag variants and reads (version v2.2)
- name: WhatsHap
  role: Phase the variants (version v1.4) after haplotagging
- name: nanoHiMe
  role: Source of the nanoHiMe-seq nanopore reads and HMM-based methylation/6mA calls
    that are analyzed per allele
  repo: github.com/YinLabTJ/nanoHiMe
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-35650-2
    title: Simultaneous profiling of histone modifications and DNA methylation via
      nanopore sequencing
  source_content_hash: sha256:d65c84feaadd24cd6c8e3dc0a31c8421a581a0dafb39f3dce3fa61013c14681d
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# read-haplotagging-allele-assignment

## Summary

Assigns individual nanopore nanoHiMe-seq reads to parental haplotypes by haplotagging heterozygous SNV/INDEL variants with Margin and phasing with WhatsHap, enabling allele-specific epigenetic analysis. Long single-molecule reads spanning multiple heterozygous sites allow far higher allele-assignment rates than short Illumina reads.

## When to use

Use when nanopore long reads (e.g. nanoHiMe-seq from HepG2) must be assigned to paternal/maternal haplotypes at feature regions such as H3K4me3 peaks, H3K27me3 peaks or CpG islands for allele-specific epigenetics.

## When NOT to use

- Loci with a low density of heterozygous SNVs/INDELs, where phasing depends on variant density and haplotype assignment is unreliable
- Regions with read coverage <20, which the article excludes from allele-specific analysis
- Short-read data where read length does not span multiple heterozygous sites — Illumina phasing rates (5.7–16.6%) are far below nanopore rates and this skill's advantage does not apply

## Inputs

- nanopore nanoHiMe-seq FASTQ/FAST5 reads from HepG2 (H3K4me3 or H3K27me3 antibody pull-down)
- hg19 reference genome FASTA (indexed)
- HepG2 heterozygous SNV/INDEL variant calls (PEPPER-Margin-DeepVariant output and Zhou et al. reference dataset)
- ENCODE HepG2 H3K4me3 and H3K27me3 ChIP-seq peak BED files (ENCFF223BZS, ENCFF632WJV, ENCFF712HMU, ENCFF546TJV, ENCFF950VUB)
- CpG island annotations

## Outputs

- haplotagged BAM with reads assigned to haplotype 1 or 2
- phased VCF of heterozygous variants
- phasing_rates.tsv tabulating the fraction of reads assigned to a haplotype per feature class (H3K4me3 peaks, H3K27me3 peaks, CGIs) per technology
- filtered high-confidence BAM (mapQ ≥20, mapped only)

## How to apply

Align nanopore reads to hg19 with minimap2 (map-ont preset, 8 threads) and retain mapped reads at mapping quality ≥20 via samtools view -b -q 20 -F 4. Call SNVs and INDELs from the filtered BAM with PEPPER-Margin-DeepVariant, using the HepG2 heterozygous variant dataset from Zhou et al. as reference calls. Haplotag variants and reads with Margin v2.2 and phase them with WhatsHap v1.4. Intersect haplotagged reads with ENCODE HepG2 H3K4me3 ChIP-seq peaks (ENCFF223BZS, ENCFF632WJV, ENCFF712HMU), H3K27me3 peaks (ENCFF546TJV, ENCFF950VUB) and CpG island annotations, restricting to regions with read coverage ≥20. Finally compute, per feature class, the fraction of reads assigned to either haplotype (e.g. 72.2% of nanopore reads over H3K4me3 peaks) and compare with Illumina-based phasing rates (14.1%, 16.6%, 5.7% for H3K4me3 CUT&Tag, H3K27me3 CUT&Tag and WGBS).

## Related tools

- **minimap2** (Align nanopore reads to hg19 with the map-ont preset (8 threads) before haplotagging)
- **samtools** (Filter to mapped reads with mapping quality ≥20 (view -b -q 20 -F 4) and index BAMs)
- **PEPPER-Margin-DeepVariant** (Call SNVs and INDELs from the filtered BAM that define heterozygous sites for phasing)
- **Margin** (Haplotag variants and reads (version v2.2))
- **WhatsHap** (Phase the variants (version v1.4) after haplotagging)
- **nanoHiMe** (Source of the nanoHiMe-seq nanopore reads and HMM-based methylation/6mA calls that are analyzed per allele) — github.com/YinLabTJ/nanoHiMe

## Examples

```
samtools view -b -q 20 -F 4 test.sorted.bam > test.sorted.q20.mapped.bam
```

## Evaluation signals

- Nanopore phasing rates reproduce reported values: 72.2% over H3K4me3 peaks, 69.6% over H3K27me3 peaks and 74.1% over CpG islands
- Computed nanopore phasing rates exceed the reported Illumina rates (14.1% H3K4me3 CUT&Tag, 16.6% H3K27me3 CUT&Tag, 5.7% WGBS)
- Filtered BAM contains only mapped reads with mapping quality ≥20 (samtools view -q 20 -F 4 criteria)
- Only feature regions with read coverage ≥20 contribute to the allele-specific phasing-rate table
- Every read in the haplotagged BAM carries a haplotype tag (HP tag) consistent with the phased VCF

## Limitations

- Haplotype phasing of 6mA-containing regions and CpG methylation depends on the density of heterozygous SNVs/INDELs, especially at genomic loci with low variant density
- Whole-genome 6mA calling outside peak regions is computationally slow, so peak-restricted analysis is preferred
- Short Illumina reads achieve much lower allele-assignment rates, limiting comparability of phased epigenetic states

## Evidence

- [discussion] Margin and WhatsHap haplotagging: "We used Margin version v2.2 and WhatsHap version v1.4 to haplotag and phase the variants"
- [discussion] Coverage ≥20 filter for allele-specific analysis: "the regions with reads coverage ≥20 were selected for the following analysis"
- [discussion] HepG2 variant reference dataset: "or the dataset (HepG2 cell) obtained by Zhou et al."
- [discussion] Variant density limitation: "especially at the genomic loci with a low density of heterozygous SNVs and/or INDELs"
- [readme] Read filtering command: "samtools view -b -q 20 -F 4 test.sorted.bam > test.sorted.q20.mapped.bam"
