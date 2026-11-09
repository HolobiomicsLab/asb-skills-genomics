---
name: nmf-matrix-decomposition-application
description: Use when starting from a somatic mutation dataset (MAF or VCF) and you
  need to extract K mutational signatures (SBS/DBS/ID) de novo, or estimate signature
  exposures with a user-defined P matrix via InverseNMF, as a precursor to gene-signature
  CCA analysis.
when_to_use_negative:
- When signature matrices P and S are already available (e.g., COSMIC v3.2 reference
  signatures) and the goal is per-gene CCA computation via cumulativeCA/genePerMutSigs
  rather than de novo extraction
- 'When speed is critical: the framework is slower than previously published software
  for de novo signature extraction due to the initial-value correction and solution-space
  building steps'
- When data have inconsistent sequencing backgrounds outside uniformly analyzed coding
  regions, which the authors note introduces limitations
edam_operation: http://edamontology.org/operation_3919
edam_topics:
- http://edamontology.org/topic_3676
- http://edamontology.org/topic_0769
tools:
- name: RNMF
  role: R package providing denovoNMF, InverseNMF, SigsInput, and cosine-similarity
    comparison for NMF decomposition of mutation count matrices
  repo: https://github.com/zhenzhang-li/RNMF
- name: R
  role: Language in which the RNMF NMF framework is implemented and executed
- name: BSgenome.Hsapiens.UCSC.hg19
  role: Bioconductor genome library used to build mutation-type information matrices
    for the hg19/Ch37 build
- name: BSgenome.Hsapiens.UCSC.hg38
  role: Bioconductor genome library used to build mutation-type information matrices
    for the hg38 build
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/cam4.4717
    title: A practical framework <scp>RNMF</scp> for exploring the association between
      mutational signatures and genes using gene cumulative contribution abundance
  source_content_hash: sha256:abb6e130e6d08c3f7786a437e04c951f37a665ead5d27954251e60dbaaf5eb7d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhenzhang-li/RNMF
  tools:
  - BSgenome.Hsapiens.UCSC.hg19
  - BSgenome.Hsapiens.UCSC.hg38
  - R
  - RNMF
  edam_operation: http://edamontology.org/operation_3919
  edam_topics:
  - http://edamontology.org/topic_3676
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# nmf-matrix-decomposition-application

## Summary

Applies non-negative matrix factorization (NMF) as implemented in the RNMF R package to decompose a mutation-type count matrix into a mutational signature matrix P and a sample abundance fraction matrix S, enabling de novo signature extraction and inverse estimation of signatures in small cohorts.

## When to use

Use when starting from a somatic mutation dataset (MAF or VCF) and you need to extract K mutational signatures (SBS/DBS/ID) de novo, or estimate signature exposures with a user-defined P matrix via InverseNMF, as a precursor to gene-signature CCA analysis.

## When NOT to use

- When signature matrices P and S are already available (e.g., COSMIC v3.2 reference signatures) and the goal is per-gene CCA computation via cumulativeCA/genePerMutSigs rather than de novo extraction
- When speed is critical: the framework is slower than previously published software for de novo signature extraction due to the initial-value correction and solution-space building steps
- When data have inconsistent sequencing backgrounds outside uniformly analyzed coding regions, which the authors note introduces limitations

## Inputs

- Somatic mutation file in MAF or VCF format (e.g., ESCC.exon.ALL.input.maf)
- Column-mapping parameters: sample.id='Tumor_Sample_Barcode', chr='Chromosome', pos='Start_position', ref='Reference_Allele', alt='Tumor_Seq_Allele2'
- Genome build (BSgenome.Hsapiens.UCSC.hg19 or hg38; genome.build='Ch37')
- AnalCOSMICSigType ('SBS', 'DBS', or 'ID')
- Range of signature numbers K (kmin, kmax) for denovoNMF
- User-defined signature matrix P (for InverseNMF)

## Outputs

- Mutational signature matrix P (e.g., SBS.Pmatrix.txt)
- Sample abundance fraction matrix S (e.g., SBS.Smatrix.txt)
- Mutational signature profiles (SBS and ID lego plots)
- Silhouette coefficient and error gradient metrics used to select K
- Cosine similarity values between extracted and COSMIC reference signatures

## How to apply

First import somatic variants from a MAF or VCF file (e.g., via SigsInput with sample.id='Tumor_Sample_Barcode', chr='Chromosome', pos='Start_position', ref='Reference_Allele', alt='Tumor_Seq_Allele2') to build the mutation-type information matrix for SBS, DBS, or ID. Run denovoNMF over a range of K (e.g., kmin=1, kmax=15, steptol=10^-9, mc.cores=20): capture random P (P>=0) and S (S>=0) matrices, optimize initial values by Dirichlet resampling over 20,000 iterations, re-update the initial value for II>=5 iterations, and rerun NMF with multiplicative update rules until P and S converge or a maximum of 100,000 iterations is reached. Repeat with I>=20 repetitions to build a solution space, then select the suitable signature number using the silhouette coefficient measure and error gradient; k-means clustering of the P and S matrices yields class-center initial values. For small datasets or single tumors, use InverseNMF to estimate the sample contribution matrix S from a user-defined signature matrix P. Compare extracted signatures to COSMIC references (v3.2 March 2021 or v2.0 March 2015) via cosine similarity to determine similarity between two mutational signatures.

## Related tools

- **RNMF** (R package providing denovoNMF, InverseNMF, SigsInput, and cosine-similarity comparison for NMF decomposition of mutation count matrices) — https://github.com/zhenzhang-li/RNMF
- **R** (Language in which the RNMF NMF framework is implemented and executed)
- **BSgenome.Hsapiens.UCSC.hg19** (Bioconductor genome library used to build mutation-type information matrices for the hg19/Ch37 build)
- **BSgenome.Hsapiens.UCSC.hg38** (Bioconductor genome library used to build mutation-type information matrices for the hg38 build)

## Examples

```
library(RNMF); denovoNMF(originalGenomes, sampleNames, subtypes, kmin = 1, kmax = 15, steptol = 10^-9, mc.cores = 20)
```

## Evaluation signals

- Multiplicative update iterations terminate only at P and S convergence or the 100,000-iteration cap, with steptol=10^-9 as the tolerance
- Solution space is generated with I>=20 repetitions for each K before signature-number selection
- Selected K is justified by silhouette coefficient and error gradient values across the K range scanned (e.g., kmin=1 to kmax=15)
- Extracted signatures match COSMIC references with high cosine similarity (COSMIC Mutational Signatures v3.2 March 2021 or v2.0 March 2015)
- Output P and S matrices are non-negative and, for denovoNMF runs, respect the randomly initialized constraints P>=0 and S>=0

## Limitations

- Compared with previously published software, the framework spends more time on de novo extraction of signature analysis due to the 'Re-updated the initial value' correction and 'build a solution space' processes; needs future optimization
- Due to the inconsistent sequencing background of the data, only coding regions were uniformly analyzed, which introduces limitations
- For very small datasets, de novo NMF is unreliable and InverseNMF with a user-defined P must be used instead

## Evidence

- [methods] these K mutational signatures could be extracted by denovoNMF (Figure 1) as following: "these K mutational signatures could be extracted by denovoNMF (Figure 1) as following"
- [methods] Iterate until P and S convergence or until the maximum number of 100,000 iterations is reached: "Iterate until P and S convergence or until the maximum number of 100,000 iterations is reached"
- [methods] we apply the k-means 36 algorithm to the set of matrices P and S to cluster the data into K clusters: "we apply the k‐means 36 algorithm to the set of matrices P and S to cluster the data into K clusters"
- [methods] we use the cosine similarity to determine the similarity between two mutational signature A and B: "we use the cosine similarity to determine the similarity between two mutational signature A and B"
- [readme] denovoNMF(originalGenomes, sampleNames, subtypes, kmin = 1, kmax = 15, steptol = 10^-9, mc.cores = 20): "denovoNMF(originalGenomes, sampleNames, subtypes, kmin = 1, kmax = 15, steptol = 10^-9, mc.cores = 20)"
- [methods] we also developed a reversible method named InverseNMF, which can identify mutational signatures within a small data set or a single tumor sample: "we also developed a reversible method named InverseNMF, which can identify mutational signatures within a small data set"
- [methods] generate a new initial mutational signature matrix P and a new initial abundance fraction matrix S through 20,000 iterations: "generate a new initial mutational signature matrix P and a new initial abundance fraction matrix S through 20,000 iterations"
