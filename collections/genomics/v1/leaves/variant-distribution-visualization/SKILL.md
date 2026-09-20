---
name: variant-distribution-visualization
description: Use when variant records (chromosome, reference position, gene assignment)
  from WGS/WES calling need to be rendered as a per-gene/per-chromosome Circos histogram
  for variant data interpretation.
when_to_use_negative:
- The analysis goal requires dynamic heat maps, gene pathways/networks, or cluster
  maps — the paper notes these advanced visualizations are not yet implemented in
  JWES.
- The user needs an interactive or clinician-friendly visualization interface — the
  current JWES version only offers a command-line interface.
- Input data is raw sequencing reads (FASTQ) rather than called/annotated variants
  — the data processing module (FastQC, Trimmomatic, BWA, HaplotypeCaller) must run
  first.
edam_operation: http://edamontology.org/operation_3208
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: Circos
  role: Renders the Circos graph image from the text-formatted input file showing
    distribution of variants across the genome
- name: JWES
  role: Pipeline whose data visualization module orchestrates query, gene aggregation,
    Ensembl coordinate lookup, Circos input generation, and plotting
  repo: github.com/drzeeshanahmed/JWES-DB
- name: JWES-DB
  role: VCF parser for JWES that stores variant records queried via SQL for visualization
  repo: github.com/drzeeshanahmed/JWES-DB
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/2211-5463.13261
    title: 'JWES: a new pipeline for whole genome/exome sequence data processing,
      management, and gene‐variant discovery, annotation, prediction, and genotyping'
  source_content_hash: sha256:9ddbc99628d5b77bb2d919de58cbb7f80d235efee652f3ee666e89c5446b3dae
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Circos
  - JWES
  - JWES-DB
  edam_operation: http://edamontology.org/operation_3208
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# variant-distribution-visualization

## Summary

A method for reconstructing the JWES data visualization module's Circos-plot generation, which presents chromosome-based variant distribution as a simplified Circos plot with two histograms: color-coded variants in protein-coding genes and black-coded variants in noncoding genes.

## When to use

Use when variant records (chromosome, reference position, gene assignment) from WGS/WES calling need to be rendered as a per-gene/per-chromosome Circos histogram for variant data interpretation.

## When NOT to use

- The analysis goal requires dynamic heat maps, gene pathways/networks, or cluster maps — the paper notes these advanced visualizations are not yet implemented in JWES.
- The user needs an interactive or clinician-friendly visualization interface — the current JWES version only offers a command-line interface.
- Input data is raw sequencing reads (FASTQ) rather than called/annotated variants — the data processing module (FastQC, Trimmomatic, BWA, HaplotypeCaller) must run first.

## Inputs

- Variant records from JWES-DB (chromosome, reference position, gene assignment) queried via SQL
- snpEff-annotated variant file (VCF) as an alternative direct input
- Ensembl API responses providing chromosome number and start/end positions for each gene and variant

## Outputs

- Text-formatted Circos input file (histogram/scatter/highlight track format) with per-gene variant counts and genomic coordinates
- Rendered Circos plot image showing chromosome-based variant distribution (two histograms: protein-coding vs noncoding)

## How to apply

Query variant records (chromosome, reference position, gene assignment) from the connected database via SQL, or load the snpEff-annotated variant file directly. Build a list of unique genes from the variants, count the total number of variants per gene, and use the Ensembl API to retrieve chromosome numbers and start/end positions of each gene and variant. Write the aggregated gene/variant counts and genomic coordinates into a text-formatted Circos input file (histogram/scatter/highlight track format) that can be passed directly to the Circos tool. Run Circos on the text input file to render the Circos graph image, divided into two histograms: a color-coded histogram for variants in protein-coding genes and a black histogram for variants in noncoding genes. Validation is qualitative: confirm the text input file exists, is parseable by Circos, and the rendered plot image displays the per-gene/per-chromosome variant distribution.

## Related tools

- **Circos** (Renders the Circos graph image from the text-formatted input file showing distribution of variants across the genome)
- **JWES** (Pipeline whose data visualization module orchestrates query, gene aggregation, Ensembl coordinate lookup, Circos input generation, and plotting) — github.com/drzeeshanahmed/JWES-DB
- **JWES-DB** (VCF parser for JWES that stores variant records queried via SQL for visualization) — github.com/drzeeshanahmed/JWES-DB

## Examples

```
circos -conf circos_variant_distribution.conf -outputdir ./plots
```

## Evaluation signals

- The generated text-formatted Circos input file exists and is parseable by Circos without errors.
- The rendered Circos plot image is produced and displays the per-gene/per-chromosome variant distribution.
- The plot shows the expected two-histogram layout: color-coded histogram for variants in protein-coding genes and black histogram for variants in noncoding genes.
- Per-gene variant counts in the Circos input file match the counts computed from the queried variant records or snpEff-annotated file.
- Gene/variant genomic coordinates retrieved from the Ensembl API (chromosome number, start/end positions) are consistent with the reference genome (hg38) used for alignment.

## Limitations

- The paper reports no numeric threshold for plot quality; validation is a qualitative acceptance check.
- The current version of JWES only offers a command-line interface; an interactive/visual interface is not yet implemented.
- Advanced visualizations such as dynamic heat maps, gene pathways and networks, and cluster maps are not yet implemented in the JWES visualization module.
- Gene/variant coordinate retrieval depends on the external Ensembl API, so availability and version of Ensembl annotations affect reproducibility.

## Evidence

- [results] JWES visualizes chromosome-based variant distribution as a simplified Circos plot divided into two histograms: a color-coded histogram for variants in protein-coding genes and a black histogram for variants in noncoding genes.: [evidence span withheld — anchor status: not_found]
- [abstract] the data visualization module supports variant data interpretation with Circos graphs: "the data visualization module supports variant data interpretation with Circos graphs"
- [discussion] The current version of the JWES offers command line but an easy-to-use interface. However, in future we are looking forward to implementing interactive interface: "The current version of the JWES offers command line but an easy"
- [discussion] we will be advancing data visualization module to produce dynamic heat maps, gene pathways and networks, and cluster maps for gene expression and variant analysis.: "advancing data visualization module to produce dynamic heat maps, gene pathways and networks, and cluster maps"
- [methods] The Burrows–Wheeler Aligner (BWA) tool is then used for mapping sequence data against the reference human genome (hg38): "mapping sequence data against the reference human genome (hg38)"
