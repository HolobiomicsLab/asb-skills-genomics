---
name: bootstrap-window-resampling
description: Use when phased, haplotagged nanoHiMe-seq methylation or 6mA calls have
  been aggregated into 2x2 haplotype-by-methylation contingency tables per region
  and the Fisher's exact result must be validated against local methylation heterogeneity
  before FDR correction.
when_to_use_negative:
- Regions with read coverage below 20 — the article selects only 'regions with reads
  coverage >=20' for this analysis
- Loci with very low density of heterozygous SNVs/INDELs, where haplotype phasing
  of 6mA-containing regions and CpG methylation is unreliable and the bootstrap cannot
  rescue unassignable reads
- Whole-genome 6mA calling without peak-region restriction (input.peak.bed unset),
  where nanoHiMe 6mA calling is slow and window-level signals are not region-anchored
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0160
tools:
- name: nanoHiMe
  role: Produces the per-read mCG log-likelihood-ratio calls and 50-bp 6mA window
    LLR calls that are aggregated into the region-level contingency tables resampled
    by the bootstrap
  repo: github.com/YinLabTJ/nanoHiMe
- name: WhatsHap
  role: Phases and haplotags variants (v1.4) so methylation counts can be assigned
    per haplotype before bootstrap testing
- name: Margin
  role: v2.2 haplotagging supporting per-haplotype assignment of reads used in the
    2x2 tables
- name: Samtools
  role: Filters alignments to mapped, MAPQ>=20 reads (samtools view -b -q 20 -F 4)
    that feed the coverage>=20 region selection
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-35650-2
    title: Simultaneous profiling of histone modifications and DNA methylation via
      nanopore sequencing
  source_content_hash: sha256:b99ae8c34da33281947f257b6b76d4af0350f0abe837e27d2432310b181c06bf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Margin
  - Samtools
  - WhatsHap
  - nanoHiMe
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# bootstrap-window-resampling

## Summary

A robustness reassessment step for allele-specific epigenetic calls in nanoHiMe-seq: within each CpG island or H3K27me3/H3K4me3 peak region already tested by Fisher's exact test, 250-bp windows are bootstrap-resampled to confirm that per-haplotype methylation (mCG) or 6mA-enrichment differences are not driven by a few read-overlapping sub-windows.

## When to use

Use when phased, haplotagged nanoHiMe-seq methylation or 6mA calls have been aggregated into 2x2 haplotype-by-methylation contingency tables per region and the Fisher's exact result must be validated against local methylation heterogeneity before FDR correction.

## When NOT to use

- Regions with read coverage below 20 — the article selects only 'regions with reads coverage >=20' for this analysis
- Loci with very low density of heterozygous SNVs/INDELs, where haplotype phasing of 6mA-containing regions and CpG methylation is unreliable and the bootstrap cannot rescue unassignable reads
- Whole-genome 6mA calling without peak-region restriction (input.peak.bed unset), where nanoHiMe 6mA calling is slow and window-level signals are not region-anchored

## Inputs

- Phased, haplotagged BAM alignments (Margin v2.2 / WhatsHap v1.4) of Q20-filtered nanoHiMe-seq reads
- Per-read CpG methylation calls from nanoHiMe mCG mode (output.mCG.eventalign.txt with Log(LIKELIHOOD_RATIO)_mCG values)
- Per-read 6mA-containing 50-bp window calls from nanoHiMe 6mA mode (test.win.6mA_region.txt with LOG(LIKELIHOOD_RATIO)_mA values)
- Region BED files: CpG islands and ENCODE H3K4me3/H3K27me3 peaks (e.g., ENCFF546TJV, ENCFF950VUB for H3K27me3)
- Per-region 2x2 contingency tables of methylated vs unmethylated counts per haplotype (coverage >=20)

## Outputs

- Per-region bootstrap-resampled significance statistics for allele-specific methylation/enrichment
- Benjamini-Hochberg-corrected (10% FDR) list of allele-specific regions per cell line and mark
- Verified per-cell-line counts of allele-specific CGIs and histone-mark regions (HepG2: 1236 CGIs, 800 H3K27me3, 476 H3K4me3; GM12878: 1292 CGIs, 1416 H3K27me3)

## How to apply

First select only regions with read coverage >=20 (after haplotagging with Margin v2.2 and WhatsHap v1.4 and per-read calling with nanoHiMe, using LLR thresholds of +1.5/-1.5 for methylated/unmethylated CpGs and 8 or 32 for 50-bp 6mA-containing windows). For each region, build the 2x2 methylated-vs-unmethylated-per-haplotype contingency table and compute Fisher's exact test as the primary statistic. Then resample 250-bp sub-windows within the region with replacement (bootstrap), recomputing the allelic statistic per resample to obtain an empirical reassessment of significance. Finally apply Benjamini-Hochberg correction at 10% FDR and call allele-specific regions only when both the primary and bootstrap-resampled evidence agree, then verify counts against reported values (e.g., 1292 CGIs and 1416 H3K27me3 regions in GM12878; 1236 CGIs, 800 H3K27me3, 476 H3K4me3 in HepG2).

## Related tools

- **nanoHiMe** (Produces the per-read mCG log-likelihood-ratio calls and 50-bp 6mA window LLR calls that are aggregated into the region-level contingency tables resampled by the bootstrap) — github.com/YinLabTJ/nanoHiMe
- **WhatsHap** (Phases and haplotags variants (v1.4) so methylation counts can be assigned per haplotype before bootstrap testing)
- **Margin** (v2.2 haplotagging supporting per-haplotype assignment of reads used in the 2x2 tables)
- **Samtools** (Filters alignments to mapped, MAPQ>=20 reads (samtools view -b -q 20 -F 4) that feed the coverage>=20 region selection)

## Evaluation signals

- All regions entering the bootstrap carry read coverage >=20; regions failing this filter must be absent from outputs
- Each region's bootstrap distribution is built from resampled 250-bp windows within the same region coordinates, not across regions
- Called allele-specific region counts reproduce the reported values: 1292 CGIs and 1416 H3K27me3 regions in GM12878; 1236 CGIs, 800 H3K27me3, 476 H3K4me3 in HepG2
- Approximately one-third of GM12878 allele-specific CGIs map to the X chromosome, matching the reported finding
- Final region calls survive Benjamini-Hochberg correction at 10% FDR

## Limitations

- Allele-specific calling is limited at genomic loci with a low density of heterozygous SNVs and/or INDELs, where phasing is unreliable
- Bootstrap reassessment only strengthens window-scale robustness; it cannot recover regions lost to insufficient haplotagging or coverage
- k-mers with partially methylated adenines are collapsed to an averaged k-mer, so 6mA window calls feeding the resampling carry parameter-assignment imprecision

## Evidence

- [discussion] Region coverage filter precedes allelic testing: "the regions with reads coverage ≥20 were selected for the following analysis"
- [discussion] Haplotagging prerequisites for per-haplotype contingency tables: "We used Margin version v2.2 and WhatsHap version v1.4 to haplotag and phase the variants"
- [discussion] nanoHiMe LLR thresholds defining methylated/unmethylated CpG and 6mA window calls that populate the resampled tables: "we set a threshold of 1.5 to call methylated CpGs and −1.5 to call unmethylated CpGs, and set 8 or 32 to call a 50 bp segment as a 6mA-containing or non-6mA window"
- [readme] README: positive LLR semantics for the per-read calls being aggregated per 250-bp resample: "A positive value in Log(LIKELIHOOD_RATIO)_mCG shows the support for mCG methylation, and a negative value indicates the support for unmethylation."
