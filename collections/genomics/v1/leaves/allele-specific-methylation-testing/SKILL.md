---
name: allele-specific-methylation-testing
description: Use when haplotagged nanopore reads carry per-read mCG log-likelihood-ratio
  or 6mA-region calls and you need to find regions (CGIs, ChIP peaks) with statistically
  significant allele-specific methylation.
when_to_use_negative:
- The locus or sample lacks a sufficient density of heterozygous SNVs/INDELs for Margin/WhatsHap
  phasing, since haplotype phasing depends on that density
- The analysis goal is strand- or haplotype-agnostic methylation quantification (e.g.
  correlation with WGBS), which instead only requires CpGs with coverage depth ≥20
  by both WGBS and nanoHiMe-seq
- Region read coverage is below 20 reads, below the paper's threshold for allele-specific
  analysis
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0160
tools:
- name: nanoHiMe
  role: per-read CpG methylation calling (mCG mode) and 6mA region calling whose log
    likelihood ratios feed the allele-specific contingency tables
  repo: github.com/YinLabTJ/nanoHiMe
- name: Margin
  role: haplotagging of phased variants (v2.2) so each read is assigned to a haplotype
- name: WhatsHap
  role: phasing of heterozygous SNV/INDEL variants (v1.4) prior to haplotagging
- name: Samtools
  role: Q20 mapped-read filtering of sorted BAMs before methylation aggregation and
    allele-specific testing
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-35650-2
    title: Simultaneous profiling of histone modifications and DNA methylation via
      nanopore sequencing
  source_content_hash: sha256:b29d057c4f1331fd82cfd4bf8351f40caa60c515ecd6448420d03f27a6feeb2e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Margin
  - Samtools
  - WhatsHap
  - nanoHiMe
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# allele-specific-methylation-testing

## Summary

Identifies genomic regions where CpG methylation or 6mA enrichment differs significantly between haplotypes, by combining haplotagged nanoHiMe-seq reads with per-region 2x2 contingency testing. As applied in this paper, it yielded 1292 allele-specific CGIs and 1416 allele-specific H3K27me3 6mA regions in GM12878.

## When to use

Use when haplotagged nanopore reads carry per-read mCG log-likelihood-ratio or 6mA-region calls and you need to find regions (CGIs, ChIP peaks) with statistically significant allele-specific methylation.

## When NOT to use

- The locus or sample lacks a sufficient density of heterozygous SNVs/INDELs for Margin/WhatsHap phasing, since haplotype phasing depends on that density
- The analysis goal is strand- or haplotype-agnostic methylation quantification (e.g. correlation with WGBS), which instead only requires CpGs with coverage depth ≥20 by both WGBS and nanoHiMe-seq
- Region read coverage is below 20 reads, below the paper's threshold for allele-specific analysis

## Inputs

- Haplotagged, phased BAM of Q20-filtered nanoHiMe-seq nanopore reads (post samtools view -b -q 20 -F 4)
- nanoHiMe mCG output (output.mCG.eventalign.txt: per-read per-CpG log likelihood ratios)
- nanoHiMe 6mA region output (test.win.6mA_region.txt: per-read 50-bp window likelihood ratios)
- CpG island annotations and ENCODE H3K4me3/H3K27me3 peak BED files (e.g. ENCFF546TJV, ENCFF950VUB)
- Heterozygous variant set (GIAB NIST v3.3.2/GRCh37 for GM12878; Zhou et al. HepG2 dataset) and reference haplotypes

## Outputs

- Per-cell-line, per-mark tables of allele-specific regions (HepG2: 1236 CGIs, 800 H3K27me3, 476 H3K4me3; GM12878: 1292 CGIs, 1416 H3K27me3)
- Per-region 2x2 contingency tables of methylated/unmethylated counts per haplotype with Fisher's exact p-values and BH-adjusted FDR values

## How to apply

Haplotype-resolve the Q20-filtered nanoHiMe-seq alignments by haplotagging and phasing heterozygous SNV/INDEL variants with Margin v2.2 and WhatsHap v1.4. Aggregate per-read CpG methylation calls (nanoHiMe mCG mode, log likelihood ratio thresholds +1.5 methylated / −1.5 unmethylated) and 6mA region calls over CpG islands and ENCODE H3K4me3/H3K27me3 peak BEDs, retaining only regions with read coverage ≥20. For each region, build a 2x2 contingency table of methylated vs unmethylated (or 6mA-enriched) counts per haplotype and test allele-specific differences with Fisher's exact test, then reassess significance by bootstrap resampling of 250-bp windows within each region. Correct p-values with Benjamini–Hochberg at 10% FDR to call allele-specific regions.

## Related tools

- **nanoHiMe** (per-read CpG methylation calling (mCG mode) and 6mA region calling whose log likelihood ratios feed the allele-specific contingency tables) — github.com/YinLabTJ/nanoHiMe
- **Margin** (haplotagging of phased variants (v2.2) so each read is assigned to a haplotype)
- **WhatsHap** (phasing of heterozygous SNV/INDEL variants (v1.4) prior to haplotagging)
- **Samtools** (Q20 mapped-read filtering of sorted BAMs before methylation aggregation and allele-specific testing)

## Examples

```
./nanoHiMe mCG ref_upper.fa test.eventalign.txt output.mCG
```

## Evaluation signals

- Region counts match reported values after 10% FDR correction: HepG2 1236 CGIs / 800 H3K27me3 / 476 H3K4me3; GM12878 1292 CGIs / 1416 H3K27me3
- Approximately one-third of the 1292 GM12878 allele-specific CGIs map to the X chromosome
- Every retained region satisfies read coverage ≥20 per haplotype before Fisher's exact testing
- Fisher's exact calls are confirmed by bootstrap resampling of 250-bp windows within each region
- mCG classification respects the ±1.5 log likelihood ratio thresholds (positive = methylated support, negative = unmethylated support)

## Limitations

- Haplotype phasing of 6mA-containing regions and CpG methylation depends on the density of heterozygous SNVs/INDELs, degrading at loci with low heterozygosity
- nanoHiMe collapses partially methylated k-mers (methyladenines at different k-mer positions) to an averaged k-mer, so trained parameters are imprecise for such positions
- Whole-genome 6mA calling without a peak BED is very slow; the paper restricts 6mA calling to peak regions when input.peak.bed is supplied

## Evidence

- [discussion] haplotag with Margin and WhatsHap: "We used Margin version v2.2 and WhatsHap version v1.4 to haplotag and phase the variants"
- [discussion] coverage filter for allele-specific analysis: "the regions with reads coverage ≥20 were selected for the following analysis"
- [discussion] phasing limitation at low heterozygosity: "especially at the genomic loci with a low density of heterozygous SNVs and/or INDELs"
- [other] expected GM12878 allele-specific region counts: [evidence span withheld — anchor status: not_found]
- [readme] mCG calling CLI: "A positive value in Log(LIKELIHOOD_RATIO)_mCG shows the support for mCG methylation, and a negative value indicates the support for unmethylation."
