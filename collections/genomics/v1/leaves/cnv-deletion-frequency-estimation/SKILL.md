---
name: cnv-deletion-frequency-estimation
description: Use when a query asks for the CNV deletion frequency of a specific gene
  in an intestinal disease and the correlation between CNV state and RNA expression,
  to interpret whether high RNA expression is driven by copy gain or persists despite
  deletions.
when_to_use_negative:
- The question concerns single-nucleotide polymorphisms (72,248 SNPs) or structural
  variations rather than gene-level CNV deletion calls
- The expression data are non-RNA-seq modalities (e.g. circular RNAs, lncRNAs, microRNAs,
  or m6A modification sites) that are not matched to CNV sample calls
- Only a single sample is available — deletion frequency and CNV-expression correlation
  require a cohort of samples
edam_topics:
- http://edamontology.org/topic_3912
- http://edamontology.org/topic_3308
tools:
- name: GutUDB
  role: Source of CNV genomics profiles (54,131 CNVs) and RNA-seq transcriptomics
    profiles used to compute deletion frequency and CNV-expression correlation; accessed
    via Browse, Query, Visualization, Download or the GitHub repo
  repo: https://github.com/Ansanqi/GutUDB
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/imt2.195
    title: 'GutUDB: A comprehensive multiomics database for intestinal diseases'
  source_content_hash: sha256:f0393b90e5403982bc980d37725cef7c07c350d14d00d4947f7307fca13d12eb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ansanqi/GutUDB
  tools:
  - GutUDB
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3912
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# cnv-deletion-frequency-estimation

## Summary

Estimates the deletion frequency of a gene's copy number variation (CNV) across disease samples and quantifies the strength of the CNV-vs-RNA-expression correlation, using CNV genomics and RNA-seq transcriptomics profiles from GutUDB (e.g. METTL3 deletion frequency 0.04 in CRC).

## When to use

Use when a query asks for the CNV deletion frequency of a specific gene in an intestinal disease and the correlation between CNV state and RNA expression, to interpret whether high RNA expression is driven by copy gain or persists despite deletions.

## When NOT to use

- The question concerns single-nucleotide polymorphisms (72,248 SNPs) or structural variations rather than gene-level CNV deletion calls
- The expression data are non-RNA-seq modalities (e.g. circular RNAs, lncRNAs, microRNAs, or m6A modification sites) that are not matched to CNV sample calls
- Only a single sample is available — deletion frequency and CNV-expression correlation require a cohort of samples

## Inputs

- METTL3-focused (or gene-of-interest) CNV genomics profiles for colorectal cancer from GutUDB
- Matching METTL3 RNA-seq transcriptomics profiles from GutUDB's 137 RNA-seq data sets
- Disease-gene association records from GutUDB (260,790 embedded associations) to scope the disease cohort

## Outputs

- CNV deletion frequency as the fraction of colorectal cancer samples with a deletion call at the gene (e.g. 0.04 for METTL3)
- CNV-vs-RNA-expression correlation coefficient (Pearson or Spearman) with weak/strong classification
- Tabular results file mettl3_cnv_results.csv

## How to apply

Retrieve the gene-focused CNV genomics profiles for the target disease (e.g. colorectal cancer) from GutUDB, which curates 54,131 CNVs, and the matching gene RNA-seq profiles from its 137 RNA-seq transcriptomics data sets via the database's Browse/Query functionality or the github.com/Ansanqi/GutUDB repository. Compute deletion frequency as the fraction of disease samples with a deletion call at the gene locus, comparing against reference values such as METTL3's 0.04 in CRC. Compute a Pearson or Spearman correlation between per-sample CNV state and RNA expression level, and classify it as weak if it falls in the weak range (roughly |r| < 0.3), as observed for METTL3 despite significantly high RNA expression. Assemble the deletion frequency and correlation coefficient into a small tabular result file (mettl3_cnv_results.csv). Rationale: GutUDB integrates genomics (SNPs, CNVs, SVs, mutated genes) with transcriptomics in one platform, so CNV and RNA layers for the same gene and disease are directly joinable.

## Related tools

- **GutUDB** (Source of CNV genomics profiles (54,131 CNVs) and RNA-seq transcriptomics profiles used to compute deletion frequency and CNV-expression correlation; accessed via Browse, Query, Visualization, Download or the GitHub repo) — https://github.com/Ansanqi/GutUDB

## Examples

```
import pandas as pd; cnv=pd.read_csv('mettl3_cnv_crc.csv'); freq=(cnv['cnv_state']=='deletion').mean(); corr=cnv['cnv_state'].astype('category').cat.codes.corr(cnv['rna_expr'],method='spearman'); pd.DataFrame({'deletion_frequency':[freq],'spearman_r':[corr]}).to_csv('mettl3_cnv_results.csv',index=False)
```

## Evaluation signals

- Deletion frequency for METTL3 in CRC reproduces the reported value of 0.04
- CNV-vs-RNA-expression correlation for METTL3 falls in the weak range (|r| < 0.3) despite significantly high RNA expression
- Output CSV (mettl3_cnv_results.csv) contains exactly the deletion frequency and correlation coefficient rows
- Both CNV and RNA-seq inputs derive from GutUDB profiles for the same disease (colorectal cancer) and the same gene

## Limitations

- The deletion-frequency value is dataset-specific to GutUDB's CNV profiles for colorectal cancer; other diseases or cohorts may differ
- GutUDB's abstract-level documentation does not specify a correlation method or a fixed weak-correlation cutoff, so the Pearson-vs-Spearman choice and 'weak' threshold must be stated explicitly
- A weak CNV-expression correlation means deletion frequency alone cannot explain the observed high RNA expression of the gene

## Evidence

- [abstract] GutUDB currently includes 72,248 single‐nucleotide polymorphisms (SNPs), 54,131 CNVs, 1097 structural variations of chromosome (SVs), and 92,888 mutated genes: "GutUDB currently includes 72,248 single‐nucleotide polymorphisms (SNPs), 54,131 CNVs, 1097 structural variations of chromosome (SVs), and 92,888 mutated genes"
- [abstract] 137 RNA-seq data sets related to 7.9 million transcriptional profiles: [evidence span withheld — anchor status: not_found]
- [abstract] four core functionalities of GutUDB: "four core functionalities—Browse, Query, Visualization, and Download—were integrated into GutUDB"
