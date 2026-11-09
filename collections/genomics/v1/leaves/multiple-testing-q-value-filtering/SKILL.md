---
name: multiple-testing-q-value-filtering
description: Use when testing whether a specific mutational signature's per-sample
  contribution abundance differs between mutation-status groups of a gene (e.g. PIK3CA
  E545k mutated vs wild-type) in MAF-annotated tumor cohorts.
when_to_use_negative:
- Input has not passed through RNMF denovoNMF/InverseNMF signature extraction and
  has no Sfile contribution abundance matrix — run cumulativeCA first
- The goal is de novo extraction of mutational signatures from raw VCF/MAF data rather
  than testing associations of pre-defined signatures with gene mutation groups
- Grouping requires clinical covariates or survival endpoints rather than a binary
  gene-mutation split (Mutfreq = 0.04, threshold = 0.06 are tuned for hotspot mutation
  grouping)
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0622
tools:
- name: RNMF
  role: R package providing samFisherSigs/samPerMutSigs for Fisher-style per-sample
    signature association testing with q-value FDR control
  repo: https://github.com/zhenzhang-li/RNMF
- name: R
  role: Language in which RNMF is implemented and invoked for the association analysis
  repo: https://github.com/zhenzhang-li/RNMF
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/cam4.4717
    title: A practical framework <scp>RNMF</scp> for exploring the association between
      mutational signatures and genes using gene cumulative contribution abundance
  source_content_hash: sha256:e48897b8e593332100b3e3f3b09e9e35baf6b4c02654741f8fe6b4f2ff89a97a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhenzhang-li/RNMF
  tools:
  - R
  - RNMF
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# multiple-testing-q-value-filtering

## Summary

A per-signature Fisher-style association analysis in the RNMF R package (samFisherSigs/samPerMutSigs) that tests whether a chosen mutational signature (e.g. SBS2*/SBS13*) is enriched in a gene-mutation group (e.g. PIK3CA E545k vs wild-type), applying Mutfreq = 0.04, threshold = 0.06, and a q-value FDR cutoff of Qvalue = 0.1 to call significant associations.

## When to use

Use when testing whether a specific mutational signature's per-sample contribution abundance differs between mutation-status groups of a gene (e.g. PIK3CA E545k mutated vs wild-type) in MAF-annotated tumor cohorts.

## When NOT to use

- Input has not passed through RNMF denovoNMF/InverseNMF signature extraction and has no Sfile contribution abundance matrix — run cumulativeCA first
- The goal is de novo extraction of mutational signatures from raw VCF/MAF data rather than testing associations of pre-defined signatures with gene mutation groups
- Grouping requires clinical covariates or survival endpoints rather than a binary gene-mutation split (Mutfreq = 0.04, threshold = 0.06 are tuned for hotspot mutation grouping)

## Inputs

- SBS.Smatrix.txt sample contribution abundance file (Sfile) from RNMF cumulative contribution abundance (CCA) model
- MAF-format somatic mutation file (e.g. Regular.ESCC.exon.WES.maf-input-nonsilent.txt) with Tumor_Sample_Barcode and Hugo_Symbol columns
- Signature name string (choose.Sigs, e.g. 'SBS2', 'SBS13') from COSMIC Mutational Signatures v2.0/v3.2
- Gene-mutation group annotation (PIK3CA E545k mutated vs wild-type per sample)

## Outputs

- Per-signature association statistics table (enrichment direction, p-value, q-value) written to sigs_association_results.csv
- Association plot per signature (plot = TRUE)

## How to apply

Load the sample contribution abundance file (Sfile) from the RNMF CCA model and restrict it to the chosen signature via choose.Sigs = isig (e.g. 'SBS2' or 'SBS13'). Provide a MAF-format mutation file keyed by Tumor_Sample_Barcode and Hugo_Symbol, from which samples are grouped as PIK3CA E545k mutated vs wild-type. Run samFisherSigs (or samPerMutSigs) with Mutfreq = 0.04, threshold = 0.06, and Qvalue = 0.1 to test enrichment of the signature's abundance between the groups, writing per-signature statistics (enrichment direction, p-value, q-value). A signature is considered closely associated when its q-value falls at or below the Qvalue = 0.1 threshold in the mutated group, as reproduced for SBS2*/SBS13* with PIK3CA E545k in the ESCC cohort.

## Related tools

- **RNMF** (R package providing samFisherSigs/samPerMutSigs for Fisher-style per-sample signature association testing with q-value FDR control) — https://github.com/zhenzhang-li/RNMF
- **R** (Language in which RNMF is implemented and invoked for the association analysis) — https://github.com/zhenzhang-li/RNMF

## Examples

```
samFisherSigs(Sfile = Sfile, choose.Sigs = 'SBS2', file = maffile, sample.id = 'Tumor_Sample_Barcode', Hugo = 'Hugo_Symbol', Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1, plot = TRUE)
```

## Evaluation signals

- SBS2* and SBS13* appear in the result table with q-values at or below Qvalue = 0.1 in the PIK3CA E545k-mutated group
- Output table sigs_association_results.csv contains enrichment direction, p-value, and q-value columns for each tested signature
- Every sample in the MAF file is assigned to either the PIK3CA E545k-mutated or wild-type group with no missing Tumor_Sample_Barcode matches against the Sfile
- Signature abundance vectors are restricted to the chosen isig (e.g. only SBS2 or SBS13) before testing, confirming choose.Sigs was applied

## Limitations

- The mechanism between APOBEC and PIK3CA is only hypothesized and needs verification by subsequent experiments
- Framework spends more time on de novo extraction of signature analysis compared with previously published software; the association step depends on upstream CCA/Sfile computation
- Analysis limited by inconsistent sequencing background of the data; only coding regions were uniformly analyzed

## Evidence

- [readme] samFisherSigs(Sfile = Sfile, choose.Sigs = isig, file = maffile, sample.id = 'Tumor_Sample_Barcode', Hugo = 'Hugo_Symbol', Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1, plot = TRUE): "samFisherSigs(Sfile = Sfile, choose.Sigs = isig, file = maffile,
                sample.id = 'Tumor_Sample_Barcode', Hugo = 'Hugo_Symbol', 
                Mutfreq = 0.04, threshold = 0.06, Qvalue ="
- [abstract] APOBEC signatures (SBS2* and SBS13*) were closely associated with PIK3CA mutation, especially the E545k mutation: "APOBEC signatures (SBS2* and SBS13*) were closely associated with PIK3CA mutation, especially the E545k mutation"
- [methods] Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1: "Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1"
- [methods] samPerMutSigs(Sfile = Sfile, choose.Sigs = isig, file = maffile: "samPerMutSigs(Sfile = Sfile, choose.Sigs = isig,
                file = maffile"
