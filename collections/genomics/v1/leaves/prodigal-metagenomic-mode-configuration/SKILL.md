---
name: prodigal-metagenomic-mode-configuration
description: Use when comparing gene content across multiple metagenomic assemblies
  (e.g., merged vs individual assembler contigs), requiring uniform gene prediction
  and per-assembly coding-density statistics.
when_to_use_negative:
- When assemblies must include partial genes at contig ends — the -c option restricts
  predictions to complete genes only
- When the input is raw reads or quality-trimmed FASTQ rather than assembled contig
  FASTA files
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: Prodigal
  role: Gene prediction in anonymous/metagenomic mode (-p meta) with complete-gene
    restriction (-c) on >= 2 kb contigs
- name: MeGAMerge
  role: Produces the merged contig assemblies whose predicted gene counts and coding
    density are being compared against individual assemblies
  repo: https://github.com/LANL-Bioinformatics/MeGAMerge
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/srep06480
    title: Improved Assemblies Using a Source-Agnostic Pipeline for MetaGenomic Assembly
      by Merging (MeGAMerge) of Contigs
  source_content_hash: sha256:dda60847d223b1394516852d6af82497b2dfd7a154be986e8a380f5b20b5cd3b
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
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# prodigal-metagenomic-mode-configuration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Configure Prodigal 2.6 in anonymous/metagenomic mode (-p meta) with complete-gene restriction (-c) to predict genes on metagenomic assembly contigs, enabling gene-count and coding-density comparison of MeGAMerge vs individual Kmer assemblies.

## When to use

Use when comparing gene content across multiple metagenomic assemblies (e.g., merged vs individual assembler contigs), requiring uniform gene prediction and per-assembly coding-density statistics.

## When NOT to use

- When assemblies must include partial genes at contig ends — the -c option restricts predictions to complete genes only
- When the input is raw reads or quality-trimmed FASTQ rather than assembled contig FASTA files

## Inputs

- Assembly FASTA files (multi-FASTA contig sets) from individual Kmer-based assemblers and the MeGAMerge merged assembly
- Contig size filter: only contigs >= 2 kb retained for gene prediction
- Prodigal 2.6 gene-coordinate output files (GFF or similar coordinate format)

## Outputs

- Per-assembly predicted gene counts and summed gene lengths parsed from Prodigal output
- gene_prediction_stats.csv with gene count and coding density (total gene length / total contig bases) per assembly

## How to apply

For each assembly, first filter the FASTA to contigs >= 2 kb, mirroring the paper's practice of removing very small contigs before Prodigal gene prediction. Run Prodigal 2.6 in anonymous/metagenomic mode with the -p meta option and -c so only complete genes that do not run off contig ends are predicted; this keeps predictions comparable across assemblies with different training characteristics. Parse the Prodigal gene-coordinate output files to count predicted genes and sum gene lengths per assembly. Compute coding density as total predicted gene length divided by total contig bases of the >= 2 kb contigs, then tabulate gene counts and coding density per assembly (including the MeGAMerge assembly) as gene_prediction_stats.csv. This method supports the paper's finding that MeGAMerge yields a consistent increase in predicted genes while coding density (~90%) stays in the expected range for microbial genomes.

## Related tools

- **Prodigal** (Gene prediction in anonymous/metagenomic mode (-p meta) with complete-gene restriction (-c) on >= 2 kb contigs)
- **MeGAMerge** (Produces the merged contig assemblies whose predicted gene counts and coding density are being compared against individual assemblies) — https://github.com/LANL-Bioinformatics/MeGAMerge

## Examples

```
prodigal -i merged_contigs.fasta -p meta -c -f gff -o genes.gff
```

## Evaluation signals

- gene_prediction_stats.csv contains the expected fields and exactly one row per assembly (row_count_equals, field_present checks)
- All computed coding density values fall in the plausible range 0.85-0.95 (value_in_range check)
- The MeGAMerge assembly shows a higher total predicted-gene count than each individual Kmer assembly, as reported in the paper
- Gene counts and gene-length sums reproduce exactly from parsing the Prodigal gene-coordinate files
- Each assembly's FASTA was pre-filtered so no contig shorter than 2 kb entered gene prediction

## Limitations

- Using -c discards partial genes running off contig ends, so gene counts undercount short contigs' coding content
- MeGAMerge's merging process can introduce new chimeras, which may affect downstream gene predictions on merged contigs
- Overlap-based consensus tools like the ones underpinning MeGAMerge have run-times that increase exponentially with increasing input dataset size

## Evidence

- [methods] Prodigal meta mode with complete-gene restriction: [evidence span withheld — anchor status: not_found]
- [results] Consistent gene-count improvement finding: [evidence span withheld — anchor status: not_found]
