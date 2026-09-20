---
name: short-read-simulation-for-baseline-comparison
description: Use when benchmarking an NRS/insertion detection method against baselines
  that require ordinary paired-end short reads, and ground-truth insertions (e.g.
  2000 sequences of 50-6000 bp inserted into GRCh38) must be tested at matched coverage.
when_to_use_negative:
- The baseline method natively accepts barcoded linked-reads (e.g. Novel-X, NUI, PopIns2)
  — use the LRSIM linked-read BAM with LongRanger alignment instead.
- The goal is to assess real-sample performance rather than a controlled simulation
  — use real datasets such as the Polaris PGx cohort, NA12878, or haploid CHM1/CHM13
  genomes.
- Insertions of interest are < 300 bp, where Novel-X's own analysis and coverage-based
  limitations apply differently.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: ART
  role: Simulates matched standard short-read data at the same coverage for the Pamir
    baseline, which cannot consume linked-reads
- name: LRSIM
  role: Simulates the barcoded synthetic long-read benchmark data from which coverage
    is matched
- name: Pamir
  role: Short-read baseline method that consumes the ART-simulated reads
- name: Novel-X
  role: Reference method evaluated on the linked-read side of the comparison
  repo: github.com/1dayac/Novel-X
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkac653
    title: Efficient detection and assembly of non-reference DNA sequences with synthetic
      long reads
  source_content_hash: sha256:879a264f45f9609daa6de45b325cb7bf5a294343d849fe505ba965f1e00e144c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ART
  - LRSIM
  - NUI
  - Novel-X
  - Pamir
  - PopIns2
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# short-read-simulation-for-baseline-comparison

## Summary

Simulate matched standard short-read data with ART at the same coverage as a linked-read benchmark so that baseline tools that cannot consume linked-reads (e.g. Pamir) can be compared on equal footing against synthetic-long-read methods like Novel-X, NUI, and PopIns2.

## When to use

Use when benchmarking an NRS/insertion detection method against baselines that require ordinary paired-end short reads, and ground-truth insertions (e.g. 2000 sequences of 50-6000 bp inserted into GRCh38) must be tested at matched coverage.

## When NOT to use

- The baseline method natively accepts barcoded linked-reads (e.g. Novel-X, NUI, PopIns2) — use the LRSIM linked-read BAM with LongRanger alignment instead.
- The goal is to assess real-sample performance rather than a controlled simulation — use real datasets such as the Polaris PGx cohort, NA12878, or haploid CHM1/CHM13 genomes.
- Insertions of interest are < 300 bp, where Novel-X's own analysis and coverage-based limitations apply differently.

## Inputs

- Simulated ground-truth genome: GRCh38 reference with 2000 randomly generated insertions of 50-6000 bp embedded
- LRSIM-generated barcoded synthetic long reads (65x coverage) from the ground-truth genome
- Target coverage level (65x) used for the matched short-read simulation

## Outputs

- ART-simulated paired-end short-read FASTQ at 65x coverage for baseline tools unable to consume linked-reads
- Pamir insertion calls derived from the ART short reads
- Per-tool precision/recall values (benchmark_results.csv), including size-bin breakdowns, e.g. Pamir precision 0.817 vs Novel-X 0.999

## How to apply

First create a simulated ground-truth genome by inserting 2000 randomly generated DNA sequences of 50-6000 bp into the GRCh38 reference genome. Simulate 65x-coverage barcoded synthetic long reads from that ground-truth genome with LRSIM for the linked-read methods (Novel-X, NUI, PopIns2, aligned with LongRanger). Then, because Pamir cannot consume linked-reads, run ART to simulate matched standard short-read data at the same 65x coverage from the same ground-truth genome and feed those reads to Pamir. This keeps insert size, coverage, and source genome identical across baselines so precision/recall differences reflect the method, not the input data.

## Related tools

- **ART** (Simulates matched standard short-read data at the same coverage for the Pamir baseline, which cannot consume linked-reads)
- **LRSIM** (Simulates the barcoded synthetic long-read benchmark data from which coverage is matched)
- **Pamir** (Short-read baseline method that consumes the ART-simulated reads)
- **Novel-X** (Reference method evaluated on the linked-read side of the comparison) — github.com/1dayac/Novel-X

## Examples

```
art_illumina -ss MSv3 -i simulated_genome_GRCh38_plus2000ins.fa -p -l 150 -f 65 -o pamir_baseline_reads
```

## Evaluation signals

- Match calls to the 2000 ground-truth insertions using the criterion that breakpoint positions differ by no more than 100 bp and insertion lengths differ by no more than 5%.
- Reproduced precision should match reported values: Novel-X 0.999, NUI 0.997, PopIns2 0.999, Pamir 0.817.
- Novel-X should show the best overall recall and precision; Pamir's lower precision (0.817) is the expected short-read-baseline behavior.
- Per-size-bin breakdowns should show short-read baselines degrading on longer insertions, consistent with the finding that short-read methods fail on insertions > 300 bp.
- Coverage must be matched (65x) between LRSIM linked reads and ART short reads; a coverage mismatch invalidates the comparison.

## Limitations

- Pamir cannot consume linked-reads, so it necessarily runs on a different read type; ART simulation plus matched coverage mitigates but does not eliminate platform differences.
- Simulated insertions are randomly generated sequences; real NRS insertions may be repetitive or structured, so results may not transfer to real data.
- Novel-X requires a substantial number of distinct barcodes; small insertions often lack sufficient coverage for anchor assembly, so size-bin comparisons should focus on insertions > 300 bp.
- Comparative performance may differ on real haploid (CHM1/CHM13) or diploid (NA12878, NA19240, HG002) genomes versus simulated data.

## Evidence

- [other] Simulate matched standard short-read data at the same coverage with ART for the Pamir baseline (which cannot consume linked-reads): [evidence span withheld — anchor status: not_found]
- [intro] short-read NRS methods list: "Subsequent short-read methods for this problem such as MindTheGap (28), ANISE and BASIL (29), PopIns2 (30) or Pamir (31)"
- [intro] short-read failure on long insertions motivates benchmark: "All of the existing short-read methods fail to correctly assemble and locate the vast majority of insertions that are > 300 bp."
- [readme] simulated ground-truth construction: [evidence span withheld — anchor status: external_doc]
- [other] matched precision values: [evidence span withheld — anchor status: not_found]
