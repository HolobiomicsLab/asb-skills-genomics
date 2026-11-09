---
name: invocation-and-execution-of-long-read-aligners
description: Use when mapping simulated or real long noisy reads (PacBio CCS/CLR,
  Oxford Nanopore) to a reference genome FASTA and SAM-format alignments are needed
  as input for downstream accuracy or variant-calling comparisons.
when_to_use_negative:
- Input reads are short Illumina reads — the compared aligners and their long-read
  presets are designed for long noisy PacBio/Oxford Nanopore reads
- The goal is de novo assembly or read clustering rather than mapping reads to a reference
  genome
- The reference genome is not available as FASTA and only pre-aligned BAM/CRAM data
  exists
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: invMap
  role: Primary aligner; compiles via make and maps long noisy reads to a reference
    genome, emitting SAM output and detecting inversion SVs via a chaining-based scoring
    method
  repo: https://github.com/zhang134/invMap.git
- name: minimap2
  role: Competing long-read mapper run with its long-read preset to benchmark location
    accuracy
- name: Winnowmap2
  role: Competing long-read mapper run with its long-read preset to benchmark location
    accuracy
- name: lra
  role: Competing long-read mapper run with its long-read preset to benchmark location
    accuracy
- name: NGMLR
  role: Competing long-read mapper run with its long-read preset to benchmark location
    accuracy
- name: PBSIM
  role: Simulates PacBio CCS and CLR read sets used as aligner inputs
- name: PBSIM2
  role: Simulates ONT read sets used as aligner inputs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad726
    title: 'invMap: a sensitive mapping tool for long noisy reads with inversion structural
      variants'
  source_content_hash: sha256:4a1d997ca4947fb5b3769449d173f575d8407720f45dd45e6b4fec1fd65e01aa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhang134/invMap.git
  tools:
  - NGMLR
  - PBSIM
  - PBSIM2
  - Winnowmap2
  - invMap
  - lra
  - minimap2
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# invocation-and-execution-of-long-read-aligners

## Summary

Compile and run long-read alignment tools (invMap, minimap2, Winnowmap2, lra, NGMLR) with their long-read presets to map simulated PacBio CCS/CLR and ONT reads back to reference genomes, producing SAM alignments used for location-accuracy benchmarking.

## When to use

Use when mapping simulated or real long noisy reads (PacBio CCS/CLR, Oxford Nanopore) to a reference genome FASTA and SAM-format alignments are needed as input for downstream accuracy or variant-calling comparisons.

## When NOT to use

- Input reads are short Illumina reads — the compared aligners and their long-read presets are designed for long noisy PacBio/Oxford Nanopore reads
- The goal is de novo assembly or read clustering rather than mapping reads to a reference genome
- The reference genome is not available as FASTA and only pre-aligned BAM/CRAM data exists

## Inputs

- Reference genome FASTA files (e.g. Neisseria meningitidis, Abortiporus biennis, GRCh37 chromosome 1)
- Simulated long reads in FASTA/FASTQ format (PacBio CCS/CLR from PBSIM, ONT from PBSIM2)
- Real sequencing datasets (e.g. NA12878 human genome dataset)

## Outputs

- SAM-format alignment files (e.g. invMap_aligned.sam) per mapper, per genome, per read platform
- Per-mapper, per-genome, per-platform location-accuracy values aggregated into location_accuracy_s4.csv

## How to apply

Compile the aligner from source with the make command, then run the help interface ('./invmap') to confirm options before executing. Map reads to the reference with invMap by redirecting SAM output ('./invMap genome.fa reads.fa > invMap_aligned.sam'), and run minimap2, Winnowmap2, lra, and NGMLR using their long-read presets so all mappers are compared under equivalent settings. Apply this to each simulated read set (PacBio CCS and CLR from PBSIM, ONT from PBSIM2) generated from each reference genome (Neisseria meningitidis, Abortiporus biennis, GRCh37 chromosome 1) at the paper's coverages. Parse the resulting SAM alignments per mapper, per genome, and per platform to compute location accuracy — the fraction of reads whose reported aligned region matches the true simulated origin region.

## Related tools

- **invMap** (Primary aligner; compiles via make and maps long noisy reads to a reference genome, emitting SAM output and detecting inversion SVs via a chaining-based scoring method) — https://github.com/zhang134/invMap.git
- **minimap2** (Competing long-read mapper run with its long-read preset to benchmark location accuracy)
- **Winnowmap2** (Competing long-read mapper run with its long-read preset to benchmark location accuracy)
- **lra** (Competing long-read mapper run with its long-read preset to benchmark location accuracy)
- **NGMLR** (Competing long-read mapper run with its long-read preset to benchmark location accuracy)
- **PBSIM** (Simulates PacBio CCS and CLR read sets used as aligner inputs)
- **PBSIM2** (Simulates ONT read sets used as aligner inputs)

## Examples

```
./invMap genome.fa reads.fa > invMap_aligned.sam
```

## Evaluation signals

- invMap attains >99% location accuracy on PacBio CCS and CLR simulated read sets
- invMap attains >98% location accuracy on ONT simulated read sets
- invMap location accuracy outperforms minimap2, Winnowmap2, lra, and NGMLR, matching Supplementary Table S4 / Fig. 4
- Each aligner produces a parseable SAM file whose aligned regions can be compared against the true simulated origin regions
- Compilation via make succeeds and './invmap' prints help information and options explanation

## Limitations

- invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes
- invMap requires approximately 800 M of available disk space on the drive or partition
- Linux is recommended; Windows 7/10 are supported but the workflow steps assume a Unix shell for redirection of SAM output

## Evidence

- [intro] compile codes using make command: "Step 1: compile codes using make command"
- [intro] invMap SAM invocation: "./invMap genome.fa reads.fa > invMap_aligned.sam"
- [intro] memory requirement: "invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes."
- [abstract] location-accuracy finding: [evidence span withheld — anchor status: not_found]
- [readme] README availability: "Input file requires fasta or fastq format"
