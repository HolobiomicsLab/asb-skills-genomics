---
name: krona-taxonomic-visualization
description: Use when per-sample Kraken2 classification results from VirDetector's
  metagenomics step (KMA-mapped, assembled viral consensus genomes) need to be summarised
  in one report and explored interactively per sample.
when_to_use_negative:
- You need taxonomic labels on raw reads instead of assembled genomes — in VirDetector,
  Kraken2 taxonomy is assigned to each assembled genome after KMA mapping
- You need phylogenetic trees with maximum likelihood or Bayesian inference — this
  is a separate VirDetector step, not the Krona visualisation
- Input is FASTA-only without reads — FASTA inputs enter the pipeline from the alignment
  step onwards, so there is no metagenomics classification/visualisation step to feed
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: VirDetector
  role: Pipeline (Nextflow, main.nf) that orchestrates sample handling, KMA mapping,
    assembly, Kraken2 classification, and Krona rendering
  repo: https://github.com/NLKaiser/VirDetector
- name: KMA
  role: Maps per-sample trimmed reads against the size-filtered viral reference database;
    mapped reads are then assembled into consensus sequences that are classified
- name: Kraken2
  role: Assigns a taxonomic label to each assembled genome against the same viral
    reference database covering segmented viruses (L, M, S entries)
- name: Krona
  role: Renders one interactive per-sample taxonomic chart from the Kraken2 classification
    output
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btaf029
    title: 'VirDetector: a bioinformatic pipeline for virus surveillance using nanopore
      sequencing'
  source_content_hash: sha256:eecc49fee494ff5432d89fa9c1889f3f9685e12843fe1505e7e1d7befb292178
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NLKaiser/VirDetector
  tools:
  - KMA
  - Kraken2
  - Krona
  - VirDetector
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# krona-taxonomic-visualization

## Summary

Render per-sample interactive taxonomic charts with Krona from VirDetector's metagenomics classification output, alongside a single summarising taxonomic report covering all samples.

## When to use

Use when per-sample Kraken2 classification results from VirDetector's metagenomics step (KMA-mapped, assembled viral consensus genomes) need to be summarised in one report and explored interactively per sample.

## When NOT to use

- You need taxonomic labels on raw reads instead of assembled genomes — in VirDetector, Kraken2 taxonomy is assigned to each assembled genome after KMA mapping
- You need phylogenetic trees with maximum likelihood or Bayesian inference — this is a separate VirDetector step, not the Krona visualisation
- Input is FASTA-only without reads — FASTA inputs enter the pipeline from the alignment step onwards, so there is no metagenomics classification/visualisation step to feed

## Inputs

- Trimmed FASTQ read files per sample (.fastq, .fq, .fastq.gz, .fq.gz) placed in the VirDetector input directory
- VirDetector viral reference database (database/sequences.fasta, processed via make_dbs.py)
- KMA-mapped reads assembled into per-sample viral consensus sequences
- Kraken2 per-sample classification output

## Outputs

- One summarising taxonomic classification report covering all samples
- One interactive Krona visualisation chart per sample

## How to apply

In VirDetector's metagenomics module (a Nextflow workflow, main.nf), load the trimmed FASTQ read files produced during upstream preprocessing and treat each file or subdirectory in the input folder as one sample. Map reads with KMA against the size-filtered viral reference database built from database/sequences.fasta via make_dbs.py, then assemble KMA-mapped reads into per-sample viral consensus sequences. Classify each assembled genome with Kraken2 against the same viral reference database, which covers segmented viruses as independent L, M, and S entries. Compile the per-sample Kraken2 results into one summarising taxonomic classification report and render one Krona interactive chart per sample from the classification output, so taxonomic labels are assigned to assembled genomes rather than raw reads.

## Related tools

- **VirDetector** (Pipeline (Nextflow, main.nf) that orchestrates sample handling, KMA mapping, assembly, Kraken2 classification, and Krona rendering) — https://github.com/NLKaiser/VirDetector
- **KMA** (Maps per-sample trimmed reads against the size-filtered viral reference database; mapped reads are then assembled into consensus sequences that are classified)
- **Kraken2** (Assigns a taxonomic label to each assembled genome against the same viral reference database covering segmented viruses (L, M, S entries))
- **Krona** (Renders one interactive per-sample taxonomic chart from the Kraken2 classification output)

## Examples

```
nextflow run main.nf
```

## Evaluation signals

- Every file or subdirectory in the input folder yields exactly one Krona chart (one chart per sample)
- The single summarising taxonomic classification report contains entries for all samples processed by the metagenomics step
- Taxonomic labels appear on assembled viral consensus genomes, and segmented viruses appear as independent L, M, and S entries consistent with the VirDetector database
- The classification database matches the size-filtered database built from database/sequences.fasta via make_dbs.py (same reference for KMA and Kraken2)
- A full pipeline run such as 'nextflow run main.nf' completes without error for the metagenomics steps

## Limitations

- If a metadata file is specified, only the samples defined in that file are passed on to subsequent processing steps, so those samples will not appear in the Krona output
- The taxonomy is limited to the viruses contained in the VirDetector database (database/sequences.fasta); viruses outside this reference cannot be labelled
- Basecalling is excluded from the default test execution of the pipeline; input must already be FASTQ/FASTA or be basecalled with Guppy or Dorado before classification

## Evidence

- [readme] Files placed or created in the *input* directory are processed as separate samples, while files in each subdirectory inside the *input* directory are automatically combined into one sample.: "Files placed or created in the *input* directory are processed as separate samples, while files in each subdirectory inside the *input* directory are automatically combined into one sample."
- [abstract] the pipeline classifies each sample taxonomically and reconstructs the viral consensus genomes: "the pipeline classifies each sample taxonomically and reconstructs the viral consensus genomes"
- [methods] The VirDetector database for phylogenetic inference currently contains the viruses:: "The VirDetector database for phylogenetic inference currently contains the viruses:"
