---
name: telomere-read-position-aggregation
description: Use when you have chromosome-end-assigned per-read telomap output (sample.telomap.tsv)
  and need per-nucleotide TVS percentage profiles across the telomeric region, or
  per-patient mean telomere lengths for correlation analyses.
when_to_use_negative:
- Input reads have not been processed by telomap — TELOMERE_START/TELOMERE_END and
  TVS annotations do not exist yet; run telomap first
- Analyzing ALT cells, whose very long telomeres exceed PacBio HiFi DNA polymerase
  processivity and cannot be measured with this method
- Comparing samples at the per-position level when the RsaI/HinfI trimming left subtelomeric
  regions too short for unambiguous chromosome-end identification against T2T-CHM13
edam_operation: http://edamontology.org/operation_2409
tools:
- name: telomap
  role: Generates the per-read telomap.tsv table whose TELOMERE_LEN, RAW_TELOMERE_LEN,
    TELOMERE_START/END and TVS annotations are aggregated per position and per sample
    by this skill
  repo: https://github.com/cytham/telomap
- name: pandas
  role: Reads sample.telomap.tsv, filters reads by chromosome end, and performs groupby
    aggregation of read-level lengths into per-patient means
- name: SciPy
  role: Computes Pearson correlation (scipy.stats.pearsonr) between aggregated per-patient
    mean telomere length and age
provenance:
  source_task_ids:
  - task_007
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-35823-7
    title: High-throughput telomere length measurement at nucleotide resolution using
      the PacBio high fidelity sequencing platform
  source_content_hash: sha256:629b40fb8282b64ceff1ef438b087132eb5673eacf66cede17b021c152219d18
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cytham/telomap
  tools:
  - SciPy
  - pandas
  - telomap
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# telomere-read-position-aggregation

## Summary

Aggregates per-read telomeric information from telomap's sample.telomap.tsv — read-level telomere lengths (TELOMERE_LEN, RAW_TELOMERE_LEN) and per-nucleotide telomeric variant sequence (TVS) positions relative to the canonical 5'-TTAGGG-3' motif — into position-resolved or sample-level summaries for downstream statistics and plotting.

## When to use

Use when you have chromosome-end-assigned per-read telomap output (sample.telomap.tsv) and need per-nucleotide TVS percentage profiles across the telomeric region, or per-patient mean telomere lengths for correlation analyses.

## When NOT to use

- Input reads have not been processed by telomap — TELOMERE_START/TELOMERE_END and TVS annotations do not exist yet; run telomap first
- Analyzing ALT cells, whose very long telomeres exceed PacBio HiFi DNA polymerase processivity and cannot be measured with this method
- Comparing samples at the per-position level when the RsaI/HinfI trimming left subtelomeric regions too short for unambiguous chromosome-end identification against T2T-CHM13

## Inputs

- sample.telomap.tsv — per-read telomap output table with RNAME, OLIGO, BARCODE, STRAND, MOTIF, RLEN, RAW_TELOMERE_LEN, TELOMERE_LEN, TELOMERE_START, TELOMERE_END columns
- Chromosome-end assignment (e.g., CHROM = Chr 7q) from telomap subtelomeric anchor mapping against T2T-CHM13
- Sample metadata (patient/sample ID, age) for per-patient aggregation and correlation

## Outputs

- Per-nucleotide TVS percentage profile over the first 3 kb of the telomeric region, per haplotype group (e.g., chr7q_tvs_signature.png)
- Pairwise Euclidean-distance TVS correlation scores between replicate samples (tvs_correlation_scores.csv)
- Per-patient mean telomere length and mean raw telomere length, and Pearson r/p values vs age (telomere_length_age_correlation.csv)

## How to apply

Load sample.telomap.tsv with pandas and filter reads to the target chromosome end (e.g., CHROM = 'Chr 7q') or sample/barcode group. For position aggregation, extract the telomeric region bounded by TELOMERE_START and TELOMERE_END, tabulate non-canonical motif occurrences detected against the canonical motif in the MOTIF column, and compute the per-nucleotide TVS percentage over the first 3 kb of the telomeric region by aggregating variant positions across reads; optionally balance haplotypes at the chromosome end with k-means clustering (k=2) on per-read TVS profiles. For length aggregation, groupby patient/sample ID on TELOMERE_LEN and RAW_TELOMERE_LEN and take means. Rationale: TVSs are dispersed throughout the telomere repeat region, so read-level events must be aggregated per nucleotide position before computing signatures or correlations (e.g., Pearson r vs age, or pairwise Euclidean-distance TVS correlation scores with replicate pairs required to exceed 0.90).

## Related tools

- **telomap** (Generates the per-read telomap.tsv table whose TELOMERE_LEN, RAW_TELOMERE_LEN, TELOMERE_START/END and TVS annotations are aggregated per position and per sample by this skill) — https://github.com/cytham/telomap
- **pandas** (Reads sample.telomap.tsv, filters reads by chromosome end, and performs groupby aggregation of read-level lengths into per-patient means)
- **SciPy** (Computes Pearson correlation (scipy.stats.pearsonr) between aggregated per-patient mean telomere length and age)

## Examples

```
import pandas as pd; df = pd.read_csv('sample.telomap.tsv', sep='\t'); sub = df[df.CHROM=='Chr 7q']; sub[['TELOMERE_LEN','RAW_TELOMERE_LEN']].mean()
```

## Evaluation signals

- Replicate sample pairs achieve a pairwise TVS correlation score > 0.90 from the Euclidean-distance comparison of TVS profiles
- The TVS signature plot (e.g., chr7q_tvs_signature.png) exists and depicts a dispersed distribution of non-canonical motifs across the telomeric region, consistent with TVS heterogeneity reported in the paper
- Computed Pearson statistics match reported values: telomere length vs age r = -0.252, p = 0.023; raw telomere length vs age r = -0.228, p = 0.041
- TELOMERE_LEN values in the aggregation are strictly ≤ RAW_TELOMERE_LEN, since TELOMERE_LEN excludes all non-telomeric sequences within the telomeric region
- Each aggregated read is bounded by TELOMERE_START defined as containing at least two consecutive telomeric repeats (5'-TTAGGGTTAGGG-3')

## Limitations

- HphI and MnlI restriction endonucleases, which cut closer to the telomere-containing region, frequently cut within TVSs, causing loss of telomere repeat regions and underestimation of telomere length
- Trimming with RsaI and HinfI may leave subtelomeric regions too short for unambiguous chromosome-end identification against T2T-CHM13, limiting position-resolved analysis at specific chromosome ends
- The method is incompatible with measuring telomere length of ALT cells whose very long telomeres exceed PacBio HiFi polymerase processivity

## Evidence

- [methods] Per-read TVS position aggregation: "Analyze the distribution of non-canonical telomeric motifs (Telomere variant sequences (TVS))"
- [readme] TELOMERE_LEN semantics: "Length of telomeric region in bases after excluding all non-telomeric sequences within RAW_TELOMERE_LEN"
- [abstract] TVS heterogeneity dispersed across the repeat region: "Our results also reveal the extreme heterogeneity of telomeric variant sequences (TVSs) that are dispersed throughout the telomere repeat region"
- [methods] TELOMERE_START definition: "the 1-based position on a read at the 5′ end of the telomeric region as defined by containing at least two consecutive telomeric repeats (5′-TTAGGGTTAGGG-3′)"
- [methods] Chromosome-end mapping reference: "Map subtelomeric anchor sequences to chromosome ends based on T2T-CHM13 assembly"
