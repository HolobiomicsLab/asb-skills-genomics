---
name: cell-barcode-demultiplexing
description: Use when raw 10x-structured nanopore or PacBio long-read FASTQ reads
  need cell barcodes found, error-corrected against the 10x v3 whitelist, and assigned
  per read before alignment/quantification.
when_to_use_negative:
- Input reads are short-read (Illumina) data — scywalker is designed for Oxford nanopore
  long reads, with PacBio supported only via appropriate options
- Reads lack 10x v3 structure (no CTACACGACGCTCTTCCGATCT adapter with adjacent 16-bp
  barcode + 12-bp UMI), so whitelist scoring and UMI extraction would be invalid
- Data is already demultiplexed (barcode-corrected FASTQ/BAM available), making the
  droplet-discovery step redundant
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scywalker
  role: 'Implements the droplet barcoding module: whitelist scoring, iterative edit-distance-1
    barcode absorption, and corrected FASTQ generation'
  repo: https://github.com/derijkp/scywalker
- name: genomecomb
  role: Framework in which scywalker is largely implemented; its cg/sw executables
    and tsv-query tools support setup and downstream inspection of demultiplexing
    outputs
  repo: https://github.com/derijkp/genomecomb
provenance:
  source_task_ids:
  - task_002
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae549
    title: 'Scywalker: scalable end-to-end data analysis workflow for long-read single-cell
      transcriptome sequencing'
  source_content_hash: sha256:163f02701b5da0eae228f536567f525890c25c28e8f99f4520d7ffeae217c86a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/derijkp/scywalker
  tools:
  - genomecomb
  - scywalker
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cell-barcode-demultiplexing

## Summary

scywalker's droplet barcoding module (Module 1) detects 10x droplet cell barcodes directly from raw nanopore (or PacBio) FASTQ reads — extracting the 16-bp barcode against the 10x v3 whitelist, absorbing single-mismatch barcodes, and emitting a corrected FASTQ with barcode/UMI prepended — without needing matched short-read data.

## When to use

Use when raw 10x-structured nanopore or PacBio long-read FASTQ reads need cell barcodes found, error-corrected against the 10x v3 whitelist, and assigned per read before alignment/quantification.

## When NOT to use

- Input reads are short-read (Illumina) data — scywalker is designed for Oxford nanopore long reads, with PacBio supported only via appropriate options
- Reads lack 10x v3 structure (no CTACACGACGCTCTTCCGATCT adapter with adjacent 16-bp barcode + 12-bp UMI), so whitelist scoring and UMI extraction would be invalid
- Data is already demultiplexed (barcode-corrected FASTQ/BAM available), making the droplet-discovery step redundant

## Inputs

- Raw Oxford nanopore (or PacBio) FASTQ of 10x-structured single-cell cDNA (e.g. GSE154870 human brain dataset)
- 10x v3 barcode whitelist file (737K-august-2016.txt.gz, bundled in the scywalker distribution)

## Outputs

- Demultiplexed FASTQ (scywalker demultiplexing step) with the corrected droplet barcode and UMI prepended before the read name and added as FASTQ comments
- List of called droplet barcodes (with read counts) from the iterative absorption loop

## How to apply

Extract the 16-bp cell barcode and 12-bp UMI from each read using the 10x v3 adapter sequence CTACACGACGCTCTTCCGATCT, scoring barcodes against the 737K-august-2016 whitelist (supplied via -sc_whitelist). Merge per-read barcode files and sort by total read count so the most abundant barcodes are considered first. Iteratively take the highest-count remaining barcode as a droplet barcode and absorb all barcodes within edit-distance cut-off 1 using cost-only dynamic programming (run as 50 precomputed parallel batches). Stop discovery when the next candidate falls below 20 reads or when 100,000 droplets have been called. Emit a demultiplexed FASTQ in which each read name/comment carries the assigned corrected droplet barcode and UMI, prepended before the read name.

## Related tools

- **scywalker** (Implements the droplet barcoding module: whitelist scoring, iterative edit-distance-1 barcode absorption, and corrected FASTQ generation) — https://github.com/derijkp/scywalker
- **genomecomb** (Framework in which scywalker is largely implemented; its cg/sw executables and tsv-query tools support setup and downstream inspection of demultiplexing outputs) — https://github.com/derijkp/genomecomb

## Examples

```
scywalker -v 1 -d 8 -dmaxmem 64G -refdir g17 -sc_expectedcells 183 -threads 6 test10x
```

## Evaluation signals

- demultiplexed_reads.fastq exists and is well-formed FASTQ (4 lines per record)
- Every prepended barcode in the demultiplexed FASTQ belongs to the set of called droplet barcodes
- Read count in the demultiplexed FASTQ reconciles with the input FASTQ read count
- Number of called droplets does not exceed the 100,000-droplet cap, and each absorbed barcode differs by at most 1 nucleotide from its assigned droplet barcode
- Downstream emptydrops filtering (using -sc_expectedcells) yields is_cell=1 cells consistent with the demultiplexed barcode read counts

## Limitations

- Barcodes with fewer than the default 20 reads are never absorbed, so very low-count droplets are discarded
- Droplet discovery stops at a default maximum of 100,000 droplets, which caps throughput for unusually heterogeneous samples
- Barcode scoring is tied to the 10x v3 whitelist (737K-august-2016) and adapter structure; non-10x protocols require reconfiguration

## Evidence

- [abstract] We developed novel scalable methods for cell barcode demultiplexing and single-cell isoform calling and quantification and incorporated these in an easily deployable package.: "We developed novel scalable methods for cell barcode demultiplexing and single-cell isoform calling and quantification and incorporated these in an easily deployable package."
- [intro] it will find and assign cellbarcodes, align reads, and reconstruct and quantify isoforms and genes, producing both bulk and per cell counts.: "It provides end-to-end analysis in one command: Starting from fastqs, it will find and assign cellbarcodes, align reads, and reconstruct"
- [methods] -sc_whitelist ~/bin/scywalker-0.112.0-linux-x86_64/whitelists/737K-august-2016.txt.gz: "-sc_whitelist ~/bin/scywalker-0.112.0-linux-x86_64/whitelists/737K-august-2016.txt.gz"
- [readme] scywalker is a package designed to analyse single cell (10x) Oxford nanopore long read data (without the need for matching short read data), but it can also analyse PacBio data with the appropriate options.: "scywalker is a package designed to analyse single cell (10x) Oxford
nanopore long read data (without the need for matching short read data),
but it can also analyse PacBio data with the appropriate"
