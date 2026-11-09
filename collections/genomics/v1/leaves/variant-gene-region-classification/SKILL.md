---
name: variant-gene-region-classification
description: Use when a snpEff-annotated VCF/CSV from the JWES pipeline needs to be
  summarized by coding vs noncoding gene regions and by chromosome, with counts and
  percentage shares per category.
when_to_use_negative:
- The VCF has not yet been functionally annotated with snpEff, so gene-region (noncoding
  vs coding) labels are unavailable
- The analysis goal is variant calling or QC rather than post-annotation summarization
  of an already-called variant set
edam_operation: http://edamontology.org/operation_3225
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3517
tools:
- name: snpEff
  role: Annotates variants with interval-forest gene-region information used to classify
    each variant as noncoding or protein-coding
- name: VariantsToTable
  role: Extracts the annotated variant records from the VCF into a tabular CSV for
    per-region and per-chromosome aggregation
- name: JWES
  role: Upstream pipeline that produced the annotated variant set (variant discovery,
    annotation, prediction, and genotyping of WGS/WES data)
  repo: https://github.com/drzeeshanahmed/JWES-DB
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/2211-5463.13261
    title: 'JWES: a new pipeline for whole genome/exome sequence data processing,
      management, and gene‐variant discovery, annotation, prediction, and genotyping'
  source_content_hash: sha256:762fc9f633743f6902038503f802bfc6841d34375cd0ce812fad6f54a5a6d987
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/drzeeshanahmed/JWES-DB
  tools:
  - JWES
  - VariantsToTable
  - snpEff
  edam_operation: http://edamontology.org/operation_3225
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# variant-gene-region-classification

## Summary

Classify snpEff-annotated variants from a JWES-processed WGS/WES sample into noncoding versus protein-coding gene regions and aggregate their per-chromosome distribution, enabling variant interpretation and reporting (e.g., 12,979 noncoding vs 7,770 protein-coding variants for SRR891275).

## When to use

Use when a snpEff-annotated VCF/CSV from the JWES pipeline needs to be summarized by coding vs noncoding gene regions and by chromosome, with counts and percentage shares per category.

## When NOT to use

- The VCF has not yet been functionally annotated with snpEff, so gene-region (noncoding vs coding) labels are unavailable
- The analysis goal is variant calling or QC rather than post-annotation summarization of an already-called variant set

## Inputs

- snpEff-annotated variant set (VCF) for a JWES-processed WGS/WES sample
- CSV variant table extracted with VariantsToTable from the annotated VCF
- snpEff interval-forest gene-region annotations (noncoding vs protein-coding)

## Outputs

- Combined chromosome-level and gene-region-level distribution table with variant counts and percentages (e.g., variant_distribution_SRR891275.csv)

## How to apply

Load the snpEff-annotated variant set (VCF or CSV extracted via VariantsToTable) produced by the JWES pipeline for the sample of interest. Classify each variant as falling in a noncoding gene or a protein-coding gene based on the snpEff interval-forest annotations. Aggregate variant counts per gene region (for SRR891275 this yields 12,979 noncoding, ~63%, and 7,770 protein-coding, ~37%, of 20,749 total variants). Then aggregate counts per chromosome and compute each chromosome's share of all variants and of protein-coding variants — chromosome 2 is the largest contributor (~11.8% of all variants, ~7.4% of protein-coding variants). Write the combined chromosome-level and gene-region-level table with counts and percentages to variant_distribution_SRR891275.csv (or the equivalent per-sample CSV).

## Related tools

- **snpEff** (Annotates variants with interval-forest gene-region information used to classify each variant as noncoding or protein-coding)
- **VariantsToTable** (Extracts the annotated variant records from the VCF into a tabular CSV for per-region and per-chromosome aggregation)
- **JWES** (Upstream pipeline that produced the annotated variant set (variant discovery, annotation, prediction, and genotyping of WGS/WES data)) — https://github.com/drzeeshanahmed/JWES-DB

## Examples

```
gatk VariantsToTable -V SRR891275_snpEff.vcf -F CHROM -F GENE -F BIOTYPE -O variants_table.csv
```

## Evaluation signals

- Recomputed gene-region counts reproduce the reported values: 12,979 (~63%) noncoding and 7,770 (~37%) protein-coding variants for SRR891275 (total 20,749)
- Chromosome 2 is the largest contributor, holding ~11.8% of all variants and ~7.4% of protein-coding-region variants
- The sum of noncoding and protein-coding variant counts equals the total variant count for the sample
- Output CSV contains both chromosome-level and gene-region-level rows with counts and percentages
- Every variant is assigned to exactly one gene-region category (no unclassified variants remain)

## Limitations

- Classification depends on snpEff's interval-forest annotations; variants outside annotated gene regions or with ambiguous annotations may not fall cleanly into the two categories
- Distribution percentages are sample-specific; values such as 12,979/7,770 and chromosome 2's ~11.8% share apply to SRR891275 and will differ for other samples

## Evidence

- [methods] noncoding vs protein-coding classification from snpEff annotations: [evidence span withheld — anchor status: not_found]
- [methods] snpEff-annotated variant set input: [evidence span withheld — anchor status: not_found]
- [results] reported distribution results: [evidence span withheld — anchor status: not_found]
- [results] chromosome-level aggregation: [evidence span withheld — anchor status: not_found]
- [abstract] JWES scope of application: "We have successfully applied JWES for processing, management, and gene-variant discovery, annotation, prediction, and genotyping of WGS and WES data"
