---
name: coding-density-calculation-from-contigs
description: Use when comparing the predicted gene content and coding density of multiple
  metagenomic assemblies (e.g., MeGAMerge merged contigs vs individual Kmer assemblies)
  to quantify assembly quality gains.
when_to_use_negative:
- The goal is total assembly size or N50/N90 comparison, which the authors note are
  not very useful for comparing metagenome assembly performance
- Input contigs are already annotated gene coordinates rather than assembled contig
  FASTA, so gene prediction would be redundant
- Contigs smaller than 2 kb dominate the assembly and the analysis target includes
  poorly covered, error-prone fragments that the paper's protocol explicitly excludes
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3174
tools:
- name: Prodigal
  role: Gene prediction tool run in anonymous/metagenomic mode ('-p meta') with '-c'
    to count complete genes and compute coding density per assembly
- name: MeGAMerge
  role: Produces the merged contig set whose gene counts and coding density are compared
    against individual Kmer assemblies
  repo: https://github.com/LANL-Bioinformatics/MeGAMerge
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/srep06480
    title: Improved Assemblies Using a Source-Agnostic Pipeline for MetaGenomic Assembly
      by Merging (MeGAMerge) of Contigs
  source_content_hash: sha256:e85941bcc929209ab946ec925b1ddeca5ee6d71ef666e7e0d0ebca0ff5ccd831
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
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Coding density calculation from contigs

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Predicts complete genes in contig sets with Prodigal 2.6 in metagenomic mode and computes coding density (total gene length / total contig bases) to compare merged vs individual metagenomic assemblies.

## When to use

Use when comparing the predicted gene content and coding density of multiple metagenomic assemblies (e.g., MeGAMerge merged contigs vs individual Kmer assemblies) to quantify assembly quality gains.

## When NOT to use

- The goal is total assembly size or N50/N90 comparison, which the authors note are not very useful for comparing metagenome assembly performance
- Input contigs are already annotated gene coordinates rather than assembled contig FASTA, so gene prediction would be redundant
- Contigs smaller than 2 kb dominate the assembly and the analysis target includes poorly covered, error-prone fragments that the paper's protocol explicitly excludes

## Inputs

- Assembly FASTA files (Multi-FASTA contig sets) for each individual assembly and the MeGAMerge merged assembly
- Prodigal 2.6 gene-coordinate output files (e.g., GFF/GBK) per assembly
- Contig length metadata per assembly (total bases of >=2 kb contigs)

## Outputs

- gene_prediction_stats.csv with one row per assembly containing gene counts and coding density
- Per-assembly predicted gene counts and summed gene lengths

## How to apply

For each assembly FASTA, first remove contigs smaller than 2 kb so gene prediction ignores poorly covered, error-prone data. Run Prodigal 2.6 with the '-p meta' option (anonymous/metagenomic mode) and '-c' to restrict predictions to complete genes that do not run off contig ends. Parse the Prodigal gene-coordinate output to count predicted genes and sum gene lengths per assembly. Compute coding density as total predicted gene length divided by total bases of the >=2 kb contigs. Tabulate gene counts and coding density per assembly (individual assemblies plus the MeGAMerge assembly) in gene_prediction_stats.csv and confirm MeGAMerge shows a higher total predicted-gene count.

## Related tools

- **Prodigal** (Gene prediction tool run in anonymous/metagenomic mode ('-p meta') with '-c' to count complete genes and compute coding density per assembly)
- **MeGAMerge** (Produces the merged contig set whose gene counts and coding density are compared against individual Kmer assemblies) — https://github.com/LANL-Bioinformatics/MeGAMerge

## Examples

```
prodigal -p meta -c -i MeGAMerge_filtered_2kb.fasta -f gff -o genes.gff
```

## Evaluation signals

- gene_prediction_stats.csv contains the expected fields and exactly one row per assembly (row_count_equals, field_present checks)
- All coding density values fall in the plausible range 0.85-0.95 (value_in_range check)
- The MeGAMerge assembly shows a higher total predicted-gene count than every individual Kmer assembly of the same data
- Coding density is ~90%, within the expected range for microbial genomes
- Contigs smaller than 2 kb are confirmed absent from every FASTA submitted to Prodigal

## Limitations

- Only complete genes are counted because '-c' restricts predictions to genes that do not run off contig ends; partial genes at contig edges are excluded
- Removing contigs <2 kb before gene prediction reduces total assembly size and affects downstream size-related metrics (N50/N90)
- MeGAMerge's merging process can introduce new chimeras even though it reduces total input chimeras

## Evidence
