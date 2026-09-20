---
name: ground-truth-sv-dataset-construction
description: Use when you need a controlled ground-truth benchmark with known insertion
  positions and lengths to compare linked-read/short-read structural variant callers,
  as done for Novel-X vs NUI, PopIns2, and Pamir.
when_to_use_negative:
- Evaluation should be on real genomes (NA12878, CHM1, CHM13, HG002) rather than simulated
  truth sets — use the published VCFs in 1dayac/novel_insertions_supplementary instead
  of re-simulating
- The method of interest targets insertions much shorter than 300 bp, since Novel-X's
  analysis focuses on insertions longer than 300 bp and small insertions often lack
  sufficient barcode coverage
- The caller cannot run on either synthetic long reads or standard short reads matched
  at the same coverage (the benchmark design assumes one of these two input types)
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3697
tools:
- name: LRSIM
  role: Simulates 65x-coverage barcoded synthetic long reads from the simulated ground-truth
    genome
- name: ART
  role: Simulates matched standard short-read data at the same coverage for baselines
    (e.g. Pamir) that cannot consume linked-reads
- name: Novel-X
  role: Insertion caller evaluated on the simulated linked-read BAM; the skill's benchmark
    validates its precision/recall
  repo: https://github.com/1dayac/Novel-X
- name: NUI
  role: Synthetic long-read NRS insertion caller used as a baseline on the same linked-read
    data
- name: PopIns2
  role: Short-read insertion caller baseline run on the same linked-read data
- name: Pamir
  role: Short-read insertion caller baseline run on ART-simulated short reads
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkac653
    title: Efficient detection and assembly of non-reference DNA sequences with synthetic
      long reads
  source_content_hash: sha256:68b1c657edd5aac3f74c1aa9b409fc3dc9fd542feb67c570e73c0f68952e9441
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/Novel-X
  tools:
  - ART
  - LRSIM
  - NUI
  - Novel-X
  - Pamir
  - PopIns2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# ground-truth-sv-dataset-construction

## Summary

Builds a simulated benchmark for non-reference sequence (NRS) insertion detection by embedding randomly generated DNA sequences into GRCh38 and simulating barcoded synthetic long reads, enabling exact precision/recall evaluation of insertion callers against known truth.

## When to use

Use when you need a controlled ground-truth benchmark with known insertion positions and lengths to compare linked-read/short-read structural variant callers, as done for Novel-X vs NUI, PopIns2, and Pamir.

## When NOT to use

- Evaluation should be on real genomes (NA12878, CHM1, CHM13, HG002) rather than simulated truth sets — use the published VCFs in 1dayac/novel_insertions_supplementary instead of re-simulating
- The method of interest targets insertions much shorter than 300 bp, since Novel-X's analysis focuses on insertions longer than 300 bp and small insertions often lack sufficient barcode coverage
- The caller cannot run on either synthetic long reads or standard short reads matched at the same coverage (the benchmark design assumes one of these two input types)

## Inputs

- Reference genome FASTA (GRCh38)
- 2000 randomly generated insertion sequences, 50-6000 bp
- LRSIM parameters for 65x-coverage barcoded synthetic long reads
- ART parameters for matched short-read simulation

## Outputs

- Simulated ground-truth genome FASTA with embedded insertions
- Simulated barcoded linked-read dataset (BAM after LongRanger alignment)
- Matched simulated short-read FASTQ/BAM for baselines
- Precision/recall comparison table (e.g. benchmark_results.csv) with per-tool and size-bin breakdowns

## How to apply

Insert 2000 randomly generated DNA sequences of 50-6000 bp into the GRCh38 reference genome to create the simulated ground-truth genome. Simulate 65x-coverage barcoded synthetic long reads from the ground-truth genome with LRSIM, and additionally simulate matched standard short-read data at the same coverage with ART for baselines like Pamir that cannot consume linked-reads. Run each caller on the appropriate BAM (Novel-X, NUI, PopIns2 on LongRanger-aligned linked-reads; Pamir on the ART short reads), then match each tool's calls to the 2000 ground-truth insertions using the criterion that breakpoint positions differ by no more than 100 bp and insertion lengths differ by no more than 5%. Compute precision and recall per tool, including size-bin breakdowns, and validate that reproduced metrics match reported values (e.g. Novel-X precision 0.999, NUI 0.997, PopIns2 0.999, Pamir 0.817).

## Related tools

- **LRSIM** (Simulates 65x-coverage barcoded synthetic long reads from the simulated ground-truth genome)
- **ART** (Simulates matched standard short-read data at the same coverage for baselines (e.g. Pamir) that cannot consume linked-reads)
- **Novel-X** (Insertion caller evaluated on the simulated linked-read BAM; the skill's benchmark validates its precision/recall) — https://github.com/1dayac/Novel-X
- **NUI** (Synthetic long-read NRS insertion caller used as a baseline on the same linked-read data)
- **PopIns2** (Short-read insertion caller baseline run on the same linked-read data)
- **Pamir** (Short-read insertion caller baseline run on ART-simulated short reads)

## Evaluation signals

- Novel-X precision reaches 0.999, NUI 0.997, PopIns2 0.999, and Pamir 0.817 on the simulated benchmark
- Novel-X shows the best overall recall and precision among the compared tools
- Every call counted as a true positive satisfies both matching criteria: breakpoint within 100 bp and insertion length within 5% of ground truth
- The comparison table (e.g. benchmark_results.csv) contains per-tool metrics including size-bin breakdowns across all 2000 embedded insertions
- Downsampled variants (e.g. 80/60/40/20% read removal as in the supplementary simulated VCFs) can be generated to probe coverage sensitivity

## Limitations

- Novel-X needs a substantial number of distinct barcodes; small insertions usually have too few associated barcodes for the coverage to support anchor assembly, biasing results toward >300 bp events
- Simulation with LRSIM may not capture all real-data artifacts of 10X Genomics, TELL-Seq and stLFR platforms, so simulated performance may overestimate real-cohort accuracy
- Contamination and low-quality reads can inflate false calls; the article's pipeline applies filters such as average base quality > 20 and discards contigs recruiting fewer than t=5 barcodes

## Evidence

- [methods] Insert 2000 randomly generated DNA sequences of 50-6000 bp into the GRCh38 reference genome to create the simulated ground-truth genome.: [evidence span withheld — anchor status: not_found]
- [methods] Simulate 65x-coverage barcoded synthetic long reads from the ground-truth genome with LRSIM.: [evidence span withheld — anchor status: not_found]
- [methods] breakpoint positions differ by no more than 100 bp and insertion lengths differ by no more than 5%: [evidence span withheld — anchor status: not_found]
- [intro] All of the existing short-read methods fail to correctly assemble and locate the vast majority of insertions that are > 300 bp.: "All of the existing short-read methods fail to correctly assemble and locate the vast majority of insertions that are > 300 bp."
- [readme] No postfix - simple simulated dataset created using LRsim.: [evidence span withheld — anchor status: external_doc]
