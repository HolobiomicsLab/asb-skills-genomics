---
name: structural-variant-simulation
description: Use when benchmarking long-read mappers or SV callers for inversions
  and indels and you need simulated ground-truth SVs with noisy CLR/ONT reads at controlled
  coverage (10x-40x) against GRCh37 chr1.
when_to_use_negative:
- The dataset is real sequencing data (e.g. GIAB HG002 PacBio reads) where mapper
  accuracy should be measured against experimentally implanted or curated truth sets
  rather than simulated reads
- You need to evaluate mapper alignment speed or base-level mapping quality rather
  than downstream SV-calling accuracy against known ground truth
- The question concerns SV types other than inversions and indels (e.g. duplications,
  translocations) that the described simulateSV setup does not cover
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0625
tools:
- name: SURVIVOR
  role: simulateSV module implants ground-truth inversions/indels into the chr1 reference
    to create the mutated reference and truth SV set
- name: PBSIM2
  role: simulates PacBio CLR long noisy reads from the mutated reference at specified
    coverage levels
- name: invMap
  role: test mapper that maps simulated reads to the original reference and detects
    inversion SVs; benchmarked against minimap2, NGMLR, Winnowmap2, lra
  repo: https://github.com/zhang134/invMap.git
- name: SVision
  role: calls structural variants from each mapper's alignments for comparison against
    the truth set
- name: samtools
  role: converts and sorts mapper SAM output to BAM before SV calling
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad726
    title: 'invMap: a sensitive mapping tool for long noisy reads with inversion structural
      variants'
  source_content_hash: sha256:37d8cfaff7497b30588e3396bb4336793538c4f0b61869492cbf34fa3a3f6be4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhang134/invMap.git
  tools:
  - NGMLR
  - Oxford Nanopore
  - PBSIM2
  - SURVIVOR
  - SVision
  - Winnowmap2
  - invMap
  - lra
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# structural-variant-simulation

## Summary

Benchmark-grade simulation of inversion and indel structural variants in a reference genome (SURVIVOR simulateSV) followed by noisy PacBio CLR/ONT long-read simulation (PBSIM2) at controlled coverage, creating ground-truth SV sets for evaluating long-read mappers such as invMap, minimap2, NGMLR, Winnowmap2, and lra.

## When to use

Use when benchmarking long-read mappers or SV callers for inversions and indels and you need simulated ground-truth SVs with noisy CLR/ONT reads at controlled coverage (10x-40x) against GRCh37 chr1.

## When NOT to use

- The dataset is real sequencing data (e.g. GIAB HG002 PacBio reads) where mapper accuracy should be measured against experimentally implanted or curated truth sets rather than simulated reads
- You need to evaluate mapper alignment speed or base-level mapping quality rather than downstream SV-calling accuracy against known ground truth
- The question concerns SV types other than inversions and indels (e.g. duplications, translocations) that the described simulateSV setup does not cover

## Inputs

- GRCh37 chromosome 1 reference sequence in FASTA format
- SURVIVOR simulateSV configuration (SV types, sizes, counts: e.g. 100 inversions or 600 indels + 300 inversions)
- Ground-truth SV set (truth VCF from simulateSV)
- Read simulator settings (PBSIM2 PacBio CLR / ONT error model, coverage: 10x/20x/30x/40x)

## Outputs

- Mutated reference genome FASTA with implanted SVs
- Simulated long noisy reads in FASTA/FASTQ (e.g. 3738 breakpoint-spanning CLR reads at 40x)
- Per-mapper alignments in SAM format, converted to sorted BAM
- Per-mapper SV calls and a benchmark table (e.g. inversion_benchmark_table.csv, mapper_accuracy_benchmark.csv) with sensitivity, precision, recall, F1

## How to apply

Use SURVIVOR's simulateSV module to embed structural variants into the GRCh37 chromosome 1 reference — e.g. 100 inversions for the inversion-sensitivity benchmark, or 900 SVs (600 indels and 300 inversions) for the coverage-accuracy benchmark — producing a mutated reference plus a truth SV set. Generate long noisy reads from the mutated chr1 with PBSIM2 using PacBio CLR (or Oxford Nanopore-style) error profiles at target coverages of 10x, 20x, 30x, and 40x; at 40x on the 100-inversion setup this yields 3738 breakpoint-spanning reads. Map every read set back against the original (non-mutated) reference with invMap ('./invMap genome.fa reads.fa > invMap_aligned.sam') and each competing mapper, convert SAMs to sorted BAMs with samtools, and call SVs (e.g. with SVision). Finally, classify predicted calls as recovered true positives against the SURVIVOR truth set and compute sensitivity (recovered/predicted), precision, and F1 per tool and coverage level.

## Related tools

- **SURVIVOR** (simulateSV module implants ground-truth inversions/indels into the chr1 reference to create the mutated reference and truth SV set)
- **PBSIM2** (simulates PacBio CLR long noisy reads from the mutated reference at specified coverage levels)
- **invMap** (test mapper that maps simulated reads to the original reference and detects inversion SVs; benchmarked against minimap2, NGMLR, Winnowmap2, lra) — https://github.com/zhang134/invMap.git
- **SVision** (calls structural variants from each mapper's alignments for comparison against the truth set)
- **samtools** (converts and sorts mapper SAM output to BAM before SV calling)

## Examples

```
./invMap genome.fa reads.fa > invMap_aligned.sam
```

## Evaluation signals

- The reproduced invMap row on the 100-inversion chr1 dataset must report 3606 predicted SVs, 3483 recovered SVs, 92.06% sensitivity, and 96.58% precision within stochastic-simulation tolerance
- The simulated read set should contain the expected number of breakpoint-spanning reads (e.g. 3738 inversion-spanning CLR reads at 40x)
- Coverage trend check: indel calling is reliable from 30x minimum coverage while inversion calling requires 40x, with inversion accuracy lower than indel accuracy at every coverage
- Mapper ranking sanity check: invMap outperforms Winnowmap2, minimap2, NGMLR, and lra on inversion recovery (e.g. 76.55% vs 75.15%/73.4%/67.10% on HG002)
- Every SAM output parses as valid SAM and converts to a sorted BAM without errors before SV calling

## Limitations

- Inversion calling accuracy is significantly lower than deletion/insertion accuracy at every coverage level, so inversion benchmarks need at least 40x coverage to be meaningful
- Simulation results are stochastic; reproduced metrics are only expected to match published values within stochastic-simulation tolerance
- invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes, which constrains machine selection

## Evidence

- [abstract] Longer reads produced by PacBio or Oxford Nanopore sequencers could more frequently span the breakpoints of structural variations (SVs): "Longer reads produced by PacBio or Oxford Nanopore sequencers could more frequently span the breakpoints of structural variations (SVs) than shorter reads"
- [abstract] invMap accuracy finding: "experimental results demonstrate that invMap is more accurate to locate aligned regions and call SVs for inversions than the competing methods"
- [readme] RAM requirement: "invMap requires a minimum of
16G of RAM for mapping sequence datasets to large reference genomes."
- [readme] running example: "./invMap genome.fa reads.fa > invMap_aligned.sam"
- [methods] simulated benchmark setup: [evidence span withheld — anchor status: not_found]
