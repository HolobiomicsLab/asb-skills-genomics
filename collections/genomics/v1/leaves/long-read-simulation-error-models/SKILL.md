---
name: long-read-simulation-error-models
description: Use when you need ground-truth-controlled synthetic PacBio CCS/CLR or
  Oxford Nanopore reads, optionally with embedded SVs (deletions/insertions/inversions),
  to benchmark mapper location accuracy or SV-calling performance across coverages.
when_to_use_negative:
- The evaluation uses real sequencing data (e.g. the NA12878 real human genome sequencing
  dataset) where simulation-based ground truth does not apply
- Short-read or error-free data is the target — PBSIM/PBSIM2 error models are calibrated
  for long noisy PacBio/ONT reads
- No reference genome or ground-truth SV set is available, so simulated-read origin
  positions cannot be validated
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0654
tools:
- name: PBSIM
  role: Simulates PacBio CCS and CLR long reads from each reference genome at the
    study's coverages
- name: PBSIM2
  role: Simulates ONT long reads with Oxford Nanopore-style error profiles
- name: SURVIVOR
  role: Its simulateSV module embeds 900 SVs (600 indels, 300 inversions) into chr1
    and produces the ground-truth SV set
- name: invMap
  role: Downstream mapper whose location accuracy and SV-calling accuracy are evaluated
    on the simulated reads
  repo: https://github.com/zhang134/invMap.git
- name: minimap2
  role: Competing long-read mapper benchmarked on the same simulated read sets with
    long-read presets
- name: Winnowmap2
  role: Competing long-read mapper benchmarked on the same simulated read sets with
    long-read presets
- name: lra
  role: Competing long-read mapper benchmarked on the same simulated read sets
- name: NGMLR
  role: Competing long-read mapper benchmarked on the same simulated read sets with
    long-read presets
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad726
    title: 'invMap: a sensitive mapping tool for long noisy reads with inversion structural
      variants'
  source_content_hash: sha256:fb141f214667dc92064e910c4edc0e494167fc8d324fef99c4c826d2afbcc6cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhang134/invMap.git
  tools:
  - NGMLR
  - Oxford Nanopore
  - PBSIM
  - PBSIM2
  - SURVIVOR
  - Winnowmap2
  - invMap
  - lra
  - minimap2
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# long-read-simulation-error-models

## Summary

Generates simulated long noisy reads with platform-specific error profiles (PacBio CCS/CLR via PBSIM, ONT via PBSIM2) and embeds structural variants via SURVIVOR's simulateSV, enabling controlled benchmarks of long-read mappers such as invMap, minimap2, Winnowmap2, lra, and NGMLR.

## When to use

Use when you need ground-truth-controlled synthetic PacBio CCS/CLR or Oxford Nanopore reads, optionally with embedded SVs (deletions/insertions/inversions), to benchmark mapper location accuracy or SV-calling performance across coverages.

## When NOT to use

- The evaluation uses real sequencing data (e.g. the NA12878 real human genome sequencing dataset) where simulation-based ground truth does not apply
- Short-read or error-free data is the target — PBSIM/PBSIM2 error models are calibrated for long noisy PacBio/ONT reads
- No reference genome or ground-truth SV set is available, so simulated-read origin positions cannot be validated

## Inputs

- Reference genome FASTA file(s) (e.g. Neisseria meningitidis, Abortiporus biennis, GRCh37 chromosome 1)
- Sequencing-platform/error-model choice: PacBio CCS, PacBio CLR, or Oxford Nanopore
- Coverage level (e.g. 10x, 20x, 30x, 40x)
- For SV benchmarks: mutated reference FASTA and ground-truth SV set from SURVIVOR simulateSV (e.g. 600 indels + 300 inversions on chr1)

## Outputs

- Simulated long-read FASTA/FASTQ files per platform, genome, and coverage (e.g. CCS, CLR, ONT read sets)
- Ground-truth origin regions of simulated reads for location-accuracy scoring
- SURVIVOR truth-set SV file and mutated reference for SV-calling benchmarks
- Aggregated accuracy tables, e.g. location_accuracy_s4.csv with per-mapper, per-genome, per-platform values

## How to apply

Download the reference genome FASTA files (e.g. Neisseria meningitidis, Abortiporus biennis, GRCh37 chromosome 1) and simulate reads with platform-matched tools: PacBio CCS and CLR reads with PBSIM and ONT reads with PBSIM2 at the coverages used in the study. For SV benchmarks, first use SURVIVOR's simulateSV to embed a defined SV payload (e.g. 900 SVs: 600 indels and 300 inversions) into a chr1 reference, producing a mutated reference plus a ground-truth SV set, then simulate noisy reads at 10x, 20x, 30x, and 40x coverage. Map each simulated set back to its reference with invMap ('./invMap genome.fa reads.fa > invMap_aligned.sam') and competing mappers (minimap2, Winnowmap2, lra, NGMLR) using their long-read presets, then parse the SAM alignments. Compute location accuracy as the fraction of reads whose reported aligned region matches the true simulated origin region; for SV benchmarks, compare called SVs against the SURVIVOR truth set with recall, precision, and F1 split by indel vs inversion. Rationale: longer PacBio/ONT reads more frequently span SV breakpoints, and controlled simulation isolates mapper accuracy from sequencing variability.

## Related tools

- **PBSIM** (Simulates PacBio CCS and CLR long reads from each reference genome at the study's coverages)
- **PBSIM2** (Simulates ONT long reads with Oxford Nanopore-style error profiles)
- **SURVIVOR** (Its simulateSV module embeds 900 SVs (600 indels, 300 inversions) into chr1 and produces the ground-truth SV set)
- **invMap** (Downstream mapper whose location accuracy and SV-calling accuracy are evaluated on the simulated reads) — https://github.com/zhang134/invMap.git
- **minimap2** (Competing long-read mapper benchmarked on the same simulated read sets with long-read presets)
- **Winnowmap2** (Competing long-read mapper benchmarked on the same simulated read sets with long-read presets)
- **lra** (Competing long-read mapper benchmarked on the same simulated read sets)
- **NGMLR** (Competing long-read mapper benchmarked on the same simulated read sets with long-read presets)

## Examples

```
./invMap genome.fa reads.fa > invMap_aligned.sam
```

## Evaluation signals

- invMap attains >99% location accuracy on simulated PacBio CCS and CLR read sets and >98% on ONT read sets (Supplementary Table S4 / Fig. 4), outperforming minimap2, Winnowmap2, lra, and NGMLR
- Per-mapper, per-genome, per-platform location-accuracy values aggregate cleanly into location_accuracy_s4.csv
- Indel SV-calling metrics (recall/precision/F1 vs the SURVIVOR truth set) become reliable at a minimum of 30x coverage, and inversion calling requires 40x, reproducing Fig. 6 trends
- Inversion calling accuracy is significantly lower than deletion and insertion accuracy at every coverage level
- Each mapper's SAM output parses without error and the fraction of reads whose aligned region matches the true simulated origin can be computed for every read set

## Limitations

- invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes and approximately 800 M of available disk space
- Inversion detection and SV calling are coverage-hungry: indel calling needs at least 30x and inversion calling at least 40x; inversion accuracy is lower than indel accuracy at every coverage
- Results are grounded in simulation with PBSIM/PBSIM2 error models and SURVIVOR-simulated SVs; real-data validation was limited to the NA12878 dataset for candidate inversion variant calls

## Evidence

- [methods] Simulate PacBio CCS and CLR reads with PBSIM and ONT reads with PBSIM2 from each reference genome at the coverages used in the paper.: [evidence span withheld — anchor status: not_found]
- [methods] SURVIVOR simulateSV payload: [evidence span withheld — anchor status: not_found]
- [abstract] Platform motivation for long-read simulation: "Longer reads produced by PacBio or Oxford Nanopore sequencers could more frequently span the breakpoints of structural variations (SVs)"
- [readme] invMap mapping invocation producing SAM: "./invMap genome.fa reads.fa > invMap_aligned.sam"
- [results] Accuracy benchmark finding: [evidence span withheld — anchor status: not_found]
