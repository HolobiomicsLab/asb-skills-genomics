---
name: phasing-rate-quantification-per-feature-class
description: Use when nanoHiMe-seq or other long-read data are haplotagged and you
  need per-feature-class allele-assignment fractions to compare nanopore vs short-read
  phasing performance.
when_to_use_negative:
- Genomic loci with a low density of heterozygous SNVs/INDELs, where haplotype phasing
  of 6mA-containing regions and CpG methylation is unreliable
- Regions with read coverage below 20, which the allele-specific analysis explicitly
  excludes
edam_operation: http://edamontology.org/operation_3454
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: minimap2
  role: Align basecalled nanopore reads to hg19 with the map-ont preset (8 threads)
  repo: https://github.com/YinLabTJ/nanoHiMe
- name: samtools
  role: Filter to mapped reads with mapping quality >=20 via 'samtools view -b -q
    20 -F 4'
  repo: https://github.com/YinLabTJ/nanoHiMe
- name: PEPPER-Margin-DeepVariant
  role: Call SNVs and INDELs from the filtered BAM, anchored to the Zhou et al. HepG2
    variant dataset
- name: Margin
  role: Haplotag variants and reads (version v2.2)
- name: WhatsHap
  role: Phase the haplotagged variants (version v1.4)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-35650-2
    title: Simultaneous profiling of histone modifications and DNA methylation via
      nanopore sequencing
  source_content_hash: sha256:417e145f709ab22df8a234bbadeb8e089326a2a70e988b8b7ccd999e54b37c0d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YinLabTJ/nanoHiMe
  tools:
  - Margin
  - PEPPER-Margin-DeepVariant
  - WhatsHap
  - minimap2
  - nanoHiMe
  - samtools
  edam_operation: http://edamontology.org/operation_3454
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# phasing-rate-quantification-per-feature-class

## Summary

Quantifies the fraction of nanopore reads that can be haplotag-assigned to parental alleles within defined genomic feature classes (e.g., H3K4me3 peaks, H3K27me3 peaks, CpG islands), enabling head-to-head comparison with Illumina-based phasing rates (CUT&Tag, WGBS).

## When to use

Use when nanoHiMe-seq or other long-read data are haplotagged and you need per-feature-class allele-assignment fractions to compare nanopore vs short-read phasing performance.

## When NOT to use

- Genomic loci with a low density of heterozygous SNVs/INDELs, where haplotype phasing of 6mA-containing regions and CpG methylation is unreliable
- Regions with read coverage below 20, which the allele-specific analysis explicitly excludes

## Inputs

- nanoHiMe-seq nanopore FAST5/FASTQ reads from HepG2 (H3K27me3 or H3K4me3 antibody)
- hg19 (UCSC) reference genome FASTA
- HepG2 heterozygous variant dataset from Zhou et al.
- ENCODE HepG2 H3K4me3 ChIP-seq peaks (ENCFF223BZS, ENCFF632WJV, ENCFF712HMU)
- ENCODE HepG2 H3K27me3 ChIP-seq peaks (ENCFF546TJV, ENCFF950VUB)
- CpG island annotations
- Illumina comparison reads (H3K4me3 CUT&Tag, H3K27me3 CUT&Tag, ENCODE WGBS e.g. ENCFF847OWL, ENCFF390OZB, ENCFF064GJQ, ENCFF369YQW)

## Outputs

- haplotagged, phased BAM of nanopore reads
- phasing_rates.tsv with fraction of reads assigned to either haplotype per feature class (H3K4me3 peaks, H3K27me3 peaks, CGIs) and per technology (nanopore vs Illumina)

## How to apply

Align HepG2 nanoHiMe-seq nanopore reads to hg19 with minimap2 (map-ont preset, 8 threads), then filter with 'samtools view -b -q 20 -F 4' to retain mapped reads with mapping quality >=20. Call SNVs/INDELs from the filtered BAM with PEPPER-Margin-DeepVariant using the HepG2 heterozygous variant dataset from Zhou et al. as reference calls, then haplotag variants and reads with Margin v2.2 and phase with WhatsHap v1.4. Intersect haplotagged reads with ENCODE HepG2 peak sets (H3K4me3: ENCFF223BZS, ENCFF632WJV, ENCFF712HMU; H3K27me3: ENCFF546TJV, ENCFF950VUB) and CpG island annotations, restricting to regions with read coverage >=20, and compute per feature class and per technology the fraction of reads assigned to either haplotype.

## Related tools

- **minimap2** (Align basecalled nanopore reads to hg19 with the map-ont preset (8 threads)) — https://github.com/YinLabTJ/nanoHiMe
- **samtools** (Filter to mapped reads with mapping quality >=20 via 'samtools view -b -q 20 -F 4') — https://github.com/YinLabTJ/nanoHiMe
- **PEPPER-Margin-DeepVariant** (Call SNVs and INDELs from the filtered BAM, anchored to the Zhou et al. HepG2 variant dataset)
- **Margin** (Haplotag variants and reads (version v2.2))
- **WhatsHap** (Phase the haplotagged variants (version v1.4))

## Examples

```
minimap2 -ax map-ont -t 8 hg19.fa test.fastq | samtools sort -o test.sorted.bam -T test.tmp; samtools view -b -q 20 -F 4 test.sorted.bam > test.sorted.q20.mapped.bam
```

## Evaluation signals

- Nanopore phasing rates over H3K4me3 peaks, H3K27me3 peaks and CGIs reproduce 72.2%, 69.6% and 74.1% respectively
- Nanopore phasing rates exceed the reported Illumina rates of 14.1% (H3K4me3 CUT&Tag), 16.6% (H3K27me3 CUT&Tag) and 5.7% (WGBS)
- Only reads with mapping quality >=20 (samtools -q 20 -F 4) enter the analysis
- Intersections are restricted to regions with read coverage >=20
- Output table phasing_rates.tsv has one fraction per feature class per technology

## Limitations

- Haplotype phasing of 6mA-containing regions and CpG methylation depends on the density of heterozygous SNVs/INDELs at the locus
- Phasing-rate comparisons require matched feature definitions (ENCODE HepG2 peaks) and a coverage >=20 cutoff, so rates are not comparable across arbitrary region sets

## Evidence

- [discussion] haplotag with Margin and WhatsHap: "We used Margin version v2.2 and WhatsHap version v1.4 to haplotag and phase the variants"
- [discussion] coverage >=20 for allele-specific analysis: "the regions with reads coverage ≥20 were selected for the following analysis"
- [methods] samtools mapped q20 filter: "samtools view -b -q 20 -F 4 test.sorted.bam > test.sorted.q20.mapped.bam"
- [discussion] mapping quality >=20 for model training reads: "For the reads with mapping quality ≥20, we performed initial alignment between events"
