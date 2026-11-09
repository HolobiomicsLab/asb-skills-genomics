---
name: variant-call-region-classification
description: Use when comparing variant-calling performance on simulated R10.4.1 nanopore
  data across genomic contexts, i.e. when you have per-simulator call sets (VCF-style)
  and need per-region true/false-positive counts.
when_to_use_negative:
- Input is raw nanopore signal (BLOW5/POD5) rather than a processed variant call set
  — signal simulation is a separate upstream task
- No region annotation (homopolymer definition or TRF bed) is available for the reference,
  making three-class classification impossible
- Calls come from real sequencing data where the simulator-comparison framing (per-simulator
  true-positive ranking) does not apply
edam_operation: http://edamontology.org/operation_3227
tools:
- name: seq2squiggle
  role: Transformer-based nanopore signal simulator whose R10.4.1 variant calls are
    one of the two call sets classified by region
  repo: https://github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- name: Python
  role: Implementation language for the interval-overlap classification of call positions
- name: pandas
  role: Used for the interval overlap logic assigning each call to homopolymer, TRF,
    or neither
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae744
    title: End-to-end simulation of nanopore sequencing signals with feed-forward
      transformers
  source_content_hash: sha256:3aef2d948404827d61a23eb4a3803d2fe44d992c5416fd0f26b06d5899dd2ad0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZKI-PH-ImageAnalysis/seq2squiggle
  tools:
  - Python
  - pandas
  - seq2squiggle
  edam_operation: http://edamontology.org/operation_3227
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# variant-call-region-classification

## Summary

Classifies variant calls (true and false positives) from nanopore simulators into homopolymer regions (runs of >=5 identical bases), Tandem Repeat Finder (TRF) repeat regions, or neither, to benchmark simulator-induced error by genomic context. It quantifies which simulator (e.g. seq2squiggle vs squigulator) yields more true-positive calls per region class.

## When to use

Use when comparing variant-calling performance on simulated R10.4.1 nanopore data across genomic contexts, i.e. when you have per-simulator call sets (VCF-style) and need per-region true/false-positive counts.

## When NOT to use

- Input is raw nanopore signal (BLOW5/POD5) rather than a processed variant call set — signal simulation is a separate upstream task
- No region annotation (homopolymer definition or TRF bed) is available for the reference, making three-class classification impossible
- Calls come from real sequencing data where the simulator-comparison framing (per-simulator true-positive ranking) does not apply

## Inputs

- VCF-style variant call sets for R10.4.1 simulated reads, one per simulator (seq2squiggle, squigulator)
- Reference genome sequence (FASTA) for defining homopolymer runs of >=5 identical bases
- Tandem Repeat Finder (TRF) repeat-region annotations

## Outputs

- region_tp_counts.csv: per-region-class true-positive counts per simulator
- Per-region true/false-positive counts and true-positive percentages for homopolymer, TRF, and neither classes

## How to apply

Load the VCF-style variant-calling results produced per simulator (here: seq2squiggle and squigulator on R10.4.1 simulated reads) from the upstream variant-calling task. Annotate each variant position against homopolymer regions defined as runs of >=5 identical bases, and against Tandem Repeat Finder (TRF) repeat regions. Assign each false-positive call to exactly one of three region classes — homopolymer, TRF, or neither — using pandas for the interval-overlap logic. Count true-positive and false-positive calls per region class per simulator and compute the true-positive percentage per region; write the per-region table of true-positive counts to region_tp_counts.csv. The expected pattern (Supplementary Fig. S9) is that homopolymers yield the lower true-positive percentage for both simulators while seq2squiggle leads in all three region classes.

## Related tools

- **seq2squiggle** (Transformer-based nanopore signal simulator whose R10.4.1 variant calls are one of the two call sets classified by region) — https://github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- **Python** (Implementation language for the interval-overlap classification of call positions)
- **pandas** (Used for the interval overlap logic assigning each call to homopolymer, TRF, or neither)

## Examples

```
python classify_calls_by_region.py --calls squigulator.vcf,seq2squiggle.vcf --fasta ref.fasta --trf trf.bed --hp-min 5 --out region_tp_counts.csv
```

## Evaluation signals

- region_tp_counts.csv exists and contains exactly the three region classes: homopolymer, TRF, neither, with per-simulator true-positive counts
- Every false-positive call is assigned to exactly one (non-overlapping) region class
- Homopolymer class shows the lower true-positive percentage for both squigulator and seq2squiggle
- seq2squiggle achieves a higher number of true-positive calls than squigulator in all three region classes, matching Supplementary Fig. S9

## Limitations

- Region classes rely on a fixed homopolymer definition (runs of >=5 identical bases) and TRF annotations, so results are sensitive to those thresholds
- Conclusions are specific to simulated R10.4.1 nanopore data and the two simulators compared (seq2squiggle, squigulator); behavior on other chemistries or simulators is not established

## Evidence

- [supplementary] Homopolymer regions yielded a lower percentage of true-positive variant calls for both squigulator and seq2squiggle, but seq2squiggle achieved a higher number of true-positive variant calls: [evidence span withheld — anchor status: not_found]
- [abstract] Seq2squiggle consistently outperforms existing tools across multiple datasets, demonstrating superior similarity to real data: "Seq2squiggle consistently outperforms existing tools across multiple datasets, demonstrating superior similarity to real data and offering a robust solution for simulating nanopore sequencing signals"
- [readme] `seq2squiggle` is a deep learning-based tool for generating artifical nanopore signals from DNA sequence data.: "`seq2squiggle` is a deep learning-based tool for generating artifical nanopore signals from DNA sequence data."
