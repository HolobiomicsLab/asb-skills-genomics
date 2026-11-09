---
name: genome-shuffling-breakpoint-simulation
description: Use when you need a controlled ground-truth benchmark of structural-variant
  callers on long-read data, with recall/FPR measured per caller across a range of
  downsampled coverage levels.
when_to_use_negative:
- You have real, not simulated, patient data with validated breakpoints — recall/FPR
  against an in-silico ground truth is not meaningful there
- You need to call SVs on a single sample rather than benchmark multiple callers —
  use the production NanoSV pipeline instead
- Short-read data is the input; this simulation workflow models nanopore long-read
  error profiles via NanoSim
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: NanoSV
  role: SV caller benchmarked on simulated long-read breakpoints
  repo: https://github.com/mroosmalen/nanosv
- name: LAST
  role: Long-read aligner producing the sorted BAMs input to all callers; chosen for
    most accurate SV-calling alignments
- name: NanoSim
  role: Read simulator trained on Patient 2 nanopore reads to generate simulated long
    reads from the rearranged reference
- name: Lumpy
  role: Comparison SV caller run on the same downsampled BAMs
- name: Sniffles
  role: Comparison SV caller run on the same downsampled BAMs
- name: sambamba
  role: Converts LAST MAF output to SAM and sorts to coordinate-sorted BAM
  repo: https://github.com/biod/sambamba
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-017-01343-4
    title: Mapping and phasing of structural variation in patient genomes using nanopore
      sequencing
  source_content_hash: sha256:488c63f2c30d056a98fa9536091227abad6fa771dc586ebde6e9f8fcb64d7182
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mroosmalen/nanosv
  tools:
  - LAST
  - Lumpy
  - NanoSV
  - NanoSim
  - Sniffles
  - sambamba
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

# genome-shuffling-breakpoint-simulation

## Summary

Benchmark SV callers by reshuffling a reference chromosome in silico to create a known set of rearrangement breakpoints, simulating nanopore long reads with NanoSim, and computing recall and false-positive rate for each caller across coverage levels (1x–44x).

## When to use

Use when you need a controlled ground-truth benchmark of structural-variant callers on long-read data, with recall/FPR measured per caller across a range of downsampled coverage levels.

## When NOT to use

- You have real, not simulated, patient data with validated breakpoints — recall/FPR against an in-silico ground truth is not meaningful there
- You need to call SVs on a single sample rather than benchmark multiple callers — use the production NanoSV pipeline instead
- Short-read data is the input; this simulation workflow models nanopore long-read error profiles via NanoSim

## Inputs

- Reference genome (e.g. GRCh37 chromosome 1 FASTA) to reshuffle in silico
- Real nanopore FASTQ reads (e.g. MinION reads from Patient 2) to train the NanoSim read model
- List of simulated rearrangement breakpoint coordinates as ground truth
- Caller output VCFs from NanoSV, Lumpy, and Sniffles at each coverage level

## Outputs

- sv_caller_recall_fpr.csv with per-caller, per-coverage recall and false-positive rate
- Sorted BAM files of downsampled simulated reads (1x–44x)
- Per-caller, per-coverage VCF call sets

## How to apply

Reshuffle GRCh37 chromosome 1 in silico to introduce simulated SV breakpoints and record their coordinates as ground truth. Train a NanoSim read model on real nanopore reads (e.g. Patient 2 MinION reads) and simulate long reads from the rearranged reference so the simulated data carry a realistic nanopore error profile. Downsample the simulated read set from 1x to 44x coverage in discrete increments, align each set with LAST, convert to sorted BAM, and run NanoSV, Lumpy, and Sniffles on every BAM. Match each caller's VCF calls against the ground-truth breakpoints to compute recall, and against the unshuffled reference to compute false-positive rate at every coverage level. LAST is preferred over other mappers because its alignments gave the most accurate results for SV calling with NanoSV.

## Related tools

- **NanoSV** (SV caller benchmarked on simulated long-read breakpoints) — https://github.com/mroosmalen/nanosv
- **LAST** (Long-read aligner producing the sorted BAMs input to all callers; chosen for most accurate SV-calling alignments)
- **NanoSim** (Read simulator trained on Patient 2 nanopore reads to generate simulated long reads from the rearranged reference)
- **Lumpy** (Comparison SV caller run on the same downsampled BAMs)
- **Sniffles** (Comparison SV caller run on the same downsampled BAMs)
- **sambamba** (Converts LAST MAF output to SAM and sorts to coordinate-sorted BAM) — https://github.com/biod/sambamba

## Examples

```
lastal -Q1 -p my-params refdb reads.fastq | last-split | maf-convert -f reference.dict sam /dev/stdin | sambamba view -h -S --format=bam /dev/stdin | sambamba sort /dev/stdin -o reads.sorted.bam
```

## Evaluation signals

- NanoSV recall on the 501 simulated breakpoints reaches 99.2% at 27x coverage with maximum FPR 1.2% at 44x
- Lumpy reports 92.4% recall / 78.8% FPR and Sniffles 92.6% recall / 97% FPR in the same benchmark
- Every caller VCF is matched against exactly 501 ground-truth breakpoints; recall cannot exceed 100%
- FPR is computed by calling against the unshuffled reference — any SV call there is a false positive
- Coverage series must span discrete increments from 1x to 44x for each caller

## Limitations

- Results depend on the NanoSim error model trained on specific patient reads; a different error profile may shift recall/FPR
- Simulation covers chromosome 1 with 501 breakpoints only; genome-wide behavior or other SV classes are not benchmarked
- The simulated-reference alignment step means callers never face reference mismatches a real sample would have

## Evidence

- [readme] README - LAST preferred for NanoSV: "We found that LAST alignments give the most accurate results for SV calling with NanoSV. However, mapping with LAST requires more compute resources."
- [readme] README - coordinate-sorted indexed BAM required: "This BAM file needs to be coordinate-sorted and indexed."
- [intro] Article - MinION sequencing of two patients: "In this work, we demonstrate sequencing of the whole diploid human genomes of two patients on the MinION sequencer at 11–16× depth of coverage."
- [intro] Article - NanoSV pipeline feasibility: "We employ a novel computational pipeline to demonstrate the feasibility of using MinION reads to detect de novo complex SV breakpoints, at high sensitivity."
- [results] Task card - benchmark result: [evidence span withheld — anchor status: not_found]
