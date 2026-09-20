---
name: barcode-primer-trimming
description: Use when MinION FASTA/FASTQ reads carry dual barcode indexes and 27-base
  primers at sequence ends and need trimming/binning before consensus calling or alignment.
when_to_use_negative:
- Reads lack dual barcode indexes and primers (e.g. already-demultiplexed or short-read
  Illumina data), so -T trimming would remove genuine sequence
- The goal is only to inspect barcode/primer match structure or edit distances without
  altering records — use -C/-D diagnostic output instead of trimming
- Input is a bulk-community metabarcoding run where the paper notes bias sources are
  undetermined and the approach is not yet reliable
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: MiniBar
  role: Demultiplexes dual-indexed MinION reads by barcode+primer match and trims
    barcodes/primers from read ends with -T
  repo: https://github.com/calacademy-research/minibar
- name: Edlib
  role: Python library used by minibar.py for edit-distance alignment of barcode and
    primer sequences
  repo: https://github.com/Martinsos/edlib
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giz006
    title: Nanopore sequencing of long ribosomal DNA amplicons enables portable and
      simple biodiversity assessments with high phylogenetic resolution across broad
      taxonomic scale
  source_content_hash: sha256:7717da4679796f696009edcb7a01cd46567eca4f6d6f15db4c53339cacdb1831
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/calacademy-research/minibar
  tools:
  - Allele Wrangler
  - Edlib
  - MiniBar
  - Python 2.7
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# barcode-primer-trimming

## Summary

Trims dual barcode indexes and primers from the ends of MinION-sequenced long rDNA amplicon reads using MiniBar's -T trim option, producing cleaned per-sample records ready for downstream consensus assembly.

## When to use

Use when MinION FASTA/FASTQ reads carry dual barcode indexes and 27-base primers at sequence ends and need trimming/binning before consensus calling or alignment.

## When NOT to use

- Reads lack dual barcode indexes and primers (e.g. already-demultiplexed or short-read Illumina data), so -T trimming would remove genuine sequence
- The goal is only to inspect barcode/primer match structure or edit distances without altering records — use -C/-D diagnostic output instead of trimming
- Input is a bulk-community metabarcoding run where the paper notes bias sources are undetermined and the approach is not yet reliable

## Inputs

- MinION FASTA or FASTQ read file of dual-indexed long rDNA amplicons (e.g. PeperomiaTestSet.fasta, gzipped or plain)
- Dual barcode/primer combination file (e.g. IndexCombinationPeperomonia.txt) with columns for sample, fwd index, fwd primer, rev index, rev primer

## Outputs

- Trimmed per-sample FASTA files (e.g. Peperomia_Jun_38, Peperomia_Jun_39, Peperomia_Jun_40, unk) with sample ID and match hit info appended to headers
- Run summary line (e.g. '750 seqs: H 750 HH 679 Hh 62 hh 0 IDs 741 Mult_IDs 0')

## How to apply

Run minibar.py with a dual-index combination file (e.g. IndexCombinationPeperomonia.txt) against the read file, supplying the -T trim option so barcode and primer sequence is removed from each end, and -F with -P (e.g. Peperomia_) to bin records into per-sample FASTA files (e.g. Peperomia_Jun_38/39/40, unk). MiniBar searches the first/last 80 bases (-l default), uses barcode edit distance 4 (-e) and primer edit distance 11 (-E) with search Method 3, which combines a start-end barcode+primer match (Method 1) with a both-ends barcode-pair match (Method 2). Trimming is done for downstream processing so polished consensus sequences reflect only the amplicon insert; sample IDs and hit-quality info (e.g. H-(0,6),H+(0,2) Jun_40) are appended to record headers. Validate that all records carry a sample ID (IDs close to total seqs, Mult_IDs 0) before proceeding to consensus assembly with Allele Wrangler at minimum coverage 30.

## Related tools

- **MiniBar** (Demultiplexes dual-indexed MinION reads by barcode+primer match and trims barcodes/primers from read ends with -T) — https://github.com/calacademy-research/minibar
- **Edlib** (Python library used by minibar.py for edit-distance alignment of barcode and primer sequences) — https://github.com/Martinsos/edlib

## Examples

```
./minibar.py -T -F -P Peperomia_ IndexCombinationPeperomonia.txt PeperomiaTestSet.fasta
```

## Evaluation signals

- Record headers show appended sample ID with hit info (e.g. 'H-(0,6),H+(0,2) Jun_40') confirming barcode/primer identification
- Run summary reports IDs near total sequence count with Mult_IDs 0 (e.g. 741 IDs from 750 seqs) and most records classified HH
- Output per-sample files follow the -P prefix convention (Peperomia_Jun_38/39/40, unk) and trimmed records no longer contain the 27-base barcode/primer segments
- Every binned specimen meets downstream requirements: minimum coverage of 30 reads per position for Allele Wrangler consensus assembly

## Limitations

- Barcode/primer matching is edit-distance-based (defaults -e 4 for barcodes, -E 11 for primers); reads whose primer exceeds 11 errors (signaled by lowercase h and -1) still bin but with a primer miss
- Duplicate sample rows in the barcode file are treated as errors unless -w is supplied
- Barcodes/primer hits depend on matches within the first/last 80 bases (-l default); reads with long low-quality leaders may fail identification

## Evidence

- [methods] Trimming barcodes and primers from sequence ends (-T): "For downstream processing, you'll probably want to trim off the barcode and primer part of the sequence. This is done for you if you supply the -T trim option"
- [methods] Binning samples into individual files: "Typically you'll want to group your records by identified sample. This can be done by using the -F option, optionally with -P to use a file prefix other than the "sample_" default."
- [readme] Sample ID appended to headers: "Sample ID is placed at end of header comment with match hit info before it."
- [readme] Edit distance defaults: "-e <int> barcode edit distance value, overrides -p (4)
        -E <int> primer edit distance value (11)
        -l <int> length to search for index and primer at start and end of sequence (80)"
- [discussion] Edlib dependency: "Other requirements: Edlib library module (https://github.com/Martinsos/edlib)"
