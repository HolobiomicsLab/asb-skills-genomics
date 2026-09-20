---
name: telomere-length-normalization
description: Use when comparing telomeric repeat content or TRF binding-site counts
  across samples or chromosome ends whose RAW_TELOMERE_LEN differ, and TVSs interspersed
  within the (5'-TTAGGG-3')n tract mean raw length alone is mis.
when_to_use_negative:
- Input data are vanilla WGS long reads without telobait capture and the question
  requires chromosome-end-specific assignment where subtelomeric regions may be too
  short for unambiguous identification against T2T-CHM13
- Measuring telomere length of ALT cells, whose very long telomeres exceed PacBio
  HiFi DNA polymerase processivity
- Raw telomere lengths are directly comparable and no TVSs interrupt the canonical
  (5'-TTAGGG-3')n tract, making density normalization unnecessary
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: telomap
  role: Produces the per-read sample.telomap.tsv table with TELOMERE_START/TELOMERE_END,
    RAW_TELOMERE_LEN, and TELOMERE_LEN used as the basis for motif counting and density
    normalization
  repo: https://github.com/cytham/telomap
- name: pandas
  role: Loads and filters sample.telomap.tsv rows by CHROM/BARCODE and computes normalized
    TRF motif density per kb
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-35823-7
    title: High-throughput telomere length measurement at nucleotide resolution using
      the PacBio high fidelity sequencing platform
  source_content_hash: sha256:a4f3d50d86971f081fe963d6646afc598988a60defa3ec64a57828ca1e66a96e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cytham/telomap
  tools:
  - pandas
  - telomap
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 3
---

# telomere-length-normalization

## Summary

Normalize TRF1/TRF2 binding-motif (5'-TTAGGGTTA-3') counts by the canonical telomere length from telomap per-read output (TELOMERE_LEN, which excludes non-telomeric/variant sequences), enabling comparison of shelterin binding-site density between chromosome ends or samples whose raw telomere lengths (RAW_TELOMERE_LEN) differ due to telomeric variant sequences (TVSs).

## When to use

Use when comparing telomeric repeat content or TRF binding-site counts across samples or chromosome ends whose RAW_TELOMERE_LEN differ, and TVSs interspersed within the (5'-TTAGGG-3')n tract mean raw length alone is misleading.

## When NOT to use

- Input data are vanilla WGS long reads without telobait capture and the question requires chromosome-end-specific assignment where subtelomeric regions may be too short for unambiguous identification against T2T-CHM13
- Measuring telomere length of ALT cells, whose very long telomeres exceed PacBio HiFi DNA polymerase processivity
- Raw telomere lengths are directly comparable and no TVSs interrupt the canonical (5'-TTAGGG-3')n tract, making density normalization unnecessary

## Inputs

- telomap per-read output table (sample.telomap.tsv) with columns RNAME, OLIGO, BARCODE, STRAND, MOTIF, RLEN, RAW_TELOMERE_LEN, TELOMERE_LEN, TELOMERE_START, TELOMERE_END, CHROM
- TRF binding motif sequence 5'-TTAGGGTTA-3'
- canonical telomere repeat definition 5'-TTAGGGTTAGGG-3' (two consecutive repeats)

## Outputs

- Per-sample chromosome-end comparison record (e.g., chr7q_trf_comparison.csv) with raw telomere length (bp), TRF motif count, and motif density normalized per kb of canonical telomere

## How to apply

Load the per-read telomap output table (sample.telomap.tsv) with pandas and filter to the chromosome end of interest (e.g., Chr 7q via the CHROM column) and samples of interest (BARCODE column, e.g., P14/P15). Extract each read's telomeric region using TELOMERE_START and TELOMERE_END coordinates, taking the canonical repeat tract defined from at least two consecutive 5'-TTAGGGTTAGGG-3' repeats. Count occurrences of the 5'-TTAGGGTTA-3' TRF binding motif within a fixed 1 kb window of canonical repeats using regex or str.find-based scanning. Normalize motif counts by TELOMERE_LEN (length after excluding all non-telomeric/variant sequences within RAW_TELOMERE_LEN) to obtain binding-site density per kb of canonical telomere. Rationale: TVSs disrupt the continuity of canonical (5'-TTAGGG-3')n repeats, affecting shelterin binding, so comparing raw lengths alone (e.g., P14 1126 bp vs P15 1627 bp) conflates TVS content with canonical repeat availability.

## Related tools

- **telomap** (Produces the per-read sample.telomap.tsv table with TELOMERE_START/TELOMERE_END, RAW_TELOMERE_LEN, and TELOMERE_LEN used as the basis for motif counting and density normalization) — https://github.com/cytham/telomap
- **pandas** (Loads and filters sample.telomap.tsv rows by CHROM/BARCODE and computes normalized TRF motif density per kb)

## Examples

```
import pandas as pd; df = pd.read_csv('sample.telomap.tsv', sep='\t'); df = df[(df['CHROM']=='chr7q') & (df['BARCODE'].isin(['P14','P15']))]; df['density'] = df.apply(lambda r: r['seq'][:1000].count('TTAGGGTTA')/(r['TELOMERE_LEN']/1000), axis=1)
```

## Evaluation signals

- Output CSV exists and contains rows for both samples (e.g., P14 and P15) with fields for raw telomere length, TRF count, and normalized density
- Computed raw telomere lengths match reported values (e.g., ~1126 bp for P14 and ~1627 bp for P15 on Chr 7q)
- Computed TRF motif counts match reported values (e.g., 81 sites for P14 and 62 for P15)
- TELOMERE_START position aligns to the 5' end defined by at least two consecutive 5'-TTAGGGTTAGGG-3' repeats
- Normalized density is consistent: TRF count / (TELOMERE_LEN/1000) per kb of canonical telomere

## Limitations

- TVSs can include sites cut by restriction enzymes such as HphI and MnlI, causing loss of telomere repeat regions and underestimation of telomere length; careful restriction enzyme selection (e.g., RsaI, HinfI) is needed
- Trimming with RsaI and HinfI may leave subtelomeric regions too short for unambiguous chromosome-end identification against T2T-CHM13
- The method is incompatible with ALT cells whose very long telomeres exceed PacBio HiFi polymerase processivity; maximum read length is limited by DNA polymerase processivity

## Evidence

- [methods] the 1-based position on a read at the 5′ end of the telomeric region as defined by containing at least two consecutive telomeric repeats (5′-TTAGGGTTAGGG-3′): "the 1-based position on a read at the 5′ end of the telomeric region as defined by containing at least two consecutive telomeric repeats (5′-TTAGGGTTAGGG-3′)"
- [readme] Length of telomeric region in bases after excluding all non-telomeric sequences within RAW_TELOMERE_LEN: "Length of telomeric region in bases after excluding all non-telomeric sequences"
- [abstract] The presence of TVSs disrupts the continuity of the canonical (5'-TTAGGG-3')n telomere repeats, which affects the binding of shelterin complexes: "The presence of TVSs disrupts the continuity of the canonical (5'-TTAGGG-3')n telomere repeats, which affects the binding of shelterin complexes at the chromosomal ends and telomere protection"
- [discussion] we found that HphI and MnlI restriction endonucleases, which are known to cut closer to the telomere-containing region, frequently cut in the TVSs: "we found that HphI and MnlI restriction endonucleases, which are known to cut closer to the telomere-containing region, frequently cut in the TVSs within the telomere repeat regions"
