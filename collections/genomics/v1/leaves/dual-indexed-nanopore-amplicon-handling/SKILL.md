---
name: dual-indexed-nanopore-amplicon-handling
description: Use when you have MinION fastq/fasta of dual barcode- and primer-flanked
  amplicons that must be assigned to samples, split into per-sample files, and trimmed
  of barcode/primer sequence before consensus building.
when_to_use_negative:
- Input reads are single-indexed or unindexed (no dual barcode/primer structure for
  minibar to match on).
- Input is already demultiplexed per-sample data, e.g. a feature/abundance table or
  trimmed consensus sequences — demultiplexing would be redundant.
- You need short-read (Illumina) demultiplexing; minibar is built for MinION amplicon
  reads.
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: MiniBar
  role: Demultiplexes dual-indexed Nanopore reads, bins them into individual sample
    files (-F), and trims barcodes/primers (-T)
  repo: https://github.com/calacademy-research/minibar
- name: Edlib
  role: Python library for edit-distance alignment used to compare consensus sequences
    after demultiplexing (pip install edlib)
  repo: https://github.com/Martinsos/edlib
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giz006
    title: Nanopore sequencing of long ribosomal DNA amplicons enables portable and
      simple biodiversity assessments with high phylogenetic resolution across broad
      taxonomic scale
  source_content_hash: sha256:f1b5288ed7f5559a2461f2dea9fd33f11eed3202c38f2a8ada89ae0f9adb7b87
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/calacademy-research/minibar
  tools:
  - Edlib
  - MiniBar
  - Python 2.7
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# dual-indexed-nanopore-amplicon-handling

## Summary

Demultiplex, bin, and trim dual-indexed Nanopore amplicon reads (e.g. long rDNA amplicons sequenced on a MinION) using minibar.py, then feed per-sample FASTA/FASTQ into downstream consensus and error analyses such as coverage sweeps scored with Edlib.

## When to use

Use when you have MinION fastq/fasta of dual barcode- and primer-flanked amplicons that must be assigned to samples, split into per-sample files, and trimmed of barcode/primer sequence before consensus building.

## When NOT to use

- Input reads are single-indexed or unindexed (no dual barcode/primer structure for minibar to match on).
- Input is already demultiplexed per-sample data, e.g. a feature/abundance table or trimmed consensus sequences — demultiplexing would be redundant.
- You need short-read (Illumina) demultiplexing; minibar is built for MinION amplicon reads.

## Inputs

- MinION FASTQ or FASTA of dual-indexed amplicon reads (gzipped or plain text)
- Barcode/index-combination file mapping sample IDs to fwd index, fwd primer, rev index, rev primer (e.g. MinIndexCombinationPeperomonia.txt)

## Outputs

- Per-sample FASTA/FASTQ files (with -F -P prefix) of demultiplexed reads
- Records with sample ID and barcode/primer hit info appended to headers
- Optionally barcode/primer-trimmed reads (with -T) for consensus assembly
- Run summary line with hit counts (H, HH, Hh, hh, IDs, Mult_IDs)

## How to apply

Run minibar.py with a barcode/index-combination file (e.g. IndexCombinationPeperomonia.txt) and the sequence file; minibar identifies samples by approximate matches of dual barcodes and primers (default barcode edit distance 4, primer edit distance 11, search length 80, Method 3), and appends hit info plus the Sample ID (e.g. 'H-(0,6),H+(0,2) Jun_40') to each record header. Use -F with a -P prefix (e.g. Peperomia_) to bin identified records into individual per-sample files, and add -T to trim the barcode and primer from each read end for downstream processing. The -C/-CC options display the found barcode and primer regions via upper/lower case (and colors), and -D prints edit distances of index and primer matches for diagnostics. Check the run summary line (e.g. '750 seqs: H 750 HH 679 Hh 62 hh 0 IDs 741 Mult_IDs 0') to confirm most reads have dual full hits and unique IDs; the resulting demultiplexed, trimmed per-sample FASTA/FASTQ are then the inputs for consensus assembly and edit-distance (Edlib) comparisons against an Illumina-derived reference consensus.

## Related tools

- **MiniBar** (Demultiplexes dual-indexed Nanopore reads, bins them into individual sample files (-F), and trims barcodes/primers (-T)) — https://github.com/calacademy-research/minibar
- **Edlib** (Python library for edit-distance alignment used to compare consensus sequences after demultiplexing (pip install edlib)) — https://github.com/Martinsos/edlib

## Examples

```
./minibar.py -T -F -P Peperomia_ IndexCombinationPeperomonia.txt PeperomiaTestSet.fasta
```

## Evaluation signals

- Run summary shows the large majority of reads have dual full hits (HH) and unique IDs, with low hh (both-miss) counts and Mult_IDs 0.
- Header comments of output records contain hit info and sample ID, e.g. 'H-(0,6),H+(0,2) Jun_40' matching the barcode file's sample names.
- With -T, barcode and primer sequences are absent from read ends (verifiable with -C upper/lower case display of found regions).
- One per-sample file is produced per identified sample ID (prefix from -P, default sample_).
- Downstream: consensus accuracy vs coverage curve derived from trimmed per-sample fastq must decrease from ~0.257% error at 10 reads to ~0.031% near 300 reads before plateauing.

## Limitations

- Barcode matches rely on edit-distance thresholds (default barcode edit distance 4, primer edit distance 11) — reads with degraded ends may fail primer matching (lowercase 'h', -1 in the hit tuple).
- Duplicate sample entries in the barcode file are an error unless -w is supplied.
- minibar.py is written in Python 2.7 (compatible with Python 3) and requires the Edlib module installed.

## Evidence

- [abstract] introduce a new software (MiniBar) to demultiplex dual indexed Nanopore reads: "introduce a new software (MiniBar) to demultiplex dual indexed Nanopore reads"
- [methods] Binning Samples into Individual Files: "Typically you'll want to group your records by identified sample. This can be done by using the -F option"
- [methods] Trimming barcodes and primers: "For downstream processing, you'll probably want to trim off the barcode and primer part of the sequence. This is done for you if you supply the -T trim option"
- [readme] minibar requirements: "**minibar.py** is written in Python version 2.7 and is compatible with Python version 3. It imports the edlib library which
you can install using **pip install edlib**"
- [readme] minibar default invocation: "minibar defaults are set so a typical run might look like:
```
./minibar.py -T -F Demultiplex.txt my_sequences.fastq
```"
