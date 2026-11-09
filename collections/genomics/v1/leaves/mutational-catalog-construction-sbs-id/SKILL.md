---
name: mutational-catalog-construction-sbs-id
description: Use when starting from raw somatic variant calls (MAF/VCF) for a cancer
  cohort and the goal is to discover the number and profiles of de novo SBS and INDEL
  (ID) mutational signatures, as done for the 508-sample WGS ESCC cohort.
when_to_use_negative:
- When the input is a very small dataset or a single tumor sample — use RNMF's InverseNMF
  instead, which was designed for small data sets or single samples with a user-defined
  P
- When the goal is only to fit known COSMIC signatures rather than discover de novo
  ones
- When the cohort has inconsistent sequencing background and only coding regions can
  be uniformly analyzed — the authors note this limits de novo extraction
edam_operation: http://edamontology.org/operation_3919
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: RNMF
  role: R package providing SigsInput to build SBS/DBS/ID count matrices from MAF/VCF
    and denovoNMF to de novo extract mutational signatures
  repo: https://github.com/zhenzhang-li/RNMF
- name: BSgenome.Hsapiens.UCSC.hg19
  role: Bioconductor genome build used by SigsInput to construct the mutation-type
    information matrix
- name: BSgenome.Hsapiens.UCSC.hg38
  role: Alternative Bioconductor genome build for SigsInput
- name: R
  role: Language in which RNMF is implemented and executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/cam4.4717
    title: A practical framework <scp>RNMF</scp> for exploring the association between
      mutational signatures and genes using gene cumulative contribution abundance
  source_content_hash: sha256:f2d6b021ef175df21f953de7f465d8c2f8a89a2f33e5e9751987ed20c0de0ae9
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
  edam_operation: http://edamontology.org/operation_3919
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# mutational-catalog-construction-sbs-id

## Summary

De novo extraction of a mutational signature catalog (SBS and ID) from somatic mutation data using the RNMF R package's denovoNMF 5-step NMF framework, validated against COSMIC reference signatures by cosine similarity.

## When to use

Use when starting from raw somatic variant calls (MAF/VCF) for a cancer cohort and the goal is to discover the number and profiles of de novo SBS and INDEL (ID) mutational signatures, as done for the 508-sample WGS ESCC cohort.

## When NOT to use

- When the input is a very small dataset or a single tumor sample — use RNMF's InverseNMF instead, which was designed for small data sets or single samples with a user-defined P
- When the goal is only to fit known COSMIC signatures rather than discover de novo ones
- When the cohort has inconsistent sequencing background and only coding regions can be uniformly analyzed — the authors note this limits de novo extraction

## Inputs

- Somatic mutation calls in MAF or VCF format (e.g., ESCC.exon.ALL.input.maf)
- Reference genome build via Bioconductor BSgenome.Hsapiens.UCSC.hg19 or BSgenome.Hsapiens.UCSC.hg38
- COSMIC Mutational Signatures reference catalogs (v3.2 March 2021 and/or v2.0 March 2015) for similarity matching

## Outputs

- De novo signature matrix P and sample-exposure matrix S (e.g., SBS.Pmatrix.txt, SBS.Smatrix.txt)
- Named catalog of de novo SBS signatures (SBS1–SBS12 in the ESCC WGS508 cohort) and ID signatures (9 IDs)
- 96-channel (SBS) and ID-channel signature plots and silhouette/error-gradient statistics used for K selection

## How to apply

Import somatic mutations from a MAF or VCF file into RNMF and run SigsInput with a Bioconductor genome build (BSgenome.Hsapiens.UCSC.hg19 or hg38) to build the mutation-type information matrices for SBS and ID. Run denovoNMF (kmin=1, kmax=15, steptol=10^-9, mc.cores=20) which executes its 5-step framework: random non-negative initialization of P and S, Dirichlet-resampling optimization over 20,000 iterations, k-means re-optimization of initial values over >=5 repeats, multiplicative-update NMF iterated to convergence or up to 100,000 iterations, and construction of a solution space repeated >=20 times per candidate K. Select the optimal signature number K using the silhouette coefficient plus error-gradient criteria (stable, no sudden decline, small standard error, gentle gradient between adjacent K). Assess the extracted signatures by cosine similarity against COSMIC references with the match threshold set to 0.9. On the ESCC WGS508 cohort this procedure stably recovered 12 SBS signatures (SBS1–SBS12) and 9 ID signatures.

## Related tools

- **RNMF** (R package providing SigsInput to build SBS/DBS/ID count matrices from MAF/VCF and denovoNMF to de novo extract mutational signatures) — https://github.com/zhenzhang-li/RNMF
- **BSgenome.Hsapiens.UCSC.hg19** (Bioconductor genome build used by SigsInput to construct the mutation-type information matrix)
- **BSgenome.Hsapiens.UCSC.hg38** (Alternative Bioconductor genome build for SigsInput)
- **R** (Language in which RNMF is implemented and executed)

## Examples

```
library(RNMF); SigsInput(file='ESCC.exon.ALL.input.maf', Filetype='MAF', AnalCOSMICSigType='SBS', genome.build='Ch37'); denovoNMF(originalGenomes, sampleNames, subtypes, kmin=1, kmax=15, steptol=10^-9, mc.cores=20)
```

## Evaluation signals

- Number of stably extracted signatures matches the reported counts for the ESCC WGS508 cohort: 12 SBS signatures (SBS1–SBS12) and 9 ID signatures
- Matched signatures reach the cosine similarity threshold of 0.9 against COSMIC reference signatures
- Silhouette coefficient and error gradient criteria hold at the chosen K: error curve stable with no sudden decline, small standard error, gentle gradient between adjacent K
- denovoNMF convergence behavior matches parameters: steptol=10^-9, multiplicative updates up to 100,000 iterations, >=20 solution-space repetitions
- P and S matrices satisfy non-negativity and column-normalization constraints from initialization through updates

## Limitations

- Compared with previously published software, the framework spends more time on de novo extraction due to the 'Re-updated the initial value' correction and 'build a solution space' processes; needs future optimization
- Due to the inconsistent sequencing background of the pooled data, only coding regions were uniformly analyzed, which introduces limitations
- The relatively small number of individuals studied is a shortcoming for model learning

## Evidence

- [methods] these K mutational signatures could be extracted by denovoNMF (Figure 1): "these K mutational signatures could be extracted by denovoNMF (Figure 1)"
- [methods] Repeat the process of steps 1 to 5 with >=20 times to generate a solution space for all the value of K: [evidence span withheld — anchor status: not_found]
- [methods] we use the cosine similarity to determine the similarity between two mutational signature A and B: "we use the cosine similarity to determine the similarity between two mutational signature A and B"
- [results] RNMF successfully identified 12 single base substitution signatures from the ESCC cohort, named SBS1 through SBS12: [evidence span withheld — anchor status: not_found]
- [readme] denovoNMF(originalGenomes, sampleNames, subtypes, kmin = 1, kmax = 15, steptol = 10^-9, mc.cores = 20): "denovoNMF(originalGenomes, sampleNames, subtypes, kmin = 1, kmax = 15, steptol = 10^-9, mc.cores = 20)"
- [methods] 508‐WGS cohort renames as WGS508: "508‐WGS cohort renames as WGS508, exon region of 508‐WGS cohort renames as EXON508, and exon region of 1073 samples renames as EXON1073"
