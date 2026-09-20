---
name: sql-variant-data-extraction
description: Use when you have a variant database or annotated VCF (e.g. snpEff output
  from JWES) and need per-gene/per-chromosome variant counts and genomic coordinates
  to build Circos histogram inputs.
when_to_use_negative:
- Input is raw FASTQ/FASTA or unaligned reads — the JWES data processing module (FastQC,
  Trimmomatic, BWA, HaplotypeCaller) must run first to produce annotated variants.
- A dynamic/interactive visualization is required — the paper notes the current JWES
  visualization is a simplified Circos plot; dynamic heat maps, gene pathways and
  networks, and cluster maps are not yet implemented.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_0625
tools:
- name: JWES-DB
  role: VCF parser that populates the variant database queried via SQL for chromosome/position/gene
    records
  repo: https://github.com/drzeeshanahmed/JWES-DB
- name: Circos
  role: Consumes the text-formatted Circos input file produced from the SQL-extracted
    variant data to render the Circos graph image
- name: JWES
  role: Pipeline whose storage module holds the variant records and whose visualization
    module consumes the Circos inputs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/2211-5463.13261
    title: 'JWES: a new pipeline for whole genome/exome sequence data processing,
      management, and gene‐variant discovery, annotation, prediction, and genotyping'
  source_content_hash: sha256:da2994850419d25829dd6aa80a944a3333fd4a8b39665b045f8edecc11016a47
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/drzeeshanahmed/JWES-DB
  tools:
  - Circos
  - JWES
  - JWES-DB
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# sql-variant-data-extraction

## Summary

Extract per-gene/per-chromosome variant records (chromosome, reference position, gene assignment) from a connected variant database via SQL — or load snpEff-annotated variant files directly — to feed the JWES Circos visualization module. JWES-DB acts as the VCF parser that stores the variant records queried here.

## When to use

Use when you have a variant database or annotated VCF (e.g. snpEff output from JWES) and need per-gene/per-chromosome variant counts and genomic coordinates to build Circos histogram inputs.

## When NOT to use

- Input is raw FASTQ/FASTA or unaligned reads — the JWES data processing module (FastQC, Trimmomatic, BWA, HaplotypeCaller) must run first to produce annotated variants.
- A dynamic/interactive visualization is required — the paper notes the current JWES visualization is a simplified Circos plot; dynamic heat maps, gene pathways and networks, and cluster maps are not yet implemented.

## Inputs

- Variant records in a connected database (chromosome, reference position, gene assignment columns)
- snpEff-annotated variant file (VCF) as a direct-load alternative
- Ensembl API responses returning chromosome number and start/end positions per gene/variant

## Outputs

- Aggregated per-gene variant counts with chromosome and start/end coordinates
- Text-formatted Circos input file (histogram/scatter/highlight track format)
- Rendered Circos plot image of chromosome-based variant distribution (downstream of the Circos tool)

## How to apply

Query the variant records (chromosome, reference position, gene assignment) from the connected database via SQL, or load the snpEff-annotated variant file directly if no database is available. Build a list of unique genes from the variants and count the total number of variants per gene. For each gene and variant, use the Ensembl API to retrieve the chromosome number and start/end positions so that counts can be anchored to genomic coordinates. Write the aggregated gene/variant counts and coordinates into a text-formatted Circos input file (histogram/scatter/highlight track format) that can be passed directly to the Circos tool. JWES-DB (github.com/drzeeshanahmed/JWES-DB) is the VCF parser that populates the underlying variant store for JWES, so the SQL layer operates on parsed VCF-derived variant records.

## Related tools

- **JWES-DB** (VCF parser that populates the variant database queried via SQL for chromosome/position/gene records) — https://github.com/drzeeshanahmed/JWES-DB
- **Circos** (Consumes the text-formatted Circos input file produced from the SQL-extracted variant data to render the Circos graph image)
- **JWES** (Pipeline whose storage module holds the variant records and whose visualization module consumes the Circos inputs)

## Examples

```
SELECT chromosome, ref_position, gene, COUNT(*) AS variant_count FROM variants WHERE gene IS NOT NULL GROUP BY gene ORDER BY chromosome, ref_position;
```

## Evaluation signals

- The generated text-formatted Circos input file exists and is parseable by the Circos tool.
- The rendered Circos plot image is produced and displays the per-gene/per-chromosome variant distribution (qualitative acceptance check; the paper reports no numeric threshold for plot quality).
- The aggregated output contains one unique gene entry per gene with a total variant count and valid Ensembl chromosome/start/end coordinates.
- SQL results are consistent with the source VCF/variant store (e.g. variant counts match those reported for the ingested dataset, such as 43,685 variants for SRR12474733).
- The plot distinguishes the two histogram tracks: color-coded histogram for variants in protein-coding genes and black histogram for variants in noncoding genes.

## Limitations

- The paper reports no numeric quality threshold for the rendered Circos plot — acceptance is a qualitative visual check only.
- The current JWES version only offers a command-line interface; an interactive interface is not yet implemented, and dynamic heat maps, gene pathways/networks, and cluster maps are not yet available in the visualization module.
- JWES-DB README provides minimal documentation ('VCF Parser for JWES'), so schema details of the variant database must be inferred from the parser/annotation output.

## Evidence

- [abstract] the data visualization module supports variant data interpretation with Circos graphs: "the data visualization module supports variant data interpretation with Circos graphs"
- [readme] VCF Parser for JWES: "# JWES-DB
VCF Parser for JWES"
- [discussion] dynamic visualizations not yet implemented: "we will be advancing data visualization module to produce dynamic heat maps, gene pathways and networks, and cluster maps for gene expression and variant analysis."
- [abstract] three modules of JWES: "JWES is a cross-platform, user-friendly, product line application, that entails three modules: (a) data processing, (b) storage, and (c) visualization"
