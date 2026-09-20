---
name: barcode-selection-threshold-tuning
description: Use when preparing a hybrid short-read/long-read scRNA-seq experiment
  (e.g., 10x Chromium split into Illumina SR and ONT LR pools) and you need a compact
  whitelist of top SR barcodes that covers most reads before trie-based LR–SR barcode
  matching.
when_to_use_negative:
- The experiment is not 10x Genomics scRNA-seq — scTagger in its current form is specific
  for 10x Genomics, and other platforms are only supported if the LR template includes
  a fixed sequence next to the barcode
- You want to avoid short reads entirely — use the extract_sr_bc_from_lr alternative
  that builds the whitelist by exact-matching 10x barcodes on long-read segments instead
- The expected number of relevant barcodes is much larger than tens of thousands —
  scTagger's design is not expected to scale well beyond that regime
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: scTagger
  role: Provides the extract_sr_bc module that performs the frequency-sorted, threshold-stopped
    barcode selection and generates the cumulative coverage plot
  repo: https://github.com/vpc-ccg/scTagger/
- name: 10x Genomics Chromium
  role: Source platform of the short-read data; Cell Ranger is used upstream to produce
    the BAM consumed by extract_sr_bc
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2022.104530
    title: Fast and accurate matching of cellular barcodes across short-reads and
      long-reads of single-cell RNA-seq experiments
  source_content_hash: sha256:562598037947aef6a6a7c11de329d5e54335812484968d493b6118cd504d9382
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/vpc-ccg/scTagger
  repo_url: https://github.com/vpc-ccg/scTagger
  tools:
  - 10x Genomics Chromium
  - scTagger
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# barcode-selection-threshold-tuning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Tunes scTagger's stage-1 short-read (SR) barcode selection: iteratively retain the most frequent SR barcodes in batches of 1,000 until an added batch contributes less than t=0.5% of total SR coverage, balancing barcode whitelist size against SR throughput loss (9–17% on the paper's real datasets).

## When to use

Use when preparing a hybrid short-read/long-read scRNA-seq experiment (e.g., 10x Chromium split into Illumina SR and ONT LR pools) and you need a compact whitelist of top SR barcodes that covers most reads before trie-based LR–SR barcode matching.

## When NOT to use

- The experiment is not 10x Genomics scRNA-seq — scTagger in its current form is specific for 10x Genomics, and other platforms are only supported if the LR template includes a fixed sequence next to the barcode
- You want to avoid short reads entirely — use the extract_sr_bc_from_lr alternative that builds the whitelist by exact-matching 10x barcodes on long-read segments instead
- The expected number of relevant barcodes is much larger than tens of thousands — scTagger's design is not expected to scale well beyond that regime

## Inputs

- Short-read BAM file produced by 10x Genomics Cell Ranger from the trimmed short-read FASTQ data
- Optional: 10x Genomics cellular barcode whitelist (e.g., 3M-february-2018.txt.gz) for the LR-only alternative extract_sr_bc_from_lr

## Outputs

- TSV file with barcode in the first column and its number of appearances (read count) in the second column
- Cumulative plot of SR coverage in batches of 1,000 barcodes
- Per-dataset selection statistics (number of selected barcodes, % SRs covered, % SRs lost), e.g., compiled into sr_barcode_selection_stats.csv

## How to apply

Run 10x Genomics Cell Ranger (genomic mapping, filtering, barcode counting, UMI counting) on the trimmed SR data to produce a BAM, then run scTagger's extract_sr_bc module on the BAM. The module sorts SR barcodes by frequency and stops retaining barcodes when the sum of SRs retained by the current 1,000-barcode increment represents less than t% (default t=0.5, i.e. --thresh 0.005) of the total number of SRs, capped at --max-barcode-cnt 25000. Use the -p option to generate a cumulative SR-coverage plot in batches of 1,000 barcodes to inspect where the stopping rule fires. From the output TSV, compute the number of selected barcodes, the percentage of SRs covered by them, and the percentage of SRs lost. For the paper's real datasets (N, NOA1, NOA2) this retained 10,000, 14,000, and 7,000 barcodes covering 80%, 55%, and 85% of SRs respectively, so a result within these ranges indicates the selection behaved as reported.

## Related tools

- **scTagger** (Provides the extract_sr_bc module that performs the frequency-sorted, threshold-stopped barcode selection and generates the cumulative coverage plot) — https://github.com/vpc-ccg/scTagger/
- **10x Genomics Chromium** (Source platform of the short-read data; Cell Ranger is used upstream to produce the BAM consumed by extract_sr_bc)

## Examples

```
./scTagger.py extract_sr_bc -i path/to/bam/file -o path/to/output/file -p path/to/output/plot --thresh 0.005 --step-size 1000 --max-barcode-cnt 25000
```

## Evaluation signals

- Output TSV has exactly two columns: barcode sequence and number of appearances of the barcode
- Number of retained barcodes stays below --max-barcode-cnt 25000 and stops per the threshold rule; on the paper's real datasets it was 10,000 (N), 14,000 (NOA1), and 7,000 (NOA2)
- Fraction of SRs covered by selected barcodes is 80% (N), 55% (NOA1), and 85% (NOA2) per Table 2, and the fraction of SRs lost to selection falls between 9% and 17%
- The last retained 1,000-barcode batch contributed at least t=0.5% of total SRs; the first dropped batch contributes less
- Cumulative coverage plot generated in batches of 1,000 barcodes matches the shape expected from the stopping rule

## Limitations

- The selection inherently discards barcodes: 9% to 17% of SR throughput is lost on the paper's real datasets, and coverage can be as low as 55% (NOA1)
- scTagger is specific to 10x Genomics scRNA-seq; extension to other platforms requires a fixed sequence adjacent to the barcode in the LR template
- The design is not expected to scale well if the number of relevant barcodes becomes much bigger than tens of thousands

## Evidence

- [discussion] if the sum of SRs retained as a result of retaining the current 1,000 barcode increment represents less than t% (by default t=0.5) of the total number of SRs, we stop retaining barcodes: "if the sum of SRs retained as a result of retaining the current 1,000 barcode increment represents less than t% (by default t=0.5) of the total number of SRs"
- [methods] The second step is to extract the top short-reads barcodes that cover most of the reads.: "The second step is to extract the top short-reads barcodes that cover most of the reads."
- [readme] '--thresh': Percentage theshold required per step to continue adding read barcodes (Optional, Default: 0.005): "Percentage theshold required per step to continue adding read barcodes (Optional, Default: 0.005)"
- [readme] A cumulative plot of SR coverage with batches of 1,000 barcodes: "A cumulative plot of SR coverage with batches of 1,000 barcodes"
