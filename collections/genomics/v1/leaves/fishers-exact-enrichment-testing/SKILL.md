---
name: fishers-exact-enrichment-testing
description: Use when comparing binary trait counts (e.g., inverted-Alu-positive vs.
  Alu-negative BSJs, tissue-specific isoform vs. gene) against a matched control set,
  and a 2x2 contingency table with scipy.stats.fisher_exact is appropriate.
when_to_use_negative:
- Counts are not binary or the comparison involves more than two categories per variable
  (e.g., multi-tissue expression magnitude itself) — use tau index or chi-square/binomial
  criteria first, not Fisher's exact test directly
- No matched negative-control or background set is available, so a 2x2 enrichment
  table cannot be constructed
- Read-level or alignment-quality assessment of isoCirc data is the goal — that requires
  the isoCirc pipeline's stringent BSJ filtering rather than an enrichment test
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0092
tools:
- name: Python (scipy.stats.fisher_exact)
  role: Runs the two-sided and one-sided Fisher's exact tests on the assembled 2x2
    contingency tables and returns p-values and odds ratios
- name: bedtools
  role: Extracts 1000-nt and 2000-nt flanking intronic windows around each BSJ and
    intersects them with the RepeatMasker Alu BED annotation to build the inverted-Alu
    counts
  repo: https://bedtools.readthedocs.io/
- name: isoCirc
  role: Supplies the isocirc.out BSJ table (with --Alu and --flank-len options, default
    flank-len 500) that feeds the enrichment analysis
  repo: https://github.com/Xinglab/isoCirc
provenance:
  source_task_ids:
  - task_004
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-20459-8
    title: isoCirc catalogs full-length circular RNA isoforms in human transcriptomes
  source_content_hash: sha256:3be5ee931b8b07028d00c8fbc1849a9901b773eefadc479c9d252e4b76310e85
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - Python (scipy.stats.fisher_exact)
  - bedtools
  - isoCirc
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# fishers-exact-enrichment-testing

## Summary

Two-sided and one-sided Fisher's exact tests on 2x2 contingency tables used in the isoCirc paper to test whether known/novel back-splice junctions are enriched for inverted Alu repeats (p < 3.6e-50) and whether tissue-specific circRNA isoforms are enriched in tissue-specific parental genes (p < 2.2e-16).

## When to use

Use when comparing binary trait counts (e.g., inverted-Alu-positive vs. Alu-negative BSJs, tissue-specific isoform vs. gene) against a matched control set, and a 2x2 contingency table with scipy.stats.fisher_exact is appropriate.

## When NOT to use

- Counts are not binary or the comparison involves more than two categories per variable (e.g., multi-tissue expression magnitude itself) — use tau index or chi-square/binomial criteria first, not Fisher's exact test directly
- No matched negative-control or background set is available, so a 2x2 enrichment table cannot be constructed
- Read-level or alignment-quality assessment of isoCirc data is the goal — that requires the isoCirc pipeline's stringent BSJ filtering rather than an enrichment test

## Inputs

- isocirc.out table with HEK293 BSJ genomic coordinates and known/novel splice-site classifications (from task_001)
- GRCh37 RepeatMasker Alu annotation as a BED file (used via the isocirc --Alu option)
- 12-tissue circRNA isoform read-count table (isoformID, chrom, coordinates, readCount per tissue)
- GTEx V8 median gene TPM values for the same 12 human tissues
- Negative-control BSJ set (10,000 random non-BSJ splice-site pairs from the same gene annotation)

## Outputs

- alu_enrichment_fisher_tests.tsv (p-values and odds ratios for known vs. control and novel vs. control inverted-Alu comparisons)
- fisher_test_results.tsv (2x2 contingency table, odds ratio, p-value for tissue-specific isoform/gene enrichment)
- 2x2 contingency tables of feature-positive vs. feature-negative counts for each case subset and controls

## How to apply

First define the case set and a matched negative-control set sampled from the same annotation space (e.g., 10,000 random non-BSJ splice-site pairs for the Alu analysis). Tabulate feature-positive vs. feature-negative counts for each case subset (known BSJs, novel BSJs, tissue-specific isoforms) and the controls into 2x2 contingency tables. Run scipy.stats.fisher_exact per comparison, choosing a two-sided test for known/novel BSJ versus control inverted-Alu enrichment and a one-sided test for enrichment of tissue-specific isoforms in tissue-specific genes. Record the odds ratio alongside each p-value in a TSV output (e.g., alu_enrichment_fisher_tests.tsv, fisher_test_results.tsv) and validate against reported thresholds: p < 3.6e-50 for all known/novel vs. control Alu comparisons and p < 2.2e-16 for the tissue-specificity enrichment. For the tissue-specific test, define gene specificity beforehand with Yanai's tau = 1 - sum_i(1 - x_i/max(x)) / (n-1) computed from GTEx V8 median TPMs across the 12 tissues, using only isoforms flagged tissue-specific by the paper's chi-square/binomial criteria (3,653 isoforms).

## Related tools

- **Python (scipy.stats.fisher_exact)** (Runs the two-sided and one-sided Fisher's exact tests on the assembled 2x2 contingency tables and returns p-values and odds ratios)
- **bedtools** (Extracts 1000-nt and 2000-nt flanking intronic windows around each BSJ and intersects them with the RepeatMasker Alu BED annotation to build the inverted-Alu counts) — https://bedtools.readthedocs.io/
- **isoCirc** (Supplies the isocirc.out BSJ table (with --Alu and --flank-len options, default flank-len 500) that feeds the enrichment analysis) — https://github.com/Xinglab/isoCirc

## Examples

```
from scipy.stats import fisher_exact; oddsr, p = fisher_exact([[a, b], [c, d]], alternative='two-sided')
```

## Evaluation signals

- All known/novel BSJ vs. negative-control inverted-Alu Fisher's exact p-values fall below the reported threshold of 3.6e-50
- Tissue-specific isoform/gene enrichment p-value satisfies p < 2.2e-16
- The set of tissue-specific circRNA isoforms flagged by the chi-square/binomial criteria totals 3,653 isoforms
- Odds ratios are recorded in the TSV outputs alongside each p-value, and known BSJs show a higher proportion of inverted Alu repeats than novel BSJs
- The negative-control set (10,000 random non-BSJ splice-site pairs) is scanned with the identical window-scan and Alu-pair classification as the case set

## Limitations

- Known BSJs show a higher proportion of inverted Alu repeats than novel BSJs, so pooled known+novel analyses would mask this difference
- isoCirc is far from saturation at current sequencing depth; the control sampling and classification depend on the GRCh37 RepeatMasker annotation and gene annotation supplied
- Stringent, asymmetric BSJ filtering (stricter for novel splice sites than known splice sites) affects which BSJs enter the case sets and hence the contingency tables

## Evidence

- [discussion] Stringent filtering before enrichment testing: "we use stringent criteria to remove potential false-positive detection"
- [abstract] Short-read limitation motivating full-length circRNA case definitions: "an inherent limitation of short-read RNA-seq is that it does not experimentally determine the full-length sequences and exact exonic compositions of circRNAs"
- [readme] isoCirc pipeline context for BSJ classification: "isoCirc is a long-read sequencing strategy coupled with an integrated computational pipeline to characterize full-length circRNA isoforms using rolling circle amplification (RCA) followed by"
- [methods] Known BSJ reference databases used to label case sets: "Number of candidate BSJs reported in existing circRNA BSJ database (circBase / MiOncoCirc)"
