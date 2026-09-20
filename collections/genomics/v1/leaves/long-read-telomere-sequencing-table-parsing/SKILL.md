---
name: long-read-telomere-sequencing-table-parsing
description: Use when you have demultiplexed telomap TSV output from WGS or telobait-captured
  long-read data and need per-patient mean telomere length metrics for group-level
  statistics such as age correlation.
when_to_use_negative:
- Input is raw FASTQ/BAM long-read data that has not yet been processed by telomap
  — run telomap first to generate sample.telomap.tsv
- Reads lack a called telomeric region (no TELOMERE_START from at least two consecutive
  TTAGGG repeats), so TELOMERE_LEN/RAW_TELOMERE_LEN are undefined
- Analyzing ALT cell telomere length, since very long ALT telomeres exceed PacBio
  HiFi read length and are not reliably measured with this method
edam_operation: http://edamontology.org/operation_3741
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: telomap
  role: Upstream tool that identifies telomeric reads in WGS or telobait-captured
    long-read data and emits the sample.telomap.tsv table parsed by this skill
  repo: github.com/cytham/telomap
- name: pandas
  role: Used to read sample.telomap.tsv files and groupby on patient/sample ID to
    compute per-patient mean TELOMERE_LEN and RAW_TELOMERE_LEN
- name: SciPy
  role: scipy.stats.pearsonr computes the Pearson correlation between per-patient
    mean telomere length metrics and age
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-35823-7
    title: High-throughput telomere length measurement at nucleotide resolution using
      the PacBio high fidelity sequencing platform
  source_content_hash: sha256:76753f04abd3b6f6d5e6e0e3f205075519c170ac1b18de826894d6480882af07
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SciPy
  - pandas
  - telomap
  edam_operation: http://edamontology.org/operation_3741
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# Long-read telomere sequencing table parsing

## Summary

Parses per-sample telomap output tables (sample.telomap.tsv) from PacBio HiFi long-read telomere sequencing, extracting read-level TELOMERE_LEN and RAW_TELOMERE_LEN columns and aggregating them into per-patient summary statistics for downstream correlation analyses.

## When to use

Use when you have demultiplexed telomap TSV output from WGS or telobait-captured long-read data and need per-patient mean telomere length metrics for group-level statistics such as age correlation.

## When NOT to use

- Input is raw FASTQ/BAM long-read data that has not yet been processed by telomap — run telomap first to generate sample.telomap.tsv
- Reads lack a called telomeric region (no TELOMERE_START from at least two consecutive TTAGGG repeats), so TELOMERE_LEN/RAW_TELOMERE_LEN are undefined
- Analyzing ALT cell telomere length, since very long ALT telomeres exceed PacBio HiFi read length and are not reliably measured with this method

## Inputs

- Per-patient/sample demultiplexed telomap output tables (sample.telomap.tsv) with columns RNAME, OLIGO, BARCODE, STRAND, MOTIF, RLEN, RAW_TELOMERE_LEN, TELOMERE_LEN
- Patient metadata table mapping sample/barcode IDs to clinical variables (e.g. age)

## Outputs

- Per-patient mean telomere length table (pandas groupby aggregation)
- Per-patient mean raw telomere length values
- Correlation results table (e.g. telomere_length_age_correlation.csv) with Pearson r and p values for both metrics vs age

## How to apply

Load each demultiplexed sample.telomap.tsv file produced upstream by telomap and extract the TELOMERE_LEN (telomeric region length after excluding non-telomeric/TVS sequences) and RAW_TELOMERE_LEN (telomeric region length between TELOMERE_START and TELOMERE_END) columns for every telomeric read. Use pandas groupby on patient/sample ID to collapse read-level values into per-patient mean telomere length and mean raw telomere length. TELOMERE_START is defined by the presence of at least two consecutive telomeric repeats (5'-TTAGGGTTAGGG-3'), so reads must contain a called telomeric region to contribute. Feed the aggregated per-patient means into downstream statistics, e.g. scipy.stats.pearsonr against patient age. Validate by reproducing reported summary statistics such as the Pearson r and p values within a small numerical tolerance.

## Related tools

- **telomap** (Upstream tool that identifies telomeric reads in WGS or telobait-captured long-read data and emits the sample.telomap.tsv table parsed by this skill) — github.com/cytham/telomap
- **pandas** (Used to read sample.telomap.tsv files and groupby on patient/sample ID to compute per-patient mean TELOMERE_LEN and RAW_TELOMERE_LEN)
- **SciPy** (scipy.stats.pearsonr computes the Pearson correlation between per-patient mean telomere length metrics and age)

## Examples

```
import pandas as pd; df = pd.read_csv('sample.telomap.tsv', sep='\t'); means = df.groupby('BARCODE')[['TELOMERE_LEN','RAW_TELOMERE_LEN']].mean()
```

## Evaluation signals

- Every parsed sample.telomap.tsv row must yield numeric TELOMERE_LEN and RAW_TELOMERE_LEN values; non-telomeric reads must be excluded
- Per-patient groupby must produce one mean per patient for both TELOMERE_LEN and RAW_TELOMERE_LEN; row counts match the number of telomeric reads per sample
- Computed Pearson r and p for telomere length vs age (r = -0.252, p = 0.023) and raw telomere length vs age (r = -0.228, p = 0.041) reproduce within small numerical tolerance
- Output CSV (telomere_length_age_correlation.csv) contains r and p values for both metrics
- TELOMERE_LEN must be ≤ RAW_TELOMERE_LEN per read, since TELOMERE_LEN excludes non-telomeric sequences within the raw telomeric region

## Limitations

- TELOMERE_LEN excludes non-telomeric sequences (e.g. TVSs) within the telomeric region, so it can diverge substantially from RAW_TELOMERE_LEN in TVS-rich telomeres
- Restriction enzyme trimming choices (RsaI/HinfI vs HphI/MnlI) affect whether telomere-containing fragments are fully captured; HphI/MnlI cut within TVSs and underestimate telomere length
- The method cannot measure ALT cell telomeres, whose length exceeds PacBio HiFi DNA polymerase processivity

## Evidence

- [methods] TELOMERE_LEN calculation excluding non-telomeric sequences within RAW_TELOMERE_LEN: "Length of telomeric region in bases after excluding all non-telomeric sequences within RAW_TELOMERE_LEN"
- [methods] TELOMERE_START defined by consecutive telomeric repeats: "the 1-based position on a read at the 5′ end of the telomeric region as defined by containing at least two consecutive telomeric repeats (5′-TTAGGGTTAGGG-3′)"
- [readme] telomap main output table columns: "sample.telomap.tsv | Main output table containing per read information"
- [readme] telomap identifies telomeric reads: "Telomap is a tool to identify and analyse telomeric reads within WGS or telobait-captured long-read sequencing data."
