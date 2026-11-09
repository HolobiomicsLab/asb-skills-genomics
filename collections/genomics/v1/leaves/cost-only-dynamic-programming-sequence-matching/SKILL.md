---
name: cost-only-dynamic-programming-sequence-matching
description: Use when correcting cell/droplet barcodes in 10x-structured nanopore
  FASTQ reads against an abundant-barcode candidate set, where single-nucleotide mismatches
  must be absorbed at scale.
when_to_use_negative:
- Input FASTQ lacks 10x-style 16-bp barcode + 12-bp UMI structure detectable via the
  10x v3 adapter CTACACGACGCTCTTCCGATCT
- Barcodes need multi-mismatch correction (edit distance > 1); the DP matching here
  is configured with a cut-off of 1
edam_operation: http://edamontology.org/operation_3187
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: scywalker
  role: Implements the droplet barcode detection and edit-distance-1 absorption of
    barcodes via cost-only dynamic programming, then writes demultiplexed FASTQs
  repo: https://github.com/derijkp/scywalker
- name: genomecomb
  role: Framework in which scywalker (and its barcode demultiplexing algorithm) is
    largely implemented; provides the cg/sw executables and dependency bundling
  repo: https://github.com/derijkp/genomecomb
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae549
    title: 'Scywalker: scalable end-to-end data analysis workflow for long-read single-cell
      transcriptome sequencing'
  source_content_hash: sha256:70ed80fff6dee14e59a85fe483d5427fa86ada0fbf3fe65574778d682a777149
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/derijkp/scywalker
  tools:
  - genomecomb
  - scywalker
  edam_operation: http://edamontology.org/operation_3187
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cost-only-dynamic-programming-sequence-matching

## Summary

A sequence-matching method used in scywalker's droplet barcoding module: barcodes within an edit-distance cut-off of 1 from a candidate droplet barcode are absorbed using cost-only dynamic programming, enabling scalable single-nucleotide-error barcode correction.

## When to use

Use when correcting cell/droplet barcodes in 10x-structured nanopore FASTQ reads against an abundant-barcode candidate set, where single-nucleotide mismatches must be absorbed at scale.

## When NOT to use

- Input FASTQ lacks 10x-style 16-bp barcode + 12-bp UMI structure detectable via the 10x v3 adapter CTACACGACGCTCTTCCGATCT
- Barcodes need multi-mismatch correction (edit distance > 1); the DP matching here is configured with a cut-off of 1

## Inputs

- Raw nanopore FASTQ from 10x-structured single-cell libraries (e.g. GSE154870)
- 10x v3 barcode whitelist (737K-august-2016.txt.gz)
- 10x v3 adapter sequence CTACACGACGCTCTTCCGATCT for barcode/UMI extraction

## Outputs

- Demultiplexed FASTQ (demultiplexed_reads.fastq) with corrected droplet barcode and UMI prepended before the read name and carried in FASTQ comments
- Set of called droplet barcodes (up to 100,000, each supported by >= 20 reads)

## How to apply

Extract the 16-bp cell barcode and 12-bp UMI from each read using the 10x v3 adapter sequence CTACACGACGCTCTTCCGATCT, scoring barcodes against the 10x v3 whitelist (737K-august-2016). Merge per-read barcode files and sort by total read count so the most abundant barcodes are considered first. Iteratively take the highest-count remaining barcode as a correct droplet barcode, then absorb barcodes differing by one nucleotide via cost-only dynamic programming with a cut-off of 1, running 50 precomputed parallel batches for scalability. Stop the loop when the next candidate falls below the default 20-read minimum or when 100,000 droplets have been called, then emit FASTQs with the corrected droplet barcode and UMI prepended to the read name/comment.

## Related tools

- **scywalker** (Implements the droplet barcode detection and edit-distance-1 absorption of barcodes via cost-only dynamic programming, then writes demultiplexed FASTQs) — https://github.com/derijkp/scywalker
- **genomecomb** (Framework in which scywalker (and its barcode demultiplexing algorithm) is largely implemented; provides the cg/sw executables and dependency bundling) — https://github.com/derijkp/genomecomb

## Examples

```
scywalker -v 1 -d 8 -dmaxmem 64G -refdir g17 -sc_whitelist ~/bin/scywalker-0.112.0-linux-x86_64/whitelists/737K-august-2016.txt.gz -sc_expectedcells 183 -threads 6 test10x
```

## Evaluation signals

- demultiplexed_reads.fastq exists and parses as well-formed FASTQ
- Every prepended barcode in the output FASTQ is among the called droplet barcodes
- Read count in the demultiplexed FASTQ reconciles with the count in the input FASTQ
- Droplet discovery stopped with fewer than the default 20 reads remaining among candidates, or after at most 100,000 droplets were called
- Each absorbed barcode differs from its assigned droplet barcode by exactly one nucleotide (edit-distance cut-off 1)

## Limitations

- Correction is limited to single-nucleotide differences (cost-only DP cut-off of 1); barcodes with >= 2 errors are not absorbed
- Droplet calling halts after a default maximum of 100,000 droplets or when candidates drop below the default 20-read threshold, so very-low-support true cells may be missed

## Evidence

- [other] Module 1 merges all barcode files, summarizes and sorts them by read count, takes the top barcode as a correct droplet barcode, and absorbs barcodes differing by one nucleotide via cost-only dynamic programming (cut-off 1): [evidence span withheld — anchor status: not_found]
- [abstract] We developed novel scalable methods for cell barcode demultiplexing and single-cell isoform calling and quantification and incorporated these in an easily deployable package.: "We developed novel scalable methods for cell barcode demultiplexing and single-cell isoform calling and quantification"
- [readme] it will find and assign cellbarcodes, align reads, and reconstruct and quantify isoforms and genes, producing both bulk and per cell counts: "it
will find and assign cellbarcodes, align reads, and reconstruct (based on
[IsoQuant](https://github.com/ablab/IsoQuant)) and quantify isoforms and
genes, producing both bulk and per cell counts."
- [intro] Starting from fastqs, it will find and assign cellbarcodes: "It provides end-to-end analysis in one command: Starting from fastqs, it will find and assign cellbarcodes, align reads, and reconstruct"
