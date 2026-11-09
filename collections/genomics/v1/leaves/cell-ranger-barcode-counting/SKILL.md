---
name: cell-ranger-barcode-counting
description: Use when the input is a 10x Genomics short-read BAM (or FASTQs that will
  be processed with Cell Ranger) from a hybrid LR-SR scRNA-seq experiment and you
  need the top SR cellular barcodes that cover most reads.
when_to_use_negative:
- 'The data are long-read-only (no SR BAM available): use the extract_sr_bc_from_lr
  alternative, which builds a whitelist from exact 10x barcode matches in LR segments
  instead'
- 'The scRNA-seq data are not from 10x Genomics: scTagger in its current form is specific
  for 10x Genomics scRNA-seq unless the LR template contains a fixed sequence next
  to the barcode'
- 'The SR barcode set is already a finalized whitelist (e.g. a small curated barcode
  list): re-selecting with the t=0.5% incremental stopping rule would re-filter an
  already-final set'
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3308
tools:
- name: scTagger
  role: Provides the extract_sr_bc module that consumes the Cell Ranger SR BAM and
    selects the top short-read barcodes covering most reads, with defaults --thresh
    0.005, --step-size 1000, --max-barcode-cnt 25000
  repo: https://github.com/vpc-ccg/scTagger
- name: 10x Genomics Chromium
  role: Source platform for the short-read data; Cell Ranger performs the upstream
    genomic mapping, filtering, barcode counting, and UMI counting that yields the
    BAM
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2022.104530
    title: Fast and accurate matching of cellular barcodes across short-reads and
      long-reads of single-cell RNA-seq experiments
  source_content_hash: sha256:fb16a1f015bca37800b84ab4d352281b1b809ddceb05fe7c8773b9a0a7ff0e3c
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# cell-ranger-barcode-counting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A scTagger stage-1 skill: run 10x Genomics Cell Ranger on trimmed 10x Chromium short reads (mapping, filtering, barcode counting, UMI counting), then apply extract_sr_bc to select the top short-read (SR) barcodes that cover most reads, stopping when an increment of 1,000 barcodes adds less than t=0.5% of total SRs.

## When to use

Use when the input is a 10x Genomics short-read BAM (or FASTQs that will be processed with Cell Ranger) from a hybrid LR-SR scRNA-seq experiment and you need the top SR cellular barcodes that cover most reads.

## When NOT to use

- The data are long-read-only (no SR BAM available): use the extract_sr_bc_from_lr alternative, which builds a whitelist from exact 10x barcode matches in LR segments instead
- The scRNA-seq data are not from 10x Genomics: scTagger in its current form is specific for 10x Genomics scRNA-seq unless the LR template contains a fixed sequence next to the barcode
- The SR barcode set is already a finalized whitelist (e.g. a small curated barcode list): re-selecting with the t=0.5% incremental stopping rule would re-filter an already-final set

## Inputs

- Short-read FASTQ files from a 10x Genomics Chromium hybrid scRNA-seq library (e.g. trimmed real datasets from Figshare 10.6084/m9.figshare.19735678.v1)
- SR BAM file produced by 10x Genomics Cell Ranger (input to extract_sr_bc)
- Optional: 10x Genomics cellular barcode whitelist (e.g. 3M-february-2018.txt.gz)

## Outputs

- TSV of selected SR barcodes (first column: barcode; second column: number of appearances of the barcode)
- Cumulative plot of SR coverage in batches of 1,000 barcodes (via the -p option)
- Per-dataset statistics: number of selected barcodes, % SRs covered, % SRs lost to selection

## How to apply

Run 10x Genomics Cell Ranger on the trimmed short-read FASTQs (e.g. from the Figshare deposit 10.6084/m9.figshare.19735678.v1) to perform genomic mapping, filtering, barcode counting, and UMI counting, producing the SR BAM. Feed that BAM to scTagger's extract_sr_bc module, which sorts barcodes by SR frequency and retains barcodes in batches of 1,000 until the summed coverage of a batch falls below the default t=0.5% threshold (--thresh 0.005) or the default cap of 25,000 barcodes (--max-barcode-cnt) is reached. The rationale is to keep low-frequency, error-prone barcodes out while retaining the barcode set that explains most SR throughput. Record the number of selected barcodes, the percentage of SRs covered, and the percentage of SRs lost to selection; on the paper's real datasets this selection retained 10,000 (N), 14,000 (NOA1), and 7,000 (NOA2) barcodes, covering 80%, 55%, and 85% of SRs respectively, with 9-17% SR loss. The selected barcode list is then passed to scTagger's match_trie stage together with extracted LR segments.

## Related tools

- **scTagger** (Provides the extract_sr_bc module that consumes the Cell Ranger SR BAM and selects the top short-read barcodes covering most reads, with defaults --thresh 0.005, --step-size 1000, --max-barcode-cnt 25000) — https://github.com/vpc-ccg/scTagger
- **10x Genomics Chromium** (Source platform for the short-read data; Cell Ranger performs the upstream genomic mapping, filtering, barcode counting, and UMI counting that yields the BAM)

## Examples

```
./scTagger.py extract_sr_bc -i "path/to/bam/file" -o "path/to/output/file" -p "path/to/output/plot"
```

## Evaluation signals

- The output TSV must contain barcodes sorted with per-barcode appearance counts, and selection must stop when a 1,000-barcode increment contributes less than t=0.5% of total SRs
- On the paper's real datasets the selected barcode counts should fall near Table 2 values (N: 10,000; NOA1: 14,000; NOA2: 7,000)
- Coverage of SRs by the selected barcodes should approach Table 2 values (N: 80%; NOA1: 55%; NOA2: 85%), with SR loss between 9% and 17%
- The -p option should produce a cumulative SR coverage plot in batches of 1,000 barcodes whose curve plateaus at the stopping point
- The selected barcode count must not exceed --max-barcode-cnt (default 25,000)

## Limitations

- Selection necessarily loses some SR throughput (9-17% on the paper's real datasets), so barcodes carrying those reads are excluded from LR-SR matching
- scTagger's design is specific to 10x Genomics scRNA-seq and not expected to scale well if the number of relevant barcodes greatly exceeds thousands to tens of thousands
- Results depend on Cell Ranger preprocessing quality; this skill does not handle UMI matching between SR and LR data (planned future work)

## Evidence

- [discussion] Cell Ranger performs the full SR preprocessing used here: "scTagger uses 10x Genomics Cell Ranger (Zheng et al., 2017) that performs genomic mapping, filtering, barcode counting, and UMI counting"
- [methods] extract_sr_bc extracts top SR barcodes: "The second step is to extract the top short-reads barcodes that cover most of the reads."
- [discussion] Default stopping threshold for barcode retention: "if the sum of SRs retained as a result of retaining the current 1,000 barcode increment represents less than t% (by default t=0.5) of the total number of SRs, we stop retaining barcodes"
- [readme] extract_sr_bc default parameters documented in the README: "* `--thresh`: Percentage theshold required per step to continue adding read barcodes (Optional, Default: 0.005)"
- [readme] Expected output includes a cumulative coverage plot: "A cumulative plot of SR coverage with batches of 1,000 barcodes"
