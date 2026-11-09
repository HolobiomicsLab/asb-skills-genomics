---
name: iterative-realignment-loop-control
description: Use when hybrid-correcting erroneous long reads (PacBio/Oxford Nanopore)
  with short reads and you need an iterative loop that fixes high-confidence corrections
  first and revisits low-confidence sites until unique k-mer counts stop improving.
when_to_use_negative:
- The goal is a single-pass (core algorithm, iteration 1) correction without iterative
  learning — HECIL's core alone suffices and avoids the extra execution time of repeated
  realignment.
- Input long reads are already Canu-corrected assemblies rather than raw erroneous
  reads — only apply if you intend to further improve Canu-corrected reads as an explicit
  follow-up step.
- No high-quality Illumina short reads are available — this is a hybrid correction
  method dependent on short-read alignments; self-correcting approaches like HGAP
  are the wrong fit here.
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: HECIL
  role: 'Core iterative correction engine: quick correction (η = 0.9), optimization-based
    correction via J_k, α-percentile confidence assignment, and realignment-based
    learning'
  repo: https://github.com/NDBL/HECIL
- name: BWA-MEM
  role: Aligns short reads to long reads initially and realigns updated long reads
    each iteration; also flags mismatch/insertion/deletion positions as questionable
- name: Jellyfish
  role: Counts unique k-mers of updated long reads at each iteration to drive the
    ε relative-improvement termination criterion
- name: Samtools
  role: Dependency for alignment processing within the HECIL pipeline (bundled in
    repo, tested on version 0.1.18)
  repo: https://github.com/NDBL/HECIL
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-018-28364-3
    title: 'HECIL: A Hybrid Error Correction Algorithm for Long Reads with Iterative
      Learning'
  source_content_hash: sha256:0da5a4bd906cdc8010b607a4e3a6f95384a3c3f53ab1d5151ba37ac201d51375
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NDBL/HECIL
  tools:
  - BWA-MEM
  - HECIL
  - Jellyfish
  - Samtools
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# iterative-realignment-loop-control

## Summary

Reconstructs HECIL's fixed iterative learning control loop: quick/optimization correction of questionable long-read bases, α-percentile confidence assignment, BWA-MEM realignment of updated long reads, and Jellyfish k-mer–based termination. This lets an agent reproduce a confidence-driven correction policy that improves across successive iterations.

## When to use

Use when hybrid-correcting erroneous long reads (PacBio/Oxford Nanopore) with short reads and you need an iterative loop that fixes high-confidence corrections first and revisits low-confidence sites until unique k-mer counts stop improving.

## When NOT to use

- The goal is a single-pass (core algorithm, iteration 1) correction without iterative learning — HECIL's core alone suffices and avoids the extra execution time of repeated realignment.
- Input long reads are already Canu-corrected assemblies rather than raw erroneous reads — only apply if you intend to further improve Canu-corrected reads as an explicit follow-up step.
- No high-quality Illumina short reads are available — this is a hybrid correction method dependent on short-read alignments; self-correcting approaches like HGAP are the wrong fit here.

## Inputs

- Uncorrected long reads in FASTA (e.g. PacBio or Oxford Nanopore reads, -l LongRead.fa)
- Corresponding Illumina short reads in FASTQ (e.g. -s ShortRead.fq)
- Short-read length parameter (e.g. -len 202)
- Tunable parameters: low-confidence flag (-lc), quick-correction consensus cutoff (-c, η = 0.9), confidence threshold percentile (-k, α > 95), k-mer improvement ε ∈ (0, 1), iteration cap n

## Outputs

- Final corrected (and uncorrected) long reads in LongRead_Corrected.fasta (task artifact: hecil_iterative_corrected_reads.fasta)
- Per-iteration unique k-mer counts from Jellyfish used to assess convergence
- Correction log of monotonically fixed high-confidence corrections across iterations

## How to apply

First align short reads to long reads with BWA-MEM and mark long-read positions with mismatches, insertions, or deletions as questionable. Per iteration, run the HECIL core: quick-correct questionable bases where short-read consensus is at least 90% (η = 0.9), then optimization-correct remaining bases by maximizing J_k = ½(ω̂_q^k + ω̂_s^k), the convex combination of PHRED-derived normalized quality weight and alignment-identity normalized similarity weight, breaking ties by highest quality. Assign confidence by taking the α-percentile (α > 95) of optimal costs J_k* over corrected positions; only top corrections are high confidence and are fixed in all subsequent iterations. Realign the updated long reads to short reads with BWA-MEM, recompute the threshold μ^(ℓ) from new optimal-cost statistics, and leave low-confidence sites for the next iteration. Count unique k-mers of updated long reads with Jellyfish each iteration and terminate when the relative improvement (#unique k-mers(ℓ−1) − #unique k-mers(ℓ)) / #unique k-mers(ℓ−1) falls below ε ∈ (0, 1) or after n maximum iterations, writing final reads to hecil_iterative_corrected_reads.fasta.

## Related tools

- **HECIL** (Core iterative correction engine: quick correction (η = 0.9), optimization-based correction via J_k, α-percentile confidence assignment, and realignment-based learning) — https://github.com/NDBL/HECIL
- **BWA-MEM** (Aligns short reads to long reads initially and realigns updated long reads each iteration; also flags mismatch/insertion/deletion positions as questionable)
- **Jellyfish** (Counts unique k-mers of updated long reads at each iteration to drive the ε relative-improvement termination criterion)
- **Samtools** (Dependency for alignment processing within the HECIL pipeline (bundled in repo, tested on version 0.1.18)) — https://github.com/NDBL/HECIL

## Examples

```
python HECIL.py -l LongRead.fa -s ShortRead.fq -len 202 -o Out
```

## Evaluation signals

- The loop terminates via the k-mer relative-improvement criterion (relative improvement below ε) or the iteration cap n — never hangs indefinitely.
- The correction log shows monotonically fixed high-confidence corrections: α-percentile (α > 95) top-cost corrections from iteration ℓ are held fixed in all subsequent iterations.
- Unique k-mer counts from Jellyfish decrease monotonically (or plateau) across iterations, with HECIL's iterative output reaching the lowest number of unique k-mers compared to single-pass correction.
- Output FASTA contains all (corrected and uncorrected) long reads with the expected filename LongRead_Corrected.fasta / hecil_iterative_corrected_reads.fasta.
- Downstream validation: corrected reads aligned to reference with BLASR and assembled with Canu show improved QUAST and MUMmer dnadiff metrics versus the uncorrected baseline.

## Limitations

- Iterative improvement is only a conjecture with no mathematical proof; proving it likely requires unrealistic stochastic assumptions.
- Saturation behavior of the iterative procedure is unvalidated speculation — improvement of metrics like unique k-mer count until saturation is not guaranteed.
- Multi-threading speedup is not yet implemented in the iterative procedure (only future extension), though the workload can be decomposed into independent data-parallel tasks; single-task runs are slow.
- Dependencies are version-sensitive: BWA 0.7.15, Samtools 0.1.18, Python 2.7.8; scripts (Align_Corr.sh, bwa, samtools) may need chmod +x after cloning.

## Evidence

- [methods] Quick correction threshold: "we adopt a stricter threshold of at least 90% consensus (η = 0.9) to be eligible for quick correction"
- [methods] Confidence percentile: "Here we select α > 95 so that a small percentage of the optimal corrections are considered to be of high confidence."
- [methods] Realignment learning: "We learn in successive iterations by realigning the updated long reads to the short reads."
- [methods] Secondary termination: "we also recommend selecting a secondary termination criterion: the maximal number n of allowable iterations for iterative learning."
- [results] k-mer counting: "We employ the widely-used k-mer counting tool Jellyfish19 to compute the number of unique k-mers obtained after each correction algorithm"
- [readme] README usage: "usage: python HECIL.py [-h] -l LR -s SR -len LENGTH_SR -o OUTPUT [-lc LOW_CONFIDENCE] [-c CUTOFF_QC] [-k CONFIDENCE_THRESHOLD]"
- [abstract] Iterative learning rationale: "an iterative learning paradigm that enhances the correction policy at each iteration by incorporating knowledge gathered from previous iterations via data-driven confidence metrics"
