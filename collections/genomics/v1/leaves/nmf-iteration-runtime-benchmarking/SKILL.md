---
name: nmf-iteration-runtime-benchmarking
description: Use when refitting a user-defined COSMIC signature matrix P to mutation
  count matrix V and you need a defensible runtime comparison of InverseNMF-style
  multiplicative-update refitting against deconstructSigs and MutationalPatterns baselines.
when_to_use_negative:
- The goal is de novo signature extraction rather than refitting a fixed signature
  set — the article notes RNMF's de novo framework (denovoNMF with initial-value optimization
  and solution-space construction) spends more time than previously published software
  and is not the fast path being benchmarked
- Input is not a single-sample or small-cohort mutation catalog requiring signature
  refitting — InverseNMF is designed to identify mutational signatures within a small
  data set or a single tumor sample
tools:
- name: RNMF
  role: Provides the InverseNMF refitting procedure whose multiplicative-update runtime
    is benchmarked against baselines
  repo: https://github.com/zhenzhang-li/RNMF
- name: R
  role: Implementation language and execution environment for RNMF, deconstructSigs,
    and MutationalPatterns benchmark runs
  repo: https://github.com/zhenzhang-li/RNMF
- name: BSgenome.Hsapiens.UCSC.hg19
  role: Bioconductor genome library used to convert the MAF/VCF catalog into the observed
    mutation count matrix V
- name: deconstructSigs
  role: Refitting baseline run on the identical ESCC catalog and COSMIC signature
    set to measure the ~12x speed ratio
- name: MutationalPatterns
  role: Refitting baseline whose runtime InverseNMF's operating speed should rival
    on the same input
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/cam4.4717
    title: A practical framework <scp>RNMF</scp> for exploring the association between
      mutational signatures and genes using gene cumulative contribution abundance
  source_content_hash: sha256:704a01771d958b2f359fe032018f961f92ba25e836f49e71d4dc4c86705285e6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhenzhang-li/RNMF
  tools:
  - BSgenome.Hsapiens.UCSC.hg19
  - MutationalPatterns
  - R
  - RNMF
  - deconstructSigs
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# nmf-iteration-runtime-benchmarking

## Summary

Benchmarks the wall-clock runtime of RNMF's InverseNMF signature-refitting procedure (fixed P, multiplicative update of S) against deconstructSigs and MutationalPatterns on the same ESCC mutation catalog, to demonstrate that InverseNMF is ~12x faster than deconstructSigs and comparable in speed to MutationalPatterns.

## When to use

Use when refitting a user-defined COSMIC signature matrix P to mutation count matrix V and you need a defensible runtime comparison of InverseNMF-style multiplicative-update refitting against deconstructSigs and MutationalPatterns baselines.

## When NOT to use

- The goal is de novo signature extraction rather than refitting a fixed signature set — the article notes RNMF's de novo framework (denovoNMF with initial-value optimization and solution-space construction) spends more time than previously published software and is not the fast path being benchmarked
- Input is not a single-sample or small-cohort mutation catalog requiring signature refitting — InverseNMF is designed to identify mutational signatures within a small data set or a single tumor sample

## Inputs

- ESCC somatic mutation catalog in MAF or VCF format (e.g. ESCC.exon.ALL.input.maf)
- User-defined COSMIC signature matrix P (e.g. COSMIC Mutational Signatures v3.2 / v2.0 SBS matrix)
- Observed mutation count matrix V built via Bioconductor BSgenome.Hsapiens.UCSC.hg19

## Outputs

- runtime_benchmark.csv with per-tool elapsed wall-clock times and InverseNMF speed ratios vs deconstructSigs and MutationalPatterns
- Converged InverseNMF contribution matrix S with sample-level signature contributions

## How to apply

Load the ESCC somatic mutation catalog (MAF or VCF) into the RNMF R package and build the observed mutation count matrix V using the Bioconductor BSgenome.Hsapiens.UCSC.hg19 library. Fix the user-defined COSMIC signature matrix P and run the InverseNMF iterative multiplicative update S' = S * P^T V / (P^T P S + gamma) until convergence or up to the 100,000,000-iteration cap, recording elapsed wall-clock time. Then run deconstructSigs and MutationalPatterns refitting on the identical ESCC catalog with the same COSMIC signature set, recording their wall-clock times. Compute InverseNMF's speed ratio against each baseline (expected ~12x over deconstructSigs, comparable to MutationalPatterns) and write per-tool runtimes and ratios to runtime_benchmark.csv, while confirming the refitted S matrix converges within the iteration cap and its contribution scores are highly consistent with both baselines.

## Related tools

- **RNMF** (Provides the InverseNMF refitting procedure whose multiplicative-update runtime is benchmarked against baselines) — https://github.com/zhenzhang-li/RNMF
- **R** (Implementation language and execution environment for RNMF, deconstructSigs, and MutationalPatterns benchmark runs) — https://github.com/zhenzhang-li/RNMF
- **BSgenome.Hsapiens.UCSC.hg19** (Bioconductor genome library used to convert the MAF/VCF catalog into the observed mutation count matrix V)
- **deconstructSigs** (Refitting baseline run on the identical ESCC catalog and COSMIC signature set to measure the ~12x speed ratio)
- **MutationalPatterns** (Refitting baseline whose runtime InverseNMF's operating speed should rival on the same input)

## Examples

```
S_new <- S * (t(P) %*% V) / (t(P) %*% (P %*% S) + gamma)
```

## Evaluation signals

- InverseNMF speedup ratio over deconstructSigs reaches approximately 12x on the same ESCC input data
- InverseNMF runtime is comparable to (rivals) MutationalPatterns on the identical catalog
- Refitted S matrix converges within the 100,000,000-iteration cap of the multiplicative update
- Contribution scores from InverseNMF are highly consistent with both deconstructSigs and MutationalPatterns baselines
- runtime_benchmark.csv contains per-tool runtimes and speed ratios for all three tools on the same signature set

## Limitations

- The 12x speedup and MutationalPatterns comparability were demonstrated on one ESCC cohort input; results depend on the specific mutation catalog and signature set used
- The paper's discussion notes the framework spends more time on de novo signature extraction (due to the 'Re-updated the initial value' correction and 'build a solution space' processes) — the fast runtime claim applies to InverseNMF refitting, not the de novo pipeline
- Runtime comparisons are wall-clock timings tied to the hardware and parallelization settings used (e.g. mc.cores in the RNMF README example)

## Evidence

- [methods] InverseNMF design and scope: "we also developed a reversible method named InverseNMF, which can identify mutational signatures within a small data set or a single tumor sample"
- [methods] Genome build dependency for building V: "Then it relies on the Bioconductor library, such as BS.genome.Hsapiens.UCSC.hg19 or BS.genome.Hsapiens.UCSC.hg38"
- [methods] Supported input formats for the mutation catalog: "somatic variants can be imported from a Variant Call Format (VCF) file or a Mutation Annotation Format (MAF) file"
- [readme] Concrete R invocation pattern from the RNMF README: "denovoNMF(originalGenomes, sampleNames, subtypes, kmin = 1, kmax = 15, steptol = 10^-9, mc.cores = 20)"
