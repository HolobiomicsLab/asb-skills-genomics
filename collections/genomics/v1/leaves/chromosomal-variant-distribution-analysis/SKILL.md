---
name: chromosomal-variant-distribution-analysis
description: Use when a snpEff-annotated VCF or variant table from a JWES run needs
  to be summarized as counts/shares per chromosome and per coding/noncoding gene region,
  with validation against known distributions.
when_to_use_negative:
- The variant set has not yet been functionally annotated with snpEff gene regions,
  so coding/noncoding classification is impossible
- The goal is raw variant discovery rather than post-hoc distribution summarization
  of an already-called, annotated VCF (use the JWES processing module instead)
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0199
tools:
- name: JWES
  role: Pipeline that produced the annotated variant set being summarized
  repo: github.com/drzeeshanahmed/JWES-DB
- name: snpEff
  role: Provides the gene-region interval-forest annotations used to classify variants
    as noncoding or protein-coding
- name: VariantsToTable
  role: Extracts CHROM/POS/gene fields from the annotated VCF into a table for aggregation
- name: JWES-DB
  role: VCF parser component for accessing JWES variant call output
  repo: github.com/drzeeshanahmed/JWES-DB
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/2211-5463.13261
    title: 'JWES: a new pipeline for whole genome/exome sequence data processing,
      management, and gene‐variant discovery, annotation, prediction, and genotyping'
  source_content_hash: sha256:c6ff03f30981b492b7656bf7dd6c9e32f99133697e70dffeafb760cd998166a2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - JWES
  - JWES-DB
  - VariantsToTable
  - snpEff
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# chromosomal-variant-distribution-analysis

## Summary

Aggregate snpEff-annotated variants from a JWES-processed WGS/WES sample into per-chromosome and noncoding vs protein-coding gene-region distribution tables with counts and percentages, e.g. reproducing SRR891275's 12,979 noncoding / 7,770 coding split across 20,749 variants.

## When to use

Use when a snpEff-annotated VCF or variant table from a JWES run needs to be summarized as counts/shares per chromosome and per coding/noncoding gene region, with validation against known distributions.

## When NOT to use

- The variant set has not yet been functionally annotated with snpEff gene regions, so coding/noncoding classification is impossible
- The goal is raw variant discovery rather than post-hoc distribution summarization of an already-called, annotated VCF (use the JWES processing module instead)

## Inputs

- snpEff-annotated variant set (VCF or CSV) for a JWES-processed sample (e.g. SRR891275, PRJNA207663)
- snpEff interval-forest gene-region annotations distinguishing noncoding vs protein-coding genes
- Genomic interval mapping of variants to chromosomes (hg38 reference coordinates)

## Outputs

- Combined chromosome-level and gene-region-level distribution table with counts and percentages (variant_distribution_SRR891275.csv)
- Per-chromosome variant shares (e.g. chromosome 2 ~11.8% of all variants, ~7.4% of protein-coding variants)
- Noncoding vs protein-coding variant counts (e.g. 12,979 ~63% / 7,770 ~37%)

## How to apply

Load the snpEff-annotated variant set (VCF/CSV) for the sample produced by the JWES pipeline, using the JWES-DB VCF parser where the variants are stored in a database. Classify each variant as falling in a noncoding gene or a protein-coding gene based on the snpEff interval-forest annotations, then aggregate counts per gene region and per chromosome. For SRR891275 the expected result is 12,979 (~63%) noncoding and 7,770 (~37%) protein-coding variants out of 20,749 total, with chromosome 2 as the largest contributor (~11.8% of all variants and ~7.4% of protein-coding variants). Compute each chromosome's share of all variants and of protein-coding variants, and write the combined chromosome-level and gene-region-level distribution table with counts and percentages to variant_distribution_<sample>.csv. Validate by recomputing the table and checking it reproduces the reported counts and percentages.

## Related tools

- **JWES** (Pipeline that produced the annotated variant set being summarized) — github.com/drzeeshanahmed/JWES-DB
- **snpEff** (Provides the gene-region interval-forest annotations used to classify variants as noncoding or protein-coding)
- **VariantsToTable** (Extracts CHROM/POS/gene fields from the annotated VCF into a table for aggregation)
- **JWES-DB** (VCF parser component for accessing JWES variant call output) — github.com/drzeeshanahmed/JWES-DB

## Examples

```
gatk VariantsToTable -V SRR891275_snpeff.vcf -F CHROM -F POS -F GENE -F Feature -O SRR891275_variants.table
```

## Evaluation signals

- Recomputed totals match the reported variant count for the sample (SRR891275: 20,749 variants)
- Gene-region split reproduces 12,979 (~63%) noncoding and 7,770 (~37%) protein-coding variants
- Chromosome-level table shows chromosome 2 as the largest contributor (~11.8% of all variants, ~7.4% of protein-coding variants)
- Counts across chromosomes sum to the total variant count and percentage columns sum to ~100% per partition
- Every variant in the distribution table is classified as exactly one of noncoding or protein-coding

## Limitations

- Validation targets are sample-specific; other samples (e.g. SRR12474733 with 43,685 variants, or the 4,803,792 variants across 14 WGS samples) will have different distributions and need their own reference values
- Classification accuracy depends entirely on the snpEff interval-forest annotations; variants outside annotated gene intervals may not be covered by the coding/noncoding partition
- JWES's current version offers a command-line interface only, with advanced visualizations such as dynamic heat maps and cluster maps not yet implemented, limiting richer distribution visualization beyond Circos graphs

## Evidence

- [results] JWES pipeline variant output: "and 20 749 variants for PRJNA207663"
- [abstract] circos visualization support: "the data visualization module supports variant data interpretation with Circos graphs"
- [readme] JWES-DB parser: "## README: drzeeshanahmed/JWES-DB
# JWES-DB
VCF Parser for JWES"
- [abstract] annotation capability: "We have successfully applied JWES for processing, management, and gene-variant discovery, annotation, prediction, and genotyping of WGS and WES data"
