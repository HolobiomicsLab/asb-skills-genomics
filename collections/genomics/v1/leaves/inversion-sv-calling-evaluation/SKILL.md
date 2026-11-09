---
name: inversion-sv-calling-evaluation
description: Use when evaluating or reproducing long-read SV-calling accuracy under
  varying coverage, especially comparing inversion detection against indel detection
  with simulated noisy reads and a ground-truth SV set.
when_to_use_negative:
- The goal is real-sample variant discovery rather than controlled benchmarking against
  a simulated truth set
- Reads are short reads, since this workflow relies on long PacBio/Nanopore reads
  that span SV breakpoints
- Reference genome has no known or simulatable SV landscape (e.g. no SURVIVOR-simulable
  chr1 reference) to serve as ground truth
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: invMap
  role: Long-read mapper used to align simulated reads and locate/call inversion SVs
    against the chr1 reference; benchmarked for recall, precision, and F1
  repo: https://github.com/zhang134/invMap.git
- name: PacBio
  role: Error-profile model used to simulate long noisy reads from the mutated chr1
    at four coverage levels
- name: Oxford Nanopore
  role: Error-profile model used to simulate long noisy reads from the mutated chr1
    at four coverage levels
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad726
    title: 'invMap: a sensitive mapping tool for long noisy reads with inversion structural
      variants'
  source_content_hash: sha256:196f1dd6b7b1daa47be4cb1b86146d3e180501961da7ea5b251e1c9e20cd9845
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhang134/invMap.git
  tools:
  - Oxford Nanopore
  - PacBio
  - invMap
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# inversion-sv-calling-evaluation

## Summary

Benchmark how sequencing coverage (10x-40x) affects long-read structural variant calling accuracy for deletions/insertions and inversions, using SURVIVOR-simulated chr1 read sets mapped and called by invMap and competing mappers against a truth SV set.

## When to use

Use when evaluating or reproducing long-read SV-calling accuracy under varying coverage, especially comparing inversion detection against indel detection with simulated noisy reads and a ground-truth SV set.

## When NOT to use

- The goal is real-sample variant discovery rather than controlled benchmarking against a simulated truth set
- Reads are short reads, since this workflow relies on long PacBio/Nanopore reads that span SV breakpoints
- Reference genome has no known or simulatable SV landscape (e.g. no SURVIVOR-simulable chr1 reference) to serve as ground truth

## Inputs

- chr1 reference sequence in FASTA format
- Simulated long noisy reads (FASTA/FASTQ) with PacBio- or Oxford Nanopore-style error profiles at 10x, 20x, 30x, and 40x coverage
- SURVIVOR ground-truth SV set (600 indels + 300 inversions embedded in mutated chr1)

## Outputs

- SAM alignment files per mapper and coverage level (e.g. invMap_aligned.sam)
- Called SV sets per tool-coverage pair
- Results table of recall, precision, and F1 for indels and inversions per coverage level

## How to apply

Use SURVIVOR's simulateSV module to embed 900 SVs (600 indels and 300 inversions) into the chr1 reference, producing a mutated reference plus a ground-truth SV set. Simulate long noisy reads from the mutated chr1 with PacBio- and Oxford Nanopore-style error profiles at 10x, 20x, 30x, and 40x coverage. Map every read set back to the chr1 reference with invMap (installed via make, run as ./invMap genome.fa reads.fa > invMap_aligned.sam) and each competing mapping tool, then call SVs from each tool's alignments against the mutated reference. Compare called SVs to the SURVIVOR truth set per tool-coverage pair, computing recall, precision, and F1 separately for indels and inversions. Expect indel calling to be reliable from 30x minimum coverage and inversion calling to require 40x, with inversion accuracy significantly lower than indel accuracy at every coverage level; invMap should be the most accurate for locating and calling inversions.

## Related tools

- **invMap** (Long-read mapper used to align simulated reads and locate/call inversion SVs against the chr1 reference; benchmarked for recall, precision, and F1) — https://github.com/zhang134/invMap.git
- **PacBio** (Error-profile model used to simulate long noisy reads from the mutated chr1 at four coverage levels)
- **Oxford Nanopore** (Error-profile model used to simulate long noisy reads from the mutated chr1 at four coverage levels)

## Examples

```
./invMap genome.fa reads.fa > invMap_aligned.sam
```

## Evaluation signals

- Indel (deletion/insertion) calling recall/precision/F1 reaches reliable levels at 30x coverage and above
- Inversion calling accuracy does not reach its best levels until 40x coverage
- Inversion call accuracy is significantly lower than deletion/insertion accuracy at every coverage level tested
- invMap yields the highest accuracy for locating aligned regions and calling inversions compared with competing mappers on the same simulated read sets
- Each mapper's alignment output is valid SAM (e.g. invMap_aligned.sam) and called SVs match the SURVIVOR truth set format for comparison

## Limitations

- Inversion calling accuracy is significantly lower than deletion and insertion calling at every coverage level, and requires 40x coverage to perform best
- invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes and approximately 800 M of available disk space
- Benchmark conclusions are grounded in simulated chr1 reads (900 SVs); real-data behavior was only illustrated on the NA12878 dataset where invMap finds more candidate inversion calls

## Evidence

- [results] Across all coverages, SV call accuracy improves with coverage, but the minimum required coverage is 30x for deletions and insertions and 40x for inversions; inversion calling accuracy is significantly lower than deletion and insertion at every coverage level.: [evidence span withheld — anchor status: not_found]
- [methods] Simulation workflow description from the source card.: [evidence span withheld — anchor status: not_found]
- [abstract] invMap accuracy claim from abstract.: "experimental results demonstrate that invMap is more accurate to locate aligned regions and call SVs for inversions than the competing methods"
- [readme] README running example.: "./invMap genome.fa reads.fa > invMap_aligned.sam"
- [readme] README memory requirement.: "invMap requires a minimum of
16G of RAM for mapping sequence datasets to large reference genomes."
