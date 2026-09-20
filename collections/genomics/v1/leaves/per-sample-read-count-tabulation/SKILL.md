---
name: per-sample-read-count-tabulation
description: Use when you have MiniBar-demultiplexed Nanopore FASTA/FASTQ records
  with sample IDs in header comments and need per-specimen read tallies, e.g. to quantify
  assignment yield across barcode edit-distance settings (-e 2/3/4).
when_to_use_negative:
- Input is a bulk-community metabarcoding sample where per-specimen read counts are
  not meaningful — the article cautions further research on taxonomic bias is required
  before long rDNA amplicon sequencing is used for bulk sample analysis
- You need phylogenetic or taxonomic identification of reads rather than demultiplexing
  yield statistics — this skill only tabulates MiniBar assignments, it does not classify
  sequences
- Reads lack dual barcode index and primer structure (e.g. already trimmed or single-indexed
  Illumina data), so MiniBar's dual barcode/primer matching at the sequence start
  and end does not apply
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: MiniBar
  role: Demultiplexer whose per-read sample ID annotations and run summary lines are
    parsed and tabulated; invoked with -e to vary barcode edit distance (default -e
    4, -E 11, -l 80, -M 3)
  repo: https://github.com/calacademy-research/minibar
- name: Edlib
  role: Python library module used by minibar.py for approximate string matching (edit
    distance) of barcodes and primers against reads
  repo: https://github.com/Martinsos/edlib
- name: Python 2.7
  role: Runtime for minibar.py (also compatible with Python 3) and for parsing sample
    IDs from record headers and assembling count tables
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giz006
    title: Nanopore sequencing of long ribosomal DNA amplicons enables portable and
      simple biodiversity assessments with high phylogenetic resolution across broad
      taxonomic scale
  source_content_hash: sha256:333c4f0dd771689269ace0e4c4d09af343b879541baf8bb6c4f925c343484ea2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/calacademy-research/minibar
  tools:
  - Edlib
  - MiniBar
  - Python 2.7
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# per-sample-read-count-tabulation

## Summary

Tally the number of MinION reads MiniBar assigns to each dual-indexed specimen, by parsing the sample ID appended to each record header and aggregating per-specimen counts (e.g. mean reads/specimen at index edit distances 2, 3, and 4).

## When to use

Use when you have MiniBar-demultiplexed Nanopore FASTA/FASTQ records with sample IDs in header comments and need per-specimen read tallies, e.g. to quantify assignment yield across barcode edit-distance settings (-e 2/3/4).

## When NOT to use

- Input is a bulk-community metabarcoding sample where per-specimen read counts are not meaningful — the article cautions further research on taxonomic bias is required before long rDNA amplicon sequencing is used for bulk sample analysis
- You need phylogenetic or taxonomic identification of reads rather than demultiplexing yield statistics — this skill only tabulates MiniBar assignments, it does not classify sequences
- Reads lack dual barcode index and primer structure (e.g. already trimmed or single-indexed Illumina data), so MiniBar's dual barcode/primer matching at the sequence start and end does not apply

## Inputs

- Nanopore FASTQ/FASTA read file (e.g. PeperomiaTestSet.fasta from dual-indexed long rDNA amplicon sequencing)
- Dual barcode/index combination file (e.g. IndexCombinationPeperomonia.txt / MinIndexCombinationPeperomonia.txt) with sample, fwd index, fwd primer, rev index, rev primer columns
- minibar.py script plus the Edlib Python module

## Outputs

- Per-specimen read count table (tab-separated) at each index edit distance tested
- Mean reads-per-specimen summary (e.g. 355 / 647 / 1,051 at edit distances 2 / 3 / 4)
- MiniBar run summary lines (seqs, HH, Hh, hh, IDs, Mult_IDs) and counts of unassigned (unk) reads

## How to apply

Run minibar.py with the barcode combination file (e.g. IndexCombinationPeperomonia.txt) against the Nanopore read file, varying the barcode edit distance via -e (2, 3, 4) with primer edit distance -E 11, search length -l 80, and match method -M 3; keep the default output so the sample ID (e.g. Jun_40) and hit-quality info (H-/h- codes) are appended to each record's header comment. Parse the sample ID from the end of each header comment, counting records with no match (unk) or multiple matches (Mult_IDs) separately from assigned reads. Capture each run's summary line (seqs, HH/Hh/hh, IDs, Mult_IDs) to confirm assignment volume. Tally reads per specimen, compute the mean reads per specimen at each edit distance, and assemble the per-specimen counts into a tab-separated table.

## Related tools

- **MiniBar** (Demultiplexer whose per-read sample ID annotations and run summary lines are parsed and tabulated; invoked with -e to vary barcode edit distance (default -e 4, -E 11, -l 80, -M 3)) — https://github.com/calacademy-research/minibar
- **Edlib** (Python library module used by minibar.py for approximate string matching (edit distance) of barcodes and primers against reads) — https://github.com/Martinsos/edlib
- **Python 2.7** (Runtime for minibar.py (also compatible with Python 3) and for parsing sample IDs from record headers and assembling count tables)

## Examples

```
./minibar.py -e 4 IndexCombinationPeperomonia.txt PeperomiaTestSet.fasta > PeperomiaTestSet_SampleIDs.fa
```

## Evaluation signals

- Mean reads per specimen at index edit distances 2, 3, and 4 with primer edit distance 11 reproduces the reported 355 / 647 / 1,051 pattern, with counts increasing significantly at larger edit distances
- Each run's summary line reports nonzero IDs counts, with Mult_IDs and unassigned (unk) reads recorded separately (README example: '750 seqs: H 750 HH 679 Hh 62 hh 0 IDs 741 Mult_IDs 0')
- Every parsed record header carries a recognizable hit-quality annotation (H/h codes with edit-distance pairs such as H-(0,6),H+(0,2)) followed by a sample ID from the barcode file
- The number of reads assigned to specimens is consistent with the IDs count in the run summary line for each -e setting
- The assembled table is tab-separated and covers all specimens present in the barcode combination file

## Limitations

- Reads with no sample match (unk) or multiple matches (Mult_IDs) cannot be tallied to a specimen and must be excluded from per-specimen counts
- Assignments depend on the barcode edit-distance threshold: -e 2 under-assigns relative to -e 4, so the metric is threshold-sensitive and not an absolute specimen abundance
- minibar.py requires the Edlib library module (pip install edlib); it is written for Python 2.7 but can be run in Python 3

## Evidence

- [readme] 750 seqs: H 750 HH 679 Hh 62 hh 0 IDs 741 Mult_IDs 0 (0.1245s): "750 seqs: H 750 HH 679 Hh 62 hh 0 IDs 741 Mult_IDs 0 (0.1245s)"
- [readme] Sample ID is placed at end of header comment with match hit info before it.: "Sample ID is placed at end of header comment with match hit info before it."
- [methods] $ minibar.py IndexCombinationPeperomonia.txt PeperomiaTestSet.fasta >PeperomiaTestSet_SampleIDs.fa: "$ minibar.py IndexCombinationPeperomonia.txt PeperomiaTestSet.fasta >PeperomiaTestSet_SampleIDs.fa"
- [readme] -e <int> barcode edit distance value, overrides -p (4): "-e <int> barcode edit distance value, overrides -p (4)
        -E <int> primer edit distance value (11)"
- [readme] minibar.py is written in Python version 2.7 and is compatible with Python version 3. It imports the edlib library: "minibar.py is written in Python version 2.7 and is compatible with Python version 3. It imports the edlib library which
you can install using **pip install edlib**"
