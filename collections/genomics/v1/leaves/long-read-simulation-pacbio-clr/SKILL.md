---
name: long-read-simulation-pacbio-clr
description: 'Use when benchmarking long-read structural-variant mappers against a
  known ground truth: simulate inversion SVs in a reference genome and generate coverage-controlled
  PacBio CLR reads spanning the breakpoints.'
when_to_use_negative:
- Reads are real sequencing data (e.g. the NA12878 human dataset), where there is
  no simulated ground truth — map and call candidate inversions directly instead of
  running the simulation benchmark
- The target SV type or scale differs (e.g. non-inversion SVs, different chromosome
  or coverage), since the 100-inversion / 40x CLR parameters are specific to this
  chr1 benchmark
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: SURVIVOR
  role: simulateSV module used to simulate the 100 inversion structural variants in
    GRCh37 chr1, creating the mutated reference ground truth
- name: PBSIM2
  role: generates 40x coverage PacBio CLR long reads against the simulated inverted
    chr1 so that 3738 reads span inversion breakpoints
- name: invMap
  role: long-read mapper under evaluation; maps simulated reads back to the original
    chr1 and is scored for inversion sensitivity/precision
  repo: https://github.com/zhang134/invMap.git
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad726
    title: 'invMap: a sensitive mapping tool for long noisy reads with inversion structural
      variants'
  source_content_hash: sha256:23510f098d2879cdb08655b01056d1f5767529fff3bba28f48d9b311fa613d77
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhang134/invMap.git
  tools:
  - PBSIM2
  - SURVIVOR
  - invMap
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# long-read-simulation-pacbio-clr

## Summary

Simulates a controlled inversion benchmark by mutating GRCh37 chr1 with SURVIVOR's simulateSV (100 inversions) and generating 40x PacBio CLR long reads with PBSIM2 so that 3738 reads span inversion breakpoints. This yields paired simulated ground truth (mutated reference + known SV coordinates) for sensitivity/precision evaluation of long-read mappers such as invMap.

## When to use

Use when benchmarking long-read structural-variant mappers against a known ground truth: simulate inversion SVs in a reference genome and generate coverage-controlled PacBio CLR reads spanning the breakpoints.

## When NOT to use

- Reads are real sequencing data (e.g. the NA12878 human dataset), where there is no simulated ground truth — map and call candidate inversions directly instead of running the simulation benchmark
- The target SV type or scale differs (e.g. non-inversion SVs, different chromosome or coverage), since the 100-inversion / 40x CLR parameters are specific to this chr1 benchmark

## Inputs

- GRCh37 chromosome 1 reference FASTA (original, non-mutated)
- SURVIVOR simulateSV configuration for 100 inversion structural variants
- PBSIM2 CLR model/parameters for 40x coverage PacBio CLR simulation
- Simulated long reads in FASTA/FASTQ format (3738 breakpoint-spanning reads)

## Outputs

- Mutated (inversion-carrying) chr1 reference FASTA serving as simulation ground truth
- 40x PacBio CLR simulated read set spanning inversion breakpoints
- Set of 100 ground-truth inversion coordinates for SV recovery classification
- Per-mapper SAM alignments and inversion_benchmark_table.csv with predicted/recovered counts, sensitivity, and precision

## How to apply

First simulate 100 inversion structural variants in the GRCh37 chromosome 1 reference using SURVIVOR's simulateSV module, producing a mutated (inverted) chr1 that serves as the simulation ground truth. Next, generate 40x coverage PacBio CLR long reads with PBSIM2 against the simulated inverted chr1, checking that 3738 reads span inversion breakpoints. Map the resulting read set back against the original non-mutated chr1 reference with each mapper under test (invMap, Winnowmap2, minimap2, NGMLR, lra), e.g. './invMap genome.fa reads.fa > invMap_aligned.sam'. Call inversion SVs from each mapper's alignments and classify predicted calls as recovered true positives against the 100 SURVIVOR ground-truth inversions. Compute sensitivity as recovered/predicted (e.g. invMap: 3483/3606 = 92.06%) plus precision, and compare against the paper's Table 1 within stochastic-simulation tolerance.

## Related tools

- **SURVIVOR** (simulateSV module used to simulate the 100 inversion structural variants in GRCh37 chr1, creating the mutated reference ground truth)
- **PBSIM2** (generates 40x coverage PacBio CLR long reads against the simulated inverted chr1 so that 3738 reads span inversion breakpoints)
- **invMap** (long-read mapper under evaluation; maps simulated reads back to the original chr1 and is scored for inversion sensitivity/precision) — https://github.com/zhang134/invMap.git

## Examples

```
./invMap genome.fa reads.fa > invMap_aligned.sam
```

## Evaluation signals

- PBSIM2 output contains 3738 reads spanning inversion breakpoints at 40x coverage
- Ground truth consists of exactly 100 SURVIVOR-simulated inversions in chr1
- Reproduced invMap row reports 3606 predicted SVs, 3483 recovered SVs, 92.06% sensitivity (3483/3606), and 96.58% precision within stochastic-simulation tolerance
- Each of the five mappers (invMap, Winnowmap2, minimap2, NGMLR, lra) produces a valid SAM alignment file against the original non-mutated chr1
- Sensitivity is computed as recovered/predicted ratio and assembled into inversion_benchmark_table.csv for direct comparison with Table 1

## Limitations

- Simulation is stochastic: reproduced counts must match Table 1 within tolerance rather than exactly
- The benchmark is limited to simulated PacBio CLR reads on chr1 inversions; on real data (NA12878) invMap was only shown to find more candidate variant calls, not validated against ground-truth accuracy
- invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes, which constrains where the downstream mapping step can run

## Evidence

- [abstract] Rationale: PacBio CLR long reads frequently span SV breakpoints, enabling inversion detection: "Longer reads produced by PacBio or Oxford Nanopore sequencers could more frequently span the breakpoints of structural variations (SVs) than shorter reads"
- [methods] Ground-truth creation step via SURVIVOR simulateSV on GRCh37 chr1: [evidence span withheld — anchor status: not_found]
- [methods] PBSIM2 CLR read generation at 40x coverage with breakpoint-spanning reads: [evidence span withheld — anchor status: not_found]
- [readme] Downstream mapping command used to align simulated reads against the original reference: "./invMap genome.fa reads.fa > invMap_aligned.sam"
