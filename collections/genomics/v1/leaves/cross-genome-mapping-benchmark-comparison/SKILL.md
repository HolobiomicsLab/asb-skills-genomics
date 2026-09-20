---
name: cross-genome-mapping-benchmark-comparison
description: Use when benchmarking a long-read mapper's location accuracy against
  competing aligners on simulated PacBio CCS, PacBio CLR, or ONT reads derived from
  reference genomes of varying sizes.
when_to_use_negative:
- The goal is detecting inversion structural variants or candidate variant calls rather
  than benchmarking mapping location accuracy — the SV-calling and NA12878 real-data
  analyses are separate tasks.
- Reads are short Illumina reads; the benchmark scope is long noisy PacBio CCS/CLR
  and Oxford Nanopore reads.
- Only real sequencing data (e.g. NA12878) is available and no ground-truth simulated
  origin regions exist to compute location accuracy.
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: invMap
  role: Mapper under test; run with './invMap genome.fa reads.fa > invMap_aligned.sam'
    to produce SAM alignments whose location accuracy is benchmarked
  repo: https://github.com/zhang134/invMap.git
- name: PBSIM
  role: Simulates PacBio CCS and CLR reads from each reference genome at the paper's
    coverages
- name: PBSIM2
  role: Simulates ONT reads from each reference genome at the paper's coverages
- name: minimap2
  role: Competing long-read mapper run with its long-read preset for accuracy comparison
- name: Winnowmap2
  role: Competing long-read mapper run with its long-read preset for accuracy comparison
- name: lra
  role: Competing long-read mapper run with its long-read preset for accuracy comparison
- name: NGMLR
  role: Competing long-read mapper run with its long-read preset for accuracy comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad726
    title: 'invMap: a sensitive mapping tool for long noisy reads with inversion structural
      variants'
  source_content_hash: sha256:7bf05460d1e0c939cf919fc021b8cf18928ee4299095e7888fa3f56570cf89ad
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
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# cross-genome-mapping-benchmark-comparison

## Summary

A benchmarking skill for comparing the location accuracy of a long-read mapper (invMap) against minimap2, Winnowmap2, lra, and NGMLR on simulated PacBio CCS/CLR and ONT reads across reference genomes of different sizes, reproducing Supplementary Table S4 / Fig. 4.

## When to use

Use when benchmarking a long-read mapper's location accuracy against competing aligners on simulated PacBio CCS, PacBio CLR, or ONT reads derived from reference genomes of varying sizes.

## When NOT to use

- The goal is detecting inversion structural variants or candidate variant calls rather than benchmarking mapping location accuracy — the SV-calling and NA12878 real-data analyses are separate tasks.
- Reads are short Illumina reads; the benchmark scope is long noisy PacBio CCS/CLR and Oxford Nanopore reads.
- Only real sequencing data (e.g. NA12878) is available and no ground-truth simulated origin regions exist to compute location accuracy.

## Inputs

- Reference genome FASTA files (Neisseria meningitidis, Abortiporus biennis, GRCh37 chromosome 1)
- Simulated PacBio CCS and CLR read sets generated with PBSIM
- Simulated ONT read sets generated with PBSIM2
- SAM alignment files from invMap, minimap2, Winnowmap2, lra, and NGMLR

## Outputs

- Per-mapper, per-genome, per-platform location-accuracy table (location_accuracy_s4.csv)
- SAM alignment files from each mapper

## How to apply

Download the three reference genome FASTA files (Neisseria meningitidis, Abortiporus biennen, and GRCh37 chromosome 1) spanning small, medium, and large genome sizes. Simulate long reads with PBSIM (PacBio CCS and CLR) and PBSIM2 (ONT) at the coverages used in the paper. Map each simulated read set back to its reference with invMap ('./invMap genome.fa reads.fa > invMap_aligned.sam') and with minimap2, Winnowmap2, lra, and NGMLR using their long-read presets. Parse each mapper's SAM output and compute location accuracy — the fraction of reads whose reported aligned region matches the true simulated origin region — then aggregate per-mapper, per-genome, per-platform values into location_accuracy_s4.csv.

## Related tools

- **invMap** (Mapper under test; run with './invMap genome.fa reads.fa > invMap_aligned.sam' to produce SAM alignments whose location accuracy is benchmarked) — https://github.com/zhang134/invMap.git
- **PBSIM** (Simulates PacBio CCS and CLR reads from each reference genome at the paper's coverages)
- **PBSIM2** (Simulates ONT reads from each reference genome at the paper's coverages)
- **minimap2** (Competing long-read mapper run with its long-read preset for accuracy comparison)
- **Winnowmap2** (Competing long-read mapper run with its long-read preset for accuracy comparison)
- **lra** (Competing long-read mapper run with its long-read preset for accuracy comparison)
- **NGMLR** (Competing long-read mapper run with its long-read preset for accuracy comparison)

## Examples

```
./invMap genome.fa reads.fa > invMap_aligned.sam
```

## Evaluation signals

- invMap attains >99% location accuracy on PacBio CCS and CLR read sets, matching Supplementary Table S4 / Fig. 4
- invMap attains >98% location accuracy on ONT read sets
- invMap's location accuracy exceeds minimap2, Winnowmap2, lra, and NGMLR on each genome/platform combination
- location_accuracy_s4.csv contains one value per mapper per genome per platform with fractions in [0,1]
- Each mapper's SAM output parses without error and covers the full simulated read set

## Limitations

- invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes.
- Accuracy claims are based on simulated reads; real-data behavior is only illustrated by NA12878 candidate inversion variant calls, not location-accuracy benchmarks.
- invMap requires approximately 800 M of available disk space on the drive or partition.

## Evidence

- [results] invMap achieves the highest location accuracy of all compared mappers, with accuracy consistently above 99% for PacBio CCS and CLR reads and above 98% for ONT reads.: [evidence span withheld — anchor status: not_found]
- [readme] invMap run command: "./invMap genome.fa reads.fa > invMap_aligned.sam"
- [readme] memory requirement: "invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes."
- [abstract] algorithm description: "invMap first locates the aligned region with a specifically designed scoring method for chaining, then checks the remaining anchors in the aligned region to discover potential inversions"
- [abstract] simulation platforms: "Longer reads produced by PacBio or Oxford Nanopore sequencers could more frequently span the breakpoints of structural variations (SVs) than shorter reads"
