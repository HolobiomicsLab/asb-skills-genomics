---
name: confidence-percentile-assignment-for-corrections
description: Use when iteratively correcting erroneous long reads and you must designate
  a small subset of corrections (α > 95 percentile of optimal costs J_k*) as high
  confidence to fix before realignment.
when_to_use_negative:
- You only need single-pass correction — HECIL's core algorithm at iteration 1 already
  yields its published k-mer results without iterative learning
- All questionable positions must remain revisitable across runs; fixing high-confidence
  sites permanently prevents revisiting them
- You expect guaranteed monotone improvement — the paper notes iterative improvement
  is a conjecture with no mathematical proof
edam_operation: http://edamontology.org/operation_0294
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: HECIL
  role: Implements the confidence assignment (α-percentile of optimal costs J_k*),
    realignment loop, and termination criteria; configurable via -k CONFIDENCE_THRESHOLD
  repo: https://github.com/NDBL/HECIL
- name: BWA-MEM
  role: Aligns the updated long reads back to the short reads each iteration so confidence
    threshold μ^(ℓ) can be recomputed
- name: Jellyfish
  role: Counts unique k-mers of updated long reads each iteration to evaluate relative
    improvement and decide loop termination
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-018-28364-3
    title: 'HECIL: A Hybrid Error Correction Algorithm for Long Reads with Iterative
      Learning'
  source_content_hash: sha256:e223dcc000d51ae873360f6e0083342c2473bf87f3ba3ef2cec08152883cc969
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NDBL/HECIL
  tools:
  - BWA-MEM
  - HECIL
  - Jellyfish
  edam_operation: http://edamontology.org/operation_0294
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# confidence-percentile-assignment-for-corrections

## Summary

HECIL's confidence assignment step ranks per-iteration corrections by their optimal decision cost J_k* and locks in only the top α-percentile (α > 95) as high confidence, fixing them in all subsequent iterative-learning iterations. It drives the realignment-based learning loop rather than one-shot correction.

## When to use

Use when iteratively correcting erroneous long reads and you must designate a small subset of corrections (α > 95 percentile of optimal costs J_k*) as high confidence to fix before realignment.

## When NOT to use

- You only need single-pass correction — HECIL's core algorithm at iteration 1 already yields its published k-mer results without iterative learning
- All questionable positions must remain revisitable across runs; fixing high-confidence sites permanently prevents revisiting them
- You expect guaranteed monotone improvement — the paper notes iterative improvement is a conjecture with no mathematical proof

## Inputs

- Long reads in FASTA (e.g. PacBio/Sequel or Oxford Nanopore reads, -l LR)
- Short reads in FASTQ (e.g. Illumina reads, -s SR)
- BWA-MEM alignments of short reads onto long reads with mismatch/insertion/deletion positions marked questionable
- Optimal correction costs J_k* per corrected position from the optimization-based correction step

## Outputs

- Set of high-confidence corrections (top α > 95 percentile of J_k*) fixed in all subsequent iterations
- Recomputed confidence threshold μ^(ℓ) per iteration ℓ
- Updated long reads for realignment in the next iteration
- Final corrected long reads written to hecil_iterative_corrected_reads.fasta (LongRead_Corrected.fasta in the README) with per-iteration unique k-mer counts

## How to apply

After each HECIL iteration's optimization-based correction, collect the optimal costs J_k* (the convex combination ½(ω̂_q + ω̂_s) of the PHRED-derived normalized quality weight and the alignment-identity normalized similarity weight) over all corrected positions. Compute the α-percentile (select α > 95) of these costs and designate only the top corrections as high confidence; fix these bases in all subsequent iterations. Realign the updated long reads to the short reads with BWA-MEM and recompute the confidence threshold μ^(ℓ) from the new optimal-cost statistics, leaving low-confidence sites to be revisited by the core algorithm (quick correction at η = 0.9 consensus, then optimization-based correction). The learning loop terminates when the relative improvement in Jellyfish unique k-mer counts falls below ε ∈ (0, 1) or when the maximal number n of allowable iterations is reached. The rationale is to accumulate only data-driven, low-cost corrections so later alignments are built on increasingly trustworthy bases.

## Related tools

- **HECIL** (Implements the confidence assignment (α-percentile of optimal costs J_k*), realignment loop, and termination criteria; configurable via -k CONFIDENCE_THRESHOLD) — https://github.com/NDBL/HECIL
- **BWA-MEM** (Aligns the updated long reads back to the short reads each iteration so confidence threshold μ^(ℓ) can be recomputed)
- **Jellyfish** (Counts unique k-mers of updated long reads each iteration to evaluate relative improvement and decide loop termination)

## Examples

```
python HECIL.py -l LongRead.fa -s ShortRead.fq -len 202 -o Out
```

## Evaluation signals

- Only corrections whose optimal cost J_k* falls above the α-percentile (α > 95) are designated high confidence and fixed in later iterations
- The correction log shows monotonically fixed high-confidence corrections accumulating across iterations
- The confidence threshold μ^(ℓ) is recomputed from new optimal-cost statistics after each BWA-MEM realignment
- The loop terminates on the k-mer relative-improvement criterion ((#unique k-mers(ℓ−1) − #unique k-mers(ℓ)) / #unique k-mers(ℓ−1) < ε) or the iteration cap n
- Per-iteration unique k-mer counts (Jellyfish) do not increase, indicating corrections are not degrading the reads

## Limitations

- Iterative improvement is a conjecture; a rigorous mathematical framework for proving it likely requires unrealistic stochastic assumptions
- Saturation behavior of the iterative procedure is unvalidated speculation
- The iterative formalism improves performance at the expense of total execution time; multi-threading speedup was not yet implemented (only decomposition into independent data-parallel tasks)

## Evidence

- [methods] Assignment of confidence uses the α-percentile of optimal costs: "Here we select α > 95 so that a small percentage of the optimal corrections are considered to be of high confidence."
- [methods] Learning proceeds by realigning updated long reads: "We learn in successive iterations by realigning the updated long reads to the short reads."
- [methods] Secondary termination criterion caps iterations: "we also recommend selecting a secondary termination criterion: the maximal number n of allowable iterations for iterative learning."
- [abstract] Iterative learning paradigm rationale: "an iterative learning paradigm that enhances the correction policy at each iteration"
- [readme] CLI exposes confidence threshold option: "python HECIL.py [-h] -l LR -s SR -len LENGTH_SR -o OUTPUT [-lc LOW_CONFIDENCE] [-c CUTOFF_QC] [-k CONFIDENCE_THRESHOLD]"
