---
name: simulated-scrnaseq-dataset-processing
description: Use when evaluating or validating cellular barcode matching accuracy
  for long-read/short-read single-cell RNA-seq tools on simulated LR-SR datasets with
  known composition, e.g. reproducing a Table 3-style accuracy comparison.
when_to_use_negative:
- You need matching on real (non-simulated) LR-SR data without a ground-truth-quality
  baseline — this skill's validation is tied to simulated datasets with Table 3 reference
  values
- The scRNA-seq platform is not 10x Genomics and the long-read template lacks a fixed
  sequence adjacent to the barcode, since scTagger is specific for 10x Genomics in
  its current form
- The number of relevant barcodes is far beyond thousands to tens of thousands, where
  scTagger's trie design is not expected to scale
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3308
tools:
- name: scTagger
  role: Runs the three-stage pipeline (extract_lr_bc, extract_sr_bc, match_trie with
    e=2) to produce LR-SR barcode matches on the simulated datasets
  repo: https://github.com/vpc-ccg/scTagger/
- name: edlib
  role: Provides exact edit-distance computation for the brute-force dynamic-programming
    baseline matching on the same LR segments and selected SR barcodes (v1.3.9)
  repo: https://github.com/Martinsos/edlib
- name: FLAMES
  role: 'Comparison tool producing barcode assignments on the simulated datasets (commit
    #18fb83c)'
  repo: https://github.com/LuyiTian/FLAMES
- name: 10x Genomics Cell Ranger
  role: Performs genomic mapping, filtering, barcode counting, and UMI counting on
    the short reads to supply the BAM used by extract_sr_bc
- name: Badread
  role: Long-read simulator used in the paper's benchmarking context for simulating
    error-prone long reads
  repo: https://github.com/rrwick/Badread
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2022.104530
    title: Fast and accurate matching of cellular barcodes across short-reads and
      long-reads of single-cell RNA-seq experiments
  source_content_hash: sha256:a92063dce7972cb5160ba8a0bca7300629c9ef7497b5da05f862b57358c214da
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
  - 10x Genomics Cell Ranger
  - Badread
  - FLAMES
  - edlib
  - scTagger
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# simulated-scrnaseq-dataset-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark scTagger's barcode matching on simulated hybrid LR-SR scRNA-seq datasets (Small, Medium, Large) against FLAMES and a brute-force edlib dynamic-programming baseline, reproducing the paper's Table 3 match/ambiguous/mismatch/skip percentages.

## When to use

Use when evaluating or validating cellular barcode matching accuracy for long-read/short-read single-cell RNA-seq tools on simulated LR-SR datasets with known composition, e.g. reproducing a Table 3-style accuracy comparison.

## When NOT to use

- You need matching on real (non-simulated) LR-SR data without a ground-truth-quality baseline — this skill's validation is tied to simulated datasets with Table 3 reference values
- The scRNA-seq platform is not 10x Genomics and the long-read template lacks a fixed sequence adjacent to the barcode, since scTagger is specific for 10x Genomics in its current form
- The number of relevant barcodes is far beyond thousands to tens of thousands, where scTagger's trie design is not expected to scale

## Inputs

- Simulated LR-SR scRNA-seq datasets (Small, Medium, Large) from Figshare 10.6084/m9.figshare.19740475.v1
- Long-read FASTQ files of the simulated datasets
- Cell Ranger BAM of the simulated short-read data
- 10x Genomics cellular barcode whitelist (3M-february-2018.txt.gz), if using extract_sr_bc_from_lr

## Outputs

- Long-read barcode segment TSV (read-id, best adapter edit distance, alignment start, 20bp segment) from extract_lr_bc
- Selected top short-read barcodes TSV (barcode, count) from extract_sr_bc
- Barcode match assignments from scTagger match_trie, edlib brute-force, and FLAMES
- CSV table of per-method per-dataset percentages for match, ambiguous, mismatch, and skip categories, validated against Table 3

## How to apply

Download the simulated LR-SR datasets (Small, Medium, Large) from Figshare (DOI 10.6084/m9.figshare.19740475.v1). For each dataset run the scTagger pipeline: extract_lr_bc on the long-read FASTQs (glocal alignment to the SR adapter CTACACGACGCTCTTCCGATCT to extract the 20bp barcode segment), extract_sr_bc on the Cell Ranger BAM (retaining barcodes in 1,000-barcodes increments while each increment adds at least 0.5% of total SRs, max 25,000 barcodes), then match_trie with the default maximum edit distance e=2 and multiple threads. Run a brute-force match of LR barcode segments against the selected SR barcodes with edlib v1.3.9 as the exact edit-distance baseline, and run FLAMES (commit #18fb83c) on the same data. Classify each long read per method as match (unique barcode), ambiguous (multiple barcodes at minimum edit distance), mismatch, or skip, compute per-category percentages, compile into a single CSV, and validate against Table 3 (e.g., Large: scTagger 53.26% matches vs FLAMES 30.05%).

## Related tools

- **scTagger** (Runs the three-stage pipeline (extract_lr_bc, extract_sr_bc, match_trie with e=2) to produce LR-SR barcode matches on the simulated datasets) — https://github.com/vpc-ccg/scTagger/
- **edlib** (Provides exact edit-distance computation for the brute-force dynamic-programming baseline matching on the same LR segments and selected SR barcodes (v1.3.9)) — https://github.com/Martinsos/edlib
- **FLAMES** (Comparison tool producing barcode assignments on the simulated datasets (commit #18fb83c)) — https://github.com/LuyiTian/FLAMES
- **10x Genomics Cell Ranger** (Performs genomic mapping, filtering, barcode counting, and UMI counting on the short reads to supply the BAM used by extract_sr_bc)
- **Badread** (Long-read simulator used in the paper's benchmarking context for simulating error-prone long reads) — https://github.com/rrwick/Badread

## Examples

```
./scTagger.py match_trie -lr path/to/lr_segments.tsv -sr path/to/sr_barcodes.tsv -o matches.tsv -t 8
```

## Evaluation signals

- Reproduced scTagger match/ambiguous/mismatch/skip percentages match Table 3 values for the simulated datasets (e.g., Large: 53.26% matches)
- FLAMES percentages match Table 3 (e.g., Large: 30.05% matches), confirming the comparison is configured correctly
- scTagger match rate is close to the brute-force edlib baseline despite the e=2 edit distance limit
- scTagger match rate substantially exceeds FLAMES on all three simulated datasets (Small, Medium, Large)
- Every long read is assigned exactly one of the four categories (match, ambiguous, mismatch, skip) and category percentages sum to 100%

## Limitations

- scTagger's default maximum edit distance is e=2 because the vast majority of LRs with >2 edits have no unique barcode match
- scTagger in its current form is specific for 10x Genomics scRNA-seq
- scTagger's current design is not expected to scale well beyond thousands to tens of thousands of relevant barcodes, and memory use is a known weakness
- Matching of UMI sequences between SR and LR data is not yet supported

## Evidence
