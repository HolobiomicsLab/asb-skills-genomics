---
name: dual-index-barcode-matching
description: Use when FASTQ/FASTA reads from MinION sequencing of dual-indexed amplicons
  must be assigned to specimens by matching dual barcode indexes plus primers, and
  per-specimen read counts or binned sample files are needed.
when_to_use_negative:
- Reads are not dual-indexed Nanopore amplicons — e.g. single-index Illumina data
  or data whose barcodes do not include flanking primers at read ends
- Input is already demultiplexed per-sample FASTQ files; binning/assignment has already
  occurred
- The task is basecalling, consensus building, or downstream phylogenetic analysis
  rather than sample assignment of raw reads
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0080
tools:
- name: MiniBar
  role: 'Core demultiplexer: matches dual barcodes and primers by edit distance and
    appends sample IDs to record headers'
  repo: https://github.com/calacademy-research/minibar
- name: Edlib
  role: Python library for approximate string matching (edit distance) used by MiniBar
    to score barcode/primer hits
  repo: https://github.com/Martinsos/edlib
- name: Python 2.7
  role: Runtime for minibar.py (also compatible with Python 3)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giz006
    title: Nanopore sequencing of long ribosomal DNA amplicons enables portable and
      simple biodiversity assessments with high phylogenetic resolution across broad
      taxonomic scale
  source_content_hash: sha256:724b64050520445bebd1108429d72c6b34d36d7f1c2e4f0b19e56ae91d28e70b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/calacademy-research/minibar
  tools:
  - Edlib
  - MiniBar
  - Python 2.7
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# dual-index-barcode-matching

## Summary

Demultiplex dual-indexed Nanopore (MinION) rDNA amplicon reads with MiniBar, which matches forward/reverse barcode indexes and primers at each read end using edit-distance search (Edlib) and appends the sample ID to each record header.

## When to use

Use when FASTQ/FASTA reads from MinION sequencing of dual-indexed amplicons must be assigned to specimens by matching dual barcode indexes plus primers, and per-specimen read counts or binned sample files are needed.

## When NOT to use

- Reads are not dual-indexed Nanopore amplicons — e.g. single-index Illumina data or data whose barcodes do not include flanking primers at read ends
- Input is already demultiplexed per-sample FASTQ files; binning/assignment has already occurred
- The task is basecalling, consensus building, or downstream phylogenetic analysis rather than sample assignment of raw reads

## Inputs

- FASTQ or FASTA file of MinION reads (plain or gzipped), e.g. PeperomiaTestSet.fasta
- Barcode/index combination file mapping sample name, forward index, forward primer, reverse index, reverse primer (e.g. IndexCombinationPeperomonia.txt)
- Edit-distance settings: index edit distance via -e (e.g. 2, 3, 4) and primer edit distance via -E (default 11)

## Outputs

- FASTA/FASTQ records with sample ID and barcode/primer hit info appended to the header comment (e.g. PeperomiaTestSet_SampleIDs.fa)
- Optional per-sample files when -F is supplied (prefix, e.g. 'sample_' or 'Peperomia_')
- Summary line per run: seqs, H/HH/Hh/hh counts, IDs, Mult_IDs
- Per-specimen read-count tally usable for a tab-separated table

## How to apply

Run minibar.py with a barcode file (e.g. IndexCombinationPeperomonia.txt mapping sample, fwd index, fwd primer, rev index, rev primer) and the FASTQ/FASTA read file; MiniBar searches the first and last 80 bases (default -l 80) of each read using Edlib approximate string matching (search method 3: barcode+primer match at the start, falling back to matched barcodes on both ends), accepting barcode hits within a configurable edit distance (-e; default 4) and primer hits within edit distance 11 (-E). Each record header receives the hit info (e.g. 'H-(0,6),H+(0,2) Jun_40') with the sample ID at the end of the comment field; uppercase H signals a barcode+primer hit and lowercase h a barcode hit with primer miss (-1). Parse the trailing sample ID from each header, rejecting records with no match or multiple matches (unk / Mult_IDs), then tally reads per specimen — e.g. in the Peperomia test set, mean reads per specimen rose from 355 at index edit distance 2 to 647 at 3 and 1,051 at 4, so choose the edit distance deliberately. Optionally use -F to bin reads into per-sample files and -T to trim barcode/primer sequence from read ends for downstream analysis.

## Related tools

- **MiniBar** (Core demultiplexer: matches dual barcodes and primers by edit distance and appends sample IDs to record headers) — https://github.com/calacademy-research/minibar
- **Edlib** (Python library for approximate string matching (edit distance) used by MiniBar to score barcode/primer hits) — https://github.com/Martinsos/edlib
- **Python 2.7** (Runtime for minibar.py (also compatible with Python 3))

## Examples

```
./minibar.py -T -F Demultiplex.txt my_sequences.fastq
```

## Evaluation signals

- Run summary line reports nonzero sequence and ID counts, e.g. '750 seqs: H 750 HH 679 Hh 62 hh 0 IDs 741 Mult_IDs 0 (0.1245s)'
- Every assigned record header ends with a known sample ID preceded by hit info like 'H-(0,6),H+(0,2)'; records with unk or Multiple_Matches are tallied separately
- Reproducing the Peperomia test set at index edit distances 2/3/4 (primer edit distance 11) yields mean reads-per-specimen near 355/647/1,051
- Uppercase/lowercase hit codes are consistent: HH records have primer matches at both ends, Hh records have a primer miss signaled by 'h' and -1 in the hit coordinates
- Per-specimen counts increase monotonically with larger index edit distance; a decrease would indicate a parameter or parsing error

## Limitations

- Index edit distance strongly trades off read yield vs assignment confidence — increasing from 2 to 4 raised mean reads per specimen from 355 to 1,051, so looser thresholds assign more reads but with greater misassignment risk
- Primer edit distance is capped (default 11 against 27-base primers); records failing the primer check at one end are marked lowercase h/Hh and counted separately
- Barcode file must match the expected column layout (default column mappings depend on 5/6/7/10+ columns) and duplicates are treated as errors unless -w is given

## Evidence

- [readme] Sample ID is placed at end of header comment with match hit info before it.: "Sample ID is placed at end of header comment with match hit info before it."
- [readme] -e barcode edit distance, -E primer edit distance 11, -l search length 80: "-e <int> barcode edit distance value, overrides -p (4)
        -E <int> primer edit distance value (11)
        -l <int> length to search for index and primer at start and end of sequence (80)"
- [readme] Summary line example: "750 seqs: H 750 HH 679 Hh 62 hh 0 IDs 741 Mult_IDs 0 (0.1245s)"
- [discussion] Minibar repository: "The program Minibar can be found at https://github.com/calacademy-research/minibar"
- [methods] Default invocation from methods: "$ minibar.py IndexCombinationPeperomonia.txt PeperomiaTestSet.fasta >PeperomiaTestSet_SampleIDs.fa"
