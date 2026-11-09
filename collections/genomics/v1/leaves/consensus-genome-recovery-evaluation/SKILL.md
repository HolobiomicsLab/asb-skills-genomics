---
name: consensus-genome-recovery-evaluation
description: 'Use when reproducing or extending the ViMOP benchmark: you need genome
  recovery metrics on FASTQ read sets (simulated viruses, host-virus mixtures, real
  SRA reads) across ViMOP, INSaFLU-TELEVIR, and VirDetector.'
when_to_use_negative:
- The input is a targeted amplicon sequencing dataset rather than untargeted/metagenomic
  nanopore reads — ViMOP's de novo assembly plus BLAST reference identification is
  designed for untargeted data
- Large DNA viruses with extensive repetitive regions (e.g., mpox) requiring finished-quality
  genomes — assemblies should be regarded as draft genomes per the README
- Only a single tool needs to be run with no cross-tool comparison — use the tool's
  own pipeline directly instead of the benchmark harness
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: ViMOP
  role: 'Pipeline under evaluation: trims reads, classifies with Centrifuge, assembles
    de novo contigs with Canu, and builds reference-guided consensus genomes from
    untargeted nanopore reads'
  repo: https://github.com/opr-group-bnitm/vimop
- name: Nextflow
  role: Workflow engine used to execute ViMOP on every benchmark dataset via a single
    Nextflow run with the default docker profile
  repo: https://www.nextflow.io/
- name: Docker
  role: Default execution profile for ViMOP across the benchmark datasets
  repo: https://www.docker.com/
- name: EPI2ME Desktop
  role: Optional ONT interface alternative to the command line for running the ViMOP
    benchmark runs
  repo: https://nanoporetech.com/software/other/epi2me-desktop-application
- name: INSaFLU-TELEVIR
  role: Comparator pipeline run on identical fastq inputs to assess ViMOP/INSaFLU-TELEVIR
    comparability
- name: VirDetector
  role: Second comparator run on the same inputs; its assembly failures are recorded
    as negative benchmark entries
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btaf687
    title: 'ViMOP: a user-friendly and field-applicable pipeline for untargeted viral
      genome nanopore sequencing'
  source_content_hash: sha256:66f755a9fb318cb56055339f3555eb1e2192582a94e6fd8385f3977506a7dfa6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/opr-group-bnitm/vimop
  tools:
  - Docker
  - EPI2ME Desktop
  - EPI2ME desktop
  - INSaFLU-TELEVIR
  - Nextflow
  - ViMOP
  - VirDetector
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# consensus-genome-recovery-evaluation

## Summary

Benchmark skill for evaluating how well ViMOP recovers viral consensus genomes from untargeted nanopore reads compared to INSaFLU-TELEVIR and VirDetector, using simulated viral reads, host-virus mixtures, and real SRA samples. It produces a per-virus, per-sample tool-comparison table mirroring the paper's Section 2.11.

## When to use

Use when reproducing or extending the ViMOP benchmark: you need genome recovery metrics on FASTQ read sets (simulated viruses, host-virus mixtures, real SRA reads) across ViMOP, INSaFLU-TELEVIR, and VirDetector.

## When NOT to use

- The input is a targeted amplicon sequencing dataset rather than untargeted/metagenomic nanopore reads — ViMOP's de novo assembly plus BLAST reference identification is designed for untargeted data
- Large DNA viruses with extensive repetitive regions (e.g., mpox) requiring finished-quality genomes — assemblies should be regarded as draft genomes per the README
- Only a single tool needs to be run with no cross-tool comparison — use the tool's own pipeline directly instead of the benchmark harness

## Inputs

- FASTQ read directories (optionally barcode01, barcode02, ... subdirectories) of simulated viral reads
- Mixed host-virus FASTQ datasets (human/mouse/mastomys host references)
- Real SRA nanopore read sets
- ViMOP_DB virus reference database (Zenodo 10.5281/zenodo.17652512)
- Benchmarking scripts (Zenodo record 17633185)
- INSaFLU-TELEVIR and VirDetector pipelines on identical fastq inputs

## Outputs

- benchmark_comparison_table.csv (tool-comparison table mirroring Section 2.11)
- Per-reference contigs from BLAST against the virus reference database
- Consensus genomes from samtools/medaka with depth and variant files
- Krona plots of Centrifuge read classification (HTML report)
- Alignment .bam files for output review

## How to apply

Retrieve the benchmarking scripts from Zenodo record 17633185 and stage the evaluation read sets together with the ViMOP_DB reference database placed in a home-directory folder called ViMOP_DB. Execute ViMOP on every dataset via a single Nextflow run with the default docker profile, setting --trim_length to remove adapters and primers, letting Centrifuge classify and filter non-viral reads, and running Canu de novo assembly with iterative re-assembly (--reassemble_max_iter). Run INSaFLU-TELEVIR on the identical fastq inputs to assess comparability, and run VirDetector on the same inputs, recording its assembly failures as negative benchmark entries. For each run, collect per-reference results (contigs identified by BLAST against the virus reference database with the highest-scoring hit used as reference genome, consensus genomes via samtools/medaka, depth and variant files) and compute genome recovery metrics per virus and per sample. Aggregate metrics into benchmark_comparison_table.csv and validate that ViMOP performs comparably to INSaFLU-TELEVIR on reconstructable genomes while VirDetector's assembly failures are documented.

## Related tools

- **ViMOP** (Pipeline under evaluation: trims reads, classifies with Centrifuge, assembles de novo contigs with Canu, and builds reference-guided consensus genomes from untargeted nanopore reads) — https://github.com/opr-group-bnitm/vimop
- **Nextflow** (Workflow engine used to execute ViMOP on every benchmark dataset via a single Nextflow run with the default docker profile) — https://www.nextflow.io/
- **Docker** (Default execution profile for ViMOP across the benchmark datasets) — https://www.docker.com/
- **EPI2ME Desktop** (Optional ONT interface alternative to the command line for running the ViMOP benchmark runs) — https://nanoporetech.com/software/other/epi2me-desktop-application
- **INSaFLU-TELEVIR** (Comparator pipeline run on identical fastq inputs to assess ViMOP/INSaFLU-TELEVIR comparability)
- **VirDetector** (Second comparator run on the same inputs; its assembly failures are recorded as negative benchmark entries)

## Examples

```
nextflow run opr-group-bnitm/vimop --fastq benchmark_reads/ --trim_length 50 -profile docker
```

## Evaluation signals

- benchmark_comparison_table.csv values reproduce the Section 2.11 reported comparison (validated by expert_review on qualitative agreement)
- ViMOP shows comparable results to INSaFLU-TELEVIR on reconstructable genomes in most cases
- VirDetector's failures to automatically assemble many of the genomes are recorded as negative benchmark entries
- Each per-reference output includes contigs matched by BLAST to the virus reference database, a consensus genome from samtools/medaka, plus depth and variant files
- Krona plot of Centrifuge read classification appears in the HTML report for each dataset

## Limitations

- VirDetector failed to automatically assemble many of the benchmark genomes
- For large DNA viruses with extensive repetitive regions (e.g., mpox), assemblies may contain inaccuracies and should be regarded as draft genomes
- Centrifuge filtering has limited accuracy and may remove some false positives; benchmark comparability is qualitative (expert_review) rather than fully quantitative
- Tools like Canu need substantial resources (recommended at least 30 GB RAM and 16 CPUs); lowering --min_cpus/--min_ram may slow or break benchmark runs

## Evidence

- [methods] de novo contig assembly then BLAST-based reference identification: "Each contig is used to for a BLAST search in the virus reference database. The highest scoring hit is then used as a reference genome."
- [methods] iterative re-assembly parameter: "After the initial assembly, an iterative re-assembly procedure is run (unless deactivated with `--reassemble_max_iter 0`)."
- [abstract] core ViMOP assembly logic: "The software assembles de novo contigs, matches contigs to known viral references and uses them to assemble consensus genomes."
- [readme] consensus building and review recommendation: "The pipeline automatically finds well fitting virus genomes and uses them as references to build reference based consensus genomes."
- [readme] draft-genome caveat for repetitive DNA viruses: "for large DNA viruses with extensive repetitive regions (e.g., mpox), assemblies may contain inaccuracies and should be regarded as draft genomes"
