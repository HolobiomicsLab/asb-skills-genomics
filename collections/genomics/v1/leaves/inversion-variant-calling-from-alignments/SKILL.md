---
name: inversion-variant-calling-from-alignments
description: Use when long noisy reads (PacBio CLR or Oxford Nanopore) must be mapped
  to a reference and inversion SVs called, or when benchmarking invMap sensitivity/precision
  against Winnowmap2, minimap2, NGMLR, and lra.
when_to_use_negative:
- Reads are short (Illumina-style) — the approach depends on longer PacBio/ONT reads
  that more frequently span SV breakpoints.
- The structural variants of interest are not inversions (e.g. pure deletions/duplications)
  — invMap is specifically designed to discover potential inversions from remaining
  anchors.
- Available RAM is below the ~16G minimum required for mapping sequence datasets to
  large reference genomes.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: invMap
  role: Inversion-aware long-read mapper producing SAM alignments from which inversion
    SVs are called; the tool under benchmark
  repo: https://github.com/zhang134/invMap.git
- name: SURVIVOR
  role: simulateSV module used to simulate 100 inversion structural variants in GRCh37
    chr1 to create the mutated reference ground truth
- name: PBSIM2
  role: Simulates 40x coverage PacBio CLR long reads against the inverted chr1 so
    that 3738 reads span inversion breakpoints
- name: Winnowmap2
  role: Competing long-read mapper in the sensitivity/precision benchmark
- name: minimap2
  role: Competing long-read mapper in the sensitivity/precision benchmark
- name: NGMLR
  role: Competing long-read mapper in the sensitivity/precision benchmark
- name: lra
  role: Competing long-read mapper in the sensitivity/precision benchmark
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad726
    title: 'invMap: a sensitive mapping tool for long noisy reads with inversion structural
      variants'
  source_content_hash: sha256:97e2ef4d6202f784841d1f7a14cf8c26ca64fa8b650f1be2097f00b157227a77
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhang134/invMap.git
  tools:
  - NGMLR
  - PBSIM2
  - SURVIVOR
  - Winnowmap2
  - invMap
  - lra
  - minimap2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# inversion-variant-calling-from-alignments

## Summary

Benchmark and apply invMap, a long-read mapping algorithm with inversion-aware chaining and anchor checking, to detect inversion structural variants from PacBio CLR/ONT alignments against a reference genome.

## When to use

Use when long noisy reads (PacBio CLR or Oxford Nanopore) must be mapped to a reference and inversion SVs called, or when benchmarking invMap sensitivity/precision against Winnowmap2, minimap2, NGMLR, and lra.

## When NOT to use

- Reads are short (Illumina-style) — the approach depends on longer PacBio/ONT reads that more frequently span SV breakpoints.
- The structural variants of interest are not inversions (e.g. pure deletions/duplications) — invMap is specifically designed to discover potential inversions from remaining anchors.
- Available RAM is below the ~16G minimum required for mapping sequence datasets to large reference genomes.

## Inputs

- Reference genome FASTA (e.g. GRCh37 chr1)
- Long-read FASTA/FASTQ dataset (PacBio CLR or Oxford Nanopore)
- Simulated mutated reference with SURVIVOR-generated inversions
- Ground-truth inversion SV list from SURVIVOR simulateSV

## Outputs

- SAM alignment file per mapper (invMap_aligned.sam)
- Inversion SV calls per mapper classified as recovered vs predicted
- Benchmark table of predicted, recovered, sensitivity, and precision per mapper (inversion_benchmark_table.csv)

## How to apply

Simulate ground-truth inversions in the GRCh37 chromosome 1 reference using SURVIVOR's simulateSV module, then generate 40x coverage PacBio CLR reads with PBSIM2 against the mutated chr1 so reads span inversion breakpoints. Map the breakpoint-spanning reads back to the original (non-mutated) reference with each mapper, invoking invMap as './invMap genome.fa reads.fa > invMap_aligned.sam' after compiling with make. invMap first locates each read's aligned region with a scoring method designed for chaining, then inspects remaining anchors within that region to discover potential inversions. Call inversion SVs from each mapper's SAM alignments and classify predictions as recovered true positives against the 100 simulated ground-truth inversions. Compute sensitivity as recovered/predicted (e.g. invMap 3483/3606 = 92.06%) and precision (invMap 96.58%), assembling the comparison table; reproduced numbers must match Table 1 within stochastic-simulation tolerance.

## Related tools

- **invMap** (Inversion-aware long-read mapper producing SAM alignments from which inversion SVs are called; the tool under benchmark) — https://github.com/zhang134/invMap.git
- **SURVIVOR** (simulateSV module used to simulate 100 inversion structural variants in GRCh37 chr1 to create the mutated reference ground truth)
- **PBSIM2** (Simulates 40x coverage PacBio CLR long reads against the inverted chr1 so that 3738 reads span inversion breakpoints)
- **Winnowmap2** (Competing long-read mapper in the sensitivity/precision benchmark)
- **minimap2** (Competing long-read mapper in the sensitivity/precision benchmark)
- **NGMLR** (Competing long-read mapper in the sensitivity/precision benchmark)
- **lra** (Competing long-read mapper in the sensitivity/precision benchmark)

## Examples

```
./invMap genome.fa reads.fa > invMap_aligned.sam
```

## Evaluation signals

- Reproduced invMap row reports 3606 predicted SVs, 3483 recovered SVs, 92.06% sensitivity, and 96.58% precision, matching Table 1 within stochastic-simulation tolerance.
- 3738 of the simulated reads span inversion breakpoints and are the mapping input.
- 100 simulated ground-truth inversions are recovered against the non-mutated chr1 reference.
- Output is valid SAM format (invMap_aligned.sam) directly from the README running example.
- invMap's sensitivity and precision exceed those of Winnowmap2, minimap2, NGMLR, and lra on the same simulated dataset.

## Limitations

- invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes.
- Simulation-based validation carries stochastic tolerance; exact recovered counts may vary between runs.
- Real-data validation (NA12878) shows more candidate inversion calls but lacks the same ground-truth precision guarantee as the simulated benchmark.

## Evidence

- [abstract] Longer reads produced by PacBio or Oxford Nanopore sequencers could more frequently span the breakpoints of structural variations (SVs) than shorter reads: "Longer reads produced by PacBio or Oxford Nanopore sequencers could more frequently span the breakpoints of structural variations (SVs) than shorter reads"
- [abstract] invMap algorithm chaining: "invMap first locates the aligned region with a specifically designed scoring method for chaining, then checks the remaining anchors in the aligned region to discover potential inversions"
- [intro] invMap RAM requirement: "invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes."
- [readme] README running example: "./invMap genome.fa reads.fa > invMap_aligned.sam"
