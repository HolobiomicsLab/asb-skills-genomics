---
name: per-read-telomap-table-filtering
description: Use when you have a telomap main output table (sample.telomap.tsv) and
  need to extract per-read telomeric regions for specific barcodes or chromosome ends
  to compute raw vs canonical telomere lengths or motif binding-site densities.
when_to_use_negative:
- Input is raw long-read FASTQ/BAM rather than telomap output — run telomap (wgs or
  telobait mode) first to generate sample.telomap.tsv
- Telomeres are from ALT cells with very long telomeres that exceed PacBio HiFi DNA
  polymerase processivity — telomap-based per-read measurement is incompatible
- The subtelomeric anchor region is too short for unambiguous chromosome-end identification
  against T2T-CHM13 (e.g. after RsaI/HinfI trimming), making CHROM assignments unreliable
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0654
tools:
- name: telomap
  role: Produces the per-read telomap.tsv table being filtered; identifies telomeric
    reads, assigns barcodes and chromosome ends, and computes RAW_TELOMERE_LEN and
    TELOMERE_LEN per read
  repo: https://github.com/cytham/telomap
- name: pandas
  role: Loads and filters the sample.telomap.tsv table and aggregates per-sample telomere
    length and motif-density statistics
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-35823-7
    title: High-throughput telomere length measurement at nucleotide resolution using
      the PacBio high fidelity sequencing platform
  source_content_hash: sha256:57f992fab62abd16de5b06e376ae49e932be0af6f72218770a7c0e9fb5c7c955
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cytham/telomap
  tools:
  - pandas
  - telomap
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# per-read-telomap-table-filtering

## Summary

Filters and interprets the per-read telomap output table (sample.telomap.tsv) to isolate telomeric reads for a given sample barcode and chromosome end, using columns such as CHROM, BARCODE, TELOMERE_START, TELOMERE_END, RAW_TELOMERE_LEN, and TELOMERE_LEN for downstream telomere-length and motif-density comparisons.

## When to use

Use when you have a telomap main output table (sample.telomap.tsv) and need to extract per-read telomeric regions for specific barcodes or chromosome ends to compute raw vs canonical telomere lengths or motif binding-site densities.

## When NOT to use

- Input is raw long-read FASTQ/BAM rather than telomap output — run telomap (wgs or telobait mode) first to generate sample.telomap.tsv
- Telomeres are from ALT cells with very long telomeres that exceed PacBio HiFi DNA polymerase processivity — telomap-based per-read measurement is incompatible
- The subtelomeric anchor region is too short for unambiguous chromosome-end identification against T2T-CHM13 (e.g. after RsaI/HinfI trimming), making CHROM assignments unreliable

## Inputs

- sample.telomap.tsv — telomap main per-read output table (RNAME, OLIGO, BARCODE, STRAND, MOTIF, RLEN, RAW_TELOMERE_LEN, TELOMERE_LEN, TELOMERE_START, TELOMERE_END, CHROM)
- Target sample barcode identifiers (e.g. P14, P15) from the telobait barcodes.fa
- Chromosome-end label (e.g. Chr 7q) resolved against the T2T-CHM13 assembly

## Outputs

- Filtered per-read subset of telomeric reads for the chosen barcode and chromosome end
- Per-read extracted telomeric region sequences and lengths (RAW_TELOMERE_LEN, TELOMERE_LEN)
- Per-sample aggregation table (e.g. chr7q_trf_comparison.csv) with raw telomere length, canonical length, TRF motif counts, and normalized binding-site density per kb

## How to apply

Load sample.telomap.tsv with pandas and filter rows by the BARCODE column (e.g. P14, P15) and the chromosome end of interest in the CHROM column (e.g. Chr 7q). For each read, extract the telomeric region using the TELOMERE_START and TELOMERE_END coordinates, where TELOMERE_START is the 1-based 5' position defined by at least two consecutive 5'-TTAGGGTTAGGG-3' repeats. Distinguish RAW_TELOMERE_LEN (the full telomeric region including telomeric variant sequences) from TELOMERE_LEN (canonical length after excluding all non-telomeric sequences within RAW_TELOMERE_LEN); this distinction is what allows detection of telomeres with identical canonical lengths (e.g. 1 kb) but different raw lengths (1126 bp vs 1627 bp) due to TVSs. Count shelterin binding motifs such as 5'-TTAGGGTTA-3' within the canonical tract and normalize by TELOMERE_LEN to obtain binding-site density per kb. Record aggregated per-sample values (raw length, canonical length, TRF motif count, density) for comparison or export as CSV.

## Related tools

- **telomap** (Produces the per-read telomap.tsv table being filtered; identifies telomeric reads, assigns barcodes and chromosome ends, and computes RAW_TELOMERE_LEN and TELOMERE_LEN per read) — https://github.com/cytham/telomap
- **pandas** (Loads and filters the sample.telomap.tsv table and aggregates per-sample telomere length and motif-density statistics)

## Examples

```
import pandas as pd; df = pd.read_csv('sample.telomap.tsv', sep='\t'); sub = df[(df['BARCODE']=='P14') & (df['CHROM']=='Chr7q')]
```

## Evaluation signals

- Filtered rows retain only the requested BARCODE and CHROM values (e.g. P14/P15 at Chr 7q)
- Per-read RAW_TELOMERE_LEN ≥ TELOMERE_LEN, with equality only for reads lacking TVSs
- TELOMERE_START coincides with at least two consecutive 5'-TTAGGGTTAGGG-3' repeats at the 5' end of the telomeric region
- Aggregated raw telomere lengths and TRF motif counts match expected reference values (e.g. ~1126 bp/81 sites for P14 and ~1627 bp/62 sites for P15 at Chr 7q despite identical 1 kb canonical length)
- Output CSV contains rows for each queried sample with raw telomere length, TRF count, and normalized density fields populated

## Limitations

- RAW_TELOMERE_LEN includes TVSs interspersed in the repeat tract; using it instead of TELOMERE_LEN overestimates canonical telomere length for TVS-rich reads
- RsaI/HinfI genomic DNA trimming can leave subtelomeric regions too short for unambiguous chromosome-end identification against T2T-CHM13, undermining CHROM-based filtering
- HphI and MnlI restriction enzymes cut within TVSs, causing loss of telomere repeat regions and underestimation of telomere length
- Method is incompatible with measuring telomere length of ALT cells whose telomeres exceed PacBio HiFi read length limits

## Evidence

- [methods] the 1-based position on a read at the 5′ end of the telomeric region as defined by containing at least two consecutive telomeric repeats (5′-TTAGGGTTAGGG-3′): "the 1-based position on a read at the 5′ end of the telomeric region as defined by containing at least two consecutive telomeric repeats (5′-TTAGGGTTAGGG-3′)"
- [methods] Length of telomeric region in bases after excluding all non-telomeric sequences within RAW_TELOMERE_LEN: "Length of telomeric region in bases after excluding all non-telomeric sequences within RAW_TELOMERE_LEN"
- [readme] Telomap is a tool to identify and analyse telomeric reads within WGS or telobait-captured long-read sequencing data.: "Telomap is a tool to identify and analyse telomeric reads within WGS or telobait-captured long-read sequencing data."
- [abstract] The presence of TVSs disrupts the continuity of the canonical (5'-TTAGGG-3')n telomere repeats, which affects the binding of shelterin complexes at the chromosomal ends: "The presence of TVSs disrupts the continuity of the canonical (5'-TTAGGG-3')n telomere repeats, which affects the binding of shelterin complexes at the chromosomal ends"
