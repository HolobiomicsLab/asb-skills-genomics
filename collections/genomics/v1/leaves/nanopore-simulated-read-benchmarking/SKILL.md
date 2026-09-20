---
name: nanopore-simulated-read-benchmarking
description: Use when comparing untargeted nanopore virus genome pipelines for robustness
  to high mutation levels or evaluating recovery of partial genomes below 50% completeness
  on simulated read sets.
when_to_use_negative:
- Input is real clinical nanopore data rather than simulated read sets with known
  ground-truth references, since recovery/completeness metrics require a known reference
- The goal is benchmarking tools outside virus genome reconstruction from untargeted
  nanopore reads (e.g. bacterial taxonomy benchmarking)
- Mutations are at low levels, where the high-mutation robustness comparison of ViMOP
  vs INSaFLU-TELEVIR is not being exercised
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: ViMOP
  role: 'Primary pipeline under test: trims reads, classifies with Centrifuge (min
    score 150), removes host/contaminants, assembles de novo contigs with Canu and
    iterative re-assembly, BLASTs contigs against the reference database, and calls
    reference-guided consensus genomes'
  repo: https://github.com/opr-group-bnitm/vimop
- name: Nextflow
  role: Execution engine for running the ViMOP pipeline with the Docker profile on
    the simulated fastq directories
  repo: https://github.com/opr-group-bnitm/vimop
- name: INSaFLU-TELEVIR
  role: Comparator pipeline run on the identical simulated read sets to benchmark
    genome recovery at high mutation levels
- name: Docker
  role: Container profile used by default to run the ViMOP Nextflow pipeline reproducibly
  repo: https://github.com/opr-group-bnitm/vimop
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaf687
    title: 'ViMOP: a user-friendly and field-applicable pipeline for untargeted viral
      genome nanopore sequencing'
  source_content_hash: sha256:9ec46126a05631a7da0a8650aece1e17c9d5c997af968077f83f1c590536800d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/opr-group-bnitm/vimop
  tools:
  - Docker
  - INSaFLU-TELEVIR
  - Nextflow
  - ViMOP
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# nanopore-simulated-read-benchmarking

## Summary

Benchmark virus genome reconstruction pipelines (e.g. ViMOP vs INSaFLU-TELEVIR) on simulated nanopore read sets generated at high mutation levels, measuring genome recovery and completeness from consensus fasta and depth outputs, including recovery of partial (<50% completeness) genomes.

## When to use

Use when comparing untargeted nanopore virus genome pipelines for robustness to high mutation levels or evaluating recovery of partial genomes below 50% completeness on simulated read sets.

## When NOT to use

- Input is real clinical nanopore data rather than simulated read sets with known ground-truth references, since recovery/completeness metrics require a known reference
- The goal is benchmarking tools outside virus genome reconstruction from untargeted nanopore reads (e.g. bacterial taxonomy benchmarking)
- Mutations are at low levels, where the high-mutation robustness comparison of ViMOP vs INSaFLU-TELEVIR is not being exercised

## Inputs

- Simulated nanopore FASTQ read directories generated at high mutation levels (Zenodo record 17632662 scripts)
- Benchmarking code from Zenodo record 17633185
- ViMOP virus reference database (Zenodo record 17652512)
- Consensus fasta and per-base depth outputs from each pipeline

## Outputs

- benchmark_high_mutation_comparison.csv summarizing genome recovery per pipeline and sample
- Per-sample completeness values (fraction of reference covered by consensus)
- Flags for partial genomes (<50% completeness) recovered by ViMOP but not INSaFLU-TELEVIR

## How to apply

Download the benchmarking code from Zenodo record 17633185 and the simulated read sets generated at high mutation levels via the scripts in Zenodo record 17632662. Run the ViMOP pipeline with Nextflow using the Docker profile on the high-mutation simulated fastq directories: it trims reads, classifies them with Centrifuge (minimum score 150 for non-viral filtering), removes host/contaminant reads, assembles de novo contigs with Canu including iterative re-assembly, BLASTs contigs against the virus reference database (Zenodo 17652512), and calls reference-guided consensus genomes. Run INSaFLU-TELEVIR on the identical simulated read sets as the comparator. For each pipeline and sample, compute genome recovery and completeness (fraction of the reference covered by the consensus) from the consensus fasta and depth outputs, and tabulate recovery, explicitly flagging partial genomes below 50% completeness recovered by one pipeline but not the other.

## Related tools

- **ViMOP** (Primary pipeline under test: trims reads, classifies with Centrifuge (min score 150), removes host/contaminants, assembles de novo contigs with Canu and iterative re-assembly, BLASTs contigs against the reference database, and calls reference-guided consensus genomes) — https://github.com/opr-group-bnitm/vimop
- **Nextflow** (Execution engine for running the ViMOP pipeline with the Docker profile on the simulated fastq directories) — https://github.com/opr-group-bnitm/vimop
- **INSaFLU-TELEVIR** (Comparator pipeline run on the identical simulated read sets to benchmark genome recovery at high mutation levels)
- **Docker** (Container profile used by default to run the ViMOP Nextflow pipeline reproducibly) — https://github.com/opr-group-bnitm/vimop

## Examples

```
nextflow run opr-group-bnitm/vimop --fastq sim_high_mutation_fastq/ -profile docker --centrifuge_filter_min_score 150
```

## Evaluation signals

- ViMOP attains higher genome recovery than INSaFLU-TELEVIR at the high mutation level
- ViMOP shows non-zero recovery of partial (<50% completeness) genomes where INSaFLU-TELEVIR recovers none
- benchmark_high_mutation_comparison.csv is produced with per-pipeline, per-sample recovery and completeness values
- Completeness values are recomputable as the fraction of the reference covered by the consensus from the consensus fasta and depth outputs
- Both pipelines were run on the identical simulated read sets so metrics are directly comparable

## Limitations

- ViMOP assemblies for large DNA viruses with extensive repetitive regions (e.g. mpox) may contain inaccuracies and should be regarded as draft genomes
- Centrifuge classification is limited in accuracy, so non-viral read filtering with minimum score 150 may also remove some false-positive viral reads
- Results are validated specifically for small and medium size RNA viruses such as Lassa, Dengue, and Ebola

## Evidence

- [abstract] The software assembles de novo contigs, matches contigs to known viral references and uses them to assemble consensus genomes.: "The software assembles de novo contigs, matches contigs to known viral references and uses them to assemble consensus genomes."
- [readme] centrifuge filter minimum score: "use `--centrifuge_filter_min_score 150` to set the minimum level of confidence (e.g. 150 in our example)."
- [abstract] benchmarking code on Zenodo: "and benchmarking code (https://zenodo.org/records/17633185)"
- [readme] iterative re-assembly: "After the initial assembly, an iterative re-assembly procedure is run (unless deactivated with `--reassemble_max_iter 0`)."
- [methods] reference identification via BLAST: "Each contig is used to for a BLAST search in the virus reference database. The highest scoring hit is then used as a reference genome."
