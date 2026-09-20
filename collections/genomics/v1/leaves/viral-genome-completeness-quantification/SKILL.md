---
name: viral-genome-completeness-quantification
description: Use when benchmarking nanopore virus genome pipelines on simulated or
  real reads and you need per-sample genome recovery metrics, especially to detect
  partial (<50% completeness) recoveries under high mutation levels.
when_to_use_negative:
- Input is raw fastq only and no consensus genome or depth output has been produced
  by any pipeline — assembly/consensus calling must happen first
- The goal is de novo discovery of novel viruses rather than quantifying recovery
  of known references relative to a reference database
- The sample contains large DNA viruses with extensive repetitive regions (e.g. mpox),
  where ViMOP output should only be treated as draft genomes and completeness metrics
  may be inaccurate
edam_operation: http://edamontology.org/operation_3434
edam_topics:
- http://edamontology.org/topic_0781
- http://edamontology.org/topic_0196
tools:
- name: ViMOP
  role: Pipeline producing reference-guided consensus genomes and depth outputs from
    untargeted nanopore reads; its consensus FASTA and depth files are the inputs
    to completeness computation
  repo: https://github.com/opr-group-bnitm/vimop
- name: Nextflow
  role: Execution framework for running the ViMOP pipeline with the Docker profile
    on simulated fastq directories
  repo: https://www.nextflow.io/
- name: Docker
  role: Default container profile used to run ViMOP reproducibly during benchmarking
  repo: https://www.docker.com/
- name: INSaFLU-TELEVIR
  role: Comparator pipeline run on identical simulated read sets whose consensus outputs
    are scored with the same completeness metric
- name: Centrifuge
  role: Classifies reads in the ViMOP pipeline; reads classified non-viral with score
    at least 150 are removed before assembly, affecting downstream recovery
- name: Canu
  role: De novo contig assembler (with iterative re-assembly) whose contigs are BLASTed
    against the virus reference database to pick reference genomes for consensus calling
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaf687
    title: 'ViMOP: a user-friendly and field-applicable pipeline for untargeted viral
      genome nanopore sequencing'
  source_content_hash: sha256:b4d356958b28939e86b996d28eb90dd66b779baf68fa4cdcd85243d3b5795f25
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/opr-group-bnitm/vimop
  tools:
  - Canu
  - Centrifuge
  - Docker
  - INSaFLU-TELEVIR
  - Nextflow
  - ViMOP
  edam_operation: http://edamontology.org/operation_3434
  edam_topics:
  - http://edamontology.org/topic_0781
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# viral-genome-completeness-quantification

## Summary

Quantify viral genome recovery and completeness (fraction of the reference covered by a consensus genome) from pipeline outputs such as ViMOP consensus FASTA and depth files, enabling comparison of recovery across pipelines (e.g. ViMOP vs INSaFLU-TELEVIR) and explicit flagging of partial genomes below 50% completeness.

## When to use

Use when benchmarking nanopore virus genome pipelines on simulated or real reads and you need per-sample genome recovery metrics, especially to detect partial (<50% completeness) recoveries under high mutation levels.

## When NOT to use

- Input is raw fastq only and no consensus genome or depth output has been produced by any pipeline — assembly/consensus calling must happen first
- The goal is de novo discovery of novel viruses rather than quantifying recovery of known references relative to a reference database
- The sample contains large DNA viruses with extensive repetitive regions (e.g. mpox), where ViMOP output should only be treated as draft genomes and completeness metrics may be inaccurate

## Inputs

- Consensus genome FASTA files per pipeline and sample (e.g. ViMOP reference-guided consensus genomes)
- Per-base depth output files for each consensus genome
- Virus reference genome sequences (ViMOP virus reference database, Zenodo 17652512)
- Simulated nanopore read fastq sets generated at high mutation levels (scripts in Zenodo 17632662)

## Outputs

- Per-sample genome recovery and completeness values (fraction of reference covered)
- Summary comparison table CSV (e.g. benchmark_high_mutation_comparison.csv) flagging partial genomes with completeness <50%

## How to apply

Run each pipeline (e.g. ViMOP via Nextflow with the Docker profile, and the comparator INSaFLU-TELEVIR) on the identical simulated fastq read sets generated at high mutation levels. For each pipeline and sample, take the reference-guided consensus genome FASTA produced by the pipeline (ViMOP assembles de novo contigs, BLASTs them against the virus reference database Zenodo 17652512, and calls reference-guided consensus genomes) together with its depth output. Compute completeness as the fraction of the reference genome covered by the consensus. Tabulate recovery at the high mutation level and explicitly flag partial genomes with completeness below 50% that one pipeline recovers but the comparator does not, writing the summary to a CSV such as benchmark_high_mutation_comparison.csv.

## Related tools

- **ViMOP** (Pipeline producing reference-guided consensus genomes and depth outputs from untargeted nanopore reads; its consensus FASTA and depth files are the inputs to completeness computation) — https://github.com/opr-group-bnitm/vimop
- **Nextflow** (Execution framework for running the ViMOP pipeline with the Docker profile on simulated fastq directories) — https://www.nextflow.io/
- **Docker** (Default container profile used to run ViMOP reproducibly during benchmarking) — https://www.docker.com/
- **INSaFLU-TELEVIR** (Comparator pipeline run on identical simulated read sets whose consensus outputs are scored with the same completeness metric)
- **Centrifuge** (Classifies reads in the ViMOP pipeline; reads classified non-viral with score at least 150 are removed before assembly, affecting downstream recovery)
- **Canu** (De novo contig assembler (with iterative re-assembly) whose contigs are BLASTed against the virus reference database to pick reference genomes for consensus calling)

## Examples

```
nextflow run opr-group-bnitm/vimop --fastq sim_high_mutation_fastq/ -profile docker --centrifuge_filter_min_score 150
```

## Evaluation signals

- Completeness per sample is computable as a fraction of reference covered and lies in [0,1]; recovery should be higher at high mutation levels for ViMOP than for INSaFLU-TELEVIR
- Samples with computed completeness below 0.5 are explicitly flagged as partial genomes in the summary table
- Non-zero recovery of partial (<50% completeness) genomes is observed for ViMOP where INSaFLU-TELEVIR recovers none, matching the expected benchmark outcome
- The output CSV (benchmark_high_mutation_comparison.csv) contains one row per pipeline/sample with recovery and completeness metrics derived from consensus FASTA and depth files
- Consensus-to-reference identity checks confirm consensus genomes map to the highest-scoring BLAST hit in the virus reference database (Zenodo 17652512)

## Limitations

- Centrifuge classification has limited accuracy, so some reads may be wrongly removed before assembly, lowering apparent recovery
- For large DNA viruses with extensive repetitive regions (e.g. mpox), assemblies may contain inaccuracies and should be regarded as draft genomes
- Results depend on the reference database: contigs are matched to the highest-scoring BLAST hit, and partial or unmatched contigs are handled separately
- Benchmark code and simulated read sets are distributed via specific Zenodo records (17633185 and 17632662); reproducing requires those exact resources

## Evidence

- [other] fraction of the reference covered by the consensus: [evidence span withheld — anchor status: not_found]
- [abstract] assembles consensus genomes: "The software assembles de novo contigs, matches contigs to known viral references and uses them to assemble consensus genomes."
- [readme] centrifuge_filter_min_score 150: "use `--centrifuge_filter_min_score 150` to set the minimum level of confidence (e.g. 150 in our example)."
- [readme] reference-guided consensus: "The pipeline automatically finds well fitting virus genomes and uses them as references to build reference based consensus genomes."
- [readme] draft genomes for repetitive DNA viruses: "for large DNA viruses with extensive repetitive regions (e.g., mpox), assemblies may contain inaccuracies and should be regarded as draft genomes."
