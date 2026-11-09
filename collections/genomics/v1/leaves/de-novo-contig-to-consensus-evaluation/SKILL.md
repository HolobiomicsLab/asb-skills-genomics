---
name: de-novo-contig-to-consensus-evaluation
description: Use when benchmarking or verifying untargeted nanopore viral pipelines
  on known-virus samples, especially at high mutation levels or with partial genomes
  (<50% completeness) where reference-tethered pipelines fail.
when_to_use_negative:
- When the goal is de novo discovery of novel, unknown viruses — ViMOP matches contigs
  to known references in its database and is intended for known virus species in untargeted
  data
- When working with large DNA viruses with extensive repetitive regions (e.g. mpox),
  where the pipeline itself warns assemblies may be inaccurate draft genomes
- When the input is not nanopore FASTQ data from untargeted/metagenomic sequencing
edam_operation: http://edamontology.org/operation_0524
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0102
tools:
- name: ViMOP
  role: Pipeline performing read trimming, Centrifuge classification/filtering, Canu
    de novo assembly with iterative re-assembly, BLAST reference matching, and reference-guided
    consensus calling
  repo: https://github.com/opr-group-bnitm/vimop
- name: Nextflow
  role: Workflow engine executing ViMOP, e.g. with the Docker profile on the simulated
    fastq directories
  repo: https://www.nextflow.io/
- name: Docker
  role: Default execution profile packaging ViMOP's dependencies
  repo: https://www.docker.com/
- name: medaka
  role: Consensus/variant-calling engine used to create consensus sequences from a
    pileup of nanopore reads against a reference sequence during consensus generation
  repo: https://github.com/nanoporetech/medaka
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaf687
    title: 'ViMOP: a user-friendly and field-applicable pipeline for untargeted viral
      genome nanopore sequencing'
  source_content_hash: sha256:7c5ac07ecb5a4ddd1f517abbc4a591f11d1f5756275ead14c6e57fc57e971596
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/opr-group-bnitm/vimop
  tools:
  - Docker
  - Nextflow
  - ViMOP
  - medaka
  edam_operation: http://edamontology.org/operation_0524
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# de-novo-contig-to-consensus-evaluation

## Summary

Evaluates ViMOP's de novo assembly-to-consensus strategy: contigs assembled from untargeted nanopore reads are BLASTed against a virus reference database to select reference genomes, which then guide reference-based consensus reconstruction; benchmarked against INSaFLU-TELEVIR and VirDetector on simulated high-mutation read sets.

## When to use

Use when benchmarking or verifying untargeted nanopore viral pipelines on known-virus samples, especially at high mutation levels or with partial genomes (<50% completeness) where reference-tethered pipelines fail.

## When NOT to use

- When the goal is de novo discovery of novel, unknown viruses — ViMOP matches contigs to known references in its database and is intended for known virus species in untargeted data
- When working with large DNA viruses with extensive repetitive regions (e.g. mpox), where the pipeline itself warns assemblies may be inaccurate draft genomes
- When the input is not nanopore FASTQ data from untargeted/metagenomic sequencing

## Inputs

- Directories of nanopore FASTQ files (optionally split into barcode01, barcode02, ... subdirectories per sample)
- Simulated high-mutation viral read sets (Zenodo record 17632662)
- ViMOP virus reference database (Zenodo record 17652512)
- Host/contaminant reference sequences (e.g. human GRCh38 GCF_000001405.39, mouse GRCm38, mastomys) for contamination filtering

## Outputs

- Per-sample reference-guided consensus genome FASTA files
- Coverage/depth outputs for each recovered genome
- Benchmark table (benchmark_high_mutation_comparison.csv) of genome recovery and completeness per pipeline and sample, with <50%-completeness partial genomes flagged

## How to apply

Run ViMOP via Nextflow (Docker profile) on the high-mutation simulated fastq directories: trim reads, classify with Centrifuge and remove non-viral reads at a centrifuge_filter_min_score of 150, strip host/contaminant reads by mapping, assemble de novo contigs with Canu including the iterative re-assembly step (controllable via --reassemble_max_iter), then BLAST each contig against the ViMOP virus reference database (Zenodo 17652512) taking the highest-scoring hit as the reference genome. Use the selected reference for reference-guided mapping and consensus calling to produce per-sample consensus FASTA and depth outputs. Run INSaFLU-TELEVIR on the identical simulated read sets as the comparator. Compute genome recovery as the fraction of the reference covered by the consensus, tabulate recovery at high mutation levels, and explicitly flag partial genomes below 50% completeness that ViMOP recovers but the comparator does not.

## Related tools

- **ViMOP** (Pipeline performing read trimming, Centrifuge classification/filtering, Canu de novo assembly with iterative re-assembly, BLAST reference matching, and reference-guided consensus calling) — https://github.com/opr-group-bnitm/vimop
- **Nextflow** (Workflow engine executing ViMOP, e.g. with the Docker profile on the simulated fastq directories) — https://www.nextflow.io/
- **Docker** (Default execution profile packaging ViMOP's dependencies) — https://www.docker.com/
- **medaka** (Consensus/variant-calling engine used to create consensus sequences from a pileup of nanopore reads against a reference sequence during consensus generation) — https://github.com/nanoporetech/medaka

## Examples

```
nextflow run opr-group-bnitm/vimop --fastq high_mutation_fastq_dir -profile docker --centrifuge_filter_min_score 150
```

## Evaluation signals

- ViMOP attains higher genome recovery than INSaFLU-TELEVIR on the high-mutation simulated read sets
- Non-zero recovery of partial genomes (<50% completeness) in ViMOP where INSaFLU-TELEVIR recovers none
- Genome recovery and completeness computed from consensus FASTA plus depth outputs cover the expected fraction of the reference
- Each BLAST-assigned reference genome is the highest-scoring hit in the virus reference database for the corresponding contig
- Centrifuge filtering removes reads classified non-viral at score >= 150, not contigs

## Limitations

- For large DNA viruses with extensive repetitive regions (e.g., mpox), assemblies may contain inaccuracies and should be regarded as draft genomes
- Centrifuge classification is limited in accuracy and non-viral read removal may also remove some false positives
- Crash when the sequence contains a non-standard nucleotide (ACGT) that is also not an N (fixed in ViMOP 1.0.1)
- Conda profile did not work with 'nextflow run' prior to ViMOP 1.0.4; bug 'Cannot invoke method optional() on null object' with Nextflow 25.10.2 (fixed in 1.0.4)

## Evidence

- [abstract] The software assembles de novo contigs, matches contigs to known viral references and uses them to assemble consensus genomes.: "The software assembles de novo contigs, matches contigs to known viral references and uses them to assemble consensus genomes."
- [methods] Each contig is used to for a BLAST search in the virus reference database. The highest scoring hit is then used as a reference genome.: "Each contig is used to for a BLAST search in the virus reference database. The highest scoring hit is then used as a reference genome."
- [methods] After the initial assembly, an iterative re-assembly procedure is run (unless deactivated with `--reassemble_max_iter 0`).: "After the initial assembly, an iterative re-assembly procedure is run (unless deactivated with `--reassemble_max_iter 0`)."
- [readme] the centrifuge classifications is used to remove reads (not contigs) that are non-viral to a given degree of confidence.: "the centrifuge classifications is used to remove reads (not contigs) that are non-viral to a given degree of confidence."
- [readme] The pipeline will remove all reads that are classified to anything that is not viral with at least this score.: "The pipeline will remove all reads that are classified to anything that is not viral with at least this score."
