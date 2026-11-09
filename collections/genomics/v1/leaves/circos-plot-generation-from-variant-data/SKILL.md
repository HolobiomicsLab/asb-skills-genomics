---
name: circos-plot-generation-from-variant-data
description: Use when you have snpEff-annotated variant calls (VCF-derived records
  with chromosome, reference position, and gene assignment) and need a genome-wide
  per-gene/per-chromosome variant distribution plot for interpretation.
when_to_use_negative:
- You need dynamic heat maps, gene pathways/networks, or cluster maps for gene expression
  and variant analysis — JWES's discussion notes these advanced visualizations are
  not yet implemented
- Input variants lack gene annotations (e.g., unannotated VCFs), since the method
  aggregates counts per gene and requires gene-to-chromosome/coordinate mapping via
  Ensembl
edam_operation: http://edamontology.org/operation_3208
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Circos
  role: Renders the Circos graph image from the text-formatted input file showing
    the distribution of variants across the genome
- name: JWES
  role: Pipeline whose data visualization module queries variant records, aggregates
    per-gene counts, and produces the Circos input file
  repo: github.com/drzeeshanahmed/JWES-DB
- name: snpEff
  role: Provides gene assignment for variants in the annotated variant file that is
    loaded directly as input
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/2211-5463.13261
    title: 'JWES: a new pipeline for whole genome/exome sequence data processing,
      management, and gene‐variant discovery, annotation, prediction, and genotyping'
  source_content_hash: sha256:6089d08198900d5ee74f7ddb41775417c28d0b5f397f644c7872c0f3394f464d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Circos
  - JWES
  - snpEff
  edam_operation: http://edamontology.org/operation_3208
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# circos-plot-generation-from-variant-data

## Summary

Generates Circos graphs from variant records to visualize chromosome-based variant distribution, as implemented in the JWES data visualization module. Variants are aggregated per gene, genomic coordinates are fetched from the Ensembl API, and a text-formatted Circos input file is rendered into a two-histogram Circos plot.

## When to use

Use when you have snpEff-annotated variant calls (VCF-derived records with chromosome, reference position, and gene assignment) and need a genome-wide per-gene/per-chromosome variant distribution plot for interpretation.

## When NOT to use

- You need dynamic heat maps, gene pathways/networks, or cluster maps for gene expression and variant analysis — JWES's discussion notes these advanced visualizations are not yet implemented
- Input variants lack gene annotations (e.g., unannotated VCFs), since the method aggregates counts per gene and requires gene-to-chromosome/coordinate mapping via Ensembl

## Inputs

- Variant records with chromosome, reference position, and gene assignment (from JWES-DB SQL database or snpEff-annotated variant file)

## Outputs

- Text-formatted Circos input file (histogram/scatter/highlight track format)
- Rendered Circos plot image showing per-gene/per-chromosome variant distribution, with color-coded histogram for protein-coding gene variants and black histogram for noncoding gene variants

## How to apply

Query variant records (chromosome, reference position, gene assignment) from the connected database via SQL, or load the snpEff-annotated variant file directly. Build a list of unique genes from the variants, count the total number of variants per gene, and use the Ensembl API to retrieve the chromosome number and start/end positions of each gene and variant. Write the aggregated gene/variant counts and genomic coordinates into a text-formatted Circos input file (histogram/scatter/highlight track format) that can be passed directly to the Circos tool. Run Circos on the text input file to render the Circos graph image. JWES renders the plot as a simplified Circos layout with two histograms: a color-coded histogram for variants in protein-coding genes and a black histogram for variants in noncoding genes.

## Related tools

- **Circos** (Renders the Circos graph image from the text-formatted input file showing the distribution of variants across the genome)
- **JWES** (Pipeline whose data visualization module queries variant records, aggregates per-gene counts, and produces the Circos input file) — github.com/drzeeshanahmed/JWES-DB
- **snpEff** (Provides gene assignment for variants in the annotated variant file that is loaded directly as input)

## Examples

```
circos -conf circos.conf -outputdir ./plots  # run Circos on the text-formatted histogram/scatter/highlight input file produced by the JWES visualization module
```

## Evaluation signals

- The generated text-formatted Circos input file exists and is parseable by Circos
- The rendered Circos plot image is produced, displaying the per-gene/per-chromosome variant distribution
- The plot shows the expected two-histogram layout: color-coded histogram for variants in protein-coding genes and black histogram for variants in noncoding genes
- Each unique gene in the input variants has a retrieved chromosome number and start/end position from the Ensembl API
- Qualitative acceptance check only — the paper reports no numeric threshold for plot quality

## Limitations

- Current JWES version offers only a command-line interface; an interactive interface is not yet implemented
- Advanced visualizations such as dynamic heat maps, gene pathways and networks, and cluster maps are not yet implemented in the visualization module
- No numeric threshold for plot quality is reported; validation is a qualitative acceptance check

## Evidence

- [abstract] the data visualization module supports variant data interpretation with Circos graphs: "the data visualization module supports variant data interpretation with Circos graphs"
- [results] JWES visualizes chromosome-based variant distribution as a simplified Circos plot divided into two histograms: [evidence span withheld — anchor status: not_found]
- [methods] Build a list of unique genes from the variants and use the Ensembl API for coordinates: [evidence span withheld — anchor status: not_found]
- [readme] README repo identity: "# JWES-DB
VCF Parser for JWES"
