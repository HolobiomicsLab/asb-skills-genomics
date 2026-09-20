---
name: kmer-based-iteration-termination-evaluation
description: Use when running HECIL's iterative learning component on long-read/short-read
  hybrid correction and you need a data-driven stopping criterion and per-iteration
  quality metric based on unique k-mer counts.
when_to_use_negative:
- You only want HECIL's core algorithm (iteration 1) result to compare against other
  hybrid correctors — no iteration loop or termination criterion is needed
- You are self-correcting long reads without short-read data (e.g. HGAP-style), since
  HECIL's correction weights and k-mer evaluation depend on aligned short reads
- Your quality assessment must come from assembly-based metrics (Canu + QUAST + MUMmer
  dnadiff) rather than per-iteration k-mer counts
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: HECIL
  role: Hybrid error-correction framework whose iterative learning loop this skill
    monitors and terminates via k-mer improvement
  repo: https://github.com/NDBL/HECIL
- name: Jellyfish
  role: k-mer counting tool used to compute the number of unique k-mers obtained after
    each correction algorithm/iteration
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-018-28364-3
    title: 'HECIL: A Hybrid Error Correction Algorithm for Long Reads with Iterative
      Learning'
  source_content_hash: sha256:4b9ad8bd2409da56e7ae5c0d3c585e2fbb5f940710ffc78c349f7656ef68dc4a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NDBL/HECIL
  tools:
  - BWA-MEM
  - HECIL
  - Jellyfish
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# kmer-based-iteration-termination-evaluation

## Summary

A decision rule for HECIL's iterative learning loop that counts unique k-mers (via Jellyfish) after each correction iteration to measure correction quality and to terminate iterations, since better corrections collapse erroneous k-mers into fewer unique k-mers.

## When to use

Use when running HECIL's iterative learning component on long-read/short-read hybrid correction and you need a data-driven stopping criterion and per-iteration quality metric based on unique k-mer counts.

## When NOT to use

- You only want HECIL's core algorithm (iteration 1) result to compare against other hybrid correctors — no iteration loop or termination criterion is needed
- You are self-correcting long reads without short-read data (e.g. HGAP-style), since HECIL's correction weights and k-mer evaluation depend on aligned short reads
- Your quality assessment must come from assembly-based metrics (Canu + QUAST + MUMmer dnadiff) rather than per-iteration k-mer counts

## Inputs

- HECIL-corrected long reads per iteration (LongRead_Corrected.fasta)
- Uncorrected long reads (FASTA) and aligned short reads (FASTQ) feeding each HECIL iteration
- User-specified epsilon (relative improvement threshold) and maximal iterations n

## Outputs

- Unique k-mer counts per correction iteration (Jellyfish)
- Termination decision at the first iteration where relative k-mer improvement < epsilon or iteration n is reached
- Per-iteration metrics table, e.g. hecil_afunestus_flowcell_metrics.csv with PI and % matched bases per flowcell and iteration

## How to apply

After each HECIL correction iteration, count unique k-mers in the corrected long reads with Jellyfish. HECIL's iterative learning fixes high-confidence corrections (alpha > 95 percentile of optimal costs) and realigns updated long reads to short reads; better corrections monotonically reduce unique k-mers, so track the relative improvement in unique k-mers between two successive iterations. Terminate when that relative improvement falls below epsilon, or apply the recommended secondary criterion: the maximal allowed number n of iterations. Because the iterative improvement property is presented as a conjecture (no rigorous proof), the k-mer series itself is your empirical check that iterations are helping. For the A. funestus (Fumoz) flowcells, log per-iteration metrics to hecil_afunestus_flowcell_metrics.csv and confirm the reproduced PI/matched-base values (e.g. PI rising from 94.5 to 98.5 on flowcell 1) alongside the monotonic k-mer decrease.

## Related tools

- **HECIL** (Hybrid error-correction framework whose iterative learning loop this skill monitors and terminates via k-mer improvement) — https://github.com/NDBL/HECIL
- **Jellyfish** (k-mer counting tool used to compute the number of unique k-mers obtained after each correction algorithm/iteration)

## Examples

```
python HECIL.py -l LongRead.fa -s ShortRead.fq -len 202 -o Out
```

## Evaluation signals

- Unique k-mer counts decrease monotonically across iterations until the relative improvement between successive iterations drops below epsilon
- Termination occurs either at epsilon saturation or at the maximal allowed iteration count n — never later
- Reproduced PI and matched-base values agree with reported flowcell results (e.g. PI 94.5 → 98.5 on flowcell 1; 87.5% matched bases with 3.8% improvement on flowcell 16 at iteration 5)
- Iteration-1 (core algorithm) unique k-mer count matches or beats the lowest k-mer count among compared tools for the dataset
- HECIL output LongRead_Corrected.fasta contains all corrected and uncorrected long reads

## Limitations

- Iterative improvement until saturation is a conjecture; no rigorous mathematical proof exists without possibly unrealistic stochastic assumptions
- epsilon and n must be chosen by the user; the article recommends n as a secondary termination criterion rather than relying on epsilon alone
- Each additional iteration costs execution time, and the iterative-learning formalism trades runtime for metric gains

## Evidence

- [results] We employ the widely-used k-mer counting tool Jellyfish19 to compute the number of unique k-mers obtained after each correction algorithm: "We employ the widely-used k-mer counting tool Jellyfish19 to compute the number of unique k-mers obtained after each correction algorithm"
- [methods] secondary termination criterion: maximal iterations: "we also recommend selecting a secondary termination criterion: the maximal number n of allowable iterations for iterative learning."
- [discussion] iterative improvement is unproven conjecture: "Note that this is a conjecture, laying a rigorous mathematical framework for proving iterative improvement likely cannot be made without making"
- [readme] README usage of HECIL.py: "python HECIL.py [-h] -l LR -s SR -len LENGTH_SR -o OUTPUT [-lc LOW_CONFIDENCE] [-c CUTOFF_QC] [-k CONFIDENCE_THRESHOLD]"
