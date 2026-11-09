---
name: edit-distance-benchmark-comparison
description: Use when evaluating whether a barcode-matching method recovers long-read
  cellular barcodes as accurately as an exact edit-distance baseline or a competing
  tool on simulated hybrid LR-SR scRNA-seq datasets.
when_to_use_negative:
- The goal is real-data performance characterization rather than controlled benchmarking
  against a published reference table
- The input barcodes come from a non-10x scRNA-seq platform without a fixed sequence
  adjacent to the barcode in the LR template
- UMI-level matching between SR and LR data is required, since scTagger does not yet
  support it
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: scTagger
  role: Trie-based barcode matcher under evaluation; run as extract_lr_bc, extract_sr_bc,
    match_trie with e=2
  repo: https://github.com/vpc-ccg/scTagger/
- name: edlib
  role: Provides exact edit-distance computation for the brute-force baseline assignments
  repo: https://github.com/Martinsos/edlib
- name: FLAMES
  role: 'Competing barcode-assignment method benchmarked against scTagger (commit
    #18fb83c)'
  repo: https://github.com/LuyiTian/FLAMES
- name: 10x Genomics Cell Ranger
  role: Produces the SR BAM used by extract_sr_bc for genomic mapping, filtering,
    barcode counting, and UMI counting
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2022.104530
    title: Fast and accurate matching of cellular barcodes across short-reads and
      long-reads of single-cell RNA-seq experiments
  source_content_hash: sha256:a36aec5462b9438c0702739398c6ea70f60b506619fab964569dd740affe9eb1
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
  - FLAMES
  - edlib
  - scTagger
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# edit-distance-benchmark-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks cellular barcode matching accuracy across a trie-based method (scTagger, e=2), an exact dynamic-programming brute-force baseline (edlib v1.3.9), and an alternative tool (FLAMES, commit #18fb83c) by classifying each long read as match, ambiguous, mismatch, or skip and comparing category percentages against published Table 3 values.

## When to use

Use when evaluating whether a barcode-matching method recovers long-read cellular barcodes as accurately as an exact edit-distance baseline or a competing tool on simulated hybrid LR-SR scRNA-seq datasets.

## When NOT to use

- The goal is real-data performance characterization rather than controlled benchmarking against a published reference table
- The input barcodes come from a non-10x scRNA-seq platform without a fixed sequence adjacent to the barcode in the LR template
- UMI-level matching between SR and LR data is required, since scTagger does not yet support it

## Inputs

- Simulated LR-SR datasets (Small, Medium, Large) from Figshare DOI 10.6084/m9.figshare.19740475.v1
- Long-read FASTQ files
- Cell Ranger BAM of short reads
- 10x Genomics cellular barcode whitelist (e.g. 3M-february-2018.txt.gz)

## Outputs

- Per-method per-dataset CSV table of match / ambiguous / mismatch / skip percentages
- Barcode match output files from scTagger match_trie, edlib brute-force, and FLAMES

## How to apply

Download the simulated LR-SR datasets (Small, Medium, Large) from Figshare DOI 10.6084/m9.figshare.19740475.v1. Run the scTagger pipeline per dataset: extract_lr_bc on long-read FASTQs, extract_sr_bc on the Cell Ranger BAM, then match_trie with the default maximum edit distance e=2. Run brute-force matching of the same LR barcode segments against the selected SR barcodes using edlib (v1.3.9) to obtain exact edit-distance assignments, and run FLAMES (commit #18fb83c) to obtain its assignments. Classify each long read per method as match (unique barcode), ambiguous (multiple barcodes at minimum edit distance), mismatch, or skip, compute percentages per category, and compile a per-method per-dataset CSV. The rationale is that most LRs with >2 edits have no unique barcode match, so e=2 is an appropriate operating point, and edlib-based brute force provides ground-truth-quality exact assignments for comparison.

## Related tools

- **scTagger** (Trie-based barcode matcher under evaluation; run as extract_lr_bc, extract_sr_bc, match_trie with e=2) — https://github.com/vpc-ccg/scTagger/
- **edlib** (Provides exact edit-distance computation for the brute-force baseline assignments) — https://github.com/Martinsos/edlib
- **FLAMES** (Competing barcode-assignment method benchmarked against scTagger (commit #18fb83c)) — https://github.com/LuyiTian/FLAMES
- **10x Genomics Cell Ranger** (Produces the SR BAM used by extract_sr_bc for genomic mapping, filtering, barcode counting, and UMI counting)

## Examples

```
./scTagger.py match_trie -lr path/to/extract/lr_segments -sr path/to/extract/sr_barcodes -o matches.tsv -t 8
```

## Evaluation signals

- Reproduced scTagger match rate on the Large simulated dataset is ~53.26% vs FLAMES ~30.05%, matching Table 3
- scTagger match/ambiguous/mismatch/skip percentages are close to the edlib brute-force baseline despite its e=2 limit
- Every long read is assigned to exactly one of the four categories (match, ambiguous, mismatch, skip) per method and percentages sum to 100%
- edlib brute-force assignments use exact minimum edit distance over the same LR segments and selected SR barcodes as scTagger, enabling a fair comparison
- Validation criterion: output_matches_reference against the article's Table 3 values for the simulated datasets

## Limitations

- scTagger's maximum edit distance of e=2 means reads with >2 edits are skipped; the vast majority of such LRs have no unique barcode match
- scTagger in its current form is specific for 10x Genomics scRNA-seq and is not expected to scale well beyond thousands to tens of thousands of relevant barcodes
- Results on simulated datasets may not transfer directly to real data; a companion benchmark on trimmed real datasets exists (Figshare 10.6084/m9.figshare.19735678.v1)

## Evidence

- [other] On the simulated datasets, scTagger significantly outperforms FLAMES in barcode matching accuracy and, despite its edit distance limit of e=2, achieves a match rate very similar to the Brute-force baseline: [evidence span withheld — anchor status: not_found]
- [discussion] we opted to use a trie data structure to compute the optimal edit distances between SR barcodes and the LR segments up to an assumed maximum edit distance of e: "we opted to use a trie data structure to compute the optimal edit distances between SR barcodes and the LR segments up to an assumed maximum edit"
- [discussion] edlib (v1.3.9): "edlib (v1.3.9)Šošić and Šikić, 2017https://github.com/Martinsos/edlib"
- [discussion] the vast majority of LRs with >2 edits have no unique barcode match and are therefore not useful for downstream analysis. Therefore, by default, we set e=2.: "the vast majority of LRs with >2 edits have no unique barcode match and are therefore not useful for downstream analysis. Therefore, by default, we set e=2."
- [readme] The whole pipeline contains three steps that you can run each part separately: "The whole pipeline contains three steps that you can run each part separately:"
