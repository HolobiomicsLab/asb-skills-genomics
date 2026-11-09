---
name: sv-caller-precision-recall-evaluation
description: Use when comparing insertion-calling tools on simulated linked-read data
  with a known ground truth, or when reproducing the reported precision/recall values
  for Novel-X versus NUI, PopIns2 and Pamir at 65x coverage.
when_to_use_negative:
- The dataset is real (non-simulated) data without a known insertion ground truth
  — precision/recall cannot be computed against the 2000 simulated insertions; use
  independent validation against assemblies (e.g. the T2T CHM13 v2.0 assembly) instead.
- The caller under test is Pamir and only linked-read data exist — Pamir cannot consume
  linked-reads, so it must be given separately simulated matched short reads.
- The analysis targets insertions below 300 bp — Novel-X's output filter (>300 bp
  with >300 bp anchor sum) and its requirement for many distinct barcodes (t=5) structurally
  bias evaluation of short insertions.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: Novel-X
  role: Primary insertion caller under evaluation; run on the LongRanger-aligned 65x
    linked-read BAM using barcode information
  repo: github.com/1dayac/Novel-X
- name: NUI
  role: Synthetic long-read insertion-caller baseline; run on the same linked-read
    data as Novel-X
- name: PopIns2
  role: Insertion-caller baseline; run on the same linked-read data in this benchmark
- name: Pamir
  role: Short-read-only insertion-caller baseline; run on the ART-simulated short
    reads because it cannot consume linked-reads
- name: LRSIM
  role: Simulates the 65x-coverage barcoded synthetic long reads from the simulated
    ground-truth genome
- name: ART
  role: Simulates the matched standard short-read data at the same coverage for the
    Pamir baseline
- name: LongRanger
  role: Aligns the linked-reads into a barcode-annotated BAM that serves as Novel-X's
    input
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkac653
    title: Efficient detection and assembly of non-reference DNA sequences with synthetic
      long reads
  source_content_hash: sha256:40bd098e448ff88ade2f9cefe9ebd6b46af1010b3a597df521aac6f7070aaa3f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ART
  - LRSIM
  - LongRanger
  - NUI
  - Novel-X
  - Pamir
  - PopIns2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# sv-caller-precision-recall-evaluation

## Summary

A benchmarking protocol that evaluates non-reference sequence (NRS) insertion callers — Novel-X, NUI, PopIns2 and Pamir — on simulated 65x barcoded linked-read data by matching each tool's calls against 2000 ground-truth GRCh38 insertions and computing precision/recall with size-bin breakdowns.

## When to use

Use when comparing insertion-calling tools on simulated linked-read data with a known ground truth, or when reproducing the reported precision/recall values for Novel-X versus NUI, PopIns2 and Pamir at 65x coverage.

## When NOT to use

- The dataset is real (non-simulated) data without a known insertion ground truth — precision/recall cannot be computed against the 2000 simulated insertions; use independent validation against assemblies (e.g. the T2T CHM13 v2.0 assembly) instead.
- The caller under test is Pamir and only linked-read data exist — Pamir cannot consume linked-reads, so it must be given separately simulated matched short reads.
- The analysis targets insertions below 300 bp — Novel-X's output filter (>300 bp with >300 bp anchor sum) and its requirement for many distinct barcodes (t=5) structurally bias evaluation of short insertions.

## Inputs

- Simulated ground-truth genome: GRCh38 with 2000 randomly generated insertions of 50-6000 bp
- LRSIM-simulated 65x barcoded synthetic long (linked) reads
- ART-simulated 65x matched standard short-read data (for the Pamir baseline)
- LongRanger-aligned linked-read BAM with barcode information
- Per-tool insertion call sets (VCF) from Novel-X, NUI, PopIns2 and Pamir

## Outputs

- benchmark_results.csv with per-tool precision and recall
- Size-bin precision/recall breakdown across the 50-6000 bp insertion-length range
- Matched call sets per tool with TP/FP/FN assignments under the 100 bp breakpoint / 5% length criterion

## How to apply

Build the ground truth by inserting 2000 randomly generated DNA sequences of 50-6000 bp into the GRCh38 reference genome, then simulate 65x-coverage barcoded synthetic long reads from that genome with LRSIM and matched standard short reads with ART (the latter because Pamir cannot consume linked-reads). Align the linked-reads with LongRanger and run Novel-X on the resulting barcode-annotated BAM, while running NUI and PopIns2 on the same linked-read data and Pamir on the ART short reads. Match each tool's insertion calls to the 2000 ground-truth insertions using the dual criterion that breakpoint positions differ by no more than 100 bp and insertion lengths differ by no more than 5%. Compute precision and recall per tool, including size-bin breakdowns across the 50-6000 bp range, and write the comparison table to benchmark_results.csv. Treat the run as correct when the metrics reproduce the reported values: Novel-X precision 0.999, NUI 0.997, PopIns2 0.999 and Pamir 0.817, with Novel-X showing the best overall recall and precision.

## Related tools

- **Novel-X** (Primary insertion caller under evaluation; run on the LongRanger-aligned 65x linked-read BAM using barcode information) — github.com/1dayac/Novel-X
- **NUI** (Synthetic long-read insertion-caller baseline; run on the same linked-read data as Novel-X)
- **PopIns2** (Insertion-caller baseline; run on the same linked-read data in this benchmark)
- **Pamir** (Short-read-only insertion-caller baseline; run on the ART-simulated short reads because it cannot consume linked-reads)
- **LRSIM** (Simulates the 65x-coverage barcoded synthetic long reads from the simulated ground-truth genome)
- **ART** (Simulates the matched standard short-read data at the same coverage for the Pamir baseline)
- **LongRanger** (Aligns the linked-reads into a barcode-annotated BAM that serves as Novel-X's input)

## Evaluation signals

- Reproduced precision matches the published values: Novel-X 0.999, NUI 0.997, PopIns2 0.999 and Pamir 0.817.
- Novel-X shows the best overall recall and precision in the reproduced comparison table.
- benchmark_results.csv contains per-tool precision/recall plus size-bin breakdowns spanning the 50-6000 bp insertion-length range.
- Every tool's calls are matched against exactly the 2000 ground-truth insertions under the 100 bp breakpoint / 5% length criterion, with no double-matched or duplicated events.
- Input consistency holds: Novel-X, NUI and PopIns2 consumed the same LongRanger-aligned linked-read BAM, while Pamir consumed ART-simulated short reads at the same 65x coverage.

## Limitations

- Pamir cannot consume linked-reads, so it must be run on separately ART-simulated short reads, making its comparison dependent on simulation fidelity.
- The matching criterion (breakpoints within 100 bp, insertion lengths within 5%) is an approximation; near-duplicate calls at the same locus can inflate or deflate per-tool precision and recall.
- Novel-X only outputs insertions longer than 300 bp with the sum of the anchors' lengths exceeding 300 bp, and it needs a substantial number of distinct barcodes — for small insertions the coverage is not sufficient — so short-insertion recall is structurally limited in this benchmark.
- Randomly generated simulated insertions may not reflect real repeat-rich or population-specific sequence biology, so absolute metric values may not transfer to real genomes.

## Evidence

- [methods] Ground-truth construction for the simulated benchmark: [evidence span withheld — anchor status: not_found]
- [methods] Linked-read simulation parameters: [evidence span withheld — anchor status: not_found]
- [methods] Call-to-truth matching criterion: [evidence span withheld — anchor status: not_found]
- [results] Expected validation values for the reproduced table: [evidence span withheld — anchor status: not_found]
- [readme] Novel-X scope relevant to the evaluated caller: "Novel-X detects and genotypes novel sequence insertions in 10X sequencing dataset using non-trivial read alignment signatures and barcode information."
