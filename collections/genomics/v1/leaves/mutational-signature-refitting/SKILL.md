---
name: mutational-signature-refitting
description: Use when you have a somatic mutation catalog (MAF/VCF) for one or few
  tumor samples and a predefined COSMIC signature set, and you need per-sample signature
  contribution scores rather than de novo signature discovery.
when_to_use_negative:
- You need to discover novel signatures rather than quantify known COSMIC signatures
  — use RNMF's denovoNMF 5-step framework instead
- Input is already a refitted contribution matrix S or a gene-level association result
  (e.g. CCA matrix) rather than raw somatic variant calls
edam_topics:
- http://edamontology.org/topic_2640
tools:
- name: RNMF
  role: Provides the InverseNMF function that estimates the sample contribution matrix
    S from a fixed user-defined signature matrix P and the mutation count matrix V
  repo: https://github.com/zhenzhang-li/RNMF
- name: R
  role: Implementation language and runtime environment for the RNMF package
  repo: https://github.com/zhenzhang-li/RNMF
- name: BSgenome.Hsapiens.UCSC.hg19
  role: Bioconductor genome library used to build the observed SBS/DBS/ID mutation
    count matrix V from the mutation catalog
- name: deconstructSigs
  role: Baseline refitting tool used for speed and contribution-consistency comparison
    on the same ESCC catalog
- name: MutationalPatterns
  role: Baseline refitting tool used for speed and contribution-consistency comparison
    on the same ESCC catalog
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/cam4.4717
    title: A practical framework <scp>RNMF</scp> for exploring the association between
      mutational signatures and genes using gene cumulative contribution abundance
  source_content_hash: sha256:28b58c9c345c4bc3ba3cfc4e2cdf8fb38a42641302fa30ed298f72461d936a69
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
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# mutational-signature-refitting

## Summary

Refit a fixed, user-defined COSMIC signature matrix P onto an observed tumor mutation count matrix V using the RNMF package's InverseNMF procedure, estimating the per-sample contribution matrix S. It enables signature quantification in a small data set or a single tumor sample and runs much faster than deconstructSigs.

## When to use

Use when you have a somatic mutation catalog (MAF/VCF) for one or few tumor samples and a predefined COSMIC signature set, and you need per-sample signature contribution scores rather than de novo signature discovery.

## When NOT to use

- You need to discover novel signatures rather than quantify known COSMIC signatures — use RNMF's denovoNMF 5-step framework instead
- Input is already a refitted contribution matrix S or a gene-level association result (e.g. CCA matrix) rather than raw somatic variant calls

## Inputs

- Somatic mutation catalog in MAF or VCF format (e.g. ESCC exon/WGS mutation calls)
- Reference genome build via Bioconductor BSgenome.Hsapiens.UCSC.hg19 or BSgenome.Hsapiens.UCSC.hg38
- User-defined COSMIC signature matrix P (COSMIC Mutational Signatures v3.2 - March 2021 or v2.0 - March 2015)

## Outputs

- Refitted sample contribution (abundance fraction) matrix S with per-sample signature scores
- Convergence/iteration and elapsed wall-clock runtime record for the refit run

## How to apply

Load the somatic mutation catalog (MAF or VCF) into the RNMF R package and build the observed mutation count matrix V using the Bioconductor BSgenome.Hsapiens.UCSC.hg19 (or hg38) library. Fix the user-defined signature matrix P (COSMIC Mutational Signatures, e.g. v3.2 March 2021 or v2.0 March 2015) and run the InverseNMF multiplicative update S' = S * P^T V / (P^T P S + gamma), iterating until S converges or up to the maximum of 100,000,000 iterations. Assess consistency of the refitted contributions against baselines (deconstructSigs, MutationalPatterns) using cosine similarity between signatures/scores. On the same ESCC catalog, InverseNMF runs about 12x faster than deconstructSigs with runtime comparable to MutationalPatterns, so it is the appropriate choice when refitting speed or single-sample scale matters.

## Related tools

- **RNMF** (Provides the InverseNMF function that estimates the sample contribution matrix S from a fixed user-defined signature matrix P and the mutation count matrix V) — https://github.com/zhenzhang-li/RNMF
- **R** (Implementation language and runtime environment for the RNMF package) — https://github.com/zhenzhang-li/RNMF
- **BSgenome.Hsapiens.UCSC.hg19** (Bioconductor genome library used to build the observed SBS/DBS/ID mutation count matrix V from the mutation catalog)
- **deconstructSigs** (Baseline refitting tool used for speed and contribution-consistency comparison on the same ESCC catalog)
- **MutationalPatterns** (Baseline refitting tool used for speed and contribution-consistency comparison on the same ESCC catalog)

## Evaluation signals

- The refitted S matrix converges within the 100,000,000-iteration cap of the multiplicative update S' = S * P^T V / (P^T P S + gamma)
- InverseNMF speedup ratio over deconstructSigs reaches approximately 12x on the same ESCC input data, with runtime comparable to MutationalPatterns
- Contribution scores are highly consistent with deconstructSigs and MutationalPatterns run on the identical catalog and signature set
- The observed count matrix V is correctly built for the chosen genome build (BSgenome.Hsapiens.UCSC.hg19/hg38) from the MAF/VCF file

## Limitations

- Runtime benchmarks and contribution consistency were demonstrated on the ESCC catalog with a COSMIC signature set; transferability to other cancer types or signature panels is not established here
- The RNMF framework spends more time on de novo signature extraction than previously published software (due to the 'Re-updated the initial value' and 'build a solution space' processes), so speed claims apply to the refitting step, not de novo extraction
- Analysis of the ESCC cohort was limited to coding regions due to inconsistent sequencing background of the data

## Evidence

- [methods] InverseNMF for single-sample refitting: "we also developed a reversible method named InverseNMF, which can identify mutational signatures within a small data set or a single tumor sample"
- [methods] Variant import formats: "somatic variants can be imported from a Variant Call Format (VCF) file or a Mutation Annotation Format (MAF) file"
- [methods] Genome build dependency: "Then it relies on the Bioconductor library, such as BS.genome.Hsapiens.UCSC.hg19 or BS.genome.Hsapiens.UCSC.hg38"
- [results] Benchmark finding on ESCC data: [evidence span withheld — anchor status: not_found]
- [methods] Similarity metric for comparing signatures: "we use the cosine similarity to determine the similarity between two mutational signature A and B"
- [readme] Package availability and loading: "library(RNMF)

file='ESCC.exon.ALL.input.maf'"
