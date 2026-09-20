---
name: haplotype-phased-read-grouping
description: Use when nanoHiMe-seq (or similar nanopore) reads must be assigned to
  haplotypes to test allele-specific CpG methylation or allele-specific 6mA enrichment
  in CpG islands and ENCODE H3K27me3/H3K4me3 peak regions.
when_to_use_negative:
- Loci with low density of heterozygous SNVs/INDELs, where the article notes phasing
  of 6mA regions and CpG methylation is unreliable
- Regions with read coverage below 20, which the article excludes from allele-specific
  testing
- Short-read or bisulfite data without per-read haplotype information — this skill
  presumes long nanopore reads jointly called for 6mA and mCpG by nanoHiMe
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3295
tools:
- name: Margin
  role: Haplotagging nanopore reads against heterozygous variant calls (version v2.2)
- name: WhatsHap
  role: Phasing heterozygous variants used to assign reads to haplotypes (version
    v1.4)
- name: nanoHiMe
  role: Produces the per-read mCG and 6mA log-likelihood-ratio calls that are grouped
    by haplotype
  repo: github.com/YinLabTJ/nanoHiMe
- name: Samtools
  role: Filters mapped reads at MAPQ ≥20 prior to haplotagging (view -b -q 20 -F 4)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-35650-2
    title: Simultaneous profiling of histone modifications and DNA methylation via
      nanopore sequencing
  source_content_hash: sha256:fb4fb39a9180cfeb2ece025de4c82945ae56d453c2af9412d4b8057cc910f47d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Margin
  - Samtools
  - WhatsHap
  - nanoHiMe
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# haplotype-phased-read-grouping

## Summary

Groups nanopore reads by parental haplotype using haplotagging and phasing of heterozygous variants (Margin v2.2, WhatsHap v1.4) so that per-read CpG methylation calls and 6mA region calls from nanoHiMe can be compared between alleles for allele-specific epigenetic analysis.

## When to use

Use when nanoHiMe-seq (or similar nanopore) reads must be assigned to haplotypes to test allele-specific CpG methylation or allele-specific 6mA enrichment in CpG islands and ENCODE H3K27me3/H3K4me3 peak regions.

## When NOT to use

- Loci with low density of heterozygous SNVs/INDELs, where the article notes phasing of 6mA regions and CpG methylation is unreliable
- Regions with read coverage below 20, which the article excludes from allele-specific testing
- Short-read or bisulfite data without per-read haplotype information — this skill presumes long nanopore reads jointly called for 6mA and mCpG by nanoHiMe

## Inputs

- Q20-filtered nanoHiMe-seq BAM alignments to hg19 (samtools view -q 20 -F 4)
- Heterozygous variant VCF per cell line (GIAB NIST v3.3.2/GRCh37 for GM12878; Zhou et al. dataset for HepG2)
- Per-read nanoHiMe mCG methylation output (output.mCG.eventalign.txt with LOG(LIKELIHOOD_RATIO)_mCG per read)
- nanoHiMe 6mA region calls (output.6mA.methylation.txt with LOG(LIKELIHOOD_RATIO)_mA per 50-bp window per read)
- Region BEDs: CpG islands and ENCODE H3K27me3 peaks (ENCFF546TJV, ENCFF950VUB) and H3K4me3 peaks (ENCFF223BZS, ENCFF632WJV, ENCFF712HMU)

## Outputs

- Haplotagged, phased BAM in which each read is assigned to a parental haplotype (HP tag)
- Per-region, per-haplotype methylated/unmethylated (or 6mA-enriched) count tables over CGIs and histone-mark peaks
- Lists of allele-specific regions at 10% BH-FDR (e.g. 1292 CGIs and 1416 H3K27me3 regions in GM12878; 1236 CGIs, 800 H3K27me3, 476 H3K4me3 regions in HepG2)

## How to apply

Starting from Q20-filtered nanoHiMe-seq BAM alignments to hg19, haplotag and phase heterozygous variants with Margin v2.2 and WhatsHap v1.4, using a matched variant truth set (GIAB NIST v3.3.2/GRCh37 for GM12878, Zhou et al. for HepG2) so each read carries an HP tag. Aggregate nanoHiMe mCG log-likelihood-ratio calls (thresholds +1.5 methylated / −1.5 unmethylated) and 6mA region calls per read over CpG islands and ENCODE H3K27me3/H3K4me3 peak BEDs, retaining only regions with read coverage ≥20. For each region build a 2×2 contingency table of methylated vs unmethylated (or 6mA-enriched) counts per haplotype, test with Fisher's exact test, then assess significance by bootstrap resampling of 250-bp windows within each region and correct with Benjamini–Hochberg at 10% FDR. Phasing power depends on local heterozygous SNV/INDEL density, so exclude low-density loci where haplotagging is unreliable.

## Related tools

- **Margin** (Haplotagging nanopore reads against heterozygous variant calls (version v2.2))
- **WhatsHap** (Phasing heterozygous variants used to assign reads to haplotypes (version v1.4))
- **nanoHiMe** (Produces the per-read mCG and 6mA log-likelihood-ratio calls that are grouped by haplotype) — github.com/YinLabTJ/nanoHiMe
- **Samtools** (Filters mapped reads at MAPQ ≥20 prior to haplotagging (view -b -q 20 -F 4))

## Examples

```
samtools view -b -q 20 -F 4 test.sorted.bam > test.sorted.q20.mapped.bam && samtools index test.sorted.q20.mapped.bam  # then Margin v2.2 + WhatsHap v1.4 haplotag/phase, Fisher's exact test, BH 10% FDR
```

## Evaluation signals

- Region counts reproduce published values: 1292 CGIs and 1416 H3K27me3 6mA-enriched regions in GM12878; 1236 CGIs, 800 H3K27me3, 476 H3K4me3 in HepG2
- Roughly one-third of the 1292 allele-specific CGIs in GM12878 fall on the X chromosome, consistent with X-inactivation-linked methylation asymmetry
- Every tested region has ≥20 reads with haplotype assignments (HP tags); regions below this coverage are excluded
- Fisher's exact p-values survive Benjamini–Hochberg correction at 10% FDR, and significance is retained after bootstrap resampling of 250-bp windows within each region
- mCG calls are thresholded consistently (LLR ≥ +1.5 methylated, ≤ −1.5 unmethylated) before building the 2×2 per-haplotype contingency tables

## Limitations

- Haplotype phasing of 6mA-containing regions and CpG methylation depends on the density of heterozygous SNVs/INDELs and degrades at loci with low variant density
- 6mA calling genome-wide is slow; constraining calling to peak regions via input.peak.bed is recommended
- Region counts depend on the chosen ENCODE peak BEDs and coverage cutoff, so results are reference-variant-set and threshold dependent

## Evidence

- [discussion] We used Margin version v2.2 and WhatsHap version v1.4 to haplotag and phase the variants: "We used Margin version v2.2 and WhatsHap version v1.4 to haplotag and phase the variants"
- [discussion] the regions with reads coverage ≥20 were selected for the following analysis: "the regions with reads coverage ≥20 were selected for the following analysis"
- [discussion] especially at the genomic loci with a low density of heterozygous SNVs and/or INDELs: "especially at the genomic loci with a low density of heterozygous SNVs and/or INDELs"
- [discussion] we set a threshold of 1.5 to call methylated CpGs and −1.5 to call unmethylated CpGs: "we set a threshold of 1.5 to call methylated CpGs and −1.5 to call unmethylated CpGs, and set 8 or 32 to call a 50 bp segment as a 6mA-containing or non-6mA window"
- [readme] samtools view -b -q 20 -F 4 test.sorted.bam > test.sorted.q20.mapped.bam: "samtools view -b -q 20 -F 4 test.sorted.bam > test.sorted.q20.mapped.bam"
