---
name: microarray-expression-normalization
description: Use when log-phase microarray expression data are available for paired
  strains that differ by a known duplication genotype, and the goal is to test whether
  genes within the duplicated region show induced expression relative to the non-duplicated
  clonal counterpart.
when_to_use_negative:
- The expression data are not paired duplicated/non-duplicated strains from the same
  clonal population (e.g. unpaired samples or different genetic backgrounds), since
  the design compares a duplicated to a non-duplicated strain within the same clonal
  population.
- The data are RNA-seq or long-read expression measurements rather than the deposited
  DNA microarray data, for which this microarray-based workflow was designed.
- The question concerns genes outside the duplicated regions or genome-wide differential
  expression across many strain contrasts rather than induction of genes contained
  within known duplications.
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0621
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/ncomms14061
    title: Transient structural variations have strong effects on quantitative traits
      and reproductive isolation in fission yeast
  source_content_hash: sha256:d2a565ac802e3041987d43f237a9eb3ac0786117cbb61cc16993bc58cb08603b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools: []
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# microarray-expression-normalization

## Summary

Compares DNA-microarray transcript expression between paired S. pombe strains (duplicated vs non-duplicated members of the same clonal population, dataset E-MTAB-4019) to quantify induction of genes inside duplications and its relationship to copy number.

## When to use

Use when log-phase microarray expression data are available for paired strains that differ by a known duplication genotype, and the goal is to test whether genes within the duplicated region show induced expression relative to the non-duplicated clonal counterpart.

## When NOT to use

- The expression data are not paired duplicated/non-duplicated strains from the same clonal population (e.g. unpaired samples or different genetic backgrounds), since the design compares a duplicated to a non-duplicated strain within the same clonal population.
- The data are RNA-seq or long-read expression measurements rather than the deposited DNA microarray data, for which this microarray-based workflow was designed.
- The question concerns genes outside the duplicated regions or genome-wide differential expression across many strain contrasts rather than induction of genes contained within known duplications.

## Inputs

- ArrayExpress E-MTAB-4019 DNA microarray expression data (log phase cultures, duplicated strain paired with non-duplicated strain from the same clonal population)
- Deposited duplication genotypes for the eight S. pombe strain pairs (copy number and duplicated-region coordinates)
- S. pombe gene annotations to map duplicated-region coordinates to gene sets

## Outputs

- per_pair_expression_induction.tsv with columns: strain pair, duplicated-gene set, induction statistic, significance
- Per-pair significance calls for induction of genes within duplications
- Spearman rank correlation between duplication copy number and expression induction across the eight pairs

## How to apply

Download the ArrayExpress E-MTAB-4019 microarray expression dataset derived from log phase cultures. For each of the eight strain pairs, load the deposited duplication genotypes and define the gene set located within the duplicated region. Compute the expression induction of those genes in the duplicated strain relative to its non-duplicated clonal counterpart from the same clonal population, and assess significance of the induction per pair. Then compute a Spearman rank correlation between duplication copy number and expression induction across the eight pairs, and write a per-pair summary table (strain pair, duplicated-gene set, induction statistic, significance) to per_pair_expression_induction.tsv.

## Evaluation signals

- In seven of the eight strain pairs, expression levels of genes within duplications are significantly induced; significantly fewer or more significant pairs suggests a processing error.
- Spearman rank correlation between duplication copy number and expression induction across the eight pairs should be reproducible when re-run on the deposited data.
- per_pair_expression_induction.tsv must contain all eight strain pairs, each with a duplicated-gene set, an induction statistic, and a significance value.
- Induction should be computed only for genes located within the defined duplicated region, and relative to the non-duplicated strain from the same clonal population.
- Some genes outside the duplicated regions may show altered expression; flagging such genes as part of the duplicated-region result indicates incorrect gene-set definition.

## Limitations

- The comparison is restricted to eight closely related strain pairs, so statistical power and generalizability are limited.
- Induction of duplicated genes does not imply expression changes are confined to the duplication: some genes outside the duplicated regions also showed altered expression levels.
- Expression was assayed from log phase cultures only; conclusions do not cover other growth conditions.

## Evidence

- [abstract] We assayed transcript expression from log phase cultures using DNA microarrays, each time comparing a duplicated to a non-duplicated strain from within the same clonal population.: "We assayed transcript expression from log phase cultures using DNA microarrays, each time comparing a duplicated to a non-duplicated strain from within the same clonal population."
- [abstract] Induction result across the eight pairs: "In seven out of the eight strain pairs, the expression levels of genes within duplications were significantly induced"
- [other] Dataset and workflow definition: [evidence span withheld — anchor status: not_found]
