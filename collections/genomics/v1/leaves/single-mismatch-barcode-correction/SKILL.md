---
name: single-mismatch-barcode-correction
description: Use when starting from raw 10x Oxford nanopore FASTQ reads that need
  droplet/cell barcode demultiplexing, i.e. 16-bp cell barcodes and 12-bp UMIs must
  be extracted, corrected against the 737K-august-2016 whitelist, and reads assigned
  to droplets.
when_to_use_negative:
- Input FASTQs lack 10x adapter structure (no CTACACGACGCTCTTCCGATCT-flanked 16-bp
  barcode/12-bp UMI) or use non-10x barcode chemistries without appropriate whitelist
  and options
- Cell barcode demultiplexing is already done upstream (e.g. reads already carry corrected
  barcodes/UMIs and only downstream alignment/isoform analysis is needed)
- The goal is short-read or matched short-read-based cell calling rather than direct
  analysis of 10x nanopore long-read data without matching short reads
edam_operation: http://edamontology.org/operation_3200
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: scywalker
  role: 'Implements the droplet barcoding module: barcode extraction, whitelist scoring,
    iterative top-barcode selection with single-mismatch absorption via cost-only
    DP, and demultiplexed FASTQ generation'
  repo: https://github.com/derijkp/scywalker
- name: genomecomb
  role: Framework within which scywalker (including the barcode correction algorithm)
    is largely implemented; distributed with scywalker and provides the cg/sw tooling
  repo: https://github.com/derijkp/genomecomb
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae549
    title: 'Scywalker: scalable end-to-end data analysis workflow for long-read single-cell
      transcriptome sequencing'
  source_content_hash: sha256:5ced8dfd268e36358a853412b9d355943ffca6c7a71cc9989c5f558e3bebd680
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/derijkp/scywalker
  tools:
  - genomecomb
  - scywalker
  edam_operation: http://edamontology.org/operation_3200
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# single-mismatch-barcode-correction

## Summary

scywalker's droplet barcoding algorithm (Module 1) detects true 10x droplet barcodes from 10x-structured nanopore FASTQ reads by iteratively picking the most abundant whitelist-scored barcode as a correct droplet barcode and absorbing all barcodes within an edit-distance cut-off of 1 via cost-only dynamic programming.

## When to use

Use when starting from raw 10x Oxford nanopore FASTQ reads that need droplet/cell barcode demultiplexing, i.e. 16-bp cell barcodes and 12-bp UMIs must be extracted, corrected against the 737K-august-2016 whitelist, and reads assigned to droplets.

## When NOT to use

- Input FASTQs lack 10x adapter structure (no CTACACGACGCTCTTCCGATCT-flanked 16-bp barcode/12-bp UMI) or use non-10x barcode chemistries without appropriate whitelist and options
- Cell barcode demultiplexing is already done upstream (e.g. reads already carry corrected barcodes/UMIs and only downstream alignment/isoform analysis is needed)
- The goal is short-read or matched short-read-based cell calling rather than direct analysis of 10x nanopore long-read data without matching short reads

## Inputs

- Raw 10x-structured Oxford nanopore FASTQ file(s) placed in a sample directory's fastq subdirectory
- 10x v3 barcode whitelist file (e.g. whitelists/737K-august-2016.txt.gz, supplied via -sc_whitelist)
- 10x v3 adapter sequence CTACACGACGCTCTTCCGATCT used to locate the 16-bp cell barcode and 12-bp UMI

## Outputs

- Demultiplexed/corrected FASTQ (demultiplexed_reads.fastq) with the corrected droplet barcode and UMI prepended before the read name and added as FASTQ comments
- Called droplet barcode list with per-barcode read counts

## How to apply

Extract the 16-bp cell barcode and 12-bp UMI from each read of the raw nanopore FASTQ (e.g. GSE154870) using the 10x v3 adapter sequence CTACACGACGCTCTTCCGATCT, and score barcodes against the 10x v3 whitelist (737K-august-2016). Merge the per-read barcode files and sort them by total read count so the most abundant barcodes are considered first. Iteratively select the highest-count remaining barcode as a correct droplet barcode and absorb every barcode differing by one nucleotide (edit-distance cut-off 1) via cost-only dynamic programming, executed in 50 precomputed parallel batches for scalability. Stop the discovery loop when the next candidate barcode falls below 20 reads (default) or when 100,000 droplets (default) have been called. Emit a corrected/demultiplexed FASTQ in which each read name/comment carries the assigned droplet barcode and UMI, prepended before the read name.

## Related tools

- **scywalker** (Implements the droplet barcoding module: barcode extraction, whitelist scoring, iterative top-barcode selection with single-mismatch absorption via cost-only DP, and demultiplexed FASTQ generation) — https://github.com/derijkp/scywalker
- **genomecomb** (Framework within which scywalker (including the barcode correction algorithm) is largely implemented; distributed with scywalker and provides the cg/sw tooling) — https://github.com/derijkp/genomecomb

## Examples

```
scywalker -v 1 -d 8 -dmaxmem 64G -sc_whitelist ~/bin/scywalker-0.112.0-linux-x86_64/whitelists/737K-august-2016.txt.gz -sc_expectedcells 183 -threads 6 test10x
```

## Evaluation signals

- demultiplexed_reads.fastq exists and parses as well-formed FASTQ (4 lines per record with '+' header line)
- Every prepended barcode in the demultiplexed FASTQ belongs to the set of called droplet barcodes
- Read count in the demultiplexed FASTQ reconciles with the read count of the input FASTQ
- The droplet discovery loop terminated at the expected boundaries: candidate barcode below 20 reads or 100,000 droplets called
- Corrected barcodes are within edit distance 1 of whitelist barcodes (single-mismatch absorption worked as specified)

## Limitations

- Absorbs only single-mismatch barcodes (edit-distance cut-off 1); barcodes with two or more sequencing errors are not corrected
- Stopping conditions (default 20-read minimum and 100,000 droplet cap) bound the number of droplets called and may miss very rare real cells or over-call when defaults are inappropriate; -sc_expectedcells can steer detection
- Algorithm is designed for 10x Oxford nanopore data; PacBio data requires appropriate options, and non-10x protocols need a user-specified whitelist

## Evidence

- [other] Module 1 merges all barcode files, summarizes and sorts them by read count, takes the top barcode as a correct droplet barcode, and absorbs barcodes differing by one nucleotide via cost-only dynamic programming: [evidence span withheld — anchor status: not_found]
- [other] extract the 16-bp cell barcode and 12-bp UMI from each read using the 10x v3 adapter sequence CTACACGACGCTCTTCCGATCT: [evidence span withheld — anchor status: not_found]
- [abstract] We developed novel scalable methods for cell barcode demultiplexing and single-cell isoform calling and quantification and incorporated these in an easily deployable package.: "We developed novel scalable methods for cell barcode demultiplexing and single-cell isoform calling and quantification"
- [intro] scywalker is a package designed to analyse single cell (10x) Oxford nanopore long read data (without the need for matching short read data): "scywalker is a package designed to analyse single cell (10x) Oxford nanopore long read data (without the need for matching short read data)"
- [intro] it will find and assign cellbarcodes, align reads, and reconstruct: "Starting from fastqs, it will find and assign cellbarcodes, align reads, and reconstruct"
- [methods] -sc_whitelist ~/bin/scywalker-0.112.0-linux-x86_64/whitelists/737K-august-2016.txt.gz: "-sc_whitelist ~/bin/scywalker-0.112.0-linux-x86_64/whitelists/737K-august-2016.txt.gz"
