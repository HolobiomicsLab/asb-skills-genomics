---
name: simulated-read-benchmark-evaluation
description: Use when a new algorithmic variant (e.g., a segmentation or alignment
  strategy) must be quantified as a speedup and quality change on simulated long-read
  data with a known reference genome.
when_to_use_negative:
- The goal is benchmarking on real reads (e.g., real ONT datasets) where no ground-truth
  error profile exists for reference-based error-rate computation
- The question concerns assembly polishing of contigs (CONSENT-polish) rather than
  self-correction of raw reads
- The algorithmic change affects overlap computation rather than the MSA/segmentation
  stage, which this ablation design does not isolate
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: CONSENT
  role: Runs the self-correction pipeline twice (with and without k-mer-chaining segmentation)
    on the simulated PB read set to isolate the segmentation strategy's runtime and
    quality effect
  repo: https://github.com/morispi/CONSENT
- name: minimap2
  role: Computes the overlaps between long reads that define alignment piles for windowed
    MSA in each correction run
- name: ELECTOR
  role: Evaluates both corrected read sets against the reference to quantify post-correction
    error rate and precision
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-020-80757-5
    title: Scalable long read self-correction and assembly polishing with multiple
      sequence alignment
  source_content_hash: sha256:8aafa48959dbd547b0ec0562fe334c8def0bef29f4ee5b5c7b1e0a8ea65ae0b8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/morispi/CONSENT
  tools:
  - CONSENT
  - ELECTOR
  - minimap2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# Simulated read benchmark evaluation

## Summary

Benchmark a long-read self-correction pipeline (CONSENT) on a simulated PacBio E. coli 50x, 12%-error read set, comparing correction runs with and without the k-mer-chaining segmentation strategy by runtime and post-correction quality metrics computed with ELECTOR.

## When to use

Use when a new algorithmic variant (e.g., a segmentation or alignment strategy) must be quantified as a speedup and quality change on simulated long-read data with a known reference genome.

## When NOT to use

- The goal is benchmarking on real reads (e.g., real ONT datasets) where no ground-truth error profile exists for reference-based error-rate computation
- The question concerns assembly polishing of contigs (CONSENT-polish) rather than self-correction of raw reads
- The algorithmic change affects overlap computation rather than the MSA/segmentation stage, which this ablation design does not isolate

## Inputs

- Simulated PacBio (PB) long-read FASTA/FASTQ at 50x coverage with 12% error profile (e.g., E. coli)
- Reference genome FASTA for post-correction quality assessment
- CONSENT parameter choices for baseline vs. segmented runs (anchorSupport, minAnchors)

## Outputs

- correctedReads.fasta per run (baseline and segmented)
- Runtime measurements per correction stage (e.g., 5 h 31 min vs. 7 min)
- ELECTOR post-correction error rate and precision per run (e.g., 1.77x error-rate reduction, ~0.15% precision increase)

## How to apply

Prepare a simulated PacBio read set at 50x coverage with a 12% error profile in FASTA/FASTQ, then run CONSENT-correct with --type PB under defaults (windowSize 500, minSupport 4, maxSupport 150, k-mer size 9) as the unsegmented baseline. Repeat the correction run with the k-mer-chaining segmentation enabled (anchorSupport 8, minAnchors 2) to measure runtime against the baseline — the article reports 5 h 31 min reduced to 7 min (47x speedup). Each run computes overlaps (via minimap2), splits alignment piles into windows, builds windowed MSAs, polishes each window consensus with a local de Bruijn graph (solid k-mer threshold 4), and locally realigns the polished consensus to the read to emit correctedReads.fasta. Evaluate both corrected read sets against the reference with ELECTOR to obtain post-correction error rate and precision; the ablation is confirmed when the segmented run reproduces the 1.77x error-rate decrease and ~0.15% precision gain while preserving the runtime reduction.

## Related tools

- **CONSENT** (Runs the self-correction pipeline twice (with and without k-mer-chaining segmentation) on the simulated PB read set to isolate the segmentation strategy's runtime and quality effect) — https://github.com/morispi/CONSENT
- **minimap2** (Computes the overlaps between long reads that define alignment piles for windowed MSA in each correction run)
- **ELECTOR** (Evaluates both corrected read sets against the reference to quantify post-correction error rate and precision)

## Examples

```
./CONSENT-correct --in example/reads.fasta --out example/correctedReads.fasta --type PB
```

## Evaluation signals

- correctedReads.fasta exists for both the baseline and segmented runs
- Segmented-run runtime reproduces the reported order-of-magnitude reduction (5 h 31 min to 7 min, ~47x)
- ELECTOR reports a ~1.77x post-correction error-rate decrease for the segmented run
- ELECTOR precision increases by almost 0.15% for the segmented run relative to the baseline
- Both runs use identical non-segmentation parameters (windowSize 500, minSupport 4, maxSupport 150, merSize 9, solid 4) so the ablation is controlled

## Limitations

- The correction step runtime of CONSENT tends to be higher than that of other state-of-the-art methods, so speedups may not translate across pipelines
- Selecting only the N highest identity overlaps per pile may be inefficient, especially when the length of the repeated regions grows longer
- Window size and minimum number of anchors significantly impact runtime, and no optimal parameter compromise is established in the article

## Evidence

- [intro] k-mer-chaining segmentation speedup: "We also introduce an efficient segmentation strategy based on k-mer chaining, which allows to reduce the time footprint of the MSA."
- [discussion] local de Bruijn graph polishing per window: "it is further polished with the help of a local de Bruijn graph, at the scale of the window"
- [readme] CONSENT-correct CLI invocation and defaults: "./CONSENT-correct --in example/reads.fasta --out example/correctedReads.fasta --type PB"
- [readme] segmentation parameter defaults: "--anchorSupport INT, -c INT:   Minimum number of sequences supporting (Ai) - (Ai+1) to keep the two anchors in the chaining. (default: 8)"
