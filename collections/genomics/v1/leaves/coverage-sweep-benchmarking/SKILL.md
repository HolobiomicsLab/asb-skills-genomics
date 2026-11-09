---
name: coverage-sweep-benchmarking
description: Use when you need to determine the minimum sequencing coverage required
  for reliable detection of a variant class (e.g., deletions, insertions, inversions)
  by sweeping coverage levels and comparing caller output to a simulated ground truth.
when_to_use_negative:
- Input is real sequencing data with no ground-truth SV set — the sweep's recall/precision/F1
  computation requires a simulated truth set
- Short-read (Illumina-style) data — the protocol is designed for long noisy PacBio/Oxford
  Nanopore reads that span SV breakpoints
- Only a single coverage level is available — the skill's purpose is the coverage
  sweep trend, not a one-off SV call
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: invMap
  role: Long-read mapper under test that locates aligned regions, discovers potential
    inversions, and produces SAM output for SV calling at each coverage level
  repo: https://github.com/zhang134/invMap.git
- name: SURVIVOR simulateSV
  role: 'Embeds the SV set (900 SVs: 600 indels, 300 inversions) into the reference
    to create the mutated reference and ground-truth SV set'
- name: PacBio
  role: Source of simulated long-read error profiles used at each coverage level
- name: Oxford Nanopore
  role: Source of simulated long-read error profiles used at each coverage level
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad726
    title: 'invMap: a sensitive mapping tool for long noisy reads with inversion structural
      variants'
  source_content_hash: sha256:78a5d622bf0919a57424bc984b2301edb6d282d272901ef33f3c2ecec3176e89
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhang134/invMap.git
  tools:
  - Oxford Nanopore
  - PacBio
  - SURVIVOR simulateSV
  - invMap
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# coverage-sweep-benchmarking

## Summary

A benchmarking protocol that simulates long noisy reads (PacBio/Oxford Nanopore error profiles) at a coverage sweep (10x, 20x, 30x, 40x) over mutated references and measures SV-calling recall, precision, and F1 per coverage level to determine the minimum coverage needed per SV type.

## When to use

Use when you need to determine the minimum sequencing coverage required for reliable detection of a variant class (e.g., deletions, insertions, inversions) by sweeping coverage levels and comparing caller output to a simulated ground truth.

## When NOT to use

- Input is real sequencing data with no ground-truth SV set — the sweep's recall/precision/F1 computation requires a simulated truth set
- Short-read (Illumina-style) data — the protocol is designed for long noisy PacBio/Oxford Nanopore reads that span SV breakpoints
- Only a single coverage level is available — the skill's purpose is the coverage sweep trend, not a one-off SV call

## Inputs

- Reference genome sequence in FASTA format (e.g., chr1)
- Embedded/mutated reference sequence with known SVs (from SURVIVOR simulateSV)
- Ground-truth SV set (VCF or SURVIVOR truth-set format)
- Simulated long noisy read sets (FASTA/FASTQ) at multiple coverage levels (10x, 20x, 30x, 40x) with PacBio- or Oxford Nanopore-style error profiles

## Outputs

- SAM alignments per mapper per coverage level (e.g., invMap_aligned.sam)
- SV call sets per mapper-coverage pair (indels and inversions)
- Results table of recall, precision, and F1 computed separately for indels and inversions at each coverage level

## How to apply

First embed a known set of SVs into a reference sequence using SURVIVOR's simulateSV module — in the invMap paper, 900 SVs (600 indels and 300 inversions) were embedded into the chr1 reference, producing a mutated reference plus a ground-truth SV set. Second, simulate long noisy reads from the mutated reference with PacBio- and Oxford Nanopore-style error profiles at four coverage levels: 10x, 20x, 30x, and 40x. Third, map every simulated read set back to the reference with the mapping tool under test (invMap and competing mappers) and call SVs from each tool's alignments against the mutated reference. Fourth, compare called SVs to the SURVIVOR truth set for each tool-coverage pair, computing recall, precision, and F1 separately for indels and inversions. Compile metrics into a results table and verify expected trends: in this study, indel calling is reliable from 30x minimum coverage, inversion calling requires 40x, and inversions are harder to call than indels at every coverage level.

## Related tools

- **invMap** (Long-read mapper under test that locates aligned regions, discovers potential inversions, and produces SAM output for SV calling at each coverage level) — https://github.com/zhang134/invMap.git
- **SURVIVOR simulateSV** (Embeds the SV set (900 SVs: 600 indels, 300 inversions) into the reference to create the mutated reference and ground-truth SV set)
- **PacBio** (Source of simulated long-read error profiles used at each coverage level)
- **Oxford Nanopore** (Source of simulated long-read error profiles used at each coverage level)

## Examples

```
./invMap genome.fa reads.fa > invMap_aligned.sam
```

## Evaluation signals

- Recall, precision, and F1 are computed per tool-coverage pair against the SURVIVOR truth set and trend upward with coverage
- Indel calling accuracy is acceptable at 30x and above; inversion calling accuracy only becomes acceptable at 40x
- Inversion calling accuracy is lower than deletion/insertion accuracy at every coverage level
- invMap yields the highest accuracy for locating aligned regions and calling inversions across all coverage levels
- Each mapper produces a valid SAM output file (e.g., invMap_aligned.sam) for every coverage level in the sweep

## Limitations

- Inversion calling accuracy is significantly lower than deletion and insertion calling at every coverage level, even at 40x
- Results are based on simulated chr1 reads with a fixed SV composition (600 indels, 300 inversions); real-genome behavior may differ
- invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes
- Real-dataset validation (NA12878) supports only candidate inversion call counts, not truth-set-based accuracy metrics

## Evidence

- [results] Across all coverages, SV call accuracy improves with coverage, but the minimum required coverage is 30x for deletions and insertions and 40x for inversions; inversion calling accuracy is significantly lower than deletion and insertion at every coverage level.: [evidence span withheld — anchor status: not_found]
- [methods] Workflow uses SURVIVOR simulateSV and four coverage levels.: [evidence span withheld — anchor status: not_found]
- [methods] Coverage sweep levels.: [evidence span withheld — anchor status: not_found]
- [abstract] invMap accuracy claim.: "experimental results demonstrate that invMap is more accurate to locate aligned regions and call SVs for inversions than the competing methods"
- [intro] invMap memory requirement.: "invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes."
- [readme] README running example.: "./invMap genome.fa reads.fa > invMap_aligned.sam"
