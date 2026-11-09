---
name: homopolymer-region-annotation
description: Use when VCF-style variant call sets from simulated nanopore reads (e.g.
  seq2squiggle vs squigulator on R10.4.1) must be attributed to homopolymer, TRF,
  or background regions to compare per-region true-positive rates.
when_to_use_negative:
- Input calls come from real (non-simulated) data with no truth set, so true-positive
  vs false-positive classification is impossible
- The analysis goal is raw signal simulation rather than post-hoc variant-call annotation
  (use seq2squiggle predict directly instead)
- Region definitions other than homopolymer runs of >=5 identical bases and TRF intervals
  are required without revalidating the three-class scheme
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: seq2squiggle
  role: Simulator that generated the R10.4.1 nanopore signals whose downstream variant
    calls are annotated by region class; one of the two simulators compared
  repo: github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- name: pandas
  role: Used in Python for the interval-overlap logic that classifies each variant
    into homopolymer, TRF, or neither
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae744
    title: End-to-end simulation of nanopore sequencing signals with feed-forward
      transformers
  source_content_hash: sha256:e1f3fa27b987236c57c3417b8b55827369f93f5cdac49cf0b7491d687c6a04cb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - pandas
  - seq2squiggle
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# homopolymer-region-annotation

## Summary

Classify variant calls (true- and false-positives) from simulated R10.4.1 nanopore data into homopolymer (runs of >=5 identical bases), Tandem Repeat Finder (TRF), or neither regions, and compute per-class true-positive counts to expose simulator-specific error profiles.

## When to use

Use when VCF-style variant call sets from simulated nanopore reads (e.g. seq2squiggle vs squigulator on R10.4.1) must be attributed to homopolymer, TRF, or background regions to compare per-region true-positive rates.

## When NOT to use

- Input calls come from real (non-simulated) data with no truth set, so true-positive vs false-positive classification is impossible
- The analysis goal is raw signal simulation rather than post-hoc variant-call annotation (use seq2squiggle predict directly instead)
- Region definitions other than homopolymer runs of >=5 identical bases and TRF intervals are required without revalidating the three-class scheme

## Inputs

- VCF-style variant call sets (true/false-positive classified) per simulator on R10.4.1 simulated reads
- Reference genome FASTA (to detect homopolymer runs of >=5 identical bases)
- Tandem Repeat Finder (TRF) repeat-region annotations (bed-style intervals)

## Outputs

- region_tp_counts.csv: per-region table of true-positive counts per simulator for homopolymer, TRF, and neither classes
- Per-region true-positive percentages (homopolymer vs TRF vs neither)

## How to apply

Load the R10.4.1 variant-calling results (VCF-style call sets for both simulators) produced by the upstream variant-calling task. Define homopolymer regions as runs of >=5 identical bases in the reference and annotate each variant position against them; annotate the same positions against Tandem Repeat Finder (TRF) repeat regions. Use pandas interval-overlap logic to assign each false-positive call to exactly one of three mutually exclusive region classes: homopolymer, TRF, or neither. Then count true-positive and false-positive calls per region class per simulator and compute the true-positive percentage per region. Write the per-region table of true-positive counts to region_tp_counts.csv; the expected pattern (Supplementary Fig. S9) is that homopolymers yield the lower true-positive percentage and seq2squiggle leads in all three region classes.

## Related tools

- **seq2squiggle** (Simulator that generated the R10.4.1 nanopore signals whose downstream variant calls are annotated by region class; one of the two simulators compared) — github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- **pandas** (Used in Python for the interval-overlap logic that classifies each variant into homopolymer, TRF, or neither)

## Examples

```
python annotate_region_tp.py --vcf seq2squiggle.vcf --fasta ref.fa --trf trf.bed --homopolymer-min-run 5 --out region_tp_counts.csv
```

## Evaluation signals

- region_tp_counts.csv exists and contains exactly the three region classes (homopolymer, TRF, neither) with per-simulator true-positive counts
- Every false-positive call is assigned to exactly one region class (no double-counting across homopolymer/TRF/neither)
- Homopolymer regions show a lower true-positive percentage than the other classes for both simulators
- seq2squiggle achieves a higher number of true-positive variant calls than squigulator across all three region classes, matching Supplementary Fig. S9

## Limitations

- Homopolymer regions are restricted to runs of >=5 identical bases; shorter runs are classified as TRF or neither
- TRF classification depends on the quality and coverage of Tandem Repeat Finder annotations
- The reported pattern (seq2squiggle leading in all three region classes) was established on simulated R10.4.1 data and may not transfer to other chemistries or real reads

## Evidence

- [supplementary] Homopolymer regions yielded a lower percentage of true-positive variant calls for both squigulator and seq2squiggle: [evidence span withheld — anchor status: not_found]
- [readme] seq2squiggle tool description: "seq2squiggle is a deep learning-based tool for generating artifical nanopore signals from DNA sequence data."
- [abstract] seq2squiggle superiority claim: "Seq2squiggle consistently outperforms existing tools across multiple datasets, demonstrating superior similarity to real data"
