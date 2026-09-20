---
name: apobec-signature-annotation
description: Use when a cohort has per-sample signature contribution abundances (S
  matrix from RNMF CCA/InverseNMF) plus a MAF mutation file, and the goal is to test
  whether specific signatures (e.g., APOBEC SBS2*/SBS13*) are enriched in samples
  carrying a given gene mutation hotspot such as PIK3CA E545k.
when_to_use_negative:
- Input is raw VCF/MAF without a precomputed signature contribution abundance matrix
  (Sfile) — first run SigsInput/denovoNMF or InverseNMF/cumulativeCA to obtain S
- The question is de novo extraction of mutational signatures rather than testing
  association of known signatures with gene mutation status (use denovoNMF instead)
- Analysis targets per-gene cumulative contribution abundance association across a
  gene list rather than per-sample grouping by a specific mutation (use genePerMutSigs
  instead)
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_3676
- http://edamontology.org/topic_0749
tools:
- name: RNMF
  role: R package providing samFisherSigs and samPerMutSigs to test enrichment of
    chosen signature abundances (SBS2/SBS13) between PIK3CA E545k mutated and wild-type
    sample groups
  repo: https://github.com/zhenzhang-li/RNMF
- name: R
  role: Language in which RNMF is implemented and the association analysis loop over
    Sigstype signatures is executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/cam4.4717
    title: A practical framework <scp>RNMF</scp> for exploring the association between
      mutational signatures and genes using gene cumulative contribution abundance
  source_content_hash: sha256:6a5c27f22212a6f98f113121bc17b7b0fab2b7d8792ce7b728473b33fd02cc40
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhenzhang-li/RNMF
  tools:
  - R
  - RNMF
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_3676
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# apobec-signature-annotation

## Summary

Per-sample association testing of APOBEC mutational signatures (SBS2*/SBS13*) against a gene hotspot status (PIK3CA E545k mutated vs wild-type) using the RNMF Fisher-style functions samFisherSigs/samPerMutSigs on CCA-derived signature abundance data.

## When to use

Use when a cohort has per-sample signature contribution abundances (S matrix from RNMF CCA/InverseNMF) plus a MAF mutation file, and the goal is to test whether specific signatures (e.g., APOBEC SBS2*/SBS13*) are enriched in samples carrying a given gene mutation hotspot such as PIK3CA E545k.

## When NOT to use

- Input is raw VCF/MAF without a precomputed signature contribution abundance matrix (Sfile) — first run SigsInput/denovoNMF or InverseNMF/cumulativeCA to obtain S
- The question is de novo extraction of mutational signatures rather than testing association of known signatures with gene mutation status (use denovoNMF instead)
- Analysis targets per-gene cumulative contribution abundance association across a gene list rather than per-sample grouping by a specific mutation (use genePerMutSigs instead)

## Inputs

- Sample contribution abundance file (Sfile, e.g. SBS.Smatrix.txt) from RNMF CCA/InverseNMF, restricted to chosen signatures via choose.Sigs
- MAF-format somatic mutation file (file = maffile) with Tumor_Sample_Barcode and Hugo_Symbol columns
- Signature choice (isig), e.g. 'SBS2' or 'SBS13' from the COSMIC v2.0/v3.2 signature set
- PIK3CA E545k mutated vs wild-type sample grouping derived from MAF variant annotations

## Outputs

- Per-signature association statistics (enrichment direction, p-value, q-value) per sample group, e.g. sigs_association_results.csv
- Per-sample plots of signature abundance stratified by PIK3CA E545k status (plot = TRUE)

## How to apply

Load the sample contribution abundance file (Sfile) from the RNMF CCA model restricted to the chosen signatures (choose.Sigs = isig, e.g. SBS2 or SBS13), and load a MAF-format mutation file keyed by Tumor_Sample_Barcode and Hugo_Symbol. Annotate each tumor sample as PIK3CA E545k mutated versus wild-type from the MAF variant annotations. Run samFisherSigs (and/or samPerMutSigs) with Mutfreq = 0.04, threshold = 0.06, and Qvalue = 0.1 to test enrichment of SBS2*/SBS13* abundance between the PIK3CA E545k groups, with plot = TRUE for per-sample visualization. The rationale is that Fisher-style per-sample testing links signature activity, quantified as cumulative contribution abundance, to binary driver-mutation status. Write the per-signature association statistics (enrichment direction, p-value, q-value) for downstream reporting; a result is considered a reproduced association when SBS2* and SBS13* reach q-values at or below the Qvalue = 0.1 threshold in the PIK3CA E545k-mutated group.

## Related tools

- **RNMF** (R package providing samFisherSigs and samPerMutSigs to test enrichment of chosen signature abundances (SBS2/SBS13) between PIK3CA E545k mutated and wild-type sample groups) — https://github.com/zhenzhang-li/RNMF
- **R** (Language in which RNMF is implemented and the association analysis loop over Sigstype signatures is executed)

## Examples

```
samFisherSigs(Sfile = Sfile, choose.Sigs = 'SBS2', file = maffile, sample.id = 'Tumor_Sample_Barcode', Hugo = 'Hugo_Symbol', Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1, plot = TRUE)
```

## Evaluation signals

- SBS2* and SBS13* appear in the result table with q-values at or below the Qvalue = 0.1 threshold in the PIK3CA E545k-mutated group
- Sample grouping correctly partitions tumors into PIK3CA E545k mutated vs wild-type based on Hugo_Symbol and MAF variant annotations
- Mutfreq = 0.04 and threshold = 0.06 parameters are applied consistently with the article's samFisherSigs/samPerMutSigs calls
- Output table contains per-signature enrichment direction, p-value, and q-value columns for each tested signature
- Plots (plot = TRUE) are generated per signature showing abundance separation between the two PIK3CA E545k groups

## Limitations

- Framework is slower than previously published software for de novo signature extraction due to the initial-value correction and solution-space processes
- Only coding regions were uniformly analyzed because of inconsistent sequencing background of the data, which introduces limitations
- The mechanism between APOBEC and PIK3CA is only hypothesized and needs verification by subsequent experiments

## Evidence

- [abstract] APOBEC signatures (SBS2* and SBS13*) were closely associated with PIK3CA mutation, especially the E545k mutation: "APOBEC signatures (SBS2* and SBS13*) were closely associated with PIK3CA mutation, especially the E545k mutation"
- [methods] samFisherSigs call with parameters: "Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1"
- [readme] samFisherSigs invocation from README: "samFisherSigs(Sfile = Sfile, choose.Sigs = isig, file = maffile,
                sample.id = 'Tumor_Sample_Barcode', Hugo = 'Hugo_Symbol', 
                Mutfreq = 0.04, threshold = 0.06, Qvalue ="
