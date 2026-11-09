---
name: dual-index-read-demultiplexing
description: Use when MinION FASTQ/FASTA amplicon reads carry dual barcode indexes
  and primers that must be identified, trimmed, and binned into per-sample files before
  consensus calling or taxonomic assignment.
when_to_use_negative:
- Reads are single-indexed or unindexed (e.g. raw metagenomic reads without dual barcodes
  and primers) — minibar is designed for dual barcode + primer identification at both
  ends
- Input is already demultiplexed and trimmed per-sample FASTA (e.g. downstream Allele
  Wrangler consensus inputs) — re-running would add no information
- Non-Nanopore reads whose library design lacks the index/primer structure expected
  at the start and end of each sequence
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: MiniBar
  role: 'Primary demultiplexing tool: matches dual barcodes and primers with Edlib
    edit distances, bins reads with -F, trims with -T'
  repo: https://github.com/calacademy-research/minibar
- name: Edlib
  role: Python library module providing the edit-distance alignment used for barcode
    and primer matching
  repo: https://github.com/Martinsos/edlib
- name: Python 2.7
  role: Runtime environment for minibar.py (compatible with Python 3)
provenance:
  source_task_ids:
  - task_002
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giz006
    title: Nanopore sequencing of long ribosomal DNA amplicons enables portable and
      simple biodiversity assessments with high phylogenetic resolution across broad
      taxonomic scale
  source_content_hash: sha256:d6afa5419e1ed3e34742efbe197ccfe9b4b177235d1a6574a0c8d725ada3ecf0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/calacademy-research/minibar
  tools:
  - Allele Wrangler
  - Edlib
  - Illumina
  - MiniBar
  - Python 2.7
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# dual-index-read-demultiplexing

## Summary

Demultiplex dual-indexed Oxford Nanopore MinION amplicon reads (FASTA/FASTQ, gzipped or plain) using minibar.py, which matches dual barcode indexes and primers at both read ends via Edlib edit distance, then bins and/or trims reads per sample.

## When to use

Use when MinION FASTQ/FASTA amplicon reads carry dual barcode indexes and primers that must be identified, trimmed, and binned into per-sample files before consensus calling or taxonomic assignment.

## When NOT to use

- Reads are single-indexed or unindexed (e.g. raw metagenomic reads without dual barcodes and primers) — minibar is designed for dual barcode + primer identification at both ends
- Input is already demultiplexed and trimmed per-sample FASTA (e.g. downstream Allele Wrangler consensus inputs) — re-running would add no information
- Non-Nanopore reads whose library design lacks the index/primer structure expected at the start and end of each sequence

## Inputs

- MinION FASTQ or FASTA reads of dual-indexed long rDNA amplicons (gzipped or plain text), e.g. PeperomiaTestSet.fasta
- Barcode/index combination file with columns for sample, fwd index, fwd primer, rev index, rev primer (e.g. IndexCombinationPeperomonia.txt; -cols overrides default column mapping)

## Outputs

- Per-sample FASTA/FASTQ files with trimmed amplicon sequences (with -F -T -P prefix)
- Records with sample ID and barcode/primer hit-quality info appended to headers
- Run summary line of match counts (e.g. '750 seqs: H 750 HH 679 Hh 62 hh 0 IDs 741 Mult_IDs 0')

## How to apply

Run minibar.py with a barcode file (e.g. IndexCombinationPeperomonia.txt) and the sequence file; defaults are barcode edit distance 4 (-e 4), primer edit distance 11 (-E 11), search length 80 bp (-l 80), and matching Method 3, which first seeks an approximate barcode+primer match at the sequence start and falls back to matching barcodes on both ends. Use -F to bin records into individual sample files (optionally -P to set a prefix such as Peperomia_) and -T to trim barcode and primer from both ends so only the amplicon is retained for downstream processing; -C shows found barcodes/primers in upper/lower case and -D reports the index and primer edit distances per record. MiniBar appends the matched sample ID and hit-quality info (e.g. H-(0,6),H+(0,2) Jun_40, with lowercase h and -1 flagging a primer miss beyond 11 errors) to each record header, and prints a summary line (H/HH/Hh/hh counts, IDs, Mult_IDs) for QC. Validate that most reads receive a unique sample ID and inspect the diagnostic output before using trimmed reads for Allele Wrangler consensus assembly or mock-community read counting.

## Related tools

- **MiniBar** (Primary demultiplexing tool: matches dual barcodes and primers with Edlib edit distances, bins reads with -F, trims with -T) — https://github.com/calacademy-research/minibar
- **Edlib** (Python library module providing the edit-distance alignment used for barcode and primer matching) — https://github.com/Martinsos/edlib
- **Python 2.7** (Runtime environment for minibar.py (compatible with Python 3))

## Examples

```
./minibar.py -T -F -P Peperomia_ IndexCombinationPeperomonia.txt PeperomiaTestSet.fasta
```

## Evaluation signals

- Run summary reports most reads assigned a unique sample ID with Mult_IDs 0 (README example: 'IDs 741 Mult_IDs 0' of 750 seqs)
- Header hit codes confirm expected matches: uppercase H for barcode+primer found, lowercase h with -1 indicating a primer miss beyond 11 edit-distance errors
- Trimmed (-T) per-sample files contain only the amplicon sequence with barcodes and primers removed, binned under the correct sample ID (e.g. Jun_38, Jun_39, Jun_40)
- Default thresholds honored unless overridden: index edit distance 4, primer edit distance 11, search length 80, Method 3
- Reads failing to match a known dual index are correctly diverted to an 'unk'/unidentified bin rather than force-assigned

## Limitations

- Barcode file must have consistent column layout; use -info cols to verify column settings and -cols to override the defaults (1,2,3,4,5 / 1,3,4,5,6 / 1,3,4,6,7 / 1,3,5,8,10)
- Duplicate sample rows in the barcode file are treated as errors unless -w is given
- Header detection of the barcode file is auto-detected; if wrong, force with -fh or -nh
- A record can match a barcode but miss the primer (h with -1), counted as Hh — such reads are lower-confidence identifications

## Evidence

- [readme] Identify MinION sequence by dual barcode indexes and primers.: "Identify MinION sequence by dual barcode indexes and primers."
- [readme] -T trims barcode and primer from each end: "-T trims barcode and primer from each end of the sequence, then outputs record"
- [readme] group records by identified sample with -F: "Typically you'll want to group your records by identified sample. This can be done by using the -F option"
- [readme] index edit distance defaults: "-e <int> barcode edit distance value, overrides -p (4)"
- [abstract] MiniBar introduced for demultiplexing: "introduce a new software (MiniBar) to demultiplex dual indexed Nanopore reads"
- [readme] example run command: "./minibar.py -T -F Demultiplex.txt my_sequences.fastq"
- [methods] trim option in article methods: "you'll probably want to trim off the barcode and primer part of the sequence. This is done for you if you supply the -T trim option"
