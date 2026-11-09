---
name: hybrid-scrnaseq-workflow-orchestration
description: Use when benchmarking runtime and memory of barcode matching in split
  hybrid LR-SR scRNA-seq experiments, or when needing a reproducible multi-stage scTagger
  workflow orchestrated across datasets with timing instrumentation.
when_to_use_negative:
- The goal is accuracy evaluation on simulated data rather than runtime/memory benchmarking
  on real hybrid datasets
- The data are not from a 10x Genomics split hybrid LR-SR scRNA-seq protocol — scTagger's
  current form is specific for 10x Genomics scRNA-seq unless the LR template contains
  a fixed sequence next to the barcode
- The number of relevant cellular barcodes greatly exceeds thousands to tens of thousands,
  where scTagger's trie design is not expected to scale
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
tools:
- name: scTagger
  role: Three-stage pipeline (extract_lr_bc, extract_sr_bc, match_trie) whose trie-based
    barcode matching is benchmarked for runtime and memory on the real datasets
  repo: https://github.com/vpc-ccg/scTagger/
- name: 'FLAMES (commit #18fb83c)'
  role: State-of-the-art baseline barcode-matching tool timed and memory-profiled
    for comparison under identical instrumentation
  repo: https://github.com/LuyiTian/FLAMES
- name: edlib (v1.3.9)
  role: Exact dynamic-programming edit-distance baseline (e<=2) run on the same inputs
    with the same timing/memory instrumentation
  repo: https://github.com/Martinsos/edlib
- name: Snakemake
  role: Workflow orchestration via the provided Snakefile and config.yaml to run the
    three scTagger stages plus Cell Ranger
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2022.104530
    title: Fast and accurate matching of cellular barcodes across short-reads and
      long-reads of single-cell RNA-seq experiments
  source_content_hash: sha256:ea84df2ba51ad772afff9b209ae2c2f8704a8135187e2a6bb22220bded3ec7e2
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
  - FLAMES
  - 'FLAMES (commit #18fb83c)'
  - Snakemake
  - edlib
  - edlib (v1.3.9)
  - scTagger
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# hybrid-scrnaseq-workflow-orchestration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Orchestrates the three-stage scTagger pipeline (extract_lr_bc, extract_sr_bc, match_trie) on hybrid long-read/short-read single-cell RNA-seq data, benchmarking runtime and peak memory against FLAMES and an edlib-based brute-force matcher on the real datasets N, NOA1, and NOA2.

## When to use

Use when benchmarking runtime and memory of barcode matching in split hybrid LR-SR scRNA-seq experiments, or when needing a reproducible multi-stage scTagger workflow orchestrated across datasets with timing instrumentation.

## When NOT to use

- The goal is accuracy evaluation on simulated data rather than runtime/memory benchmarking on real hybrid datasets
- The data are not from a 10x Genomics split hybrid LR-SR scRNA-seq protocol — scTagger's current form is specific for 10x Genomics scRNA-seq unless the LR template contains a fixed sequence next to the barcode
- The number of relevant cellular barcodes greatly exceeds thousands to tens of thousands, where scTagger's trie design is not expected to scale

## Inputs

- Long-read FASTQ files (trimmed real datasets N, NOA1, NOA2 from Figshare 10.6084/m9.figshare.19735678.v1)
- Short-read BAM files from Cell Ranger mapping
- 10x Genomics cellular barcode whitelist (e.g., 3M-february-2018.txt.gz) for the extract_sr_bc_from_lr alternative

## Outputs

- LR barcode-segment TSVs from extract_lr_bc (read-id, edit distance, adapter alignment start, LR segment)
- Top SR barcode TSVs from extract_sr_bc (barcode, appearance count)
- trie-based LR-to-barcode match output from match_trie
- table4_runtime_memory_benchmark.csv with real time, user time, and peak memory per tool per dataset

## How to apply

Install scTagger from Bioconda (conda create -n sctagger-env -c bioconda sctagger) and download the trimmed real datasets from Figshare (10.6084/m9.figshare.19735678.v1). For each dataset, run extract_lr_bc on the long-read FASTQ files with the default SR adapter CTACACGACGCTCTTCCGATCT and default 20bp segment extraction to produce LR barcode-segment TSVs, then run extract_sr_bc on the short-read BAM with default filtering (--thresh 0.005, --step-size 1000, --max-barcode-cnt 25000) to obtain the top SR barcode lists. Next run match_trie with defaults (-mr 2, -m 16.0 GB, -bl 16, -t 16 threads) wrapping each stage with time/memory instrumentation, and run the edlib (v1.3.9) exact dynamic-programming baseline (e<=2) and FLAMES (commit #18fb83c) under identical instrumentation. A Snakemake Snakefile with config.yaml can orchestrate the three scTagger stages plus Cell Ranger. Aggregate real time, user time, and peak memory per tool per dataset into a CSV and validate that scTagger's measured times are orders of magnitude lower than the brute-force baseline and FLAMES, consistent with scTagger's multi-core advantage.

## Related tools

- **scTagger** (Three-stage pipeline (extract_lr_bc, extract_sr_bc, match_trie) whose trie-based barcode matching is benchmarked for runtime and memory on the real datasets) — https://github.com/vpc-ccg/scTagger/
- **FLAMES (commit #18fb83c)** (State-of-the-art baseline barcode-matching tool timed and memory-profiled for comparison under identical instrumentation) — https://github.com/LuyiTian/FLAMES
- **edlib (v1.3.9)** (Exact dynamic-programming edit-distance baseline (e<=2) run on the same inputs with the same timing/memory instrumentation) — https://github.com/Martinsos/edlib
- **Snakemake** (Workflow orchestration via the provided Snakefile and config.yaml to run the three scTagger stages plus Cell Ranger)

## Examples

```
conda create -n sctagger-env -c bioconda sctagger && ./scTagger.py match_trie -lr lr_segments.tsv -sr top_sr_barcodes.tsv -o matches.tsv -t 16
```

## Evaluation signals

- scTagger's measured real/user times are orders of magnitude lower than the edlib brute-force baseline and FLAMES on all three datasets (N, NOA1, NOA2)
- match_trie completes within the 16.0 GB memory cap (-m 16.0) and peak memory is recorded per run
- extract_lr_bc TSVs contain the expected four columns (read-id, best edit distance, adapter alignment start, 20bp segment)
- extract_sr_bc output retains the top barcodes until a 1,000-barcode increment contributes less than t=0.5% of total SRs, capped at --max-barcode-cnt 25000
- The benchmark CSV contains one row per tool per dataset with real time, user time, and peak memory populated for every cell

## Limitations

- Memory use is a known weakness; streamlining how scTagger reads and processes input is planned future work
- scTagger in its current form is specific for 10x Genomics scRNA-seq; support for other platforms requires a fixed sequence next to the barcode in the LR template
- scTagger's current design is not expected to scale well if the number of relevant barcodes becomes much larger than thousands to tens of thousands
- No changelog is available for the synthesized scTagger source, a reproducibility gap for pinning the exact benchmarked version

## Evidence

- [discussion] Memory improvement is planned future work: [evidence span withheld — anchor status: not_found]
