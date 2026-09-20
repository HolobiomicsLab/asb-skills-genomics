---
name: yanai-tau-index-computation
description: Use when you need to classify genes as tissue-specific from a per-tissue
  expression matrix (e.g., GTEx V8 median TPM for the same 12 tissues used by isoCirc)
  to test enrichment of tissue-specific isoforms among tissue-specific parental genes.
when_to_use_negative:
- Expression data covers a different tissue panel than the 12 isoCirc tissues — tau
  computed on mismatched tissues cannot be cross-tabulated against isoCirc isoform
  specificity calls
- Input is per-isoform rather than per-gene expression — tau classifies parental-gene
  specificity, not isoform-level specificity, which must instead be called with the
  paper's chi-square/binomial criteria
- Read counts are raw rather than normalized TPM — tau is sensitive to library-size
  scaling differences across tissues
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Python
  role: Implementation language for the tau computation, contingency-table construction,
    and Fisher's exact test (pandas DataFrame, scipy.stats.fisher_exact)
- name: isoCirc
  role: Source of the 12-tissue full-length circRNA isoform catalog and per-tissue
    isoform read counts whose parental genes are classified by tau
  repo: github.com/Xinglab/isoCirc
- name: scipy.stats.fisher_exact
  role: One-sided Fisher's exact test on the 2x2 contingency table to assess enrichment
    of tissue-specific isoforms in tissue-specific genes
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-20459-8
    title: isoCirc catalogs full-length circular RNA isoforms in human transcriptomes
  source_content_hash: sha256:8021060f9e28ed4fdabd97a5eee917d10d9923efa14b258797df65e5d6e5d79d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - isoCirc
  - scipy.stats.fisher_exact
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# yanai-tau-index-computation

## Summary

Computes Yanai's tissue-specificity index (tau) per parental gene from median TPM expression across the 12 human tissues, enabling a 2x2 contingency analysis of whether tissue-specific circRNA isoforms (3,653 identified by isoCirc) are enriched in tissue-specific parental genes.

## When to use

Use when you need to classify genes as tissue-specific from a per-tissue expression matrix (e.g., GTEx V8 median TPM for the same 12 tissues used by isoCirc) to test enrichment of tissue-specific isoforms among tissue-specific parental genes.

## When NOT to use

- Expression data covers a different tissue panel than the 12 isoCirc tissues — tau computed on mismatched tissues cannot be cross-tabulated against isoCirc isoform specificity calls
- Input is per-isoform rather than per-gene expression — tau classifies parental-gene specificity, not isoform-level specificity, which must instead be called with the paper's chi-square/binomial criteria
- Read counts are raw rather than normalized TPM — tau is sensitive to library-size scaling differences across tissues

## Inputs

- GTEx V8 median gene TPM values for the same 12 human tissues analyzed by isoCirc
- 12-tissue circRNA isoform read-count table from isoCirc (isoformID, chrom, coordinates, readCount per tissue)
- Mapping of circRNA isoforms to their parental genes

## Outputs

- Per-gene tau index values with tissue-specific gene classifications
- 2x2 contingency table (tissue-specific isoform yes/no vs tissue-specific gene yes/no)
- Fisher's exact test odds ratio and p-value written to fisher_test_results.tsv

## How to apply

Load GTEx V8 median gene TPM values for the same 12 human tissues profiled by isoCirc into a pandas DataFrame aligned to the parental genes of the 3,653 tissue-specific circRNA isoforms (flagged from the per-tissue read-count table using the paper's chi-square/binomial criteria). For each parental gene, compute Yanai's tau index: tau = 1 - sum_i(1 - x_i/max(x)) / (n-1), where x_i is the gene's TPM in tissue i and n = 12; genes with high tau are classified as tissue-specific. Cross-tabulate tissue-specific isoform status (yes/no) against tissue-specific parental gene status (yes/no) into a 2x2 contingency table. Run a one-sided Fisher's exact test (scipy.stats.fisher_exact) to test enrichment of tissue-specific isoforms in tissue-specific genes, expecting p < 2.2e-16 as reported in the paper. The rationale is that isoCirc cataloged 107,147 full-length circRNA isoforms across 12 human tissues and HEK293, providing the per-tissue isoform read counts that tau-based gene classification must be matched against.

## Related tools

- **Python** (Implementation language for the tau computation, contingency-table construction, and Fisher's exact test (pandas DataFrame, scipy.stats.fisher_exact))
- **isoCirc** (Source of the 12-tissue full-length circRNA isoform catalog and per-tissue isoform read counts whose parental genes are classified by tau) — github.com/Xinglab/isoCirc
- **scipy.stats.fisher_exact** (One-sided Fisher's exact test on the 2x2 contingency table to assess enrichment of tissue-specific isoforms in tissue-specific genes)

## Examples

```
from scipy.stats import fisher_exact; tau = 1 - sum(1 - tpm/tpm.max())/(len(tpm)-1); odds, p = fisher_exact([[a, b], [c, d]], alternative='greater')
```

## Evaluation signals

- Applying the paper's chi-square/binomial criteria to the per-tissue isoform read counts recovers exactly 3,653 tissue-specific circRNA isoforms
- The one-sided Fisher's exact test reproduces the reported enrichment with p-value < 2.2e-16
- fisher_test_results.tsv contains the contingency table, odds ratio, and p-value, and the p-value satisfies the p < 2.2e-16 threshold
- tau values fall in [0, 1] for every gene, with tau = 0 for genes expressed equally across all 12 tissues and tau approaching 1 for single-tissue genes
- The parental-gene set matched to GTEx TPM covers the same 12 tissues as the isoCirc read-count table with no tissue-name mismatches

## Limitations

- isoCirc is far from saturation at current sequencing depth, so additional circRNA isoforms (and their parental-gene classifications) would be expected with deeper sequencing
- Tissue-specificity calls depend on isoCirc isoform read counts in only 12 tissues plus HEK293; genes specific to unprofiled tissues may receive misleadingly high tau
- tau on GTEx median TPM measures whole-cell gene specificity; the isoCirc catalog was generated from whole cells and tissues, so subcellular isoform localization is not captured

## Evidence

- [other] tau formula applied per parental gene: [evidence span withheld — anchor status: not_found]
- [other] Fisher's exact test enrichment finding: [evidence span withheld — anchor status: not_found]
- [abstract] isoCirc 12-tissue catalog that the analysis builds on: "we generate a comprehensive catalog of 107,147 full-length circRNA isoforms across 12 human tissues and one human cell line (HEK293)"
- [readme] isoCirc pip installation for generating/processing the isoform read-count input: "**isoCirc** is written with `python`, please use `pip` to install **isoCirc**:"
