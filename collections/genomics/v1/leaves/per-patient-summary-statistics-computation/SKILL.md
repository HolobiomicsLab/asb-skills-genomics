---
name: per-patient-summary-statistics-computation
description: Use when you have per-read telomap TSV output for multiple barcoded patient
  PBL samples and need read-to-patient aggregation (mean telomere length, mean raw
  telomere length) followed by a Pearson correlation of those summaries against patient
  age.
when_to_use_negative:
- Input is read-level data that has not yet been processed by telomap (no TELOMERE_LEN/RAW_TELOMERE_LEN
  columns exist yet)
- The association of interest is non-linear or the goal is per-chromosome-end analysis
  rather than a single per-patient mean summary
- Samples are ALT cells or very long telomeres that exceed PacBio HiFi read length,
  where telomap lengths are not reliable
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: telomap
  role: Produces the per-read sample.telomap.tsv tables whose TELOMERE_LEN and RAW_TELOMERE_LEN
    columns are aggregated per patient
  repo: https://github.com/cytham/telomap
- name: pandas
  role: Groupby aggregation of read-level telomere length values into per-patient
    means
- name: SciPy
  role: scipy.stats.pearsonr computes the Pearson correlation between per-patient
    mean telomere lengths and age
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-35823-7
    title: High-throughput telomere length measurement at nucleotide resolution using
      the PacBio high fidelity sequencing platform
  source_content_hash: sha256:29761419eb74058a60b8ca5c7150ff786da4891c75e6c0d74767aec0b93ab54f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cytham/telomap
  tools:
  - SciPy
  - pandas
  - telomap
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# per-patient-summary-statistics-computation

## Summary

Aggregates read-level telomere length measurements from per-sample telomap output tables into per-patient summary statistics (mean TELOMERE_LEN and mean RAW_TELOMERE_LEN), then tests their association with clinical covariates such as age via Pearson correlation.

## When to use

Use when you have per-read telomap TSV output for multiple barcoded patient PBL samples and need read-to-patient aggregation (mean telomere length, mean raw telomere length) followed by a Pearson correlation of those summaries against patient age.

## When NOT to use

- Input is read-level data that has not yet been processed by telomap (no TELOMERE_LEN/RAW_TELOMERE_LEN columns exist yet)
- The association of interest is non-linear or the goal is per-chromosome-end analysis rather than a single per-patient mean summary
- Samples are ALT cells or very long telomeres that exceed PacBio HiFi read length, where telomap lengths are not reliable

## Inputs

- Per-patient demultiplexed telomap output tables (sample.telomap.tsv) with columns RNAME, OLIGO, BARCODE, STRAND, MOTIF, RLEN, RAW_TELOMERE_LEN, TELOMERE_LEN
- Patient metadata table mapping sample/barcode to patient age

## Outputs

- Per-patient summary table of mean telomere length and mean raw telomere length
- Correlation results table (telomere_length_age_correlation.csv) with Pearson r and p-values for telomere length vs age and raw telomere length vs age

## How to apply

Load each demultiplexed sample.telomap.tsv produced by telomap and extract the TELOMERE_LEN and RAW_TELOMERE_LEN columns for every telomeric read; TELOMERE_LEN is the telomeric region after excluding non-telomeric sequences, while RAW_TELOMERE_LEN spans TELOMERE_START to TELOMERE_END. Use pandas groupby on the patient/sample ID (derived from the demultiplexed BARCODE/sample file) to compute per-patient means of both metrics. Run scipy.stats.pearsonr twice: per-patient mean telomere length vs age, and mean raw telomere length vs age. Write the resulting r and p values for both metrics to a CSV (e.g. telomere_length_age_correlation.csv) and check that they reproduce the reported correlations within a small numerical tolerance.

## Related tools

- **telomap** (Produces the per-read sample.telomap.tsv tables whose TELOMERE_LEN and RAW_TELOMERE_LEN columns are aggregated per patient) — https://github.com/cytham/telomap
- **pandas** (Groupby aggregation of read-level telomere length values into per-patient means)
- **SciPy** (scipy.stats.pearsonr computes the Pearson correlation between per-patient mean telomere lengths and age)

## Examples

```
import pandas as pd; from scipy.stats import pearsonr; df = pd.concat([pd.read_csv(f, sep='\t') for f in samples]); m = df.groupby('patient')[['TELOMERE_LEN','RAW_TELOMERE_LEN']].mean(); r, p = pearsonr(m['TELOMERE_LEN'], ages)
```

## Evaluation signals

- Computed Pearson r and p for telomere length vs age match the reported values (r = -0.252, p = 0.023) within a small numerical tolerance
- Computed Pearson r and p for raw telomere length vs age match the reported values (r = -0.228, p = 0.041) within tolerance
- Each patient contributes exactly one aggregated row per metric (groupby on patient/sample ID yields the expected number of patients)
- telomere_length_age_correlation.csv contains r and p values for both metrics with the expected schema
- Mean telomere length values fall within biologically plausible ranges consistent with the RAW_TELOMERE_LEN/TELOMERE_LEN column definitions in the telomap output

## Limitations

- RAW_TELOMERE_LEN includes interspersed non-telomeric sequences; TELOMERE_LEN excludes them, so the two metrics give different (though similarly negative) correlations with age
- Mean-based summaries discard the full telomere length distribution, including over-representation of extremely short telomeres that the method detected in cell lines
- Patient samples must be demultiplexed telobait-capture data; barcode matching uses a minimum alignment score fraction (--barscore, default 1), and reducing it may cause multiple barcode mappings per read and high read omission

## Evidence

- [readme] TELOMERE_LEN definition excluding non-telomeric sequence: "TELOMERE_LEN | Length of telomeric region in bases after excluding all non-telomeric sequences"
- [readme] telomap main output table: "sample.telomap.tsv | Main output table containing per read information"
- [other] Pearson correlation step of the source workflow: [evidence span withheld — anchor status: not_found]
- [other] Reported correlation finding: [evidence span withheld — anchor status: not_found]
- [readme] Barcode matching caveat: "Warning: Reducing this value may lead to multiple barcode mapping per read, causing high read omission."
