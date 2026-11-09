---
name: silhouette-coefficient-signature-number-selection
description: Decision support for choosing the number of de novo SBS/ID mutational
  signatures (K) when running RNMF denovoNMF on a cohort's somatic mutation count
  matrix and needing a stable, data-driven K rather than an arbitrary choice.
when_to_use_negative:
- When the analysis goal is signature assignment to a small dataset or a single tumor
  sample with a user-defined signature matrix — use RNMF's InverseNMF instead of de
  novo K selection
- When signatures should simply be matched against a fixed COSMIC reference set rather
  than discovered de novo, since this skill only selects K for de novo extraction
- 'When compute time is critical: the article notes this framework spends more time
  on de novo extraction than previously published software due to the re-updated initial
  value and solution-space processes'
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3316
tools:
- name: RNMF
  role: R package providing SigsInput and denovoNMF whose Step 5 builds the solution
    space and selects K via the silhouette coefficient measure and error-gradient
    criteria
  repo: https://github.com/zhenzhang-li/RNMF
- name: R
  role: Language in which RNMF and the silhouette-based K-selection procedure are
    implemented
- name: BSgenome.Hsapiens.UCSC.hg19
  role: Bioconductor genome build used by SigsInput to construct the SBS/ID count
    matrices that feed the K-selection workflow
- name: BSgenome.Hsapiens.UCSC.hg38
  role: Alternative Bioconductor genome build for constructing the mutation-type information
    matrix
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/cam4.4717
    title: A practical framework <scp>RNMF</scp> for exploring the association between
      mutational signatures and genes using gene cumulative contribution abundance
  source_content_hash: sha256:f078b7e861d860563e24cb39e76807102eadecd790124a08cbff36d6e7ab0283
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhenzhang-li/RNMF
  tools:
  - BSgenome.Hsapiens.UCSC.hg19
  - BSgenome.Hsapiens.UCSC.hg38
  - Bioconductor
  - R
  - RNMF
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# silhouette-coefficient-signature-number-selection

## Summary

A model-selection procedure within the RNMF denovoNMF framework that determines the optimal number of de novo mutational signatures (K) by building a solution space of repeated NMF runs and selecting K using the silhouette coefficient measure combined with error-gradient stability criteria.

## When to use

Decision support for choosing the number of de novo SBS/ID mutational signatures (K) when running RNMF denovoNMF on a cohort's somatic mutation count matrix and needing a stable, data-driven K rather than an arbitrary choice.

## When NOT to use

- When the analysis goal is signature assignment to a small dataset or a single tumor sample with a user-defined signature matrix — use RNMF's InverseNMF instead of de novo K selection
- When signatures should simply be matched against a fixed COSMIC reference set rather than discovered de novo, since this skill only selects K for de novo extraction
- When compute time is critical: the article notes this framework spends more time on de novo extraction than previously published software due to the re-updated initial value and solution-space processes

## Inputs

- Mutation-type information matrix (SBS and ID count matrices) from SigsInput, built from somatic MAF or VCF files (e.g. 508-sample WGS ESCC cohort, WGS508)
- Candidate K range passed to denovoNMF (e.g. kmin = 1, kmax = 15)
- Reference COSMIC Mutational Signatures (v3.2 - March 2021 and v2.0 - March 2015) for downstream cosine-similarity validation

## Outputs

- Solution space of NMF solutions (P and S matrices) for all candidate K values from >=20 repetitions
- Selected optimal number of signatures K (e.g. 12 SBS signatures, SBS1–SBS12, and 9 ID signatures for WGS508)
- Silhouette coefficient and error-gradient (standard error) diagnostics used to justify the selected K

## How to apply

After building the SBS and ID mutation-type information matrices with SigsInput (from MAF/VCF somatic variants against BSgenome.Hsapiens.UCSC.hg19 or hg38), run the denovoNMF 5-step framework (random non-negative P/S initialization with column normalization, Dirichlet-resampling optimization over 20,000 iterations, k-means re-optimization over >=5 repeats, multiplicative update rules until convergence or 100,000 iterations) for every candidate K. Repeat steps 1–5 at least 20 times (I >= 20) to generate a solution space of solutions for all candidate K values. Then select the suitable K from this solution space using the silhouette coefficient measure together with the error-gradient criteria: the error curve should be stable with no sudden decline, show a small standard error, and exhibit a gentle gradient between adjacent K values. In the ESCC WGS508 application (kmin=1, kmax=15 in the README), this procedure identified 12 SBS signatures (SBS1–SBS12) and 9 ID signatures, subsequently compared to COSMIC reference signatures by cosine similarity with a 0.9 threshold.

## Related tools

- **RNMF** (R package providing SigsInput and denovoNMF whose Step 5 builds the solution space and selects K via the silhouette coefficient measure and error-gradient criteria) — https://github.com/zhenzhang-li/RNMF
- **R** (Language in which RNMF and the silhouette-based K-selection procedure are implemented)
- **BSgenome.Hsapiens.UCSC.hg19** (Bioconductor genome build used by SigsInput to construct the SBS/ID count matrices that feed the K-selection workflow)
- **BSgenome.Hsapiens.UCSC.hg38** (Alternative Bioconductor genome build for constructing the mutation-type information matrix)

## Examples

```
denovoNMF(originalGenomes, sampleNames, subtypes, kmin = 1, kmax = 15, steptol = 10^-9, mc.cores = 20)
```

## Evaluation signals

- The number of stable extracted signatures matches the reported counts (12 SBS signatures SBS1–SBS12 and 9 ID signatures for the WGS508 ESCC cohort)
- The error curve across candidate K is stable: no sudden decline, small standard error, and a gentle gradient between adjacent K values
- Silhouette coefficients computed over the >=20-repetition solution space support the chosen K as the optimum
- Matched signatures reach the cosine similarity threshold of 0.9 against COSMIC reference signatures (v3.2 / v2.0)
- denovoNMF convergence observed before the maximum of 100,000 multiplicative-update iterations (steptol = 10^-9 in the README example)

## Limitations

- Compared with previously published software, the framework spends more time on de novo signature extraction because of the 'Re-updated the initial value' correction and 'build a solution space' processes; future optimization is needed
- Selection quality depends on repetitions of the 5-step process (I >= 20), which multiplies runtime; the README example uses mc.cores = 20 to parallelize
- Results can be affected by inconsistent sequencing backgrounds of pooled cohorts; the article uniformly analyzed coding regions only due to this inconsistency

## Evidence

- [methods] Repeat the process of steps 1 to 5 with Ι^>=20 times to generate a solution space for all the value of K. Then suitable selection in this solution space using the silhouette coefficient measure and: [evidence span withheld — anchor status: not_found]
- [methods] these K mutational signatures could be extracted by denovoNMF (Figure 1) as following: "these K mutational signatures could be extracted by denovoNMF (Figure 1) as following"
- [other] RNMF successfully identified 12 single base substitution signatures from the ESCC cohort, named SBS1 through SBS12, which were compared with the COSMIC signatures.: [evidence span withheld — anchor status: not_found]
- [readme] denovoNMF(originalGenomes, sampleNames, subtypes, kmin = 1, kmax = 15, steptol = 10^-9, mc.cores = 20): "denovoNMF(originalGenomes, sampleNames, subtypes, kmin = 1, kmax = 15, steptol = 10^-9, mc.cores = 20)"
- [methods] Iterate until P and S convergence or until the maximum number of 100,000 iterations is reached: "Iterate until P and S convergence or until the maximum number of 100,000 iterations is reached"
