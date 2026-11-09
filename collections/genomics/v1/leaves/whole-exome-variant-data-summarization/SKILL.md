---
name: whole-exome-variant-data-summarization
description: Use when you have a snpEff-annotated VCF/CSV variant set from a JWES-processed
  sample and need coding/noncoding and per-chromosome distribution counts with percentages.
when_to_use_negative:
- Input is a raw (unannotated) VCF lacking snpEff gene-region annotations — coding/noncoding
  classification cannot be made
- The goal is de novo variant calling or QC/trimming of FASTQ reads rather than summarizing
  already-called variants
- Sample size or variant count is so small that percentage shares per chromosome are
  not meaningful
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: JWES
  role: Pipeline that produced the annotated variant calls being summarized
  repo: https://github.com/drzeeshanahmed/JWES-DB
- name: snpEff
  role: Annotates variants with gene-region (interval-forest) classifications used
    for the noncoding/protein-coding split
- name: VariantsToTable
  role: Exports variant records from the VCF into a tabular form for aggregation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/2211-5463.13261
    title: 'JWES: a new pipeline for whole genome/exome sequence data processing,
      management, and gene‐variant discovery, annotation, prediction, and genotyping'
  source_content_hash: sha256:185f7bb680e17d22874562fd8e7ec419e9e63f3e1d25269b4a3b85eb53a11c3d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/drzeeshanahmed/JWES-DB
  tools:
  - JWES
  - VariantsToTable
  - snpEff
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# whole-exome-variant-data-summarization

## Summary

Summarize snpEff-annotated variant calls from a JWES-processed WGS/WES sample by gene region (noncoding vs protein-coding) and by chromosome, producing a counts-and-percentages distribution table. This reproduces the SRR891275 analysis where 20,749 variants were split into 12,979 noncoding (~63%) and 7,770 protein-coding (~37%).

## When to use

Use when you have a snpEff-annotated VCF/CSV variant set from a JWES-processed sample and need coding/noncoding and per-chromosome distribution counts with percentages.

## When NOT to use

- Input is a raw (unannotated) VCF lacking snpEff gene-region annotations — coding/noncoding classification cannot be made
- The goal is de novo variant calling or QC/trimming of FASTQ reads rather than summarizing already-called variants
- Sample size or variant count is so small that percentage shares per chromosome are not meaningful

## Inputs

- snpEff-annotated variant set (VCF or CSV) from a JWES-processed WGS/WES sample
- snpEff interval-forest gene-region annotations (noncoding vs protein-coding classification per variant)

## Outputs

- Combined chromosome-level and gene-region-level distribution table with counts and percentages (CSV, e.g. variant_distribution_SRR891275.csv)

## How to apply

Load the snpEff-annotated variant set (VCF or CSV exported via VariantsToTable) for the JWES-processed sample. Classify each variant as falling in a noncoding gene or a protein-coding gene using the snpEff interval-forest annotations attached to each record. Aggregate variant counts per gene region and per chromosome, computing each chromosome's share of all variants and of protein-coding variants. Write the combined chromosome-level and gene-region-level table with counts and percentages (e.g. variant_distribution_SRR891275.csv). Validate by recomputing and comparing against reported values — for SRR891275: 12,979 (~63%) noncoding, 7,770 (~37%) protein-coding, with chromosome 2 the largest contributor (~11.8% of all variants, ~7.4% of protein-coding variants).

## Related tools

- **JWES** (Pipeline that produced the annotated variant calls being summarized) — https://github.com/drzeeshanahmed/JWES-DB
- **snpEff** (Annotates variants with gene-region (interval-forest) classifications used for the noncoding/protein-coding split)
- **VariantsToTable** (Exports variant records from the VCF into a tabular form for aggregation)

## Evaluation signals

- Recomputed gene-region counts match reported values: 12,979 (~63%) noncoding and 7,770 (~37%) protein-coding of 20,749 total variants for SRR891275
- Gene-region counts sum to the total variant count for the sample (12,979 + 7,770 = 20,749)
- Chromosome 2 is the largest contributor, holding ~11.8% of all variants and ~7.4% of protein-coding variants
- All per-chromosome percentages sum to approximately 100% of total variants
- Output CSV contains both chromosome-level and gene-region-level rows with counts and percentages

## Limitations

- Validation depends on reproducing the exact reported values from the specific sample (SRR891275); other samples will have different distributions
- Classification accuracy depends on the snpEff annotation quality and the reference genome build (hg38 in JWES)
- The current JWES version only offers a command-line interface, so summarization steps must be scripted manually

## Evidence

- [results] Of the 20,749 variants identified in SRR891275, 12,979 (~63%) affected noncoding genes and 7,770 (~37%) affected protein-coding genes: [evidence span withheld — anchor status: not_found]
- [results] chromosome 2 carried the largest share (~11.8% of noncoding-region variants and ~7.4% of protein-coding-region variants): [evidence span withheld — anchor status: not_found]
- [results] we downloaded SRR891275 from NCBI Sequence Read Architecture (SRA) database: "we downloaded SRR891275 from NCBI Sequence Read Architecture (SRA) database"
