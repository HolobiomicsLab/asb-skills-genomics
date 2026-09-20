---
name: cross-pipeline-genome-recovery-comparison
description: Use when comparing viral genome reconstruction pipelines on identical
  (e.g., simulated, high-mutation) read sets and you must quantify which tool recovers
  more reference genome, including partial (<50% completeness) genomes.
when_to_use_negative:
- The goal is de novo discovery of novel viruses rather than benchmarking detection/reconstruction
  of known virus species against a reference database
- Input data are already consensus genomes or assembly artifacts rather than raw/simulated
  FASTQ read sets requiring full pipeline runs
- Large DNA viruses with extensive repetitive regions (e.g., mpox) are targeted, where
  ViMOP assemblies may be inaccurate draft genomes and comparisons may mislead
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0781
- http://edamontology.org/topic_3673
tools:
- name: ViMOP
  role: Primary pipeline under evaluation; runs trimming, Centrifuge filtering (min
    score 150), de novo assembly, BLAST reference identification, and reference-guided
    consensus calling
  repo: https://github.com/opr-group-bnitm/vimop
- name: Nextflow
  role: Workflow engine used to execute ViMOP with the Docker profile on the simulated
    fastq directories
  repo: https://www.nextflow.io/
- name: Docker
  role: Default execution profile for ViMOP runs during benchmarking
  repo: https://www.docker.com/
- name: INSaFLU-TELEVIR
  role: Comparator pipeline run on the identical simulated read sets to compute relative
    genome recovery
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaf687
    title: 'ViMOP: a user-friendly and field-applicable pipeline for untargeted viral
      genome nanopore sequencing'
  source_content_hash: sha256:f310795a815fa466ec6412bb5bd0267c9ee3c779f35fb49ecb80089cab405236
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/opr-group-bnitm/vimop
  tools:
  - Docker
  - INSaFLU-TELEVIR
  - Nextflow
  - ViMOP
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0781
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# cross-pipeline-genome-recovery-comparison

## Summary

Benchmarks ViMOP against comparator pipelines such as INSaFLU-TELEVIR on simulated nanopore read sets by computing per-sample genome recovery and completeness from consensus fasta and depth outputs, with emphasis on robustness at high mutation levels and recovery of partial genomes below 50% completeness.

## When to use

Use when comparing viral genome reconstruction pipelines on identical (e.g., simulated, high-mutation) read sets and you must quantify which tool recovers more reference genome, including partial (<50% completeness) genomes.

## When NOT to use

- The goal is de novo discovery of novel viruses rather than benchmarking detection/reconstruction of known virus species against a reference database
- Input data are already consensus genomes or assembly artifacts rather than raw/simulated FASTQ read sets requiring full pipeline runs
- Large DNA viruses with extensive repetitive regions (e.g., mpox) are targeted, where ViMOP assemblies may be inaccurate draft genomes and comparisons may mislead

## Inputs

- Simulated nanopore FASTQ read sets generated at high mutation levels (via Zenodo 17632662 scripts)
- Virus reference database (Zenodo record 17652512)
- Benchmarking code (Zenodo record 17633185)
- Per-pipeline consensus FASTA and per-reference depth outputs

## Outputs

- Per-pipeline, per-sample genome recovery and completeness metrics (fraction of reference covered by consensus)
- Flagged list of partial genomes with completeness <50% recovered by ViMOP but not by the comparator
- Summary table benchmark_high_mutation_comparison.csv

## How to apply

Download the benchmarking code from Zenodo record 17633185 and simulated read sets generated at high mutation levels via scripts in Zenodo record 17632662. Run ViMOP with Nextflow (Docker profile) on the fastq directories — reads are trimmed, classified with Centrifuge (minimum score 150 for non-viral read removal), host/contaminant reads are removed, contigs are assembled de novo with Canu including iterative re-assembly, BLASTed against the virus reference database (Zenodo 17652512), and reference-guided consensus genomes are called. Run the comparator (INSaFLU-TELEVIR) on the identical simulated read sets. For each pipeline and sample, compute genome recovery and completeness as the fraction of the reference covered by the consensus, derived from the consensus fasta and depth outputs. Tabulate recovery, explicitly flagging partial genomes with completeness below 50% that one pipeline recovers but the other does not, and write the summary to a CSV (e.g., benchmark_high_mutation_comparison.csv).

## Related tools

- **ViMOP** (Primary pipeline under evaluation; runs trimming, Centrifuge filtering (min score 150), de novo assembly, BLAST reference identification, and reference-guided consensus calling) — https://github.com/opr-group-bnitm/vimop
- **Nextflow** (Workflow engine used to execute ViMOP with the Docker profile on the simulated fastq directories) — https://www.nextflow.io/
- **Docker** (Default execution profile for ViMOP runs during benchmarking) — https://www.docker.com/
- **INSaFLU-TELEVIR** (Comparator pipeline run on the identical simulated read sets to compute relative genome recovery)

## Examples

```
nextflow run opr-group-bnitm/vimop --fastq sim_reads_high_mutation/ -profile docker --centrifuge_filter_min_score 150
```

## Evaluation signals

- ViMOP attains higher genome recovery than INSaFLU-TELEVIR on the high-mutation simulated read sets
- ViMOP shows non-zero recovery of partial genomes with completeness <50% where INSaFLU-TELEVIR recovers none
- Completeness per sample equals the fraction of the reference covered by the consensus, reproducible from consensus fasta + depth outputs
- Both pipelines were run on identical fastq inputs, so differences are attributable to the pipelines themselves
- Output CSV (benchmark_high_mutation_comparison.csv) exists with one row per pipeline/sample and valid completeness values in [0,1]

## Limitations

- Centrifuge classification is limited in accuracy and may remove some false-positive viral reads during non-viral filtering (min score 150)
- For large DNA viruses with extensive repetitive regions (e.g., mpox), ViMOP assemblies may contain inaccuracies and should be regarded as draft genomes
- ViMOP is intended for small and medium size RNA viruses such as Lassa, Dengue, and Ebola; results outside this scope may differ

## Evidence

- [abstract] ViMOP assembles de novo contigs, matches them to known viral references, and uses them to assemble consensus genomes: "The software assembles de novo contigs, matches contigs to known viral references and uses them to assemble consensus genomes."
- [full_text] On simulated data, ViMOP is more robust at high mutation levels than the compared tools: [evidence span withheld — anchor status: not_found]
- [methods] Centrifuge filtering with score threshold 150: "Deactivate this with `--centrifuge_do_filter false` and use `--centrifuge_filter_min_score 150` to set the minimum level of confidence"
- [methods] Each contig is used for a BLAST search in the virus reference database: "Each contig is used to for a BLAST search in the virus reference database. The highest scoring hit is then used as a reference genome."
- [readme] ViMOP is intended for small and medium size RNA viruses: "This works well for small and medium size RNA viruses such as Lassa, Dengue, Ebola and many others."
