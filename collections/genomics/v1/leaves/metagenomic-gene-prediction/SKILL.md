---
name: metagenomic-gene-prediction
description: Use when comparing merged (MeGAMerge) vs individual metagenomic assemblies
  by predicted gene counts and coding density, or when annotating contigs >=2 kb without
  a reference genome.
when_to_use_negative:
- When the input is raw reads rather than assembled Multi-FASTA contigs — gene prediction
  here operates on assembled contigs only
- When the goal is assembly merging itself (use the MeGAMerge/Minimus2 pipeline, not
  gene prediction)
- When contigs <2 kb must be annotated, since the paper's protocol removes contigs
  smaller than 2 kb prior to Prodigal
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_0622
tools:
- name: Prodigal
  role: Predicts complete protein-coding genes on >=2 kb contigs in anonymous/metagenomic
    mode (-p meta -c) to produce gene counts and lengths for coding-density computation
- name: MeGAMerge
  role: Produces the merged contig set whose gene counts and coding density are compared
    against the individual Kmer assemblies
  repo: https://github.com/LANL-Bioinformatics/MeGAMerge
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/srep06480
    title: Improved Assemblies Using a Source-Agnostic Pipeline for MetaGenomic Assembly
      by Merging (MeGAMerge) of Contigs
  source_content_hash: sha256:ffca8959b6fe004783e8e9b334074fc98c096181f69d7372adbc12b2cecd3d94
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# metagenomic-gene-prediction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Predict protein-coding genes on metagenomic assemblies (e.g., MeGAMerge merged vs. individual Kmer assemblies) with Prodigal in anonymous/metagenomic mode, then quantify predicted gene counts and coding density to compare assembly quality.

## When to use

Use when comparing merged (MeGAMerge) vs individual metagenomic assemblies by predicted gene counts and coding density, or when annotating contigs >=2 kb without a reference genome.

## When NOT to use

- When the input is raw reads rather than assembled Multi-FASTA contigs — gene prediction here operates on assembled contigs only
- When the goal is assembly merging itself (use the MeGAMerge/Minimus2 pipeline, not gene prediction)
- When contigs <2 kb must be annotated, since the paper's protocol removes contigs smaller than 2 kb prior to Prodigal

## Inputs

- Assembly FASTA files (Multi-FASTA contigs) from individual assemblers (SOAPdenovo, CLC Bio, Velvet, IDBA, Ray, Newbler) and the MeGAMerge merged assembly
- Contig size filter threshold (2 kb) applied before prediction

## Outputs

- Prodigal gene-coordinate files (predicted CDS coordinates and lengths) per assembly
- gene_prediction_stats.csv with per-assembly total predicted gene counts, summed gene lengths, and coding density

## How to apply

First filter each assembly FASTA, keeping only contigs >=2 kb, since the paper removed contigs smaller than 2 kb before gene prediction to avoid poorly covered, error-prone sequence. Run Prodigal 2.6 in anonymous/metagenomic mode with the -p meta option, adding -c so only complete genes that do not run off contig ends are predicted. Parse the gene-coordinate output to count predicted genes and sum gene lengths per assembly. Compute coding density as total predicted gene length divided by total contig bases of the >=2 kb contigs. Tabulate gene counts and coding density per individual assembly and the MeGAMerge assembly (e.g., gene_prediction_stats.csv); rationale is that MeGAMerge should yield increased numbers of predicted genes without replication of data.

## Related tools

- **Prodigal** (Predicts complete protein-coding genes on >=2 kb contigs in anonymous/metagenomic mode (-p meta -c) to produce gene counts and lengths for coding-density computation)
- **MeGAMerge** (Produces the merged contig set whose gene counts and coding density are compared against the individual Kmer assemblies) — https://github.com/LANL-Bioinformatics/MeGAMerge

## Examples

```
prodigal -i merged_contigs.min2kb.fasta -p meta -c -f gff -o genes.gff
```

## Evaluation signals

- gene_prediction_stats.csv contains the expected fields and exactly one row per assembly analyzed (row_count_equals, field_present)
- Coding density values fall in the plausible microbial range 0.85-0.95 (~90% expected)
- MeGAMerge assembly shows a higher total predicted-gene count than any individual Kmer assembly of the same data, as reported in the paper
- Verify only contigs >=2 kb were included: total contig bases in the denominator matches the >=2 kb filtered FASTA

## Limitations

- Only complete genes (with -c) are counted, so partial genes running off contig ends are excluded from gene counts and coding density
- Removal of small contigs (<2 kb) impacts total assembly size and N50/N90 metrics, which the paper calls not very useful for comparing metagenome assembly performance
- Merging can introduce new chimeras, so predicted genes on merged contigs may include chimeric sequence
- Overlap-based consensus tools' run-times increase exponentially with increasing input dataset size, limiting upstream MeGAMerge scalability

## Evidence
