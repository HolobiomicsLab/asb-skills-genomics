---
name: cnv-expression-correlation-analysis
description: Use when a gene-focused question requires pairing CNV deletion calls
  with RNA-seq expression profiles in GutUDB to test whether copy-number loss explains
  (or fails to explain) transcript levels.
when_to_use_negative:
- The question concerns SNPs, structural variations, or mutated genes rather than
  CNV deletion calls (GutUDB curates these as separate genomics categories)
- Only one data modality is available — this skill requires both CNV genomics and
  RNA-seq transcriptomics profiles for the same gene and cohort
- The goal is drug-disease or microbe-metabolite association queries, which GutUDB
  handles through its THERAPY and metabolite modules instead
edam_operation: http://edamontology.org/operation_3233
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3912
tools:
- name: GutUDB
  role: Source database supplying the CNV genomics profiles and RNA-seq transcriptomics
    profiles queried for the gene–disease pair
  repo: github.com/Ansanqi/GutUDB
- name: manual expert review
  role: Curation/verification step used to confirm the computed deletion frequency
    and weak correlation against the reported values
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/imt2.195
    title: 'GutUDB: A comprehensive multiomics database for intestinal diseases'
  source_content_hash: sha256:cfff952e764a6d621d07ab698e3122612bb03608e02df5460eae017285766734
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GutUDB
  - manual expert review
  edam_operation: http://edamontology.org/operation_3233
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3912
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# cnv-expression-correlation-analysis

## Summary

Quantifies how copy number variation (CNV) deletion frequency of a gene relates to its RNA expression in intestinal-disease cohorts using GutUDB's genomics and transcriptomics profiles, e.g. METTL3's 0.04 deletion frequency and weak CNV–RNA correlation in colorectal cancer.

## When to use

Use when a gene-focused question requires pairing CNV deletion calls with RNA-seq expression profiles in GutUDB to test whether copy-number loss explains (or fails to explain) transcript levels.

## When NOT to use

- The question concerns SNPs, structural variations, or mutated genes rather than CNV deletion calls (GutUDB curates these as separate genomics categories)
- Only one data modality is available — this skill requires both CNV genomics and RNA-seq transcriptomics profiles for the same gene and cohort
- The goal is drug-disease or microbe-metabolite association queries, which GutUDB handles through its THERAPY and metabolite modules instead

## Inputs

- GutUDB CNV genomics profiles for the gene of interest in the chosen intestinal disease cohort (github.com/Ansanqi/GutUDB)
- Matching GutUDB RNA-seq transcriptomics profiles for the same gene and disease

## Outputs

- CNV deletion frequency (fraction of samples with a deletion call, e.g. 0.04 for METTL3 in CRC)
- Correlation coefficient (Pearson or Spearman) between CNV state and RNA expression
- Tabular result file (e.g. mettl3_cnv_results.csv) containing deletion frequency and correlation coefficient

## How to apply

Retrieve the METTL3-focused CNV genomics profiles for the target intestinal disease (e.g. colorectal cancer) from GutUDB (github.com/Ansanqi/GutUDB), which aggregates 54,131 CNVs and 72,248 SNPs across 56 intestinal diseases. Pull the matching METTL3 RNA-seq transcriptomics profiles from GutUDB's 137 RNA-seq data sets (7.9 million transcriptional profiles). Compute CNV deletion frequency as the fraction of disease samples carrying a deletion call at the gene locus, then compute a Pearson or Spearman correlation between CNV state and RNA expression level. Compare the deletion frequency against reference values (e.g. METTL3 = 0.04 in CRC) and check the correlation falls in the weak range despite significantly high RNA expression. Assemble the deletion frequency and correlation coefficient into a tabular result file (e.g. mettl3_cnv_results.csv) via GutUDB's Browse, Query, Visualization, and Download functionality or manual expert review.

## Related tools

- **GutUDB** (Source database supplying the CNV genomics profiles and RNA-seq transcriptomics profiles queried for the gene–disease pair) — github.com/Ansanqi/GutUDB
- **manual expert review** (Curation/verification step used to confirm the computed deletion frequency and weak correlation against the reported values)

## Examples

```
python compute_cnv_rna_correlation.py --gene METTL3 --disease colorectal_cancer --cnv GutUDB/genomics/cnv_crc.csv --rna GutUDB/transcriptomics/rnaseq_crc.csv --out mettl3_cnv_results.csv
```

## Evaluation signals

- Computed deletion frequency matches the reference value (METTL3 = 0.04 in colorectal cancer)
- Correlation coefficient between METTL3 CNV and RNA expression falls in the weak range
- Result file (mettl3_cnv_results.csv) contains both the deletion frequency and correlation coefficient in a small tabular schema
- CNV and RNA-seq profiles come from the same disease cohort and gene within GutUDB (56 intestinal diseases, 137 RNA-seq data sets)
- High RNA expression of the gene is confirmed independently of the CNV state, reproducing the reported dissociation

## Limitations

- Weak CNV–RNA correlation means deletion frequency alone cannot explain elevated RNA expression; other regulatory mechanisms must be considered
- Findings are cohort- and database-specific (GutUDB's ~9 million profiles across 56 intestinal diseases and six species); other repositories may yield different frequencies
- Workflow relies on manual expert review rather than an automated pipeline, so reproducibility depends on consistent querying of GutUDB

## Evidence

- [other] METTL3 shows a CNV deletion frequency of only 0.04 in CRC and a weak correlation between CNV and RNA expression, despite significantly high RNA expression of the gene.: [evidence span withheld — anchor status: not_found]
- [abstract] GutUDB genomics content includes 72,248 SNPs, 54,131 CNVs, 1097 structural variations, and 92,888 mutated genes: "GutUDB currently includes 72,248 single‐nucleotide polymorphisms (SNPs), 54,131 CNVs, 1097 structural variations"
- [abstract] GutUDB transcriptomics comprises 137 RNA-seq data sets with 7.9 million transcriptional profiles, plus 62 circular RNAs, 182 lncRNAs, and 58 microRNAs: "137 RNA‐seq data sets related to 7.9 million transcriptional profiles"
- [abstract] GutUDB integrates four core functionalities: Browse, Query, Visualization, and Download: "four core functionalities—Browse, Query, Visualization, and Download—were integrated into GutUDB"
