---
name: gene-variant-count-aggregation
description: Use when variant calls annotated with gene assignments (e.g., snpEff-annotated
  VCF or variant records in JWES-DB) must be summarized per gene and rendered as a
  Circos plot of chromosome-based variant distribution.
when_to_use_negative:
- Variant data are not yet annotated with gene assignments (e.g., raw VCF straight
  from HaplotypeCaller before snpEff annotation) — aggregation per gene cannot be
  performed
- The goal is dynamic heat maps, gene pathways/networks, or cluster maps — the article
  states these advanced visualizations are not yet implemented in JWES
- A per-sample quality-metric plot is needed (e.g., AnalyzeCovariates lines/bar plots)
  rather than a genomic-coordinate variant distribution
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0091
tools:
- name: JWES
  role: Pipeline whose visualization module performs the variant-to-Circos aggregation
    and plotting; JWES-DB provides the VCF parser that reads variant records
  repo: github.com/drzeeshanahmed/JWES-DB
- name: Circos
  role: Renders the aggregated per-gene/per-chromosome variant counts (supplied as
    text histogram/scatter/highlight input) into the Circos graph image
- name: Ensembl API
  role: Source of chromosome number and start/end positions for each gene and variant
    used to build the Circos coordinate tracks
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/2211-5463.13261
    title: 'JWES: a new pipeline for whole genome/exome sequence data processing,
      management, and gene‐variant discovery, annotation, prediction, and genotyping'
  source_content_hash: sha256:a2648dc6e2c4c197f3f332681c4427ccadd4436d4bf62328f8304a4e986adf84
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Circos
  - Ensembl API
  - JWES
  edam_operation: http://edamontology.org/operation_3204
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# gene-variant-count-aggregation

## Summary

Aggregates per-gene variant counts and genomic coordinates from snpEff-annotated variant records (VCF) into text-formatted Circos histogram/scatter/highlight input files, enabling JWES's Circos-based visualization of chromosome-level variant distribution split into protein-coding and noncoding gene tracks.

## When to use

Use when variant calls annotated with gene assignments (e.g., snpEff-annotated VCF or variant records in JWES-DB) must be summarized per gene and rendered as a Circos plot of chromosome-based variant distribution.

## When NOT to use

- Variant data are not yet annotated with gene assignments (e.g., raw VCF straight from HaplotypeCaller before snpEff annotation) — aggregation per gene cannot be performed
- The goal is dynamic heat maps, gene pathways/networks, or cluster maps — the article states these advanced visualizations are not yet implemented in JWES
- A per-sample quality-metric plot is needed (e.g., AnalyzeCovariates lines/bar plots) rather than a genomic-coordinate variant distribution

## Inputs

- snpEff-annotated variant VCF file
- Variant records from JWES-DB (chromosome, reference position, gene assignment) accessible via SQL
- Gene and variant genomic coordinates retrieved from the Ensembl API

## Outputs

- Text-formatted Circos input file (histogram/scatter/highlight track format) with per-gene variant counts and genomic coordinates
- Circos plot image showing chromosome-based variant distribution, with color-coded (protein-coding) and black (noncoding) histogram tracks

## How to apply

Query the variant records (chromosome, reference position, gene assignment) from the JWES-DB database via SQL, or load the snpEff-annotated variant file (VCF) directly with the JWES VCF parser. Build a list of unique genes from the variants and count the total number of variants per gene. Use the Ensembl API to retrieve the chromosome number and start/end positions of each gene and variant, since Circos requires absolute genomic coordinates. Write the aggregated gene/variant counts and genomic coordinates into a text-formatted Circos input file (histogram, scatter, or highlight track format): a color-coded histogram track for variants in protein-coding genes and a black histogram track for variants in noncoding genes. Pass the file directly to Circos to render the graph; JWES's visualization module exists specifically to support variant data interpretation with Circos graphs.

## Related tools

- **JWES** (Pipeline whose visualization module performs the variant-to-Circos aggregation and plotting; JWES-DB provides the VCF parser that reads variant records) — github.com/drzeeshanahmed/JWES-DB
- **Circos** (Renders the aggregated per-gene/per-chromosome variant counts (supplied as text histogram/scatter/highlight input) into the Circos graph image)
- **Ensembl API** (Source of chromosome number and start/end positions for each gene and variant used to build the Circos coordinate tracks)

## Examples

```
curl -s "https://rest.ensembl.org/lookup/gene/BRCA1?content-type=application/json;expand=1"  # retrieve chr/start/end for each unique gene before writing the Circos histogram track
```

## Evaluation signals

- The generated text-formatted Circos input file exists and is parseable by Circos (no errors on Circos run)
- The rendered Circos plot image is produced and displays the per-gene/per-chromosome variant distribution (qualitative acceptance check; the paper reports no numeric threshold for plot quality)
- Per-gene variant counts in the Circos input sum to the number of annotated variant records from the VCF/database query
- Every gene track carries a chromosome number and start/end coordinate pair from the Ensembl API (no missing/unmapped coordinates)
- Protein-coding and noncoding variants are separated into the color-coded and black histogram tracks respectively

## Limitations

- The paper reports no numeric threshold for Circos plot quality, so acceptance is a qualitative visual check
- The current JWES version only offers a command-line interface; an interactive interface is not yet implemented
- Advanced visualizations such as dynamic heat maps, gene pathways/networks, and cluster maps are not yet implemented in the JWES visualization module
- The JWES-DB repository README is minimal ('VCF Parser for JWES'), offering little documentation beyond the parser's purpose

## Evidence

- [abstract] The visualization module supports variant data interpretation with Circos graphs: "the data visualization module supports variant data interpretation with Circos graphs"
- [abstract] JWES comprises three modules including visualization: "JWES is a cross-platform, user-friendly, product line application, that entails three modules: (a) data processing, (b) storage, and (c) visualization"
- [readme] JWES-DB README identifies the repo as the JWES VCF parser: "JWES-DB
VCF Parser for JWES"
