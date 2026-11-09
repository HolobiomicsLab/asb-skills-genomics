---
name: paired-strain-differential-expression
description: Use when you have expression data from strains that carry a duplication
  paired with a non-duplicated strain of the same clonal population, and you need
  to test whether duplicated-region genes are induced and whether induction tracks
  copy number.
when_to_use_negative:
- Expression comparisons are not clonally paired (duplicated and non-duplicated strains
  come from different clonal populations), so genetic background confounds the induction
  estimate
- The variant of interest is not a duplication/CNV (e.g. translocations or inversions,
  which the authors excluded from genotyping re-examination because they were more
  challenging to accurately genotype)
- The data are RNA-seq rather than the deposited DNA microarray expression data, requiring
  a different quantification pipeline
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: SURVIVOR
  role: Consensus merging/comparison of SV caller VCFs that produced the duplication
    genotypes defining the duplicated gene sets
  repo: https://github.com/fritzsedlazeck/SURVIVOR
- name: cn.MOPS
  role: Copy-number inference whose calls were manually curated to establish duplication
    genotypes
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/ncomms14061
    title: Transient structural variations have strong effects on quantitative traits
      and reproductive isolation in fission yeast
  source_content_hash: sha256:eae5669a52d75e62d47076602abb63ddd85fc1907af041a66d3fa03efe731a7a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fritzsedlazeck/SURVIVOR
  tools:
  - SURVIVOR
  - cn.MOPS
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# Paired-strain differential expression from microarray data

## Summary

Tests whether genes inside naturally occurring duplications are induced in expression, by comparing log-phase microarray transcript levels of each duplicated S. pombe strain against a non-duplicated strain from the same clonal population, and correlating induction with duplication copy number across the eight strain pairs.

## When to use

Use when you have expression data from strains that carry a duplication paired with a non-duplicated strain of the same clonal population, and you need to test whether duplicated-region genes are induced and whether induction tracks copy number.

## When NOT to use

- Expression comparisons are not clonally paired (duplicated and non-duplicated strains come from different clonal populations), so genetic background confounds the induction estimate
- The variant of interest is not a duplication/CNV (e.g. translocations or inversions, which the authors excluded from genotyping re-examination because they were more challenging to accurately genotype)
- The data are RNA-seq rather than the deposited DNA microarray expression data, requiring a different quantification pipeline

## Inputs

- ArrayExpress E-MTAB-4019 microarray expression data (log phase cultures, duplicated strain paired with non-duplicated strain from same clonal population)
- Duplication genotypes / SV catalogue giving duplicated regions per strain (curated 113-SV callset: 23 deletions, 64 duplications, 11 inversions, 15 translocations)
- S. pombe gene annotations to map duplicated regions to gene sets

## Outputs

- per_pair_expression_induction.tsv — per-pair summary table (strain pair, duplicated-gene set, induction statistic, significance)
- Spearman rank correlation between duplication copy number and expression induction across the eight pairs

## How to apply

Download the deposited microarray dataset (ArrayExpress E-MTAB-4019), which assayed transcript expression from log phase cultures with each comparison being a duplicated vs. non-duplicated strain from within the same clonal population. Load the curated duplication genotypes (from the 113-SV catalogue established with Delly, Lumpy, Pindel, cn.MOPs and SURVIVOR) and, for each of the eight strain pairs, define the gene set located within the duplicated region. For each pair, compute the expression induction of those genes in the duplicated strain relative to its clonal counterpart and assess significance of the induction, noting that genes outside duplicated regions can also show altered expression. Finally, compute a Spearman rank correlation between duplication copy number and expression induction across the eight pairs, and write a per-pair summary table (strain pair, duplicated-gene set, induction statistic, significance) to per_pair_expression_induction.tsv. The expected outcome, serving as a check, is significant induction in seven of the eight strain pairs.

## Related tools

- **SURVIVOR** (Consensus merging/comparison of SV caller VCFs that produced the duplication genotypes defining the duplicated gene sets) — https://github.com/fritzsedlazeck/SURVIVOR
- **cn.MOPS** (Copy-number inference whose calls were manually curated to establish duplication genotypes)

## Examples

```
wget -r -np https://www.ebi.ac.uk/arrayexpress/experiments/E-MTAB-4019/  # log-phase microarray data for the eight duplicated/non-duplicated clonal strain pairs
```

## Evaluation signals

- Per-pair table contains exactly eight rows, one per duplicated vs. non-duplicated clonal strain pair
- Significant induction of duplicated-region genes is reproduced in 7 of 8 strain pairs (as reported in the paper)
- Spearman rank correlation between copy number and induction is computed across all eight pairs with a reported ρ and P value
- Each duplicated-gene set matches the annotated genes falling inside the curated duplication coordinates for that strain
- Output file parses as a TSV with the required columns: strain pair, duplicated-gene set, induction statistic, significance

## Limitations

- Induction was only tested within clonally paired strains, so results do not generalize to unpaired strain comparisons
- Some genes outside the duplicated regions also showed altered expression levels, so induced expression is not exclusive to duplicated genes
- The SV callset prioritized high specificity over high sensitivity, so not all duplications present in the strains may be captured in the genotype inputs

## Evidence

- [abstract] In seven of the eight strain pairs, duplicated-gene expression is induced: "In seven out of the eight strain pairs, the expression levels of genes within duplications were significantly induced"
- [abstract] Microarray expression was assayed in log phase with clonal duplicated vs non-duplicated pairs: "We assayed transcript expression from log phase cultures using DNA microarrays, each time comparing a duplicated to a non-duplicated strain from within the same clonal population."
- [abstract] Expression changes also occur outside duplicated regions: "some genes outside the duplicated regions also showed altered expression levels"
- [readme] SURVIVOR merges and compares SV calls into consensus genotypes: "SURVIVOR is a tool set for simulating/evaluating SVs, merging and comparing SVs within and among samples"
