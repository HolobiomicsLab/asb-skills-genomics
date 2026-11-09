---
name: escc-sample-group-annotation
description: Use when you have a MAF-format mutation file keyed by Tumor_Sample_Barcode
  and Hugo_Symbol and need binary sample group labels (e.g., PIK3CA E545k vs wild-type)
  to run RNMF per-sample signature association tests.
when_to_use_negative:
- You need de novo signature extraction rather than testing a predefined group association
  — use RNMF's denovoNMF 5-step framework instead.
- Your input is a per-gene cumulative contribution abundance table (Gfile) rather
  than a per-sample Sfile and MAF; the gene-level question calls for genePerMutSigs
  with its own Gfile input.
- The mutation file lacks standardized Tumor_Sample_Barcode and Hugo_Symbol keys,
  so samples cannot be reliably mapped to groups.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0749
tools:
- name: RNMF
  role: R package providing samFisherSigs/samPerMutSigs, which consume the group-annotated
    samples together with the Sfile signature abundance matrix (Mutfreq = 0.04, threshold
    = 0.06, Qvalue = 0.1) to test APOBEC signature–PIK3CA E545k association
  repo: https://github.com/zhenzhang-li/RNMF
- name: R
  role: Language in which the annotation and the RNMF Fisher-style association functions
    are executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/cam4.4717
    title: A practical framework <scp>RNMF</scp> for exploring the association between
      mutational signatures and genes using gene cumulative contribution abundance
  source_content_hash: sha256:a49b6e4d10850b516c963f4649e35e2f9310c92189c80bd722554d3d26f9e1d7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhenzhang-li/RNMF
  tools:
  - R
  - RNMF
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# escc-sample-group-annotation

## Summary

Annotates each tumor sample in an ESCC MAF dataset as PIK3CA E545k-mutated versus wild-type (or other gene/hotspot groups), producing the group labels needed to test whether signature contribution abundances (e.g., SBS2*/SBS13*) differ between mutation groups via RNMF's samFisherSigs/samPerMutSigs.

## When to use

Use when you have a MAF-format mutation file keyed by Tumor_Sample_Barcode and Hugo_Symbol and need binary sample group labels (e.g., PIK3CA E545k vs wild-type) to run RNMF per-sample signature association tests.

## When NOT to use

- You need de novo signature extraction rather than testing a predefined group association — use RNMF's denovoNMF 5-step framework instead.
- Your input is a per-gene cumulative contribution abundance table (Gfile) rather than a per-sample Sfile and MAF; the gene-level question calls for genePerMutSigs with its own Gfile input.
- The mutation file lacks standardized Tumor_Sample_Barcode and Hugo_Symbol keys, so samples cannot be reliably mapped to groups.

## Inputs

- MAF-format somatic mutation file with Tumor_Sample_Barcode and Hugo_Symbol columns (e.g., Regular.ESCC.exon.WES.maf-input-nonsilent.txt)
- Sample contribution abundance matrix (Sfile, e.g., SBS.Smatrix.txt) from the RNMF cumulative contribution abundance (CCA) model
- Signature list to test (e.g., SBS2/SBS2* and SBS13/SBS13* from COSMIC Mutational Signatures v2.0)

## Outputs

- Binary sample group annotation (PIK3CA E545k-mutated vs wild-type) per Tumor_Sample_Barcode
- Per-signature association statistics table (enrichment direction, p-value, q-value) from samFisherSigs/samPerMutSigs, e.g., sigs_association_results.csv

## How to apply

Import somatic variants from the MAF file (somatic variants can be imported from a VCF or MAF file), using Tumor_Sample_Barcode as the sample key and Hugo_Symbol as the gene key. For each sample, inspect the MAF variant annotations for the target gene (PIK3CA) and its hotspot codon to assign the E545k-mutated versus wild-type label. Combine this group assignment with the sample contribution abundance matrix (Sfile) derived from the RNMF CCA model, restricted to the chosen signatures (choose.Sigs = isig, e.g., SBS2* and SBS13*). Then run samFisherSigs/samPerMutSigs with the article's thresholds Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1 to test enrichment of signature abundance between the groups. Correct application means the group file partitions all samples unambiguously and the resulting association table reports enrichment direction, p-value, and q-value per signature.

## Related tools

- **RNMF** (R package providing samFisherSigs/samPerMutSigs, which consume the group-annotated samples together with the Sfile signature abundance matrix (Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1) to test APOBEC signature–PIK3CA E545k association) — https://github.com/zhenzhang-li/RNMF
- **R** (Language in which the annotation and the RNMF Fisher-style association functions are executed)

## Examples

```
samFisherSigs(Sfile = Sfile, choose.Sigs = 'SBS2', file = maffile, sample.id = 'Tumor_Sample_Barcode', Hugo = 'Hugo_Symbol', Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1, plot = TRUE)
```

## Evaluation signals

- SBS2* and SBS13* both appear in the result table with q-values at or below Qvalue = 0.1 in the PIK3CA E545k-mutated group, reproducing the reported close APOBEC–PIK3CA E545k association.
- Every Tumor_Sample_Barcode in the MAF file is assigned exactly one group label (E545k-mutated vs wild-type) with no unmapped or duplicated samples.
- The association table includes enrichment direction, p-value, and q-value columns for each chosen signature (choose.Sigs = isig).
- The Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1 parameters match the article's reported settings for the Fisher-style test.
- The Sfile matrix rows align with the sample identifiers used in the group annotation (sample.id = 'Tumor_Sample_Barcode').

## Limitations

- The mechanism between APOBEC and PIK3CA is only hypothesized and needs verification by subsequent experiments; the group-association result is correlative, not causal.
- Due to the inconsistent sequencing background of the data, only coding regions were uniformly analyzed, which may affect mutation-based group assignment.
- The relatively small number of individuals studied limits statistical power of group comparisons.

## Evidence

- [abstract] APOBEC signatures (SBS2* and SBS13*) were closely associated with PIK3CA mutation, especially the E545k mutation: "APOBEC signatures (SBS2* and SBS13*) were closely associated with PIK3CA mutation, especially the E545k mutation"
- [readme] samFisherSigs call with parameters: "samFisherSigs(Sfile = Sfile, choose.Sigs = isig, file = maffile,
                sample.id = 'Tumor_Sample_Barcode', Hugo = 'Hugo_Symbol', 
                Mutfreq = 0.04, threshold = 0.06, Qvalue ="
- [methods] MAF/VCF import: "somatic variants can be imported from a Variant Call Format (VCF) file or a Mutation Annotation Format (MAF) file"
- [methods] Thresholds: "Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1"
