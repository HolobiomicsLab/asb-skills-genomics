---
name: tandem-repeat-region-annotation
description: Use when you have VCF-style variant call sets from two or more simulators
  (e.g. seq2squiggle vs squigulator) and need to stratify TP/FP calls by repeat context
  (homopolymer, TRF, neither) to explain error patterns.
when_to_use_negative:
- Input is raw nanopore signal data (BLOW5/POD5) rather than a processed variant call
  set — annotation requires called variant positions
- The reference genome has no TRF annotation available and generating one is out of
  scope; homopolymer-only stratification would be a simpler alternative
- Variant calls come from a single simulator, preventing the per-simulator comparison
  that motivates the classification
edam_operation: http://edamontology.org/operation_3281
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: seq2squiggle
  role: Transformer-based nanopore signal simulator whose R10.4.1 variant-calling
    output provides one of the two call sets stratified by repeat region
  repo: github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- name: Python
  role: Implementation language for the interval-overlap and counting logic
- name: pandas
  role: Dataframe library used to implement the interval overlap logic and aggregate
    TP/FP counts per region class
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae744
    title: End-to-end simulation of nanopore sequencing signals with feed-forward
      transformers
  source_content_hash: sha256:4e9c1093b92dbb1ebd6e652b42ed74c0d51828732627b2e83099858f9fb4ec7e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - pandas
  - seq2squiggle
  edam_operation: http://edamontology.org/operation_3281
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# tandem-repeat-region-annotation

## Summary

Annotate variant call positions against tandem repeat (TRF) and homopolymer (runs of >=5 identical bases) regions, then classify true-positive and false-positive SNV/indel calls per region class to evaluate simulator-dependent variant-calling accuracy.

## When to use

Use when you have VCF-style variant call sets from two or more simulators (e.g. seq2squiggle vs squigulator) and need to stratify TP/FP calls by repeat context (homopolymer, TRF, neither) to explain error patterns.

## When NOT to use

- Input is raw nanopore signal data (BLOW5/POD5) rather than a processed variant call set — annotation requires called variant positions
- The reference genome has no TRF annotation available and generating one is out of scope; homopolymer-only stratification would be a simpler alternative
- Variant calls come from a single simulator, preventing the per-simulator comparison that motivates the classification

## Inputs

- VCF-style variant call sets (true-positive and false-positive SNV/indel calls) per simulator, from R10.4.1 simulated data
- Reference genome FASTA for detecting homopolymer runs (>=5 identical bases)
- Tandem Repeat Finder (TRF) repeat-region annotations (BED-style intervals)

## Outputs

- region_tp_counts.csv — per-region-class table of true-positive counts per simulator
- Per-region true-positive percentage summary across region classes (homopolymer, TRF, neither)

## How to apply

Load the R10.4.1 variant-calling results (VCF-style call sets per simulator) from the upstream variant-calling task. Annotate each variant position against homopolymer regions defined as runs of >=5 identical bases and against Tandem Repeat Finder (TRF) repeat regions. Classify each call into exactly one of three mutually exclusive region classes — homopolymer, TRF region, or neither — implementing the interval-overlap logic with pandas. Count true-positive and false-positive calls per region class per simulator and compute the true-positive percentage per region. Write the per-region true-positive table to region_tp_counts.csv and check it reproduces the reported pattern (lower TP percentage in homopolymers; seq2squiggle leading in all three classes, Supplementary Fig. S9).

## Related tools

- **seq2squiggle** (Transformer-based nanopore signal simulator whose R10.4.1 variant-calling output provides one of the two call sets stratified by repeat region) — github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- **Python** (Implementation language for the interval-overlap and counting logic)
- **pandas** (Dataframe library used to implement the interval overlap logic and aggregate TP/FP counts per region class)

## Examples

```
python annotate_regions.py --calls squigulator.vcf seq2squiggle.vcf --trf trf_regions.bed --reference ref.fasta --homopolymer-min 5 --output region_tp_counts.csv
```

## Evaluation signals

- region_tp_counts.csv exists and contains exactly the three region classes: homopolymer, TRF, neither
- Each false-positive call is assigned to exactly one region class (no overlaps, no unclassified rows)
- True-positive percentage is lower in the homopolymer class than in the other classes for both simulators
- seq2squiggle shows a higher number of true-positive variant calls than squigulator in all three region classes
- TP and FP counts summed over the three region classes equal the totals in the input call sets (partition invariant)

## Limitations

- Homopolymer definition is fixed at runs of >=5 identical bases; alternative thresholds change the classification
- Region classes are mutually exclusive with a fixed precedence, so variants overlapping both a homopolymer and a TRF region are counted once rather than in both
- Conclusions are specific to R10.4.1 chemistry simulated reads; behaviour on real reads or other pore chemistries is not established by this analysis

## Evidence

- [other] workflow step: annotate homopolymers and TRF classes: [evidence span withheld — anchor status: not_found]
- [other] workflow step: interval overlap and output: [evidence span withheld — anchor status: not_found]
- [other] expected pattern per Supplementary Fig. S9: [evidence span withheld — anchor status: not_found]
- [readme] pandas dependency from repository setup: "poetry add scikit-learn
poetry add pandas"
