---
name: signature-sample-association-testing
description: Use when you have per-sample signature abundance values (an Sfile from
  RNMF's CCA/InverseNMF) plus a MAF mutation file and want to test whether specific
  signatures are enriched in samples carrying a given gene mutation or hotspot variant.
when_to_use_negative:
- You want de novo signature extraction rather than testing pre-estimated signatures
  against a sample grouping — use RNMF's denovoNMF instead
- Your goal is gene-level cumulative contribution abundance association rather than
  per-sample grouping tests — use genePerMutSigs with a Gfile from cumulativeCA instead
- Input lacks per-sample signature abundance (Sfile) or a MAF file with Tumor_Sample_Barcode
  and Hugo_Symbol identifiers needed to define the mutated vs wild-type groups
edam_operation: http://edamontology.org/operation_3564
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0199
tools:
- name: RNMF
  role: R package providing samFisherSigs and samPerMutSigs for per-sample Fisher-style
    signature association testing with Mutfreq = 0.04, threshold = 0.06, Qvalue =
    0.1
  repo: https://github.com/zhenzhang-li/RNMF
- name: R
  role: Implementation language; the skill is invoked as R library(RNMF) calls on
    Sfile and maffile inputs
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/cam4.4717
    title: A practical framework <scp>RNMF</scp> for exploring the association between
      mutational signatures and genes using gene cumulative contribution abundance
  source_content_hash: sha256:01bbc25997f15db085fd19e9ebf6fad6ab5fb6f09512702cbc4022a234174775
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhenzhang-li/RNMF
  tools:
  - R
  - RNMF
  edam_operation: http://edamontology.org/operation_3564
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# signature-sample-association-testing

## Summary

Fisher-style per-sample testing of mutational signature contribution abundance (from the RNMF CCA model) against a mutation-based binary grouping of samples, e.g. PIK3CA E545k mutated vs wild-type, to detect signatures like SBS2*/SBS13* enriched in the mutated group.

## When to use

Use when you have per-sample signature abundance values (an Sfile from RNMF's CCA/InverseNMF) plus a MAF mutation file and want to test whether specific signatures are enriched in samples carrying a given gene mutation or hotspot variant.

## When NOT to use

- You want de novo signature extraction rather than testing pre-estimated signatures against a sample grouping — use RNMF's denovoNMF instead
- Your goal is gene-level cumulative contribution abundance association rather than per-sample grouping tests — use genePerMutSigs with a Gfile from cumulativeCA instead
- Input lacks per-sample signature abundance (Sfile) or a MAF file with Tumor_Sample_Barcode and Hugo_Symbol identifiers needed to define the mutated vs wild-type groups

## Inputs

- Sample contribution abundance file (Sfile, e.g. SBS.Smatrix.txt) from the RNMF cumulative contribution abundance (CCA) model
- MAF-format somatic mutation file (e.g. Regular.ESCC.exon.WES.maf-input-nonsilent.txt) with Tumor_Sample_Barcode and Hugo_Symbol columns
- Chosen signature names for choose.Sigs (e.g. 'SBS2', 'SBS13' from COSMIC Mutational Signatures v2.0/v3.2)

## Outputs

- Per-signature association statistics table (enrichment direction, p-value, q-value), e.g. sigs_association_results.csv
- Optional per-signature association plots (plot = TRUE)

## How to apply

Load the sample contribution abundance file (Sfile) derived from the CCA model and restrict to the chosen signatures via choose.Sigs (e.g. SBS2* and SBS13* for the APOBEC test). Load the MAF file keyed by Tumor_Sample_Barcode and Hugo_Symbol, and annotate each tumor sample as PIK3CA E545k mutated vs wild-type from the MAF variant annotations. Run RNMF's samFisherSigs (or samPerMutSigs) with Mutfreq = 0.04, threshold = 0.06, and Qvalue = 0.1 to test enrichment of the signature abundance between the two groups. The function returns per-signature statistics (enrichment direction, p-value, q-value) and can plot results; write these to a results table such as sigs_association_results.csv. The association is considered reproduced when SBS2* and SBS13* show q-values at or below the 0.1 threshold in the PIK3CA E545k-mutated group.

## Related tools

- **RNMF** (R package providing samFisherSigs and samPerMutSigs for per-sample Fisher-style signature association testing with Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1) — https://github.com/zhenzhang-li/RNMF
- **R** (Implementation language; the skill is invoked as R library(RNMF) calls on Sfile and maffile inputs)

## Examples

```
samFisherSigs(Sfile = Sfile, choose.Sigs = 'SBS2', file = maffile, sample.id = 'Tumor_Sample_Barcode', Hugo = 'Hugo_Symbol', Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1, plot = TRUE)
```

## Evaluation signals

- SBS2* and SBS13* appear in the result table with q-values at or below Qvalue = 0.1 in the PIK3CA E545k-mutated group (reproduces the reported close APOBEC–PIK3CA E545k association)
- Output table contains, for each chosen signature, an enrichment direction, a p-value, and a q-value computed at the stated parameters (Mutfreq = 0.04, threshold = 0.06)
- Sample identifiers in the Sfile and MAF file agree via Tumor_Sample_Barcode, and PIK3CA E545k annotation cleanly partitions samples into mutated vs wild-type groups
- Rerunning samFisherSigs and samPerMutSigs on the same Sfile/maffile yields concordant association calls for the same signatures

## Limitations

- Mechanism between APOBEC and PIK3CA is only hypothesized and needs verification by subsequent experiments
- Due to the inconsistent sequencing background of the data, only coding regions were uniformly analyzed, which introduces limitations
- Framework spends more time than previously published software for de novo extraction, due to the 'Re-updated the initial value' correction and 'build a solution space' processes

## Evidence

- [abstract] APOBEC signatures (SBS2* and SBS13*) were closely associated with PIK3CA mutation, especially the E545k mutation: "APOBEC signatures (SBS2* and SBS13*) were closely associated with PIK3CA mutation, especially the E545k mutation"
- [methods] samPerMutSigs call with parameters: "samPerMutSigs(Sfile = Sfile, choose.Sigs = isig,
                file = maffile"
- [methods] Mutfreq/threshold/Qvalue parameters: "Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1"
- [readme] README example invocation: "samFisherSigs(Sfile = Sfile, choose.Sigs = isig, file = maffile,
                sample.id = 'Tumor_Sample_Barcode', Hugo = 'Hugo_Symbol', 
                Mutfreq = 0.04, threshold = 0.06, Qvalue ="
- [discussion] Coding-region-only analysis limitation: "Due to the inconsistent sequencing background of the data, we uniformly analyzed the data of coding regions"
