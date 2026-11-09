---
name: viral-genome-assembly-benchmarking
description: Use when evaluating or comparing de novo/consensus viral genome assembly
  pipelines on untargeted nanopore reads, especially when genome recovery per virus
  per sample must be quantified against published results.
when_to_use_negative:
- Input is targeted amplicon sequencing data rather than untargeted (metagenomic)
  nanopore reads — ViMOP's de novo assembly and Centrifuge-based filtering are designed
  for untargeted data
- The goal is a single-sample clinical run with an HTML report, not a multi-tool benchmark
  — use the pipeline directly rather than the benchmark harness
- Comparing pipelines designed for short-read or Illumina data whose assembly dynamics
  differ fundamentally from the nanopore-based comparators
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_3673
tools:
- name: ViMOP
  role: 'Pipeline under test: detects known virus species in untargeted nanopore reads
    and reconstructs consensus genomes for benchmark comparison'
  repo: https://github.com/opr-group-bnitm/vimop
- name: Nextflow
  role: Executes ViMOP on every benchmark dataset via a single command with the default
    docker profile
  repo: https://github.com/opr-group-bnitm/vimop
- name: Docker
  role: Default execution profile for running ViMOP reproducibly on benchmark datasets
  repo: https://github.com/opr-group-bnitm/vimop
- name: Canu
  role: De novo assembler used iteratively on filtered nanopore read sets; assembly
    failures by comparator VirDetector are recorded as negative benchmark entries
  repo: https://github.com/opr-group-bnitm/vimop
- name: EPI2ME desktop
  role: Optional ONT interface for running ViMOP on benchmark inputs
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btaf687
    title: 'ViMOP: a user-friendly and field-applicable pipeline for untargeted viral
      genome nanopore sequencing'
  source_content_hash: sha256:ac16377efc1a06bf3da6369983f8413f7399f0faed75ce2657722761f3efd108
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/opr-group-bnitm/vimop
  tools:
  - Canu
  - Docker
  - EPI2ME desktop
  - Nextflow
  - ViMOP
  - cd-hit-est
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# viral-genome-assembly-benchmarking

## Summary

A benchmarking protocol for comparing nanopore viral genome reconstruction pipelines (e.g., ViMOP vs INSaFLU-TELEVIR vs VirDetector) across simulated viral reads, host-virus mixtures, and real SRA samples, producing a tool-comparison table of genome recovery metrics.

## When to use

Use when evaluating or comparing de novo/consensus viral genome assembly pipelines on untargeted nanopore reads, especially when genome recovery per virus per sample must be quantified against published results.

## When NOT to use

- Input is targeted amplicon sequencing data rather than untargeted (metagenomic) nanopore reads — ViMOP's de novo assembly and Centrifuge-based filtering are designed for untargeted data
- The goal is a single-sample clinical run with an HTML report, not a multi-tool benchmark — use the pipeline directly rather than the benchmark harness
- Comparing pipelines designed for short-read or Illumina data whose assembly dynamics differ fundamentally from the nanopore-based comparators

## Inputs

- FASTQ read sets: simulated viral reads, mixed host-virus datasets, and real Sequence Read Archive (SRA) samples
- ViMOP_DB virus reference database
- Benchmarking scripts from Zenodo record 17633185
- Host/contamination reference FASTA files (e.g., human GRCh38) used by contamination filters

## Outputs

- benchmark_comparison_table.csv with per-virus, per-sample genome recovery metrics for each tool
- Per-reference results: BLAST-identified contigs, consensus genomes, depth and variant files
- Documented assembly failures for pipelines that fail to automatically assemble genomes (e.g., VirDetector)

## How to apply

Retrieve benchmarking scripts (Zenodo record 17633185) and stage evaluation read sets — simulated viral reads, mixed host-virus datasets, and real SRA samples — together with the ViMOP_DB reference database. Run the pipeline under test (e.g., ViMOP via a single Nextflow command with the default docker profile, using --trim_length read-end trimming, Centrifuge read classification, Canu de novo assembly, and iterative re-assembly controlled by --reassemble_max_iter), then run each comparator (INSaFLU-TELEVIR, VirDetector) on identical fastq inputs, recording VirDetector's automatic assembly failures as negative benchmark entries. For each run, collect per-reference outputs — contigs identified by BLAST against the virus reference database, consensus genomes from samtools/medaka, plus depth and variant files — and compute genome recovery metrics per virus and per sample. Aggregate the metrics into a single tool-comparison table (benchmark_comparison_table.csv) mirroring the article's Section 2.11 table.

## Related tools

- **ViMOP** (Pipeline under test: detects known virus species in untargeted nanopore reads and reconstructs consensus genomes for benchmark comparison) — https://github.com/opr-group-bnitm/vimop
- **Nextflow** (Executes ViMOP on every benchmark dataset via a single command with the default docker profile) — https://github.com/opr-group-bnitm/vimop
- **Docker** (Default execution profile for running ViMOP reproducibly on benchmark datasets) — https://github.com/opr-group-bnitm/vimop
- **Canu** (De novo assembler used iteratively on filtered nanopore read sets; assembly failures by comparator VirDetector are recorded as negative benchmark entries) — https://github.com/opr-group-bnitm/vimop
- **EPI2ME desktop** (Optional ONT interface for running ViMOP on benchmark inputs)

## Examples

```
nextflow run opr-group-bnitm/vimop --fastq benchmark_reads/ --db ViMOP_DB -profile docker
```

## Evaluation signals

- benchmark_comparison_table.csv reproduces the Section 2.11 reported comparison: ViMOP and INSaFLU-TELEVIR show comparable results in most cases
- VirDetector's failure to automatically assemble many genomes is recorded as negative benchmark entries
- Each run produces the expected per-reference outputs (contigs identified by BLAST against the virus reference database, consensus genomes from samtools/medaka, depth and variant files)
- Genome recovery metrics are computed per virus and per sample for all three tools on identical fastq inputs
- Qualitative agreement with the published benchmark figures confirmed by expert review

## Limitations

- VirDetector failed to automatically assemble many of the genomes, so its benchmark rows reflect assembly failures rather than reconstruction quality
- ViMOP and INSaFLU-TELEVIR showed comparable results only 'in most cases' — genome reconstruction is not guaranteed for every virus in every dataset
- For large DNA viruses with extensive repetitive regions (e.g., mpox), assemblies may contain inaccuracies and should be regarded as draft genomes, which affects benchmark interpretations
- Centrifuge classification accuracy is limited; false-positive read removal can affect upstream assembly and thus benchmark outcomes

## Evidence

- [other] In the benchmark, VirDetector failed to automatically assemble many of the genomes, while ViMOP and INSaFLU-TELEVIR showed comparable results in most cases.: [evidence span withheld — anchor status: not_found]
- [abstract] benchmarking code Zenodo record: "the scripts used to generate it (https://zenodo.org/records/17632662) and benchmarking code (https://zenodo.org/records/17633185)"
- [abstract] Execution via single Nextflow command: "Executed with a single Nextflow command or via the EPI2ME Desktop interface (ONT), results are summarized in an HTML report."
- [readme] Iterative re-assembly in ViMOP: "After the initial assembly, an iterative re-assembly procedure is run (unless deactivated with `--reassemble_max_iter 0`)."
- [readme] ViMOP purpose statement: "ViMOP is a pipeline for detecting known virus species in untargeted Oxford Nanopore sequencing data and reconstructing their genomes."
- [readme] Draft genome caveat for large DNA viruses: "for large DNA viruses with extensive repetitive regions (e.g., mpox), assemblies may contain inaccuracies and should be regarded as draft genomes."
