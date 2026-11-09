---
name: cross-tool-performance-comparison
description: Use when a new signature-refitting tool claims speed advantages and you
  need a controlled same-input runtime comparison against baseline tools, recording
  per-tool elapsed wall-clock times and ratios.
when_to_use_negative:
- When the goal is de novo signature extraction rather than refitting — the article
  states the RNMF de novo framework 'spends more time on a de novo extraction of signature
  analysis' than previously published software
- When the mutation catalog or signature set differs across tools, since the speed
  comparison is only meaningful on the identical ESCC input and the same COSMIC signature
  set
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: RNMF
  role: Provides the InverseNMF refitting procedure whose runtime is benchmarked against
    the baselines
  repo: https://github.com/zhenzhang-li/RNMF
- name: R
  role: Language in which RNMF, the timing harness, and the comparison are implemented
- name: BSgenome.Hsapiens.UCSC.hg19
  role: Bioconductor library used to build the observed mutation count matrix V from
    the ESCC catalog
- name: deconstructSigs
  role: Refitting baseline run on the identical ESCC catalog with the same COSMIC
    signature set; expected to be ~12x slower than InverseNMF
- name: MutationalPatterns
  role: Refitting baseline run on the identical ESCC catalog; InverseNMF's runtime
    is expected to be comparable to it
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/cam4.4717
    title: A practical framework <scp>RNMF</scp> for exploring the association between
      mutational signatures and genes using gene cumulative contribution abundance
  source_content_hash: sha256:e6de12f6608800daa3d2f17ada96a0d07a80c59d13fdb49380427a296dadef14
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
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# cross-tool-performance-comparison

## Summary

Benchmark the operating speed of a new method (RNMF's InverseNMF signature-refitting procedure) against established baselines (deconstructSigs, MutationalPatterns) by running all tools on the identical ESCC mutation catalog and COSMIC signature set, then computing wall-clock speed ratios.

## When to use

Use when a new signature-refitting tool claims speed advantages and you need a controlled same-input runtime comparison against baseline tools, recording per-tool elapsed wall-clock times and ratios.

## When NOT to use

- When the goal is de novo signature extraction rather than refitting — the article states the RNMF de novo framework 'spends more time on a de novo extraction of signature analysis' than previously published software
- When the mutation catalog or signature set differs across tools, since the speed comparison is only meaningful on the identical ESCC input and the same COSMIC signature set

## Inputs

- ESCC somatic mutation catalog as a MAF or VCF file (e.g. ESCC.exon.ALL.input.maf)
- COSMIC signature matrix P (user-defined refitting signature set, e.g. COSMIC Mutational Signatures v3.2 or v2.0)
- BSgenome.Hsapiens.UCSC.hg19 reference genome library (or hg38) to build the observed mutation count matrix V

## Outputs

- runtime_benchmark.csv with per-tool runtimes for InverseNMF, deconstructSigs, and MutationalPatterns plus speed ratios
- Refitted contribution matrix S from InverseNMF (converged within the 100,000,000-iteration cap)
- Per-tool contribution scores enabling cross-tool consistency checks

## How to apply

Load the ESCC somatic mutation catalog (MAF or VCF) into the RNMF R package and use the Bioconductor BSgenome.Hsapiens.UCSC.hg19 library to build the observed mutation count matrix V. Fix the user-defined COSMIC signature matrix P and run InverseNMF's iterative update S' = S * P^T V / (P^T P S + gamma) until convergence or the 100,000,000-iteration cap, recording elapsed wall-clock time. Run deconstructSigs and MutationalPatterns on the identical catalog with the same COSMIC signature set, recording their elapsed wall-clock times. Compute speed ratios of InverseNMF against each baseline and verify that contribution scores remain highly consistent across tools, so the comparison reflects speed rather than divergent results.

## Related tools

- **RNMF** (Provides the InverseNMF refitting procedure whose runtime is benchmarked against the baselines) — https://github.com/zhenzhang-li/RNMF
- **R** (Language in which RNMF, the timing harness, and the comparison are implemented)
- **BSgenome.Hsapiens.UCSC.hg19** (Bioconductor library used to build the observed mutation count matrix V from the ESCC catalog)
- **deconstructSigs** (Refitting baseline run on the identical ESCC catalog with the same COSMIC signature set; expected to be ~12x slower than InverseNMF)
- **MutationalPatterns** (Refitting baseline run on the identical ESCC catalog; InverseNMF's runtime is expected to be comparable to it)

## Evaluation signals

- InverseNMF speedup ratio over deconstructSigs reaches approximately 12x on the same ESCC input data
- InverseNMF runtime is comparable to MutationalPatterns on the identical catalog and signature set
- The refitted S matrix converges within the 100,000,000-iteration cap
- Contribution scores from InverseNMF are highly consistent with those from deconstructSigs and MutationalPatterns, confirming the comparison is speed-only
- runtime_benchmark.csv contains all per-tool runtimes and both speed ratios

## Limitations

- The benchmark is tied to a specific ESCC cohort and COSMIC signature set; ratios may not transfer to other cancer types, catalogs, or signature panel sizes
- Wall-clock times are hardware-dependent (the README example runs with mc.cores = 20), so baselines must run on the same environment for a fair ratio
- The RNMF framework is acknowledged as slower than previously published software for de novo signature extraction; the speed advantage applies specifically to the InverseNMF refitting procedure

## Evidence

- [methods] we also developed a reversible method named InverseNMF, which can identify mutational signatures within a small data set or a single tumor sample: "we also developed a reversible method named InverseNMF, which can identify mutational signatures within a small data set or a single tumor sample"
- [methods] Then it relies on the Bioconductor library, such as BS.genome.Hsapiens.UCSC.hg19 or BS.genome.Hsapiens.UCSC.hg38: "Then it relies on the Bioconductor library, such as BS.genome.Hsapiens.UCSC.hg19 or BS.genome.Hsapiens.UCSC.hg38"
- [abstract] We developed a simple and practical R package called RNMF (https://github.com/zhenzhang-li/RNMF) for mutational signature analysis: "We developed a simple and practical R package called RNMF (https://github.com/zhenzhang-li/RNMF) for mutational signature analysis"
- [readme] denovoNMF(originalGenomes, sampleNames, subtypes, kmin = 1, kmax = 15, steptol = 10^-9, mc.cores = 20): "denovoNMF(originalGenomes, sampleNames, subtypes, kmin = 1, kmax = 15, steptol = 10^-9, mc.cores = 20)"
