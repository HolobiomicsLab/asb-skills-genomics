---
name: telomere-repeat-motif-counting
description: Use when you have a telomap sample.telomap.tsv with per-read RAW_TELOMERE_LEN,
  TELOMERE_LEN, TELOMERE_START/END coordinates and need to count TRF1/TRF2 binding
  motifs and compute binding-site density per kb of canonical telomere.
when_to_use_negative:
- Input is raw long reads (FASTA/FASTQ/BAM) rather than an annotated telomap output
  table — run telomap first to identify telomeric reads and TELOMERE_START/END coordinates
- Telomeres are from ALT cells, whose very long telomeres exceed PacBio HiFi DNA polymerase
  processivity and cannot be fully measured by this workflow
- The goal is total telomere length measurement rather than motif/binding-site density
  — use RAW_TELOMERE_LEN/TELOMERE_LEN directly instead of motif counting
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: telomap
  role: Produces the per-read sample.telomap.tsv table (TELOMERE_START/END, RAW_TELOMERE_LEN,
    TELOMERE_LEN, BARCODE, CHROM) that motif counting is performed on
  repo: https://github.com/cytham/telomap
- name: pandas
  role: Loads and filters sample.telomap.tsv rows by CHROM and BARCODE and aggregates
    motif counts and densities per sample
provenance:
  source_task_ids:
  - task_008
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-35823-7
    title: High-throughput telomere length measurement at nucleotide resolution using
      the PacBio high fidelity sequencing platform
  source_content_hash: sha256:2937cd6a81c2042f158aeccc7a22723b5ec6135e040a41b5883232dd8f653c2c
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
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# telomere-repeat-motif-counting

## Summary

Counts occurrences of shelterin binding motifs (e.g. the TRF1/TRF2 motif 5'-TTAGGGTTA-3') within extracted telomeric repeat tracts from telomap per-read output, and normalizes counts by canonical telomere length to compare binding-site density across samples. This quantifies how telomeric variant sequences (TVSs) disrupt canonical (5'-TTAGGG-3')n continuity and thus shelterin binding at chromosome ends.

## When to use

Use when you have a telomap sample.telomap.tsv with per-read RAW_TELOMERE_LEN, TELOMERE_LEN, TELOMERE_START/END coordinates and need to count TRF1/TRF2 binding motifs and compute binding-site density per kb of canonical telomere.

## When NOT to use

- Input is raw long reads (FASTA/FASTQ/BAM) rather than an annotated telomap output table — run telomap first to identify telomeric reads and TELOMERE_START/END coordinates
- Telomeres are from ALT cells, whose very long telomeres exceed PacBio HiFi DNA polymerase processivity and cannot be fully measured by this workflow
- The goal is total telomere length measurement rather than motif/binding-site density — use RAW_TELOMERE_LEN/TELOMERE_LEN directly instead of motif counting

## Inputs

- telomap main output table sample.telomap.tsv with columns RNAME, OLIGO, BARCODE, STRAND, MOTIF, RLEN, RAW_TELOMERE_LEN, TELOMERE_LEN, TELOMERE_START, TELOMERE_END and chromosome-end assignment (CHROM)
- Chromosome-end designation (e.g. Chr 7q) mapped via subtelomeric anchors against the T2T-CHM13 assembly
- Sample identifiers from telobait barcodes (e.g. BARCODE values P14, P15)

## Outputs

- Per-sample motif count table of TRF binding motif (5'-TTAGGGTTA-3') occurrences within 1 kb canonical telomere windows
- Binding-site density per kb of canonical telomere (motif count normalized by TELOMERE_LEN)
- Structured comparison record (e.g. chr7q_trf_comparison.csv) with raw telomere length, TRF motif count, and normalized density per sample

## How to apply

Load sample.telomap.tsv with pandas and filter rows to the chromosome end of interest (CHROM column) and the samples being compared (BARCODE column, e.g. P14 vs P15). Extract each read's telomeric region using TELOMERE_START and TELOMERE_END coordinates, taking the first 1 kb of the canonical repeat tract, where TELOMERE_START is defined by at least two consecutive 5'-TTAGGGTTAGGG-3' repeats per telomap's definition. Scan that window with regex/str.find for the TRF binding motif 5'-TTAGGGTTA-3' and count occurrences per read. Normalize motif counts by TELOMERE_LEN (canonical length after excluding non-telomeric/variant sequences) to get density per kb, then aggregate per sample — e.g. P14 (1126 bp raw, 81 sites) vs P15 (1627 bp raw, 62 sites) at Chr 7q — and write a structured comparison record such as chr7q_trf_comparison.csv. The rationale is that TVSs dispersed throughout the repeat region interrupt canonical (TTAGGG)n tracts, so identical canonical-repeat lengths can still carry different numbers of shelterin binding sites.

## Related tools

- **telomap** (Produces the per-read sample.telomap.tsv table (TELOMERE_START/END, RAW_TELOMERE_LEN, TELOMERE_LEN, BARCODE, CHROM) that motif counting is performed on) — https://github.com/cytham/telomap
- **pandas** (Loads and filters sample.telomap.tsv rows by CHROM and BARCODE and aggregates motif counts and densities per sample)

## Examples

```
import pandas as pd; df = pd.read_csv('sample.telomap.tsv', sep='\t'); p7q = df[(df.CHROM=='chr7q') & (df.BARCODE.isin(['P14','P15']))]; p7q['trf'] = p7q.apply(lambda r: r.SEQ[r.TELOMERE_START:r.TELOMERE_START+1000].count('TTAGGGTTA'), axis=1)
```

## Evaluation signals

- Output CSV exists and contains one row per expected sample (e.g. P14 and P15) with fields for raw telomere length, TRF motif count, and normalized density
- Computed raw telomere lengths match reported values (e.g. 1126 bp for P14 and 1627 bp for P15 at Chr 7q)
- Computed TRF motif counts match reported values (e.g. 81 sites for P14 and 62 for P15) despite both samples having an identical 1 kb of canonical telomere repeats
- Every counted read's TELOMERE_START satisfies telomap's definition of at least two consecutive 5'-TTAGGGTTAGGG-3' repeats
- Density per kb is consistent with count/TELOMERE_LEN for each sample (recomputable from output columns)

## Limitations

- TELOMERE_LEN excludes all non-telomeric (variant) sequences within RAW_TELOMERE_LEN, so motif density depends on how TVSs are classified as non-canonical
- Shelterin binding density inferred from the 5'-TTAGGGTTA-3' motif is a sequence-based proxy and does not directly measure protein occupancy
- Motif counts within a fixed 1 kb window of the canonical tract may differ from counts over the full telomere, and the paper notes TVSs are extremely heterogeneous and dispersed throughout the repeat region

## Evidence

- [abstract] TVSs disrupt the continuity of canonical (5'-TTAGGG-3')n telomere repeats, affecting shelterin complex binding at chromosomal ends and telomere protection: "The presence of TVSs disrupts the continuity of the canonical (5'-TTAGGG-3')n telomere repeats, which affects the binding of shelterin complexes at the chromosomal ends and telomere protection"
- [methods] TELOMERE_START requires two consecutive canonical repeats: "the 1-based position on a read at the 5′ end of the telomeric region as defined by containing at least two consecutive telomeric repeats (5′-TTAGGGTTAGGG-3′)"
- [readme] TELOMERE_LEN excludes non-telomeric sequence: "Length of telomeric region in bases after excluding all non-telomeric sequences"
- [readme] telomap output table schema: "sample.telomap.tsv | Main output table containing per read information"
