---
name: coverage-downsampling
description: Use when evaluating the minimum depth at which a structural-variant caller
  (e.g. NanoSV) reaches target recall/FPR on long-read data, by generating per-coverage
  BAM/VCF call sets from a single simulated read set.
when_to_use_negative:
- Input is real patient data with no simulated ground-truth breakpoints, since recall
  against the 501 simulated junctions cannot be computed
- Short-read Illumina data — this benchmark is specific to nanopore long reads and
  callers tuned for split/gapped long-read alignments
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: NanoSV
  role: SV caller run on each downsampled BAM to produce per-coverage VCF call sets
  repo: https://github.com/mroosmalen/nanosv
- name: LAST
  role: Mapper producing the sorted BAMs that are downsampled; reported to give the
    most accurate NanoSV results
- name: NanoSim
  role: Simulates nanopore reads with a Patient 2-trained error model from the rearranged
    reference before downsampling
- name: Lumpy
  role: Comparison SV caller run on every downsampled BAM
- name: Sniffles
  role: Comparison SV caller run on every downsampled BAM
- name: sambamba
  role: Converts and sorts LAST MAF-derived SAM into coordinate-sorted BAMs used for
    downsampling and SV calling
  repo: https://github.com/biod/sambamba
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-017-01343-4
    title: Mapping and phasing of structural variation in patient genomes using nanopore
      sequencing
  source_content_hash: sha256:e6f589efb4a175b304908ca10976c761f6f94dff75aaf02c979a497579301629
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
yes_rate: 0.875
n_claims: 8
---

# Coverage downsampling

## Summary

Systematically downsample a simulated long-read set (here NanoSim nanopore reads of a rearranged GRCh37 chromosome 1) from 1x to 44x coverage in discrete increments to benchmark how SV-caller recall and false-positive rate depend on read depth.

## When to use

Use when evaluating the minimum depth at which a structural-variant caller (e.g. NanoSV) reaches target recall/FPR on long-read data, by generating per-coverage BAM/VCF call sets from a single simulated read set.

## When NOT to use

- Input is real patient data with no simulated ground-truth breakpoints, since recall against the 501 simulated junctions cannot be computed
- Short-read Illumina data — this benchmark is specific to nanopore long reads and callers tuned for split/gapped long-read alignments

## Inputs

- Simulated nanopore FASTQ reads from a rearranged reference with recorded ground-truth SV coordinates
- Sorted, indexed BAM of LAST alignments of the full-coverage simulated read set
- Rearranged reference genome (GRCh37 chromosome 1) with 501 ground-truth breakpoint coordinates
- Unshuffled GRCh37 reference for FPR estimation

## Outputs

- Per-coverage-level downsampled BAM files (1x–44x)
- Per-caller, per-coverage VCF call sets from NanoSV, Lumpy, and Sniffles
- Recall and false-positive rate table per caller per coverage (sv_caller_recall_fpr.csv)

## How to apply

First simulate long reads from a rearranged reference with a known ground truth (here 501 reshuffled breakpoints on GRCh37 chromosome 1) using NanoSim trained on real Patient 2 nanopore reads, then align the full read set with LAST and convert to sorted BAM. Downsample the BAM in discrete coverage increments spanning 1x to 44x, and run each caller (NanoSV, Lumpy, Sniffles) on every downsampled BAM. Match each caller's VCF calls against the 501 ground-truth breakpoints to compute recall per coverage level, and against the unshuffled reference to compute false-positive rate. In this benchmark, NanoSV reached 99.2% recall at 27x with a maximum FPR of 1.2% at 44x, versus Lumpy (92.4% recall / 78.8% FPR) and Sniffles (92.6% recall / 97% FPR); results are tabulated per caller and coverage in sv_caller_recall_fpr.csv.

## Related tools

- **NanoSV** (SV caller run on each downsampled BAM to produce per-coverage VCF call sets) — https://github.com/mroosmalen/nanosv
- **LAST** (Mapper producing the sorted BAMs that are downsampled; reported to give the most accurate NanoSV results)
- **NanoSim** (Simulates nanopore reads with a Patient 2-trained error model from the rearranged reference before downsampling)
- **Lumpy** (Comparison SV caller run on every downsampled BAM)
- **Sniffles** (Comparison SV caller run on every downsampled BAM)
- **sambamba** (Converts and sorts LAST MAF-derived SAM into coordinate-sorted BAMs used for downsampling and SV calling) — https://github.com/biod/sambamba

## Examples

```
NanoSV -t 4 -o calls_27x.vcf reads_27x.sorted.bam
```

## Evaluation signals

- Recall against the 501 ground-truth breakpoints reproduces the reported figures: NanoSV 99.2% at 27x, Lumpy 92.4%, Sniffles 92.6%
- False-positive rate (calls matched against the unshuffled reference) matches reported maxima: NanoSV 1.2% at 44x, Lumpy 78.8%, Sniffles 97%
- Recall should be monotonically non-decreasing with increasing coverage for a well-behaved caller
- Output sv_caller_recall_fpr.csv contains one row per caller per coverage level across the full 1x–44x range
- All downsampled BAMs are coordinate-sorted and indexed before NanoSV runs, as required by the caller

## Limitations

- Results depend on the NanoSim error model trained on a specific patient's nanopore reads; other error profiles may shift the coverage at which recall saturates
- Benchmark limited to simulated rearrangements of chromosome 1 (501 breakpoints); genome-wide or real chromothripsis behavior may differ
- Article notes sensitivity is limited by medium coverage and intrinsic nanopore sequencing biases; stringency of post-calling filtering trades precision for sensitivity

## Evidence

- [results] NanoSV achieves 99.2% recall at 27x coverage with a maximum false positive rate of 1.2% (at 44x coverage).: [evidence span withheld — anchor status: not_found]
- [readme] This BAM file needs to be coordinate-sorted and indexed.: "This BAM file needs to be coordinate-sorted and indexed."
- [readme] We found that LAST alignments give the most accurate results for SV calling with NanoSV.: "We found that LAST alignments give the most accurate results for SV calling with NanoSV."
- [discussion] Due to the medium coverage, some intrinsic nanopore sequencing biases and for benchmarking purposes: "Due to the medium coverage, some intrinsic nanopore sequencing biases and for benchmarking purposes"
