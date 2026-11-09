---
name: dual-index-amplicon-demultiplexing-context
description: Use when raw MinION FASTQ/FASTA amplicon reads carry dual barcode indexes
  and primers at both ends and must be assigned to samples, trimmed, and quality/length
  filtered before consensus calling.
when_to_use_negative:
- Input reads are already demultiplexed per sample or lack dual barcode/primer structure
  at sequence ends
- Data are short-read Illumina reads rather than MinION long amplicon sequences —
  the pipeline targets multi-kilobase rDNA amplicons and a >3 kb length threshold
- Input is already a consensus or filtered sequence set rather than raw indexed Nanopore
  reads
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: MiniBar (minibar.py)
  role: Identifies reads by dual barcode indexes and primers, bins them into per-sample
    files (-F), and trims barcodes/primers (-T)
  repo: https://github.com/calacademy-research/minibar
- name: NanoFilt
  role: Filters demultiplexed per-sample reads by minimum average Phred quality (-q
    13) and minimum length (-l 3000)
- name: Edlib
  role: Python library minibar.py imports to compute edit distances for barcode and
    primer matching
  repo: https://github.com/Martinsos/edlib
- name: Python 2.7
  role: Runtime for minibar.py (compatible with Python 3) and for counting fastq records
    in filtered files
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giz006
    title: Nanopore sequencing of long ribosomal DNA amplicons enables portable and
      simple biodiversity assessments with high phylogenetic resolution across broad
      taxonomic scale
  source_content_hash: sha256:b547c6279334c91243dfe495b103543935ace3a4116afdbbc9672c1eda6352a7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/calacademy-research/minibar
  tools:
  - Edlib
  - MiniBar
  - MiniBar (minibar.py)
  - NanoFilt
  - Python
  - Python 2.7
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# dual-index-amplicon-demultiplexing-context

## Summary

Demultiplex dual-indexed MinION Nanopore amplicon reads (e.g., long rDNA barcodes) with minibar.py using dual barcode indexes and primers, then quality/length filter the per-sample files with NanoFilt (Phred >13, >3 kb) so downstream rDNA consensus analysis starts from clean, correctly assigned reads.

## When to use

Use when raw MinION FASTQ/FASTA amplicon reads carry dual barcode indexes and primers at both ends and must be assigned to samples, trimmed, and quality/length filtered before consensus calling.

## When NOT to use

- Input reads are already demultiplexed per sample or lack dual barcode/primer structure at sequence ends
- Data are short-read Illumina reads rather than MinION long amplicon sequences — the pipeline targets multi-kilobase rDNA amplicons and a >3 kb length threshold
- Input is already a consensus or filtered sequence set rather than raw indexed Nanopore reads

## Inputs

- MinION FASTQ or FASTA file of dual-indexed long rDNA amplicon reads (plain or gzipped)
- Barcode file mapping sample IDs to forward/reverse index and primer columns (e.g., Demultiplex.txt, MinIndexCombinationPeperomonia.txt)
- Per-sample demultiplexed fastq files from minibar.py -F binning

## Outputs

- Per-sample FASTQ/FASTA files of identified reads with sample ID and hit info appended to headers (e.g., PeperomiaTestSet_SampleIDs.fa, sample_<ID> files)
- Unassigned reads file (*_unk)
- Quality- and length-filtered per-sample fastq files (Phred >13, length >3 kb)
- Read counts summed across samples (~245,433 filtered reads expected for the main Nanopore run)

## How to apply

Run minibar.py with a barcode combination file (e.g., MinIndexCombinationPeperomonia.txt) and the MinION sequence file, using defaults of barcode edit distance 4, primer edit distance 11, search length 80, and matching Method 3; supply -F to bin records into individual sample files (optional -P prefix) and -T to trim barcode and primer sequence from each read end. Reads matching a sample get the sample ID appended to the header with hit info such as H+(2,5),h-(3,-1) Jun_38; unmatched reads go to the *_unk file. Then pipe each per-sample demultiplexed FASTQ through NanoFilt with -q 13 (minimum average Phred quality 13) and -l 3000 (minimum read length 3000 bp, matching the multi-kilobase rDNA amplicon) to discard low-quality and short reads. Finally count fastq records in the filtered files (e.g., with a short Python script) and sum them across samples.

## Related tools

- **MiniBar (minibar.py)** (Identifies reads by dual barcode indexes and primers, bins them into per-sample files (-F), and trims barcodes/primers (-T)) — https://github.com/calacademy-research/minibar
- **NanoFilt** (Filters demultiplexed per-sample reads by minimum average Phred quality (-q 13) and minimum length (-l 3000))
- **Edlib** (Python library minibar.py imports to compute edit distances for barcode and primer matching) — https://github.com/Martinsos/edlib
- **Python 2.7** (Runtime for minibar.py (compatible with Python 3) and for counting fastq records in filtered files)

## Examples

```
./minibar.py -T -F Demultiplex.txt my_sequences.fastq
```

## Evaluation signals

- Minibar run summary line reports expected structure, e.g. '750 seqs: H 750 HH 679 Hh 62 hh 0 IDs 741 Mult_IDs 0 (0.1245s)' with Mult_IDs 0
- Record headers carry sample ID with hit info such as 'H-(0,6),H+(0,2) Jun_40'; lowercase h and -1 flag primer misses
- Per-sample output files exist for each identified sample plus an *_unk file for unassigned reads
- Summed read count over quality-filtered per-sample files matches the reported ~245,433 reads for the main run
- Trimmed (-T) reads no longer contain the barcode/primer segments at sequence ends

## Limitations

- Reads whose primer match exceeds the allowed edit distance (default 11) are flagged with lowercase h and -1 in the header, indicating an incomplete barcode/primer hit
- Unassigned reads are not recoverable by sample and accumulate in the *_unk file
- Demultiplexing quality depends on barcode file column layout; -cols defaults vary (1,2,3,4,5 for 5 cols; 1,3,4,5,6 for 6 cols; etc.) and must match the file format

## Evidence

- [readme] README usage example: "./minibar.py -T -F Demultiplex.txt my_sequences.fastq"
- [readme] Binning option description: "Typically you'll want to group your records by identified sample. This can be done by using the -F option, optionally with -P"
- [readme] Trimming behavior: "-T trims barcode and primer from each end of the sequence, then outputs record"
- [methods] Article methods description: "Identify MinION sequence by dual barcode indexes and primers."
- [methods] Downstream trimming rationale: "For downstream processing, you'll probably want to trim off the barcode and primer part of the sequence."
- [other] Source card finding on NanoFilt thresholds and read yield: [evidence span withheld — anchor status: not_found]
