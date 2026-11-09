---
name: umi-and-barcode-extraction-from-nanopore-reads
description: Use when raw 10x-structured Oxford nanopore FASTQ must be demultiplexed
  by cell barcode and UMI before read alignment or isoform/gene reconstruction, and
  no matching short-read data is available.
when_to_use_negative:
- Input reads lack the 10x structure (no CTACACGACGCTCTTCCGATCT-flanked 16-bp barcode
  + 12-bp UMI), e.g. user-specified non-10x protocols or PacBio data without the appropriate
  options
- Matching short-read data is available and already provides demultiplexed cell barcodes
  (scywalker targets analysis 'without the need for matching short read data')
- Reads are already demultiplexed FASTQs with barcode/UMI in read names — proceed
  directly to alignment/isoform calling instead
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0196
tools:
- name: scywalker
  role: 'Executes the droplet barcoding module: adapter-guided extraction of 16-bp
    barcode and 12-bp UMI, whitelist scoring, and iterative droplet barcode calling
    with edit-distance-1 correction'
  repo: https://github.com/derijkp/scywalker
- name: genomecomb
  role: Framework in which scywalker's demultiplexing algorithms are implemented;
    provides the cg/sw executables and tsv utilities
  repo: https://github.com/derijkp/genomecomb
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae549
    title: 'Scywalker: scalable end-to-end data analysis workflow for long-read single-cell
      transcriptome sequencing'
  source_content_hash: sha256:f23eff534739eef047528427ef1691031ad0b1a65a8a4e6b3eefde0fe0d0387d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/derijkp/scywalker
  tools:
  - genomecomb
  - scywalker
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# umi-and-barcode-extraction-from-nanopore-reads

## Summary

Scywalker's droplet barcoding module extracts the 16-bp cell barcode and 12-bp UMI from 10x-structured Oxford nanopore FASTQ reads using the 10x v3 adapter (CTACACGACGCTCTTCCGATCT), then calls droplet barcodes by iterative top-count selection with edit-distance-1 correction against the 10x 737K-august-2016 whitelist. It emits a demultiplexed FASTQ in which the corrected barcode and UMI are carried with each read for downstream single-cell isoform analysis.

## When to use

Use when raw 10x-structured Oxford nanopore FASTQ must be demultiplexed by cell barcode and UMI before read alignment or isoform/gene reconstruction, and no matching short-read data is available.

## When NOT to use

- Input reads lack the 10x structure (no CTACACGACGCTCTTCCGATCT-flanked 16-bp barcode + 12-bp UMI), e.g. user-specified non-10x protocols or PacBio data without the appropriate options
- Matching short-read data is available and already provides demultiplexed cell barcodes (scywalker targets analysis 'without the need for matching short read data')
- Reads are already demultiplexed FASTQs with barcode/UMI in read names — proceed directly to alignment/isoform calling instead

## Inputs

- Raw Oxford nanopore FASTQ files of full-length single-cell/single-nuclei cDNA in 10x v3 structure (16-bp cell barcode + 12-bp UMI preceded by adapter CTACACGACGCTCTTCCGATCT), placed in a sample/project 'fastq' subdirectory
- 10x v3 barcode whitelist 737K-august-2016.txt.gz (bundled with scywalker 0.112.0)

## Outputs

- Demultiplexed FASTQ (e.g. demultiplexed_reads.fastq) with corrected droplet barcode and UMI prepended to read names and included as FASTQ comments
- Called droplet barcode set (up to 100,000 droplets by default) used for subsequent cell-level counting and emptydrops filtering

## How to apply

Run scywalker (implemented within genomecomb) on a sample or project directory whose fastq subdirectory holds the raw nanopore FASTQ. Scywalker extracts the 16-bp cell barcode and 12-bp UMI from each read using the 10x v3 adapter sequence CTACACGACGCTCTTCCGATCT and scores barcodes against the 10x v3 whitelist (737K-august-2016, shipped with the distribution and given via -sc_whitelist). Per-read barcode files are merged and sorted by total read count, then the highest-count remaining barcode is taken as a correct droplet barcode; barcodes differing by one nucleotide are absorbed via cost-only dynamic programming (edit-distance cut-off 1), running 50 precomputed parallel batches. The loop stops when the next candidate falls below 20 reads or 100,000 droplets have been called (both defaults tunable). The demultiplexing step (e.g. demultiplexed_reads.fastq) prepends the corrected droplet barcode and UMI before each read name and adds them as FASTQ comments, ready for alignment with minimap2_splice and IsoQuant-based isoform reconstruction.

## Related tools

- **scywalker** (Executes the droplet barcoding module: adapter-guided extraction of 16-bp barcode and 12-bp UMI, whitelist scoring, and iterative droplet barcode calling with edit-distance-1 correction) — https://github.com/derijkp/scywalker
- **genomecomb** (Framework in which scywalker's demultiplexing algorithms are implemented; provides the cg/sw executables and tsv utilities) — https://github.com/derijkp/genomecomb

## Examples

```
scywalker -v 1 -d 8 -dmaxmem 64G -refdir g17 -sc_expectedcells 183 -cellmarkerfile markers_chr17.tsv -threads 6 test10x
```

## Evaluation signals

- demultiplexed_reads.fastq exists and is well-formed FASTQ (4 lines per record)
- The barcode prepended to each read name/comment is among the called droplet barcodes
- Read count of the demultiplexed FASTQ reconciles with the input FASTQ
- Called droplets respect the stopping rule: candidates below 20 reads are not taken once the loop stops, and at most 100,000 droplets are called
- Corrected barcodes should all match the 10x v3 737K-august-2016 whitelist (mismatches collapsed to their edit-distance-1 correct barcode)

## Limitations

- Barcode discovery assumes roughly 10x chemistry structure; the 10x v3 adapter, 16-bp barcode, and 12-bp UMI lengths are hard assumptions of this module
- Single-mismatch absorption (edit-distance cut-off 1) may misassign barcode sequences that differ from true droplets by one nucleotide
- IsoQuant-based downstream code can hang or crash on organelle chromosomes, so organelles must be excluded/flagged in the reference directory (not specific to demultiplexing but relevant to the scywalker run)

## Evidence

- [abstract] We developed novel scalable methods for cell barcode demultiplexing and single-cell isoform calling and quantification: "We developed novel scalable methods for cell barcode demultiplexing and single-cell isoform calling and quantification and incorporated these in an easily deployable package."
- [intro] Starting from fastqs, it will find and assign cellbarcodes: "It provides end-to-end analysis in one command: Starting from fastqs, it will find and assign cellbarcodes, align reads, and reconstruct"
- [methods] -sc_whitelist 737K-august-2016.txt.gz: "-sc_whitelist ~/bin/scywalker-0.112.0-linux-x86_64/whitelists/737K-august-2016.txt.gz"
- [readme] scywalker analyses single cell (10x) Oxford nanopore long read data: "scywalker is a package designed to analyse single cell (10x) Oxford"
