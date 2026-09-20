---
name: nanopore-read-processing
description: Use when you have untargeted metagenomic nanopore FASTQ data (optionally
  barcode-demultiplexed into barcode01, barcode02, ... subdirectories) from MinION
  sequencing of clinical or animal samples and need to detect known viral species
  and reconstruct their consensus genomes.
when_to_use_negative:
- The input is already assembled contigs or consensus genomes rather than raw nanopore
  FASTQ reads — Centrifuge filtering and Canu assembly operate on reads, not contigs
- The target is de novo discovery of novel or divergent viruses with no close reference
  — ViMOP relies on BLAST matches to known viral references in ViMOP_DB
- The organism is a large DNA virus with extensive repetitive regions (e.g. mpox)
  where only inaccurate draft genomes are expected
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: ViMOP
  role: pipeline that orchestrates trimming, Centrifuge filtering, contamination removal,
    Canu assembly, iterative re-assembly, BLAST reference identification and consensus
    generation
  repo: https://github.com/opr-group-bnitm/vimop
- name: Nextflow
  role: workflow engine to run the whole pipeline with a single command (docker, conda,
    mamba or apptainer profiles)
  repo: https://www.nextflow.io/
- name: Docker
  role: default containerization profile used to execute the pipeline steps reproducibly
  repo: https://www.docker.com/
- name: EPI2ME Desktop
  role: optional graphical ONT interface to launch the same Nextflow pipeline
  repo: https://nanoporetech.com/software/other/epi2me-desktop-application
- name: medaka
  role: neural-network consensus sequence and variant calling from a pileup of nanopore
    reads against the identified reference genome
  repo: https://github.com/nanoporetech/medaka
- name: seqtk
  role: lightweight FASTA/FASTQ manipulation tool used in nanopore read processing
  repo: https://github.com/lh3/seqtk
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btaf687
    title: 'ViMOP: a user-friendly and field-applicable pipeline for untargeted viral
      genome nanopore sequencing'
  source_content_hash: sha256:77d9a4d7d9f7840bcbfcee264b294c575ba956159065032f9497a4b661dc6777
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/opr-group-bnitm/vimop
  tools:
  - Docker
  - EPI2ME Desktop
  - EPI2ME desktop
  - Nextflow
  - ViMOP
  - medaka
  - seqtk
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# nanopore-read-processing

## Summary

End-to-end processing of untargeted Oxford Nanopore FASTQ reads for viral genome reconstruction, as implemented in ViMOP: read-end trimming (adapter/primer removal), Centrifuge-based taxonomic classification and non-viral read filtering, host/contaminant read removal by mapping, de novo assembly with Canu, iterative re-assembly, BLAST reference identification, and reference-guided consensus generation (samtools/medaka).

## When to use

Use when you have untargeted metagenomic nanopore FASTQ data (optionally barcode-demultiplexed into barcode01, barcode02, ... subdirectories) from MinION sequencing of clinical or animal samples and need to detect known viral species and reconstruct their consensus genomes.

## When NOT to use

- The input is already assembled contigs or consensus genomes rather than raw nanopore FASTQ reads — Centrifuge filtering and Canu assembly operate on reads, not contigs
- The target is de novo discovery of novel or divergent viruses with no close reference — ViMOP relies on BLAST matches to known viral references in ViMOP_DB
- The organism is a large DNA virus with extensive repetitive regions (e.g. mpox) where only inaccurate draft genomes are expected

## Inputs

- directory of untargeted Oxford Nanopore FASTQ read files (optionally with barcode01, barcode02, ... subdirectories)
- ViMOP_DB virus reference database (placed by default in ~/ViMOP_DB)
- contamination.yaml database config with reference sets (human, mouse, reagent, mastomys, etc.)
- optional target list of virus species/groups (e.g. MARV, EBOV, FILO)

## Outputs

- consensus virus genomes (per reference/sample)
- alignment .bam files of reads mapped to identified references
- per-contig BLAST hits against the virus reference database
- per-reference depth and variant (VCF) files
- Centrifuge read classification with Krona plot in the HTML report

## How to apply

Start by trimming a fixed number of bases from both read ends with --trim_length to remove adapters and primers. Classify reads with Centrifuge and drop reads classified as non-viral with a confidence score of at least --centrifuge_filter_min_score (default example 150), then remove host and contaminant reads by mapping against the contamination.yaml reference sets (e.g. --contamination_filters "reagent,mouse"; human is the default), optionally enriching for target viruses with --targets (e.g. "MARV,EBOV,FILO"). Run de novo assembly (Canu) per filtered read set plus the unfiltered set, followed by the iterative re-assembly procedure (controlled by --reassemble_max_iter; set 0 to deactivate). BLAST each contig against the ViMOP_DB virus reference database, take the highest-scoring hit as reference, map reads to it, and call consensus genomes with samtools/medaka, reporting depth and variant files. Execute the whole workflow with a single Nextflow run using the docker profile (or conda/mamba/apptainer profiles on Linux), or via EPI2ME Desktop; results are summarized in an HTML report with a Krona plot of Centrifuge classifications.

## Related tools

- **ViMOP** (pipeline that orchestrates trimming, Centrifuge filtering, contamination removal, Canu assembly, iterative re-assembly, BLAST reference identification and consensus generation) — https://github.com/opr-group-bnitm/vimop
- **Nextflow** (workflow engine to run the whole pipeline with a single command (docker, conda, mamba or apptainer profiles)) — https://www.nextflow.io/
- **Docker** (default containerization profile used to execute the pipeline steps reproducibly) — https://www.docker.com/
- **EPI2ME Desktop** (optional graphical ONT interface to launch the same Nextflow pipeline) — https://nanoporetech.com/software/other/epi2me-desktop-application
- **medaka** (neural-network consensus sequence and variant calling from a pileup of nanopore reads against the identified reference genome) — https://github.com/nanoporetech/medaka
- **seqtk** (lightweight FASTA/FASTQ manipulation tool used in nanopore read processing) — https://github.com/lh3/seqtk

## Examples

```
nextflow run opr-group-bnitm/vimop --fastq <fastq_dir> --trim_length <bases> --targets "MARV,EBOV,FILO" -profile docker
```

## Evaluation signals

- Centrifuge classification summary and Krona plot are produced in the HTML report; --centrifuge_do_classify false only when speed is needed, so a missing classification block is a red flag
- Reads classified as non-viral with a Centrifuge score at or above --centrifuge_filter_min_score (e.g. 150) must be absent from the assembly input read sets
- Each reconstructed genome must have a BLAST match in the ViMOP reference database used as its reference genome; check per-reference results (contigs identified by BLAST, consensus from samtools/medaka, depth and variant files)
- Iterative re-assembly converges within --reassemble_max_iter iterations; output should include one assembly per filtered read set plus the unfiltered set
- Benchmark expectation: genome recovery metrics should be comparable to INSaFLU-TELEVIR on reconstructable genomes (per Section 2.11 comparison table)

## Limitations

- Large DNA viruses with extensive repetitive regions (e.g. mpox) yield assemblies with inaccuracies that should be treated as draft genomes; outputs (e.g. alignment .bam files) should be reviewed manually
- Centrifuge classification is limited in accuracy, so the non-viral read filter may also remove some false-positive viral reads
- Resource-intensive steps such as Canu need considerable RAM/CPU; recommended at least 30 GB RAM and 16 CPUs, and lowering --min_cpus/--min_ram may fail or run much longer
- Conda profile did not work with 'nextflow run' prior to ViMOP 1.0.4, and a 'Cannot invoke method optional() on null object' bug occurs with Nextflow 25.10.2 (fixed in 1.0.4)

## Evidence

- [methods] trimming+classification: "At the beginning the pipeline trims the ends of the reads to remove adapter sequences and primers."
- [readme] centrifuge filter: "The pipeline will remove all reads that are classified to anything that is not viral with at least this score."
- [abstract] assembly+consensus: "The software assembles de novo contigs, matches contigs to known viral references and uses them to assemble consensus genomes."
- [readme] iterative reassembly: "After the initial assembly, an iterative re-assembly procedure is run (unless deactivated with `--reassemble_max_iter 0`)."
- [abstract] run interface: "Executed with a single Nextflow command or via the EPI2ME Desktop interface (ONT), results are summarized in an HTML report."
