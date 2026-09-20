---
name: cross-tool-comparison-metrics-tabulation
description: Use when comparing genome-reconstruction performance of viral nanopore
  assembly pipelines (ViMOP, INSaFLU-TELEVIR, VirDetector) on the same read sets and
  you need a consolidated benchmark comparison table.
when_to_use_negative:
- The goal is de novo discovery of novel viruses rather than benchmarking known-virus
  genome reconstruction against reference databases
- Input data are not untargeted nanopore fastq read sets shared across all comparator
  pipelines, so identical-input comparisons cannot be made
- Large DNA viruses with extensive repetitive regions (e.g., mpox) are the target,
  since ViMOP assemblies for these may be inaccurate drafts that would confound recovery
  metrics
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0781
- http://edamontology.org/topic_3174
tools:
- name: ViMOP
  role: Primary pipeline under evaluation; run via a single Nextflow command with
    docker profile to produce contigs and consensus genomes
  repo: https://github.com/opr-group-bnitm/vimop
- name: Nextflow
  role: Execution engine to run ViMOP on every benchmark dataset in a single run
  repo: https://github.com/opr-group-bnitm/vimop
- name: Docker
  role: Default container profile for ViMOP execution during the benchmark
- name: EPI2ME desktop
  role: Optional ONT interface for executing ViMOP instead of the command line
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btaf687
    title: 'ViMOP: a user-friendly and field-applicable pipeline for untargeted viral
      genome nanopore sequencing'
  source_content_hash: sha256:784443c325c70cd7381f2f148d38856f444cd220ba74e2bf9a41554f70ba5411
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/opr-group-bnitm/vimop
  tools:
  - Docker
  - EPI2ME desktop
  - Nextflow
  - ViMOP
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0781
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# cross-tool-comparison-metrics-tabulation

## Summary

Reproduces the ViMOP benchmark against INSaFLU-TELEVIR and VirDetector by running each pipeline on identical fastq inputs (simulated viral reads, host-virus mixtures, real SRA samples) and tabulating per-virus, per-sample genome recovery metrics into a single tool-comparison table mirroring Section 2.11 of the paper.

## When to use

Use when comparing genome-reconstruction performance of viral nanopore assembly pipelines (ViMOP, INSaFLU-TELEVIR, VirDetector) on the same read sets and you need a consolidated benchmark comparison table.

## When NOT to use

- The goal is de novo discovery of novel viruses rather than benchmarking known-virus genome reconstruction against reference databases
- Input data are not untargeted nanopore fastq read sets shared across all comparator pipelines, so identical-input comparisons cannot be made
- Large DNA viruses with extensive repetitive regions (e.g., mpox) are the target, since ViMOP assemblies for these may be inaccurate drafts that would confound recovery metrics

## Inputs

- Evaluation fastq read sets: simulated viral reads, mixed host-virus datasets, and real SRA samples
- ViMOP_DB virus reference database
- Benchmarking scripts from Zenodo record 17633185
- Per-run outputs from ViMOP, INSaFLU-TELEVIR, and VirDetector

## Outputs

- benchmark_comparison_table.csv aggregating genome recovery metrics per virus and per sample across tools
- Per-reference results: BLAST-identified contigs, consensus genomes from samtools/medaka, depth and variant files
- Records of VirDetector assembly failures as negative benchmark entries

## How to apply

Retrieve the benchmarking scripts from Zenodo record 17633185 and stage the evaluation read sets (simulated viral reads, mixed host-virus datasets, real SRA samples) together with the ViMOP_DB reference database. Execute ViMOP on every dataset via a single Nextflow run with the default docker profile, trimming read ends with --trim_length and classifying reads with Centrifuge before de novo assembly with Canu and iterative re-assembly (--reassemble_max_iter). Run INSaFLU-TELEVIR on the identical fastq inputs to assess comparability, and run VirDetector on the same inputs, recording its assembly failures as negative benchmark entries. For each run, collect per-reference results (contigs identified by BLAST against the virus reference database, consensus genomes from samtools/medaka, depth and variant files) and compute genome recovery metrics per virus and per sample. Aggregate the metrics into benchmark_comparison_table.csv and accept it when the values reproduce the Section 2.11 reported comparison, verified qualitatively against published figures.

## Related tools

- **ViMOP** (Primary pipeline under evaluation; run via a single Nextflow command with docker profile to produce contigs and consensus genomes) — https://github.com/opr-group-bnitm/vimop
- **Nextflow** (Execution engine to run ViMOP on every benchmark dataset in a single run) — https://github.com/opr-group-bnitm/vimop
- **Docker** (Default container profile for ViMOP execution during the benchmark)
- **EPI2ME desktop** (Optional ONT interface for executing ViMOP instead of the command line)

## Examples

```
nextflow run opr-group-bnitm/vimop --fastq evaluation_readsets/ --db ViMOP_DB --trim_length 30 --reassemble_max_iter 5 -profile docker
```

## Evaluation signals

- benchmark_comparison_table.csv values reproduce the Section 2.11 reported comparison (expert_review on qualitative agreement with published figures)
- ViMOP and INSaFLU-TELEVIR show comparable results on reconstructable genomes in most cases
- VirDetector's failure to automatically assemble many genomes appears as documented negative benchmark entries
- Per-virus and per-sample genome recovery metrics are computed for every run from contigs, consensus genomes, depth and variant files
- All three comparator pipelines were executed on identical fastq inputs

## Limitations

- VirDetector fails to automatically assemble many of the genomes, limiting its comparability on those entries
- For large DNA viruses with extensive repetitive regions (e.g., mpox), ViMOP assemblies may contain inaccuracies and should be regarded as draft genomes
- Centrifuge classification used for read filtering is limited in accuracy and may remove some false positives

## Evidence

- [other] In the benchmark, VirDetector failed to automatically assemble many of the genomes, while ViMOP and INSaFLU-TELEVIR showed comparable results in most cases.: [evidence span withheld — anchor status: not_found]
- [abstract] benchmarking code on Zenodo: "and benchmarking code (https://zenodo.org/records/17633185)"
- [readme] ViMOP description in README: "The pipeline automatically finds well fitting virus genomes and uses them as references to build reference based consensus genomes."
- [readme] mpox limitation in README: "However, for large DNA viruses with extensive repetitive regions (e.g., mpox), assemblies may contain inaccuracies and should be regarded as draft genomes."
- [abstract] consensus assembly workflow in article: "The software assembles de novo contigs, matches contigs to known viral references and uses them to assemble consensus genomes."
