---
name: fisher-exact-contingency-testing
description: Use when reads have been haplotagged/phased (e.g. Margin v2.2 + WhatsHap
  v1.4) and per-region methylation or 6mA counts per haplotype are needed to call
  allele-specific CGIs or histone-marked regions.
when_to_use_negative:
- Regions with read coverage below 20 reads — the article explicitly selects only
  regions with reads coverage ≥20 for this analysis
- Genomic loci with a low density of heterozygous SNVs/INDELs, where haplotype phasing
  of 6mA-containing regions and CpG methylation is unreliable
- Data not phased/haplotagged — without haplotype assignment of reads no per-allele
  2×2 table can be constructed
tools:
- name: nanoHiMe
  role: Produces per-read CpG methylation log-likelihood ratios and 6mA-containing
    region calls that populate the contingency-table cells
  repo: https://github.com/YinLabTJ/nanoHiMe
- name: Margin
  role: Haplotags Q20-filtered nanoHiMe-seq alignments (v2.2) to assign each read
    to a parental haplotype before tabulation
- name: WhatsHap
  role: Phases heterozygous variants (v1.4) so per-haplotype methylation counts can
    be formed for Fisher's exact testing
- name: Samtools
  role: Filters reads to mapping quality ≥20 (-q 20 -F 4) prior to haplotagging and
    counting
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-35650-2
    title: Simultaneous profiling of histone modifications and DNA methylation via
      nanopore sequencing
  source_content_hash: sha256:5134188976e2d573a086587063e72e8ad80639575c36145dbb527a4e698243d1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YinLabTJ/nanoHiMe
  tools:
  - Margin
  - Samtools
  - WhatsHap
  - nanoHiMe
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# fisher-exact-contingency-testing

## Summary

Detects allele-specific CpG methylation and 6mA enrichment in haplotagged nanoHiMe-seq data by building per-region 2×2 contingency tables of methylated/unmethylated (or 6mA/non-6mA) read counts across the two phased haplotypes and testing them with Fisher's exact test, followed by Benjamini–Hochberg correction at 10% FDR.

## When to use

Use when reads have been haplotagged/phased (e.g. Margin v2.2 + WhatsHap v1.4) and per-region methylation or 6mA counts per haplotype are needed to call allele-specific CGIs or histone-marked regions.

## When NOT to use

- Regions with read coverage below 20 reads — the article explicitly selects only regions with reads coverage ≥20 for this analysis
- Genomic loci with a low density of heterozygous SNVs/INDELs, where haplotype phasing of 6mA-containing regions and CpG methylation is unreliable
- Data not phased/haplotagged — without haplotype assignment of reads no per-allele 2×2 table can be constructed

## Inputs

- Haplotagged, phased nanoHiMe-seq BAM (Margin v2.2 + WhatsHap v1.4)
- Per-read CpG methylation calls from nanoHiMe mCG mode (output.mCG.eventalign.txt)
- 6mA-containing region calls from nanoHiMe 6mA mode (test.win.6mA_region.txt)
- CpG island BED annotations
- ENCODE H3K4me3 and H3K27me3 ChIP-seq peak BEDs (e.g. ENCFF223BZS, ENCFF546TJV)

## Outputs

- Per-region 2×2 contingency tables of methylated/unmethylated read counts per haplotype
- Fisher's exact test p-values with Benjamini–Hochberg FDR (10%) corrections
- Tables of allele-specific regions per cell line and mark (e.g. 1292 CGIs and 1416 H3K27me3 regions in GM12878)

## How to apply

First haplotype-resolve Q20-filtered nanoHiMe-seq alignments by haplotagging and phasing heterozygous variants with Margin v2.2 and WhatsHap v1.4. Aggregate per-read CpG methylation calls (nanoHiMe mCG mode, log likelihood ratio thresholds +1.5 methylated / −1.5 unmethylated) and 6mA calls (thresholds 8 or 32 per 50 bp window) over CpG islands and ENCODE H3K4me3/H3K27me3 peak BEDs, retaining only regions with read coverage ≥20. For each retained region, build a 2×2 contingency table of methylated vs unmethylated counts per haplotype and apply Fisher's exact test to test allele-specific differences. Reassess borderline calls by bootstrap resampling of 250-bp windows within each region, then correct p-values with Benjamini–Hochberg at 10% FDR to define allele-specific regions.

## Related tools

- **nanoHiMe** (Produces per-read CpG methylation log-likelihood ratios and 6mA-containing region calls that populate the contingency-table cells) — https://github.com/YinLabTJ/nanoHiMe
- **Margin** (Haplotags Q20-filtered nanoHiMe-seq alignments (v2.2) to assign each read to a parental haplotype before tabulation)
- **WhatsHap** (Phases heterozygous variants (v1.4) so per-haplotype methylation counts can be formed for Fisher's exact testing)
- **Samtools** (Filters reads to mapping quality ≥20 (-q 20 -F 4) prior to haplotagging and counting)

## Examples

```
from scipy.stats import fisher_exact; odds, p = fisher_exact([[met_hap1, unmet_hap1], [met_hap2, unmet_hap2]])
```

## Evaluation signals

- Region counts reproduce the published values: 1236 CGIs, 800 H3K27me3 and 476 H3K4me3 allele-specific regions in HepG2, and 1292 CGIs and 1416 H3K27me3 regions in GM12878
- Approximately one-third of the 1292 allele-specific CGIs in GM12878 fall on the X chromosome, consistent with female-cell X-inactivation
- Every region entering the test has read coverage ≥20 as enforced before Fisher's exact testing
- Reported calls survive Benjamini–Hochberg correction at 10% FDR and bootstrap reassessment over 250-bp windows

## Limitations

- Haplotype phasing of 6mA-containing regions and CpG methylation depends on the density of heterozygous SNVs/INDELs at the locus, failing at low-variant loci
- Assignment of trained parameters to k-mers with methyladenine(s) in different positions is imprecise, so such k-mers are collapsed to an averaged k-mer, adding noise to 6mA counts
- Megalodon could not accurately identify methylated CpG sites, so methylation inputs to the contingency tables should come from nanoHiMe rather than ONT default callers

## Evidence

- [discussion] Haplotagging and phasing of variants with Margin and WhatsHap, followed by allele-specific methylation/enrichment testing: "We used Margin version v2.2 and WhatsHap version v1.4 to haplotag and phase the variants"
- [discussion] Coverage filter before allele-specific testing: "the regions with reads coverage ≥20 were selected for the following analysis"
- [discussion] Per-read methylation classification thresholds feeding the contingency tables: "we set a threshold of 1.5 to call methylated CpGs and −1.5 to call unmethylated CpGs, and set 8 or 32 to call a 50 bp segment as a 6mA-containing or non-6mA window"
- [readme] Q20 read filtering prior to haplotagging: "samtools view -b -q 20 -F 4 test.sorted.bam > test.sorted.q20.mapped.bam"
