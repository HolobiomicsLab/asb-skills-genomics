---
name: pearson-correlation-clinical-age-analysis
description: Use when you have per-read telomere length tables (sample.telomap.tsv)
  from telomap runs on patient PBL samples plus per-patient ages, and need to test
  whether mean telomere length correlates with age.
when_to_use_negative:
- Input is raw long-read FASTQ/BAM that has not yet been processed by telomap — no
  per-read TELOMERE_LEN/RAW_TELOMERE_LEN columns exist yet
- Data are from ALT cell lines or samples with telomeres exceeding PacBio HiFi read
  length, where per-read lengths are unreliable
- Sample sizes are too small or distributions are non-normal/non-linear, where Pearson
  r is inappropriate
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: telomap
  role: Upstream tool producing the per-read sample.telomap.tsv tables whose TELOMERE_LEN
    and RAW_TELOMERE_LEN columns are aggregated per patient
  repo: github.com/cytham/telomap
- name: pandas
  role: Groupby aggregation of read-level telomere lengths into per-patient means
    keyed on patient/sample ID
- name: SciPy
  role: scipy.stats.pearsonr computes the Pearson r and p value between per-patient
    mean telomere lengths and age
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-35823-7
    title: High-throughput telomere length measurement at nucleotide resolution using
      the PacBio high fidelity sequencing platform
  source_content_hash: sha256:f126a781143cb572dd72cf5760861d4fb1509cf59b62c8703b94d00bdf7c1c83
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SciPy
  - pandas
  - telomap
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# pearson-correlation-clinical-age-analysis

## Summary

Aggregates read-level telomere length measurements (TELOMERE_LEN and RAW_TELOMERE_LEN columns from telomap's sample.telomap.tsv) into per-patient means and tests their association with patient age using Pearson correlation, to detect age-related telomere attrition in clinical PBL samples.

## When to use

Use when you have per-read telomere length tables (sample.telomap.tsv) from telomap runs on patient PBL samples plus per-patient ages, and need to test whether mean telomere length correlates with age.

## When NOT to use

- Input is raw long-read FASTQ/BAM that has not yet been processed by telomap — no per-read TELOMERE_LEN/RAW_TELOMERE_LEN columns exist yet
- Data are from ALT cell lines or samples with telomeres exceeding PacBio HiFi read length, where per-read lengths are unreliable
- Sample sizes are too small or distributions are non-normal/non-linear, where Pearson r is inappropriate

## Inputs

- Per-patient telomap output tables (sample.telomap.tsv) with TELOMERE_LEN and RAW_TELOMERE_LEN columns per telomeric read
- Per-patient age metadata for the clinical PBL cohort

## Outputs

- Per-patient mean telomere length and mean raw telomere length table (pandas groupby output)
- telomere_length_age_correlation.csv containing Pearson r and p values for telomere length vs age and raw telomere length vs age

## How to apply

Load each patient's demultiplexed telomap output table (sample.telomap.tsv) and extract the TELOMERE_LEN and RAW_TELOMERE_LEN columns for every telomeric read; TELOMERE_LEN excludes non-telomeric sequences within the telomeric region, whereas RAW_TELOMERE_LEN is the full region demarcated by TELOMERE_START and TELOMERE_END. Aggregate read-level values into per-patient mean telomere length and mean raw telomere length with pandas groupby on patient/sample ID. Compute the Pearson correlation of each per-patient mean against age using scipy.stats.pearsonr, then write the resulting r and p values for both metrics to a summary table (e.g. telomere_length_age_correlation.csv). A negative, statistically significant r (in the source data, r = -0.252, p = 0.023 for telomere length and r = -0.228, p = 0.041 for raw telomere length) confirms age-related telomere shortening measured at nucleotide resolution by PacBio HiFi sequencing.

## Related tools

- **telomap** (Upstream tool producing the per-read sample.telomap.tsv tables whose TELOMERE_LEN and RAW_TELOMERE_LEN columns are aggregated per patient) — github.com/cytham/telomap
- **pandas** (Groupby aggregation of read-level telomere lengths into per-patient means keyed on patient/sample ID)
- **SciPy** (scipy.stats.pearsonr computes the Pearson r and p value between per-patient mean telomere lengths and age)

## Examples

```
from scipy.stats import pearsonr; r, p = pearsonr(patient_means['mean_telomere_len'], patient_means['age'])
```

## Evaluation signals

- Computed Pearson r and p for telomere length vs age reproduce the reported values (r = -0.252, p = 0.023) within numerical tolerance
- Computed Pearson r and p for raw telomere length vs age reproduce the reported values (r = -0.228, p = 0.041) within numerical tolerance
- Both correlations are negative and statistically significant (p < 0.05), consistent with age-related telomere attrition
- telomere_length_age_correlation.csv contains exactly four values: r and p for each of the two length metrics
- Per-patient groupby produces one mean per patient/sample ID with no missing or duplicated patient entries

## Limitations

- Telomeric variant sequences (TVSs) interspersed in telomere repeats are excluded from TELOMERE_LEN but included in RAW_TELOMERE_LEN, so the two metrics can diverge
- Restriction enzymes cutting within TVSs (HphI, MnlI) can cause loss of telomere repeat regions and underestimation of telomere length
- The method is incompatible with measuring telomere length of ALT cells, whose very long telomeres exceed PacBio HiFi DNA polymerase processivity

## Evidence

- [methods] workflow steps 1-4: [evidence span withheld — anchor status: not_found]
- [readme] column semantics: "Length of telomeric region in bases after excluding all non-telomeric sequences"
- [abstract] clinical cohort context: "clinical patient samples and measure their length accurately at nucleotide resolution"
- [readme] source of read-level metrics: "Telomap is a tool to identify and analyse telomeric reads within WGS or telobait-captured long-read sequencing data."
