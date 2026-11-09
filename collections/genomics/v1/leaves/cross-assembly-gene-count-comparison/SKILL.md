---
name: cross-assembly-gene-count-comparison
description: Use when you have a merged (e.g. MeGAMerge) assembly and one or more
  individual assembler contig FASTAs for the same metagenomic data and need to quantify
  gene-count and coding-density improvements.
when_to_use_negative:
- When assemblies have not been filtered to a common contig-size cutoff — comparing
  gene counts over unfiltered contig sets (<2 kb included) conflates merging gains
  with retained error-prone small contigs
- When the goal is a general assembly-quality metric comparison (N50, total assembly
  size), which the paper describes as 'not very useful for comparing metagenome assembly
  performance' — this skill targets gene prediction specifically
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: Prodigal
  role: Gene predictor run in anonymous/metagenomic mode (-p meta) with -c to restrict
    to complete genes on filtered >=2 kb contigs
- name: MeGAMerge
  role: Produces the merged contig set whose gene counts and coding density are compared
    against individual assemblies
  repo: https://github.com/LANL-Bioinformatics/MeGAMerge
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/srep06480
    title: Improved Assemblies Using a Source-Agnostic Pipeline for MetaGenomic Assembly
      by Merging (MeGAMerge) of Contigs
  source_content_hash: sha256:6ad8c3b1ce06ded0f51f4eb04bf54b80345d09ee086dd5d8645262d2d3c55b02
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: cc-by-nc-sa
    url: https://github.com/LANL-Bioinformatics/MeGAMerge
  repo_url: https://github.com/LANL-Bioinformatics/MeGAMerge
  tools:
  - MeGAMerge
  - Prodigal
  edam_operation: http://edamontology.org/operation_2454
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# cross-assembly-gene-count-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compares predicted gene counts and coding density across individual metagenomic assemblies versus their MeGAMerge merged assembly using Prodigal 2.6 in metagenomic mode, to quantify whether contig merging yields more predicted genes at microbial-plausible coding density.

## When to use

Use when you have a merged (e.g. MeGAMerge) assembly and one or more individual assembler contig FASTAs for the same metagenomic data and need to quantify gene-count and coding-density improvements.

## When NOT to use

- When assemblies have not been filtered to a common contig-size cutoff — comparing gene counts over unfiltered contig sets (<2 kb included) conflates merging gains with retained error-prone small contigs
- When the goal is a general assembly-quality metric comparison (N50, total assembly size), which the paper describes as 'not very useful for comparing metagenome assembly performance' — this skill targets gene prediction specifically

## Inputs

- Individual assembler contig FASTA files (e.g. SOAPdenovo, CLC Bio, Velvet, IDBA, Ray, Newbler outputs) for the same metagenomic sample
- MeGAMerge merged contig FASTA (e.g. MergedContigs.fasta output directory)
- Prodigal 2.6 gene-coordinate output files per assembly

## Outputs

- gene_prediction_stats.csv with one row per assembly: predicted gene count, total gene length, total >=2 kb contig bases, and coding density
- Per-assembly Prodigal gene prediction files (gene coordinates and protein translations)

## How to apply

For each assembly being compared, remove contigs smaller than 2 kb before gene prediction, mirroring the paper's cutoff that removes poorly covered or error-prone contigs. Run Prodigal 2.6 with '-p meta' (anonymous/metagenomic mode) and '-c' so only complete genes that do not run off contig ends are predicted. Parse the Prodigal gene-coordinate output to count predicted genes and sum gene lengths per assembly, then compute coding density as total predicted gene length divided by total bases of the >=2 kb contigs. Tabulate gene counts and coding density per assembly (individual and merged) in gene_prediction_stats.csv; the paper's expectation is consistently higher gene counts for merged contigs with coding density around ~90%.

## Related tools

- **Prodigal** (Gene predictor run in anonymous/metagenomic mode (-p meta) with -c to restrict to complete genes on filtered >=2 kb contigs)
- **MeGAMerge** (Produces the merged contig set whose gene counts and coding density are compared against individual assemblies) — https://github.com/LANL-Bioinformatics/MeGAMerge

## Examples

```
prodigal -i MergedContigs.fasta -p meta -c -f gff -o merged_genes.gff -a merged_proteins.faa
```

## Evaluation signals

- gene_prediction_stats.csv contains the expected fields and exactly one row per assembly (row_count_equals, field_present checks)
- Coding density values for all assemblies fall in the plausible microbial range 0.85–0.95 (value_in_range check)
- The MeGAMerge assembly shows a higher total predicted-gene count than each individual Kmer-based assembly of the same data
- Per-assembly gene counts are computed on contigs >=2 kb only, matching the paper's 'contigs smaller than 2 kb were removed from the assembly' filter
- Prodigal runs used both '-p meta' and '-c', so predicted genes are complete and do not run off contig ends

## Limitations

- MeGAMerge merging can introduce new chimeras even though it reduces the total number of chimeras input, so elevated gene counts may include chimeric sequence
- Very small contigs (<2 kb) are excluded, so the comparison ignores poorly covered, difficult-to-assemble, or error-prone contigs by design
- Overlap-based consensus tools' run-times increase exponentially with increasing input dataset size, limiting scalability of the upstream merge step for very large contig pools

## Evidence

- [methods] For each assembly analyzed, remove contigs smaller than 2 kb from the assembly FASTA before gene prediction.: [evidence span withheld — anchor status: not_found]
- [methods] Run Prodigal 2.6 in anonymous/metagenomic mode: [evidence span withheld — anchor status: not_found]
- [results] coding density (~90%) within the expected range for microbial genomes: [evidence span withheld — anchor status: not_found]
