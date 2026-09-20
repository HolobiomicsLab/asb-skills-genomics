---
name: leukemia-subtype-annotation
description: Use when a bulk RNA-seq cohort of AML/MDS samples (e.g. Beat AML from
  vizome.org) needs WHO subtype/lesion annotation and quantitative lineage scoring
  to verify that signature scores peak in expected subtypes.
when_to_use_negative:
- Input is single-cell data rather than bulk RNA-seq profiles — the z-score/mean-of-genes
  signature approach was validated on bulk Beat AML profiles
- The cohort lacks reliable WHO subtype or lesion-status annotations, making per-subtype
  score summarization impossible
- The goal is genotyping (SNVs, fusions, isoforms) rather than lineage-expression
  scoring — that requires the long-read nanoranger workflow, not expression signatures
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: nanoranger
  role: Optional complementary long-read single-cell genotyping pipeline (ONT/PacBio
    from 10x Genomics cDNA libraries) for lineage-defining features such as SNVs,
    fusions and isoforms in AML subclones, supplementing expression-based subtype
    annotation
  repo: https://github.com/mehdiborji/nanoranger
- name: nanoranger.R
  role: R package for downstream processing of nanoranger output (SNVs, indels, fusion
    genes/CAR sequences, isoforms) when molecular lesion annotation of subtypes is
    needed beyond expression signatures
  repo: https://github.com/liviuspenter/nanoranger.R
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-44137-7
    title: Integrative genotyping of cancer and immune phenotypes by long-read sequencing
  source_content_hash: sha256:16362ad66779e3981547094eebb34ec8f40961ab1f19ed7df78028ea356c6890
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mehdiborji/nanoranger
  tools:
  - nanoranger
  - nanoranger.R
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# leukemia-subtype-annotation

## Summary

Annotate bulk leukemia transcriptomic samples (e.g. the 646 Beat AML RNA-seq profiles) with WHO subtype and lesion status, then compute z-score-based lineage signature scores (5-gene erythroid: GATA1, CA1, HBA1, HBB, ALAD; 5-gene megakaryocytic: GATA2, ZFPM1, FLI1, NFE2, PF4) to map lineage identity across AML/MDS subtypes.

## When to use

Use when a bulk RNA-seq cohort of AML/MDS samples (e.g. Beat AML from vizome.org) needs WHO subtype/lesion annotation and quantitative lineage scoring to verify that signature scores peak in expected subtypes.

## When NOT to use

- Input is single-cell data rather than bulk RNA-seq profiles — the z-score/mean-of-genes signature approach was validated on bulk Beat AML profiles
- The cohort lacks reliable WHO subtype or lesion-status annotations, making per-subtype score summarization impossible
- The goal is genotyping (SNVs, fusions, isoforms) rather than lineage-expression scoring — that requires the long-read nanoranger workflow, not expression signatures

## Inputs

- Bulk RNA-seq expression matrix for the Beat AML cohort (from vizome.org), subset to the 646 profiles of interest
- Per-sample WHO subtype annotations (AML/MDS subtypes)
- Per-sample lesion status (e.g. CEBPA mutation, inv(3)/RPN1-EVI1)

## Outputs

- Per-sample erythroid signature score (mean z-scored expression of GATA1, CA1, HBA1, HBB, ALAD)
- Per-sample megakaryocytic signature score (mean z-scored expression of GATA2, ZFPM1, FLI1, NFE2, PF4)
- Per-subtype summary table of mean signature scores with WHO subtype and lesion-status annotations

## How to apply

Retrieve the bulk RNA-seq expression matrix (Beat AML cohort from vizome.org) and subset to the target profiles (e.g. the 646 used in the paper). Z-score normalize expression of each signature gene per cohort, then compute a per-sample erythroid score as the mean normalized expression of GATA1, CA1, HBA1, HBB and ALAD, and a megakaryocytic score as the mean of GATA2, ZFPM1, FLI1, NFE2 and PF4. Annotate each sample with its WHO subtype and lesion status (e.g. CEBPA mutation, inv(3)/RPN1-EVI1). Finally summarize mean signature scores per subtype and confirm the expected pattern: highest erythroid and megakaryocytic scores in essential thrombocythemia and acute erythroid/megakaryoblastic leukemia, with elevated scores in CEBPA-mutated AML and inv(3)/RPN1-EVI1 cases.

## Related tools

- **nanoranger** (Optional complementary long-read single-cell genotyping pipeline (ONT/PacBio from 10x Genomics cDNA libraries) for lineage-defining features such as SNVs, fusions and isoforms in AML subclones, supplementing expression-based subtype annotation) — https://github.com/mehdiborji/nanoranger
- **nanoranger.R** (R package for downstream processing of nanoranger output (SNVs, indels, fusion genes/CAR sequences, isoforms) when molecular lesion annotation of subtypes is needed beyond expression signatures) — https://github.com/liviuspenter/nanoranger.R

## Examples

```
scores <- cbind(erythroid=rowMeans(z[,c('GATA1','CA1','HBA1','HBB','ALAD')]), megakaryocytic=rowMeans(z[,c('GATA2','ZFPM1','FLI1','NFE2','PF4')])); aggregate(scores, by=list(subtype), FUN=mean)
```

## Evaluation signals

- Erythroid and megakaryocytic scores are highest in essential thrombocythemia and in acute erythroid/megakaryoblastic leukemia samples
- CEBPA-mutated AML and inv(3)/RPN1-EVI1 cases show the paper-reported elevated signature score pattern
- Signature scores are computed from z-score normalized expression (mean of the 5 genes per signature) for every one of the 646 profiles
- Each sample carries a non-empty WHO subtype and lesion-status annotation before score summarization
- Per-subtype mean scores reproduce the published ranking when re-derived from the vizome.org Beat AML matrix

## Limitations

- Expression profiles alone can be insufficient to unambiguously distinguish malignant from physiologic populations, which the paper notes is 'oftentimes impossible based on gene expression profiles alone'; genotyping barcodes may be needed
- Unlisted or ambiguous WHO subtype assignments in the cohort prevent correct per-subtype aggregation
- Signature scores are cohort-dependent: z-score normalization must be recomputed within the retrieved Beat AML matrix rather than imported from another cohort

## Evidence

- [intro] A crucial prerequisite of cancer single cell studies is the unambiguous identification of malignant versus physiologic populations: "A crucial prerequisite of cancer single cell studies is the unambiguous identification of malignant versus physiologic populations"
- [abstract] nanoranger applied to AML subclones and immune-cell isoforms: "we apply nanoranger to longitudinal tracking of subclones of acute myeloid leukemia (AML)"
- [methods] nanoranger input from 10x Genomics cDNA libraries or targeted amplicons: "The input data can be obtained through sequencing of 10x Genomics whole-transcriptome cDNA libraries"
- [abstract] nanoranger detects lineage-defining features from single-cell cDNA libraries: "starts from intermediate single-cell cDNA libraries to detect cell lineage-defining features"
- [readme] nanoranger.R processes SNVs, indels, fusions and isoforms from long-read amplicons: "read out single nucleotide variants, insertions/deletions,
fusion genes/CAR sequences and isoforms from amplicons sequenced with long-read sequencing"
