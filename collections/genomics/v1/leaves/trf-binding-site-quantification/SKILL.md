---
name: trf-binding-site-quantification
description: Use when comparing shelterin (TRF1/TRF2) binding-site density between
  samples or chromosome ends whose canonical (TTAGGG)n telomere lengths are identical
  but whose raw telomere lengths differ due to TVSs.
when_to_use_negative:
- Telomere length of ALT cells is being measured — the article states PacBio HiFi
  reads are incompatible with measuring ALT cell telomeres because their very long
  telomeres exceed polymerase processivity
- The telomap table lacks chromosome-end assignment (CHROM) because subtelomeric regions
  were too short for unambiguous identification against T2T-CHM13
- The question is absolute telomere length distribution rather than motif-density
  comparison — use telomap's own telomere length outputs (RAW_TELOMERE_LEN / TELOMERE_LEN)
  directly instead
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0654
tools:
- name: telomap
  role: Produces the per-read sample.telomap.tsv table with TELOMERE_START, TELOMERE_END,
    RAW_TELOMERE_LEN, and TELOMERE_LEN fields consumed by this skill; also analyzes
    TVS distribution
  repo: https://github.com/cytham/telomap
- name: pandas
  role: Loads and filters sample.telomap.tsv rows by CHROM and BARCODE, and aggregates
    per-sample motif counts and densities
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-35823-7
    title: High-throughput telomere length measurement at nucleotide resolution using
      the PacBio high fidelity sequencing platform
  source_content_hash: sha256:23a54b0c74f127630e9e2f5c65740e1f97667b421615c3fd95daa35879769f22
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cytham/telomap
  tools:
  - pandas
  - telomap
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# trf-binding-site-quantification

## Summary

Quantifies TRF1/TRF2 binding-motif (5'-TTAGGGTTA-3') occurrences per kb of canonical telomere repeats on a specific chromosome end (e.g. Chr 7q) using per-read telomap output, to test whether telomeric variant sequences (TVSs) disrupt shelterin binding despite identical canonical repeat lengths.

## When to use

Use when comparing shelterin (TRF1/TRF2) binding-site density between samples or chromosome ends whose canonical (TTAGGG)n telomere lengths are identical but whose raw telomere lengths differ due to TVSs.

## When NOT to use

- Telomere length of ALT cells is being measured — the article states PacBio HiFi reads are incompatible with measuring ALT cell telomeres because their very long telomeres exceed polymerase processivity
- The telomap table lacks chromosome-end assignment (CHROM) because subtelomeric regions were too short for unambiguous identification against T2T-CHM13
- The question is absolute telomere length distribution rather than motif-density comparison — use telomap's own telomere length outputs (RAW_TELOMERE_LEN / TELOMERE_LEN) directly instead

## Inputs

- telomap per-read output table (sample.telomap.tsv) with columns RNAME, OLIGO, BARCODE, STRAND, MOTIF, RLEN, RAW_TELOMERE_LEN, TELOMERE_LEN, TELOMERE_START, TELOMERE_END, CHROM
- Sample identifiers (e.g. BARCODE values P14, P15) and target chromosome end (e.g. Chr 7q)

## Outputs

- Structured comparison CSV (e.g. chr7q_trf_comparison.csv) with per-sample raw telomere length (bp), TRF motif count, and TRF binding-site density per kb of canonical telomere

## How to apply

Load the per-read telomap output table (sample.telomap.tsv) with pandas and filter to the target chromosome end (CHROM column) and samples of interest (BARCODE column). For each read, extract the telomeric region using TELOMERE_START and TELOMERE_END coordinates, where TELOMERE_START is the 1-based position at the 5' end of the region containing at least two consecutive 5'-TTAGGGTTAGGG-3' repeats. Take the first 1 kb of the canonical repeat tract and count occurrences of the TRF binding motif 5'-TTAGGGTTA-3' via regex or str.find-based scanning. Normalize motif counts by TELOMERE_LEN (canonical length after excluding non-telomeric/variant sequences within RAW_TELOMERE_LEN) to obtain binding-site density per kb. Aggregate per sample and write a structured comparison CSV (e.g. chr7q_trf_comparison.csv) with raw telomere length, TRF count, and normalized density per sample. This is grounded in the paper's finding that TVSs disrupt the continuity of canonical (TTAGGG)n repeats, which affects shelterin complex binding at chromosomal ends.

## Related tools

- **telomap** (Produces the per-read sample.telomap.tsv table with TELOMERE_START, TELOMERE_END, RAW_TELOMERE_LEN, and TELOMERE_LEN fields consumed by this skill; also analyzes TVS distribution) — https://github.com/cytham/telomap
- **pandas** (Loads and filters sample.telomap.tsv rows by CHROM and BARCODE, and aggregates per-sample motif counts and densities)

## Examples

```
import pandas as pd; df = pd.read_csv('sample.telomap.tsv', sep='\t'); sub = df[(df['CHROM']=='chr7q') & (df['BARCODE'].isin(['P14','P15']))]; sub['TRF_count'] = sub.apply(lambda r: seq[r.TELOMERE_START-1:r.TELOMERE_START-1+1000].count('TTAGGGTTA'), axis=1)
```

## Evaluation signals

- Output CSV exists and contains rows for both samples (e.g. P14 and P15) with fields for raw telomere length, TRF count, and normalized density
- Computed raw telomere lengths match reported values (e.g. ~1126 bp for P14 and ~1627 bp for P15)
- Computed TRF motif counts match reported values (e.g. 81 sites for P14 and 62 sites for P15)
- Normalized density values are finite, positive, and consistent with TRF count divided by canonical TELOMERE_LEN per kb
- Each read's telomeric window starts at a position with at least two consecutive TTAGGGTTAGGG repeats, matching telomap's TELOMERE_START definition

## Limitations

- TVSs within the telomere repeat region disrupt continuity of canonical (TTAGGG)n repeats, so RAW_TELOMERE_LEN and TELOMERE_LEN differ and motif counts must be normalized to canonical length
- Restriction enzymes cutting closer to the telomere (HphI, MnlI) frequently cut within TVSs, causing loss of telomere repeat regions and underestimation of telomere length
- Trimming with RsaI and HinfI may yield subtelomeric regions too short for unambiguous identification of specific chromosome ends against T2T-CHM13
- The method is incompatible with measuring telomere length of ALT cells, whose very long telomeres exceed PacBio HiFi DNA polymerase processivity

## Evidence

- [methods] the 1-based position on a read at the 5′ end of the telomeric region as defined by containing at least two consecutive telomeric repeats (5′-TTAGGGTTAGGG-3′): "the 1-based position on a read at the 5′ end of the telomeric region as defined by containing at least two consecutive telomeric repeats (5′-TTAGGGTTAGGG-3′)"
- [methods] TELOMERE_LEN excluding non-telomeric sequences: "Length of telomeric region in bases after excluding all non-telomeric sequences within RAW_TELOMERE_LEN"
- [abstract] TVSs disrupt shelterin binding: "The presence of TVSs disrupts the continuity of the canonical (5'-TTAGGG-3')n telomere repeats, which affects the binding of shelterin complexes at the chromosomal ends"
- [readme] telomap output table in README: "sample.telomap.tsv | Main output table containing per read information"
- [discussion] ALT cells limitation: "Another caveat of this method is the incompatibility of using PacBio HiFi sequencing reads in measuring telomere length of ALT cells"
