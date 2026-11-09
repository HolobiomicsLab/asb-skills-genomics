---
name: bed-file-coordinate-matching
description: Use when you have two BED-format modification-site tables (e.g. mAFiA
  predictions with modRatio and GLORI reference stoichiometry) and need a coordinate-matched
  benchmark set to compute stoichiometry correlations.
when_to_use_negative:
- Sites are reported in different genome builds or transcript coordinates that have
  not been lifted over to a common reference (e.g. GRCh38)
- The goal is read-level modification probability comparison rather than site-level
  stoichiometry aggregation
- Site coverage in the dRNA-Seq data falls below the minimum of 50 reads required
  for comparison against GLORI measurements
tools:
- name: mAFiA
  role: Produces mAFiA.sites.bed containing per-site coverage, modRatio, and confidence
    columns that are matched against GLORI_chrX.bed
  repo: https://github.com/dieterich-lab/mAFiA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-47661-2
    title: Detecting m6A at single-molecular resolution via direct RNA sequencing
      and realistic training data
  source_content_hash: sha256:54e9f909d628a3427fb7f83c8eab2b43298037c723bd35fe827a519f0508c3c9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dieterich-lab/mAFiA
  tools:
  - mAFiA
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# bed-file-coordinate-matching

## Summary

Match predicted m6A sites in mAFiA.sites.bed to reference GLORI_chrX.bed stoichiometry by genomic coordinate (chromosome, position, strand) so that site-level modRatio values can be benchmarked against orthogonal chemical measurements.

## When to use

Use when you have two BED-format modification-site tables (e.g. mAFiA predictions with modRatio and GLORI reference stoichiometry) and need a coordinate-matched benchmark set to compute stoichiometry correlations.

## When NOT to use

- Sites are reported in different genome builds or transcript coordinates that have not been lifted over to a common reference (e.g. GRCh38)
- The goal is read-level modification probability comparison rather than site-level stoichiometry aggregation
- Site coverage in the dRNA-Seq data falls below the minimum of 50 reads required for comparison against GLORI measurements

## Inputs

- mAFiA.sites.bed with columns chrom, position, strand, coverage, modRatio, confidence
- GLORI_chrX.bed reference modification sites with stoichiometry values (GRCh38 coordinates)
- Coverage threshold parameter (min_coverage = 50 for high-precision comparisons)

## Outputs

- Coordinate-matched benchmark table of 5925 HEK293 WT sites with paired mAFiA modRatio and GLORI stoichiometry
- Pearson and Spearman correlation coefficients between mAFiA and GLORI stoichiometry (expected Pearson 0.86)
- Comparison table against published CHEUI (0.64) and m6Anet (0.80) correlations

## How to apply

Load mAFiA.sites.bed produced by the mAFiA pileup step and retain only sites with coverage above the minimum threshold of 50, extracting the 'modRatio' column (ratio of single nucleotides with modification probability ≥0.5). Load the GLORI_chrX.bed reference deposit of query modification sites in bed format, which corresponds to sites listed in the GLORI publication (Liu et al, Nat Biotechnol 2023). Match records between the two BED files by (chromosome, position, strand) to build the benchmark set — 5925 HEK293 WT transcriptome sites in the published comparison. Compute Pearson (and Spearman) correlation between mAFiA modRatio and GLORI stoichiometry over the matched sites, and tabulate alongside the published comparator values (m6Anet = 0.80, CHEUI = 0.64). Matching must be strand-aware and coordinate-exact; mismatches in genome build (GRCh38) will silently drop pairs.

## Related tools

- **mAFiA** (Produces mAFiA.sites.bed containing per-site coverage, modRatio, and confidence columns that are matched against GLORI_chrX.bed) — https://github.com/dieterich-lab/mAFiA

## Examples

```
matched = pd.merge(mafia_bed, glori_bed, on=['chrom','start','strand']); r, p = scipy.stats.pearsonr(matched['modRatio'], matched['glori_stoich'])
```

## Evaluation signals

- The coordinate-matched table contains exactly 5925 HEK293 WT transcriptome sites
- Recomputed Pearson correlation between mAFiA modRatio and GLORI stoichiometry reproduces the reported S_mAFiA/S_GLRORI = 0.86 within numerical tolerance
- Every retained mAFiA site has coverage ≥ 50 reads per the GLORI comparison rule
- Matching is strand-consistent: no pairs are joined across opposite strands
- Spearman correlation should agree in direction/magnitude with the Pearson value, confirming no rank-order divergence

## Limitations

- Comparison restricted to sites with minimum dRNA-Seq coverage of 50, so low-coverage sites are excluded from benchmarking
- For high-precision measurements the mAFiA README recommends additionally a minimum confidence of 80% (ratio of nucleotides with modification probability ≥0.75 or <0.25), which would further reduce the matched site count
- Correlation depends on the overlap between mAFiA DRACH-motif coverage and GLORI-reported sites; mismatched coordinate systems silently reduce the matched set

## Evidence

- [results] A minimum dRNA-Seq coverage of 50 is required for a site to be compared against GLORI measurements: "A minimum dRNA-Seq coverage of 50 is required for a site to be compared against GLORI measurements"
- [methods] GLORI_chrX.bed is the bed-format query modification site file from GLORI: "GLORI_chrX.bed: query modification sites in bed format. This file specifically corresponds to those listed in"
- [readme] mAFiA.sites.bed gains coverage, modRatio, and confidence columns after pileup: [evidence span withheld — anchor status: external_doc]
- [readme] modRatio is defined as the ratio of nucleotides at the site with modification probability ≥0.5: [evidence span withheld — anchor status: external_doc]
