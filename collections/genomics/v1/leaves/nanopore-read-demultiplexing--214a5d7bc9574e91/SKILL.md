---
name: nanopore-read-demultiplexing
description: Use when you have dual-indexed MinION FASTA/FASTQ amplicon reads (e.g.
  long rDNA amplicons) plus a barcode/index combination file and need to assign each
  read to a specimen by matching forward/reverse barcodes and primers.
when_to_use_negative:
- Input reads are already demultiplexed or single-indexed without dual barcodes and
  primers at both read ends — MiniBar's search logic expects dual indexes plus primers
  at the start and end of each MinION read
- Input is a post-clustering OTU/ASV table or per-sample read-count matrix — no demultiplexing
  remains to be done
- Downstream processing that requires fully trimmed, consensus-called sequences is
  the goal without needing sample assignment — use the trimming/consensus steps of
  the pipeline instead
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: MiniBar
  role: Dual barcode and primer demultiplexer for MinION sequenced reads; assigns
    sample IDs to reads based on -e barcode and -E primer edit distances
  repo: https://github.com/calacademy-research/minibar
- name: Edlib
  role: Python library module providing the fast edit-distance (Levenshtein) alignment
    MiniBar uses to match barcodes and primers
  repo: https://github.com/Martinsos/edlib
- name: Python 2.7
  role: Runtime for minibar.py (compatible with Python 3)
- name: Oxford Nanopore MinION
  role: Instrument generating the dual-indexed long rDNA amplicon reads that this
    demultiplexing skill processes
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giz006
    title: Nanopore sequencing of long ribosomal DNA amplicons enables portable and
      simple biodiversity assessments with high phylogenetic resolution across broad
      taxonomic scale
  source_content_hash: sha256:0950e204279470cb166c8ae38bd86649ebfb45c62c848caf5ba36f7c3d1b07ef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/calacademy-research/minibar
  tools:
  - Edlib
  - MiniBar
  - Oxford Nanopore MinION
  - Python 2.7
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# nanopore-read-demultiplexing

## Summary

Demultiplex dual-indexed Oxford Nanopore MinION rDNA amplicon reads with MiniBar (minibar.py, Python 2.7/3 with Edlib), matching dual barcode indexes and primers at the sequence ends and appending per-read sample IDs to record headers.

## When to use

Use when you have dual-indexed MinION FASTA/FASTQ amplicon reads (e.g. long rDNA amplicons) plus a barcode/index combination file and need to assign each read to a specimen by matching forward/reverse barcodes and primers.

## When NOT to use

- Input reads are already demultiplexed or single-indexed without dual barcodes and primers at both read ends — MiniBar's search logic expects dual indexes plus primers at the start and end of each MinION read
- Input is a post-clustering OTU/ASV table or per-sample read-count matrix — no demultiplexing remains to be done
- Downstream processing that requires fully trimmed, consensus-called sequences is the goal without needing sample assignment — use the trimming/consensus steps of the pipeline instead

## Inputs

- Nanopore MinION amplicon reads in FASTA or FASTQ format (gzipped or plain), e.g. PeperomiaTestSet.fasta
- Dual-index barcode combination file with sample, forward index, forward primer, reverse index, reverse primer columns (e.g. IndexCombinationPeperomonia.txt / MinIndexCombinationPeperomonia.txt)
- Barcode edit distance setting (-e: 2, 3, or 4) and primer edit distance (-E, default 11)

## Outputs

- FASTA/FASTQ records with sample ID and hit quality info (e.g. H-(0,6),H+(0,2) Jun_40) appended to the header comment, e.g. PeperomiaTestSet_SampleIDs.fa
- Run summary line such as '750 seqs: H 750 HH 679 Hh 62 hh 0 IDs 741 Mult_IDs 0 (0.1245s)'
- Optionally, individual per-sample files via -F (default prefix 'sample_', e.g. 'Peperomia_' with -P)
- Per-specimen read counts used to compute mean reads per specimen (reported: 355 at e=2, 647 at e=3, 1051 at e=4)

## How to apply

Obtain minibar.py from the calacademy-research/minibar repository and install the Edlib module (pip install edlib). Run minibar.py with the barcode file (e.g. IndexCombinationPeperomonia.txt) and the Nanopore read file using default settings: search length 80 (-l), search method 3 (-M 3, which tries approximate barcode+primer match at the start and falls back to matching barcodes on both ends), primer edit distance 11 (-E 11), and vary the barcode edit distance via -e (2, 3, 4). MiniBar appends match hit info plus the sample ID to the end of each record header comment; parse that field and reject records with no sample match (unk) or multiple matches (Mult_IDs). Use -T to trim barcodes and primers for downstream processing and -F (optionally -P prefix) to bin records into per-sample files. Note the run summary line (seqs, HH/Hh/hh counts, IDs, Mult_IDs) — lowercase h and -1 in the hit annotation (e.g. h-(3,-1)) signal a barcode match whose primer did not match within the allowed 11 edit distance.

## Related tools

- **MiniBar** (Dual barcode and primer demultiplexer for MinION sequenced reads; assigns sample IDs to reads based on -e barcode and -E primer edit distances) — https://github.com/calacademy-research/minibar
- **Edlib** (Python library module providing the fast edit-distance (Levenshtein) alignment MiniBar uses to match barcodes and primers) — https://github.com/Martinsos/edlib
- **Python 2.7** (Runtime for minibar.py (compatible with Python 3))
- **Oxford Nanopore MinION** (Instrument generating the dual-indexed long rDNA amplicon reads that this demultiplexing skill processes)

## Examples

```
./minibar.py -T -F IndexCombinationPeperomonia.txt PeperomiaTestSet.fasta -e 4 -E 11 > PeperomiaTestSet_SampleIDs.fa
```

## Evaluation signals

- Each run's summary line reports nonzero H, HH, Hh counts and IDs with Mult_IDs and unassigned (unk) reads recorded separately
- Matched records carry hit info plus a sample ID at the end of the header comment (e.g. 'H-(0,6),H+(0,2) Jun_40'); primer misses appear as lowercase h with -1 (e.g. 'h-(3,-1)')
- Mean reads per specimen at index edit distances 2, 3, and 4 with primer edit distance 11 reproduce the reported 355 / 647 / 1,051 pattern, with read numbers increasing at larger edit distances
- Identified records with perfect both-end barcode+primer matches are tallied as HH, one-end primer misses as Hh, in the summary line
- Reference behavior: the README example run on PeperomiaTestSet.fasta yields '750 seqs: H 750 HH 679 Hh 62 hh 0 IDs 741 Mult_IDs 0'

## Limitations

- Reads with no sample match (unk) or multiple sample matches (Mult_IDs) cannot be assigned to a specimen and must be recorded separately
- A barcode hit can pass at the allowed -e distance while its primer fails the -E 11 limit, leaving the record in a lower-quality Hh class
- Read assignment counts increase substantially as the index edit distance is relaxed (355 → 647 → 1,051), so larger -e values admit more reads but risk more misassignment
- Duplicate entries in the barcode file are treated as errors unless the -w flag is used; column positions for sample/index/primer fields must match the barcode file layout (default -cols mapping varies for 5, 6, 7, or 10+ columns)

## Evidence

- [readme] Sample ID is placed at end of header comment with match hit info before it.: "Sample ID is placed at end of header comment with match hit info before it."
- [readme] 750 seqs: H 750 HH 679 Hh 62 hh 0 IDs 741 Mult_IDs 0 (0.1245s): "750 seqs: H 750 HH 679 Hh 62 hh 0 IDs 741 Mult_IDs 0 (0.1245s)"
- [methods] minibar.py IndexCombinationPeperomonia.txt PeperomiaTestSet.fasta >PeperomiaTestSet_SampleIDs.fa: "$ minibar.py IndexCombinationPeperomonia.txt PeperomiaTestSet.fasta >PeperomiaTestSet_SampleIDs.fa"
- [abstract] introduce a new software (MiniBar) to demultiplex dual indexed Nanopore reads: "introduce a new software (MiniBar) to demultiplex dual indexed Nanopore reads"
- [discussion] Programming language: Python 2.7 (but can be run in Python 3): "Programming language: Python 2.7 (but can be run in Python 3)"
