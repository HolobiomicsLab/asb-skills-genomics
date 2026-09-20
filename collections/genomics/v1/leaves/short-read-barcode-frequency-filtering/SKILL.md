---
name: short-read-barcode-frequency-filtering
description: Use when you have a 10x Genomics short-read BAM (from Cell Ranger mapping/filtering/UMI
  counting) and need a compact whitelist of the most frequent cellular barcodes that
  cover the bulk of short reads before barcode matching to long reads.
when_to_use_negative:
- You want to avoid short reads entirely — use the extract_sr_bc_from_lr module instead,
  which builds a barcode whitelist directly from long-read segments against the 10x
  Genomics barcode whitelist (e.g. 3M-february-2018.txt.gz).
- The experiment is not 10x Genomics scRNA-seq; the article notes scTagger in its
  current form is specific for 10x Genomics scRNA-seq (other platforms only work if
  the LR template has a fixed sequence next to the barcode).
- The number of relevant barcodes is much larger than tens of thousands — the article
  states scTagger's current design is not expected to scale well in that regime.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scTagger
  role: Provides the extract_sr_bc module that performs frequency-sorted, batched
    barcode selection from the short-read BAM
  repo: https://github.com/vpc-ccg/scTagger/
- name: 10x Genomics Chromium
  role: Upstream protocol whose Cell Ranger output BAM (mapped, filtered, UMI-counted)
    is the input for barcode selection
provenance:
  source_task_ids:
  - task_006
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2022.104530
    title: Fast and accurate matching of cellular barcodes across short-reads and
      long-reads of single-cell RNA-seq experiments
  source_content_hash: sha256:5cc3ceb23aabe1b989260cfc1f39f50df378847b637eb0467ab3ec0b40dea36c
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
  - FLAMES
  - Snakemake
  - edlib
  - scTagger
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# short-read-barcode-frequency-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Selects the top short-read cellular barcodes (10x Genomics Chromium) that cumulatively cover most SR reads, stopping when an additional 1,000-barcodes batch adds less than t% (default 0.5%) of total SRs. This is scTagger's extract_sr_bc stage, which builds a tractable SR barcode list for later trie-based matching against LR barcode segments.

## When to use

Use when you have a 10x Genomics short-read BAM (from Cell Ranger mapping/filtering/UMI counting) and need a compact whitelist of the most frequent cellular barcodes that cover the bulk of short reads before barcode matching to long reads.

## When NOT to use

- You want to avoid short reads entirely — use the extract_sr_bc_from_lr module instead, which builds a barcode whitelist directly from long-read segments against the 10x Genomics barcode whitelist (e.g. 3M-february-2018.txt.gz).
- The experiment is not 10x Genomics scRNA-seq; the article notes scTagger in its current form is specific for 10x Genomics scRNA-seq (other platforms only work if the LR template has a fixed sequence next to the barcode).
- The number of relevant barcodes is much larger than tens of thousands — the article states scTagger's current design is not expected to scale well in that regime.

## Inputs

- Short-read BAM file produced by 10x Genomics Cell Ranger (genomic mapping, filtering, barcode counting, UMI counting)
- Selection parameters: --thresh (default 0.005), --step-size (default 1000), --max-barcode-cnt (default 25000)

## Outputs

- TSV file with one barcode per row (column 1: barcode, column 2: number of appearances of the barcode)
- Cumulative plot of SR coverage in batches of 1,000 barcodes

## How to apply

Run scTagger's extract_sr_bc module on the Cell Ranger short-read BAM (./scTagger.py extract_sr_bc -i <bam> -o <out.tsv> -p <plot>). The module sorts barcodes by SR frequency and processes them in batches of --step-size 1000 barcodes; it stops retaining barcodes when the sum of SRs contributed by the current 1,000-barcode increment falls below --thresh 0.005 (t=0.5%) of total SRs, with a hard cap at --max-barcode-cnt 25000. Generate the cumulative SR coverage plot in 1,000-barcode batches via -p to inspect where the retention cutoff lands. From the output TSV compute the number of selected barcodes, the percentage of SRs covered, and the percentage of SRs lost by the selection (paper reports 10,000; 14,000; 7,000 barcodes covering 80%, 55%, 85% of SRs on the N, NOA1, NOA2 datasets, with 9–17% of SR throughput lost). The rationale is to shrink the search space so the subsequent match_trie stage compares each LR segment only against a few thousand high-frequency barcodes instead of all 10x whitelist barcodes.

## Related tools

- **scTagger** (Provides the extract_sr_bc module that performs frequency-sorted, batched barcode selection from the short-read BAM) — https://github.com/vpc-ccg/scTagger/
- **10x Genomics Chromium** (Upstream protocol whose Cell Ranger output BAM (mapped, filtered, UMI-counted) is the input for barcode selection)

## Examples

```
./scTagger.py extract_sr_bc -i "path/to/bam/file" -o "path/to/output/file" -p "path/to/output/plot" --thresh 0.005 --step-size 1000 --max-barcode-cnt 25000
```

## Evaluation signals

- The selected barcode count on the real datasets should fall in the thousands, consistent with Table 2 values (N: 10,000; NOA1: 14,000; NOA2: 7,000 selected barcodes) and never exceed --max-barcode-cnt 25000.
- The cumulative coverage plot should show the retention stopping when a 1,000-barcode batch adds less than 0.5% of total SRs; SR coverage of selected barcodes should land around the reported 80%, 55%, 85% on N, NOA1, NOA2.
- Output TSV schema check: exactly two columns, barcode string in column 1 and appearance count in column 2, sorted by descending frequency.
- Downstream check: the selected list feeds match_trie (-sr argument) and should yield SR coverage loss in the 9–17% range rather than near-total SR loss.

## Limitations

- The selection discards 9–17% of short reads (SRs lost by the stage-1 selection), so low-frequency/error-prone barcodes are excluded by design.
- scTagger in its current form is specific for 10x Genomics scRNA-seq, so this filtering strategy inherits that platform specificity.
- The approach is not expected to scale well if the number of relevant barcodes becomes much bigger than thousands to tens of thousands.

## Evidence
