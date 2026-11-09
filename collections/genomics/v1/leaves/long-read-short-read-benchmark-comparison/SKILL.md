---
name: long-read-short-read-benchmark-comparison
description: Use when you need to quantify how a fast barcode-matching method (scTagger,
  e=2) deviates from an exact DP baseline and from FLAMES on hybrid LR-SR scRNA-seq
  data, e.g. reproducing Figure 6 / Table S1.
when_to_use_negative:
- You want to actually assign cellular barcodes to a new experiment rather than measure
  assignment accuracy — run scTagger's pipeline directly instead of the brute-force/FLAMES
  comparison
- The data are not from a 10x Genomics scRNA-seq hybrid LR-SR protocol, since scTagger's
  current form is specific to 10x Genomics scRNA-seq
- The task requires matching UMI sequences between SR and LR data, which scTagger
  does not support
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3168
tools:
- name: scTagger
  role: Fast trie-based barcode matcher under evaluation; run as extract_lr_bc, extract_sr_bc,
    match_trie with default e=2
  repo: https://github.com/vpc-ccg/scTagger/
- name: edlib
  role: Provides exact edit-distance computation for the brute-force dynamic-programming
    reference matcher (v1.3.9)
  repo: https://github.com/Martinsos/edlib
- name: FLAMES
  role: 'State-of-the-art long-read-to-barcode annotation tool used as a comparison
    baseline (commit #18fb83c)'
  repo: https://github.com/LuyiTian/FLAMES
- name: Cell Ranger
  role: Upstream SR processing (mapping, filtering, barcode counting, UMI counting)
    supplying the BAM input to extract_sr_bc
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2022.104530
    title: Fast and accurate matching of cellular barcodes across short-reads and
      long-reads of single-cell RNA-seq experiments
  source_content_hash: sha256:b41e1d4b41858b33b8a9dbd7785b707d7f5731f98f0933f05458933249fba10c
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
  - Cell Ranger
  - FLAMES
  - edlib
  - scTagger
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# long-read-short-read-benchmark-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking skill that quantifies how scTagger's trie-based barcode matching (default maximum edit distance e=2) mirrors an exact dynamic-programming brute-force baseline and FLAMES on hybrid LR-SR single-cell RNA-seq datasets, measuring lost unambiguous matches and assignment discordance.

## When to use

Use when you need to quantify how a fast barcode-matching method (scTagger, e=2) deviates from an exact DP baseline and from FLAMES on hybrid LR-SR scRNA-seq data, e.g. reproducing Figure 6 / Table S1.

## When NOT to use

- You want to actually assign cellular barcodes to a new experiment rather than measure assignment accuracy — run scTagger's pipeline directly instead of the brute-force/FLAMES comparison
- The data are not from a 10x Genomics scRNA-seq hybrid LR-SR protocol, since scTagger's current form is specific to 10x Genomics scRNA-seq
- The task requires matching UMI sequences between SR and LR data, which scTagger does not support

## Inputs

- Gzipped FASTQ long-read files from hybrid LR-SR scRNA-seq experiments (trimmed real datasets, Figshare 10.6084/m9.figshare.19735678.v1)
- Cell Ranger BAM of short-read data for extract_sr_bc
- 10x Genomics cellular barcode whitelist (e.g. 3M-february-2018.txt.gz)
- Long-read segment TSV produced by extract_lr_bc (read-id, best adapter edit distance, alignment start, 20bp segment)

## Outputs

- sctagger_bruteforce_flames_comparison.csv tabulating LR loss from e=2, scTagger-vs-brute-force discordance, and FLAMES-missed shared matches
- edit_distance_distribution.png bar plot of long-read counts by minimum edit distance
- scTagger match_trie gzipped TSV of LR-to-barcode assignments

## How to apply

Run the scTagger pipeline (extract_lr_bc, extract_sr_bc, then match_trie with default maximum edit distance e=2, barcode length 16, gzipped TSV output) on the trimmed real hybrid LR-SR datasets (Figshare 10.6084/m9.figshare.19735678.v1). As the accuracy reference, run a brute-force exact dynamic-programming matcher built on edlib v1.3.9 to compute unambiguous minimum edit-distance barcode assignments for the same LR segments. Run FLAMES (commit #18fb83c) on the same datasets to obtain its long-read-to-cellular-barcode annotations. Quantify three metrics: the fraction of LRs with unambiguous matches lost due to scTagger's e=2 limit, the discordance between scTagger-only and brute-force assignments, and the fraction of shared scTagger/brute-force matches missed by FLAMES. The rationale is that scTagger sets e=2 because the vast majority of LRs with >2 edits have no unique barcode match, so the benchmark tests whether that speed-motivated cutoff costs accuracy in practice.

## Related tools

- **scTagger** (Fast trie-based barcode matcher under evaluation; run as extract_lr_bc, extract_sr_bc, match_trie with default e=2) — https://github.com/vpc-ccg/scTagger/
- **edlib** (Provides exact edit-distance computation for the brute-force dynamic-programming reference matcher (v1.3.9)) — https://github.com/Martinsos/edlib
- **FLAMES** (State-of-the-art long-read-to-barcode annotation tool used as a comparison baseline (commit #18fb83c)) — https://github.com/LuyiTian/FLAMES
- **Cell Ranger** (Upstream SR processing (mapping, filtering, barcode counting, UMI counting) supplying the BAM input to extract_sr_bc)

## Examples

```
./scTagger.py match_trie -lr "path/to/output/extract/long-read/segment" -sr "path/to/output/extract/top/short-read" -o "path/to/output/file" -t "number of threads"
```

## Evaluation signals

- Fraction of LRs with unambiguous matches lost due to scTagger's e=2 limit falls within the reported 0.8%-2.2% range on real datasets
- Discordance between scTagger-only and brute-force dynamic-programming assignments is approximately 0.1%
- FLAMES misses approximately 26-28% of the shared scTagger/brute-force matches
- extract_lr_bc output TSV has the four expected columns: read-id, best adapter edit distance, alignment start position, and extracted long-read segment
- edit_distance_distribution.png shows the long-read count per minimum edit distance and supports the claim that LRs with >2 edits rarely have unique barcode matches

## Limitations

- scTagger's default e=2 discards a small but nonzero fraction (0.8%-2.2%) of LRs that do have unambiguous brute-force matches on real datasets
- scTagger in its current form is specific for 10x Genomics scRNA-seq and its design is not expected to scale well beyond thousands to tens of thousands of relevant barcodes
- Memory use is a known weakness of scTagger, and matching of UMI sequences between SR and LR data is not yet supported

## Evidence
