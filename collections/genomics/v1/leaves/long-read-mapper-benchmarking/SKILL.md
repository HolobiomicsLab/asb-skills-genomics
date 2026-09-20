---
name: long-read-mapper-benchmarking
description: Use when comparing long-read mappers on datasets containing inversion
  structural variants, needing sensitivity/precision/accuracy tables (e.g., Table
  1 style) or wall-clock runtime rankings across coverage levels.
when_to_use_negative:
- Reads have no expected structural variants and only mapping throughput/identity
  matters without a truth set for SV recovery
- Input data is short Illumina reads — the compared mappers are designed for PacBio/Oxford
  Nanopore long noisy reads
- Only one mapper's output is needed (e.g., production mapping) rather than a head-to-head
  comparison
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: invMap
  role: Long-read mapper under evaluation; detects inversions via chaining-based aligned-region
    location and anchor checking; produces SAM output
  repo: https://github.com/zhang134/invMap.git
- name: SURVIVOR
  role: simulateSV module used to implant known inversions (100 in chr1; 1000 genome-wide)
    into GRCh37 to create mutated references and truth sets
- name: PBSIM2
  role: Simulates 40x PacBio CLR long reads against the mutated chr1 so that 3738
    reads span inversion breakpoints
- name: minimap2
  role: Competing baseline mapper; fastest at all coverages in runtime benchmarking
- name: Winnowmap2
  role: Competing baseline mapper for accuracy and runtime comparison
- name: lra
  role: Competing baseline mapper for accuracy and runtime comparison
- name: NGMLR
  role: Competing baseline mapper; slowest, with invMap significantly faster
- name: SVision
  role: Calls structural variants from each mapper's BAM alignments for comparison
    against the truth set
- name: samtools
  role: Converts and sorts each mapper's SAM output to BAM before SV calling
provenance:
  source_task_ids:
  - task_003
  - task_006
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad726
    title: 'invMap: a sensitive mapping tool for long noisy reads with inversion structural
      variants'
  source_content_hash: sha256:195151ae8d51b351a4e6d7a5cb90a0d64a071520bbd78c14d388059d4fafc1c9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhang134/invMap.git
  tools:
  - NGMLR
  - PBSIM2
  - SURVIVOR
  - SVision
  - Winnowmap2
  - invMap
  - lra
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# long-read-mapper-benchmarking

## Summary

Benchmark long-read mapping tools (invMap, minimap2, Winnowmap2, lra, NGMLR) for inversion SV detection accuracy and runtime on SURVIVOR-implanted simulated and real PacBio datasets, as done in the invMap paper.

## When to use

Use when comparing long-read mappers on datasets containing inversion structural variants, needing sensitivity/precision/accuracy tables (e.g., Table 1 style) or wall-clock runtime rankings across coverage levels.

## When NOT to use

- Reads have no expected structural variants and only mapping throughput/identity matters without a truth set for SV recovery
- Input data is short Illumina reads — the compared mappers are designed for PacBio/Oxford Nanopore long noisy reads
- Only one mapper's output is needed (e.g., production mapping) rather than a head-to-head comparison

## Inputs

- Reference genome FASTA (e.g., GRCh37 chr1)
- SURVIVOR-generated mutated reference with truth VCF of implanted inversions (100 or 1000 SVs)
- PBSIM2-simulated PacBio CLR reads at 10x/20x/30x/40x coverage, or real PacBio reads (GIAB HG002)
- Mapper binaries: invMap, minimap2, Winnowmap2, lra, NGMLR; SV caller (SVision); samtools

## Outputs

- Per-mapper SAM/BAM alignment files (e.g., invMap_aligned.sam)
- Inversion benchmark table with predicted, recovered, sensitivity, precision (inversion_benchmark_table.csv; invMap: 3606 predicted, 3483 recovered, 92.06% sensitivity, 96.58% precision)
- Per-mapper accuracy table for real reads (mapper_accuracy_benchmark.csv; invMap 76.55%, minimap2 75.15%, NGMLR 73.4%, Winnowmap2 67.10%, lra 63.25%)
- Runtime table per coverage level and mapper (mapper_runtime_benchmark.csv)

## How to apply

Implant a known number of inversions into a reference genome with SURVIVOR's simulateSV module (e.g., 100 inversions in GRCh37 chr1, or 1000 genome-wide) to create a mutated reference plus a truth VCF. Simulate long reads against the mutated reference with PBSIM2 (e.g., 40x PacBio CLR coverage yielding 3738 inversion-spanning reads), or use real reads such as the GIAB HG002 PacBio dataset. Map reads against the original (non-mutated) reference with each mapper using 64 threads, run invMap with './invMap genome.fa reads.fa > invMap_aligned.sam', convert SAM to sorted BAM with samtools, call SVs (e.g., with SVision), then classify predictions as recovered true positives against the truth set and compute sensitivity (e.g., 3483/3606 = 92.06% for invMap) and precision (96.58%). Record wall-clock runtimes per coverage (10x, 20x, 30x, 40x) to rank mappers; validation expects minimap2 fastest, then lra and Winnowmap2, with invMap faster than NGMLR, matching Supplementary Fig. S7 ordering.

## Related tools

- **invMap** (Long-read mapper under evaluation; detects inversions via chaining-based aligned-region location and anchor checking; produces SAM output) — https://github.com/zhang134/invMap.git
- **SURVIVOR** (simulateSV module used to implant known inversions (100 in chr1; 1000 genome-wide) into GRCh37 to create mutated references and truth sets)
- **PBSIM2** (Simulates 40x PacBio CLR long reads against the mutated chr1 so that 3738 reads span inversion breakpoints)
- **minimap2** (Competing baseline mapper; fastest at all coverages in runtime benchmarking)
- **Winnowmap2** (Competing baseline mapper for accuracy and runtime comparison)
- **lra** (Competing baseline mapper for accuracy and runtime comparison)
- **NGMLR** (Competing baseline mapper; slowest, with invMap significantly faster)
- **SVision** (Calls structural variants from each mapper's BAM alignments for comparison against the truth set)
- **samtools** (Converts and sorts each mapper's SAM output to BAM before SV calling)

## Examples

```
./invMap genome.fa reads.fa > invMap_aligned.sam
```

## Evaluation signals

- Simulated-chr1 invMap row reproduces 3606 predicted SVs, 3483 recovered SVs, 92.06% sensitivity, 96.58% precision within stochastic-simulation tolerance
- HG002 real-read accuracy ordering holds: invMap 76.55% > minimap2 75.15% > NGMLR 73.4% > Winnowmap2 67.10% > lra 63.25%
- Runtime ranking matches Supplementary Fig. S7: minimap2 fastest at all coverages, followed by lra and Winnowmap2, with invMap faster than NGMLR
- Recovered/predicted ratio is reproducible from the predicted and recovered SV counts (e.g., 3483/3606 = 92.06%)
- All mapper outputs are valid SAM converted to sorted BAM without errors before SV calling

## Limitations

- Simulated-read counts (e.g., 3738 breakpoint-spanning reads) vary stochastically, so exact reproduced numbers tolerate simulation noise
- invMap requires a minimum of 16G of RAM for mapping to large reference genomes and ~800 M disk space
- SV calling accuracy depends on the downstream caller (SVision); runtime comparisons assume identical threading (64 threads) across mappers

## Evidence

- [other] invMap predicted 3606 inversions, correctly recovered 3483 of them, and achieved a recover sensitivity of 92.06%: [evidence span withheld — anchor status: not_found]
- [other] On the real HG002 PacBio dataset with implanted inversions, invMap achieves 76.55% accuracy: [evidence span withheld — anchor status: not_found]
- [other] Minimap2 is consistently the fastest tool at all coverages, followed by lra and Winnowmap2, while invMap is significantly faster than NGMLR.: [evidence span withheld — anchor status: not_found]
- [abstract] invMap first locates the aligned region with a specifically designed scoring method for chaining: "invMap first locates the aligned region with a specifically designed scoring method for chaining, then checks the remaining anchors in the aligned region to discover potential inversions"
- [readme] ./invMap genome.fa reads.fa > invMap_aligned.sam: "./invMap genome.fa reads.fa > invMap_aligned.sam"
- [intro] invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes.: "invMap requires a minimum of
16G of RAM for mapping sequence datasets to large reference genomes."
