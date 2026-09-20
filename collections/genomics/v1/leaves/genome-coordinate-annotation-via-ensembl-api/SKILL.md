---
name: genome-coordinate-annotation-via-ensembl-api
description: Use when variant records carry gene assignments but lack genomic coordinates,
  and you must annotate each gene/variant with chromosome, start, and end positions
  to build Circos histogram/scatter/highlight inputs.
when_to_use_negative:
- Variant records already carry complete chromosome, start, and end coordinates for
  all genes, so no external coordinate lookup is needed
- The goal is plotting rather than coordinate annotation — the downstream Circos rendering
  step is a separate skill
- The organism/reference is not covered by Ensembl gene annotations (JWES uses the
  human reference genome hg38)
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: Ensembl API
  role: Retrieves the chromosome number and start/end positions of each gene and variant
    identified from the variant records
- name: JWES
  role: Hosts the visualization module that queries variant records, aggregates per-gene
    counts, and writes the Circos input file
  repo: https://github.com/drzeeshanahmed/JWES-DB
- name: Circos
  role: Consumes the generated text input file to render the chromosome-based Circos
    graph of variant distribution
- name: snpEff
  role: Provides the annotated variant file (with gene assignments) that can be loaded
    directly as an alternative to SQL database queries
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/2211-5463.13261
    title: 'JWES: a new pipeline for whole genome/exome sequence data processing,
      management, and gene‐variant discovery, annotation, prediction, and genotyping'
  source_content_hash: sha256:d3472a744c9600f278db377abda36d56e21a2588e22392b9da3c5206b254085c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/drzeeshanahmed/JWES-DB
  tools:
  - Circos
  - Ensembl API
  - JWES
  - snpEff
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# Genome coordinate annotation via Ensembl API

## Summary

Retrieves chromosome numbers and start/end coordinates for genes and variants (from snpEff-annotated VCF records or SQL-queried variant records) using the Ensembl API, so per-gene variant counts and genomic positions can be aggregated into Circos-format track files for the JWES visualization module.

## When to use

Use when variant records carry gene assignments but lack genomic coordinates, and you must annotate each gene/variant with chromosome, start, and end positions to build Circos histogram/scatter/highlight inputs.

## When NOT to use

- Variant records already carry complete chromosome, start, and end coordinates for all genes, so no external coordinate lookup is needed
- The goal is plotting rather than coordinate annotation — the downstream Circos rendering step is a separate skill
- The organism/reference is not covered by Ensembl gene annotations (JWES uses the human reference genome hg38)

## Inputs

- Variant records (chromosome, reference position, gene assignment) from the JWES connected database via SQL
- snpEff-annotated variant file (VCF with gene assignments)

## Outputs

- List of unique genes with per-gene variant counts
- Chromosome number and start/end positions for each gene and variant (from Ensembl API)
- Text-formatted Circos input file (histogram/scatter/highlight track format)

## How to apply

Start from variant records (chromosome, reference position, gene assignment) queried via SQL from the connected database, or load the snpEff-annotated variant file directly. Build a list of unique genes from the variants and count the total number of variants per gene. For each unique gene and each variant, call the Ensembl API to retrieve the chromosome number and start/end positions. Write the aggregated gene/variant counts together with the returned genomic coordinates into a text-formatted Circos input file (histogram/scatter/highlight track format), separating protein-coding variants (color-coded histogram track) from noncoding variants (black histogram track), which can be passed directly to Circos to render the plot.

## Related tools

- **Ensembl API** (Retrieves the chromosome number and start/end positions of each gene and variant identified from the variant records)
- **JWES** (Hosts the visualization module that queries variant records, aggregates per-gene counts, and writes the Circos input file) — https://github.com/drzeeshanahmed/JWES-DB
- **Circos** (Consumes the generated text input file to render the chromosome-based Circos graph of variant distribution)
- **snpEff** (Provides the annotated variant file (with gene assignments) that can be loaded directly as an alternative to SQL database queries)

## Examples

```
circos -conf jwes_circos.conf -outputdir ./circos_out  # after writing histogram track file from Ensembl-annotated genes/variants
```

## Evaluation signals

- The generated text-formatted Circos input file exists and is parseable by Circos
- The rendered Circos plot image is produced showing the per-gene/per-chromosome variant distribution
- The Circos plot divides into two histograms: a color-coded histogram for variants in protein-coding genes and a black histogram for variants in noncoding genes
- Every unique gene in the input variant set has an Ensembl-retrieved chromosome number and start/end coordinate
- Qualitative acceptance check only — the paper reports no numeric threshold for plot quality

## Limitations

- The paper reports no numeric threshold for Circos plot quality, so validation is a qualitative acceptance check
- The current JWES version offers only a command-line interface; an easy-to-use interactive interface is not yet implemented
- Advanced visualizations such as dynamic heat maps, gene pathways/networks, and cluster maps are not yet implemented in the JWES visualization module
- Coordinate annotation depends on Ensembl API coverage of the reference (JWES maps against hg38), and retrieval requires network access to Ensembl

## Evidence

- [other] Build a list of unique genes from the variants, count the total number of variants per gene, and use the Ensembl API to retrieve the chromosome number and start/end positions of each gene and variant.: [evidence span withheld — anchor status: not_found]
- [other] Write the aggregated gene/variant counts and genomic coordinates into a text-formatted Circos input file (e.g., histogram/scatter/highlight track format) that can be passed directly to the Circos tool.: [evidence span withheld — anchor status: not_found]
- [other] JWES visualizes chromosome-based variant distribution as a simplified Circos plot divided into two histograms: a color-coded histogram for variants in protein-coding genes and a black histogram for variants in noncoding genes.: [evidence span withheld — anchor status: not_found]
- [abstract] the data visualization module supports variant data interpretation with Circos graphs: "the data visualization module supports variant data interpretation with Circos graphs"
