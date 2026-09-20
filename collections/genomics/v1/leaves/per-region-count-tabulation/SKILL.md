---
name: per-region-count-tabulation
description: Use when you have variant-calling results (e.g. VCF-style call sets from
  two simulators on R10.4.
when_to_use_negative:
- You only need genome-wide aggregate precision/recall with no stratification by genomic
  context — a single confusion matrix suffices
- The call set has not yet been resolved into true-positive vs false-positive calls
  (run the upstream variant-calling/truth-comparison task first)
- Regions of interest are defined by something other than sequence-context classes
  (e.g. gene bodies or GC bins), for which this three-class scheme does not apply
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3673
tools:
- name: pandas
  role: Implements the interval-overlap classification of each variant position into
    exactly one of the three region classes and builds the per-region count table
- name: seq2squiggle
  role: One of the two simulators whose R10.4.1 variant call sets are stratified by
    region class; its TP counts per region are compared against squigulator
  repo: github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- name: squigulator
  role: Comparator simulator whose per-region true-positive and false-positive counts
    are tabulated alongside seq2squiggle's
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae744
    title: End-to-end simulation of nanopore sequencing signals with feed-forward
      transformers
  source_content_hash: sha256:de5940f8392bfc971565df261e9389e47156d80eb2b77f899b0617df89d1d935
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - pandas
  - seq2squiggle
  - squigulator
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# per-region-count-tabulation

## Summary

Tabulates true-positive and false-positive variant calls per genomic region class (homopolymer runs >=5 identical bases, TRF tandem repeats, or neither) for each simulator, quantifying where call accuracy degrades. It converts a flat VCF-style call set into a per-region TP/FP contingency table that exposes context-dependent simulator error.

## When to use

Use when you have variant-calling results (e.g. VCF-style call sets from two simulators on R10.4.1 data) and need to know whether false-positive SNV/indel calls concentrate in homopolymer or tandem-repeat regions versus neutral sequence, or must compare simulators by true-positive counts per region class.

## When NOT to use

- You only need genome-wide aggregate precision/recall with no stratification by genomic context — a single confusion matrix suffices
- The call set has not yet been resolved into true-positive vs false-positive calls (run the upstream variant-calling/truth-comparison task first)
- Regions of interest are defined by something other than sequence-context classes (e.g. gene bodies or GC bins), for which this three-class scheme does not apply

## Inputs

- VCF-style variant call sets (true positives and false positives) per simulator on R10.4.1 simulated data
- Reference genome sequence for detecting homopolymer runs of >=5 identical bases
- Tandem Repeat Finder (TRF) repeat-region annotations (BED/interval style)

## Outputs

- region_tp_counts.csv — per-region-class table of true-positive counts per simulator
- Per-region-class true-positive percentage and false-positive counts for homopolymer, TRF, and neither classes

## How to apply

Load the upstream variant-calling results (VCF-style call sets for both squigulator and seq2squiggle on R10.4.1 simulated data). Annotate each variant position against homopolymer regions defined as runs of >=5 identical bases, then against Tandem Repeat Finder (TRF) repeat regions. Classify each false-positive call into exactly one of three region classes — homopolymer, TRF, or neither — using pandas interval-overlap logic so classes are mutually exclusive. Count true-positive and false-positive calls per region class per simulator and compute the true-positive percentage per region. Write the per-region table of true-positive counts to region_tp_counts.csv; expect homopolymers to yield the lowest TP percentage and seq2squiggle to lead in all three classes (Supplementary Fig. S9).

## Related tools

- **pandas** (Implements the interval-overlap classification of each variant position into exactly one of the three region classes and builds the per-region count table)
- **seq2squiggle** (One of the two simulators whose R10.4.1 variant call sets are stratified by region class; its TP counts per region are compared against squigulator) — github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- **squigulator** (Comparator simulator whose per-region true-positive and false-positive counts are tabulated alongside seq2squiggle's)

## Examples

```
python stratify_by_region.py --vcf squigulator.vcf seq2squiggle.vcf --reference ref.fa --trf trf.bed --homopolymer-min 5 --out region_tp_counts.csv
```

## Evaluation signals

- region_tp_counts.csv exists and contains all three region classes (homopolymer, TRF, neither) with per-simulator true-positive counts
- Every false-positive call is assigned to exactly one region class — no call counted in two classes, none unclassified
- Homopolymer class shows a lower true-positive percentage than the other classes for both simulators, matching Supplementary Fig. S9
- seq2squiggle's true-positive count exceeds squigulator's in all three region classes, reproducing the reported pattern
- True-positive percentage per region computed as TP / (TP + FP) reconciles with the raw per-class counts in the table

## Limitations

- The region classes are binary and exhaustive only by construction (homopolymer >=5 bp, TRF, otherwise 'neither'); variants near region boundaries depend on the interval-overlap tie-breaking implemented in pandas
- The '>=5 identical bases' homopolymer threshold is fixed; results are not directly comparable to analyses using different homopolymer definitions
- Interpretation is limited to the R10.4.1 simulated data and the specific variant caller used upstream; error profiles may differ on real reads or other chemistries

## Evidence

- [other] Annotate each variant position against homopolymer regions defined as runs of >=5 identical bases.: [evidence span withheld — anchor status: not_found]
- [other] Classify each false-positive call into exactly one of three region classes.: [evidence span withheld — anchor status: not_found]
- [results] Homopolymer regions yielded a lower percentage of true-positive variant calls.: [evidence span withheld — anchor status: not_found]
- [abstract] Seq2squiggle consistently outperforms existing tools across multiple datasets.: "Seq2squiggle consistently outperforms existing tools across multiple datasets, demonstrating superior similarity to real data and offering a robust solution for simulating nanopore sequencing signals"
- [readme] seq2squiggle requires Python >= 3.10.: [evidence span withheld — anchor status: external_doc]
