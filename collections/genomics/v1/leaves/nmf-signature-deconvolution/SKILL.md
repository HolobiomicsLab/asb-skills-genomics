---
name: nmf-signature-deconvolution
description: Use when starting from raw somatic mutation calls (MAF/VCF) and you need
  to de novo extract and enumerate single base substitution (SBS) and insertion/deletion
  (ID) mutational signatures without relying on a fixed reference catalog.
when_to_use_negative:
- When the dataset is small (a single tumor sample or small cohort) — use the companion
  InverseNMF method, which estimates the sample contribution matrix S from a user-defined
  signature matrix P
- When the goal is association of pre-defined COSMIC signatures with genes rather
  than de novo extraction — use cumulativeCA / genePerMutSigs downstream functions
  instead
- When the input is already a mutation count/context matrix and you need a fast extractor
  — the article notes this framework spends more time on de novo extraction than previously
  published software due to the initial-value correction and solution-space processes
edam_operation: http://edamontology.org/operation_3919
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: RNMF
  role: R package providing SigsInput to build SBS/DBS/ID count matrices from mutation
    data and denovoNMF for de novo signature extraction
  repo: https://github.com/zhenzhang-li/RNMF
- name: R
  role: Language in which RNMF is implemented and executed
  repo: https://github.com/zhenzhang-li/RNMF
- name: BSgenome.Hsapiens.UCSC.hg19
  role: Bioconductor genome build used by SigsInput to derive trinucleotide contexts
    (Ch37)
- name: BSgenome.Hsapiens.UCSC.hg38
  role: Alternative Bioconductor genome build for trinucleotide context annotation
    (hg38)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/cam4.4717
    title: A practical framework <scp>RNMF</scp> for exploring the association between
      mutational signatures and genes using gene cumulative contribution abundance
  source_content_hash: sha256:24b02760cedb0176e1e0f127616f58f9c15d269724258603010af36e56925345
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
n_claims: 12
---

# nmf-signature-deconvolution

## Summary

De novo extraction of mutational signatures (SBS and ID) from somatic mutation datasets using the RNMF R package's denovoNMF, a 5-step non-negative matrix factorization framework with Dirichlet resampling, k-means re-initialization, and multiplicative update rules.

## When to use

Use when starting from raw somatic mutation calls (MAF/VCF) and you need to de novo extract and enumerate single base substitution (SBS) and insertion/deletion (ID) mutational signatures without relying on a fixed reference catalog.

## When NOT to use

- When the dataset is small (a single tumor sample or small cohort) — use the companion InverseNMF method, which estimates the sample contribution matrix S from a user-defined signature matrix P
- When the goal is association of pre-defined COSMIC signatures with genes rather than de novo extraction — use cumulativeCA / genePerMutSigs downstream functions instead
- When the input is already a mutation count/context matrix and you need a fast extractor — the article notes this framework spends more time on de novo extraction than previously published software due to the initial-value correction and solution-space processes

## Inputs

- Somatic mutation calls in MAF or VCF format (e.g. ESCC.exon.ALL.input.maf)
- Reference genome build for trinucleotide context: BSgenome.Hsapiens.UCSC.hg19 (Ch37) or BSgenome.Hsapiens.UCSC.hg38
- Candidate range of signature numbers K (e.g. kmin=1, kmax=15)

## Outputs

- Signature matrix P (mutational signatures, e.g. SBS.Pmatrix.txt) and abundance matrix S (sample exposures, e.g. SBS.Smatrix.txt)
- De novo SBS and ID signature profiles (12 SBS and 9 ID on the WGS508 ESCC cohort), named SBS1-SBS12 for COSMIC comparison
- Silhouette coefficient / error-gradient diagnostics for selecting the optimal K
- Cosine similarity assignments of de novo signatures to COSMIC references at threshold 0.9

## How to apply

Import somatic variants from MAF or VCF and run SigsInput with BSgenome.Hsapiens.UCSC.hg19 (Ch37) or hg38 to build the mutation type information matrices (SBS, DBS, ID). Run denovoNMF over a range of candidate signature numbers (e.g. kmin=1, kmax=15): Step 1 randomly generates non-negative matrices P and S with column-normalization constraints; Step 2 optimizes the initial value by Dirichlet-distribution resampling over 20,000 iterations; Step 3 repeats Steps 1-2 at least 5 times and applies k-means clustering to the P and S sets to obtain class-center initial values; Step 4 reruns NMF with multiplicative update rules until convergence (steptol=10^-9) or a maximum of 100,000 iterations; Step 5 builds a solution space by repeating the whole process >=20 times. Select the optimal K using the silhouette coefficient plus error gradient criteria (stable solution, no sudden decline, small standard error, gentle gradient between adjacent K). Validate extracted signatures by cosine similarity against COSMIC references (v3.2 March 2021 or v2.0 March 2015) with the match threshold set to 0.9; on the 508-sample WGS ESCC cohort (WGS508) this procedure recovered 12 SBS signatures (SBS1-SBS12) and 9 ID signatures.

## Related tools

- **RNMF** (R package providing SigsInput to build SBS/DBS/ID count matrices from mutation data and denovoNMF for de novo signature extraction) — https://github.com/zhenzhang-li/RNMF
- **R** (Language in which RNMF is implemented and executed) — https://github.com/zhenzhang-li/RNMF
- **BSgenome.Hsapiens.UCSC.hg19** (Bioconductor genome build used by SigsInput to derive trinucleotide contexts (Ch37))
- **BSgenome.Hsapiens.UCSC.hg38** (Alternative Bioconductor genome build for trinucleotide context annotation (hg38))

## Examples

```
denovoNMF(originalGenomes, sampleNames, subtypes, kmin = 1, kmax = 15, steptol = 10^-9, mc.cores = 20)
```

## Evaluation signals

- Number of stable extracted signatures matches the reported counts for the cohort (12 SBS and 9 ID signatures on WGS508 ESCC)
- Cosine similarity of matched signatures against COSMIC references reaches the 0.9 threshold
- Silhouette coefficient is maximal / error gradient is stable across candidate K values (no sudden decline, small standard error, gentle gradient between adjacent K)
- NMF iteration converges (steptol=10^-9) or terminates at the 100,000-iteration maximum with non-negative P and S satisfying column-normalization constraints
- Solution space built from >=20 repetitions yields consistent signature counts across repeats

## Limitations

- Compared with previously published software, the framework spends more time on de novo signature extraction due to the 'Re-updated the initial value' correction process and 'build a solution space' process; needs future optimization
- Due to inconsistent sequencing background of the pooled data, only coding regions were uniformly analyzed, which introduces limitations
- Accuracy of the selected number of signatures depends on silhouette coefficient and error gradient criteria being interpreted across a sufficiently wide K range

## Evidence

- [methods] generate a new initial mutational signature matrix P and a new initial abundance fraction matrix S through 20,000 iterations: "generate a new initial mutational signature matrix P and a new initial abundance fraction matrix S through 20,000 iterations"
- [methods] Iterate until P and S convergence or until the maximum number of 100,000 iterations is reached: "Iterate until P and S convergence or until the maximum number of 100,000 iterations is reached"
- [methods] Repeat the process of steps 1 to 5 with Ι^>=20 times to generate a solution space for all the value of K.: [evidence span withheld — anchor status: not_found]
- [methods] we use the cosine similarity to determine the similarity between two mutational signature A and B: "we use the cosine similarity to determine the similarity between two mutational signature A and B"
- [readme] denovoNMF(originalGenomes, sampleNames, subtypes, kmin = 1, kmax = 15, steptol = 10^-9, mc.cores = 20): "denovoNMF(originalGenomes, sampleNames, subtypes, kmin = 1, kmax = 15, steptol = 10^-9, mc.cores = 20)"
- [methods] our designed RNMF software provides the ability to obtain the count matrix of SBS, DBS, and ID directly from mutation data sets: "our designed RNMF software provides the ability to obtain the count matrix of SBS, DBS, and ID directly from mutation data sets"
