---
name: simulated-vs-real-read-alignment-comparison
description: Use when benchmarking a barcode-extraction or adapter-alignment step
  on real vs simulated long reads, and you need the proportion of LRs with a successful
  SR-adapter alignment plus the cumulative edit-distance distribution (Figure 5-style).
when_to_use_negative:
- The goal is matching barcodes between SR and LR data (use the full scTagger pipeline
  with match_trie, not the adapter-alignment comparison alone)
- The LR platform does not include the fixed 10x adapter sequence next to the barcode,
  since scTagger's current form is specific for 10x Genomics scRNA-seq
- The question is UMI matching between SR and LR data, which scTagger does not support
  yet
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: scTagger
  role: Runs extract_lr_bc to glocal-align the Illumina SR adapter to each long read,
    reject out-of-range alignments, and report best edit distances used to build the
    cumulative distributions
  repo: https://github.com/vpc-ccg/scTagger/
- name: edlib
  role: Sequence-alignment library (v1.3.9) used by scTagger for adapter alignment
    / edit-distance computation
  repo: https://github.com/Martinsos/edlib
- name: Badread
  role: Long-read simulator available for generating simulated LR FASTQ inputs against
    which real-data adapter-alignment coverage is compared
  repo: https://github.com/rrwick/Badread
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2022.104530
    title: Fast and accurate matching of cellular barcodes across short-reads and
      long-reads of single-cell RNA-seq experiments
  source_content_hash: sha256:38bafad821ffb2076795bd2b9e038d97a1fcfe9fd2520c07e794fc737489a23f
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
  - Badread
  - edlib
  - scTagger
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# simulated-vs-real-read-alignment-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies and compares the fraction of long reads with a successfully aligned Illumina SR adapter in real (trimmed) versus simulated LR FASTQ datasets, by aggregating best adapter-alignment edit distances from scTagger's extract_lr_bc output and plotting cumulative distributions (as in the paper's Figure 5).

## When to use

Use when benchmarking a barcode-extraction or adapter-alignment step on real vs simulated long reads, and you need the proportion of LRs with a successful SR-adapter alignment plus the cumulative edit-distance distribution (Figure 5-style).

## When NOT to use

- The goal is matching barcodes between SR and LR data (use the full scTagger pipeline with match_trie, not the adapter-alignment comparison alone)
- The LR platform does not include the fixed 10x adapter sequence next to the barcode, since scTagger's current form is specific for 10x Genomics scRNA-seq
- The question is UMI matching between SR and LR data, which scTagger does not support yet

## Inputs

- Real (trimmed) long-read FASTQ files from Figshare 10.6084/m9.figshare.19735678.v1
- Simulated long-read FASTQ files from Figshare 10.6084/m9.figshare.19740475.v1
- Default Illumina SR adapter sequence CTACACGACGCTCTTCCGATCT (hardcoded default of extract_lr_bc)

## Outputs

- Per-dataset TSV from extract_lr_bc: read-id, best edit distance with the SR adapter, adapter-alignment start position, extracted 20 bp LR segment
- Adapter-alignment location plots produced by extract_lr_bc
- Cumulative adapter-alignment edit-distance distributions (bar/cumulative plots) for real and simulated datasets, reproducing Figure 5

## How to apply

Download the real trimmed LR FASTQ files (Figshare 10.6084/m9.figshare.19735678.v1) and simulated LR FASTQ files (Figshare 10.6084/m9.figshare.19740475.v1). Run scTagger's extract_lr_bc module on each dataset with the default short-read adapter CTACACGACGCTCTTCCGATCT, automatic detection of adapter-location ranges, and the default 20 bp segment extracted after the adapter alignment; alignments falling outside the automatically detected ranges around mapping-location peaks are rejected. Parse the output TSV (read-id, best edit distance, alignment start, extracted segment), aggregate LRs by best adapter-alignment edit distance, and compute the cumulative fraction of LRs at each edit distance for real and simulated datasets. Plot the two cumulative distributions and validate that successful adapter alignment exceeds 75% of LRs for real data and reaches approximately 92-93% for simulated data, matching the reported Figure 5 outcome.

## Related tools

- **scTagger** (Runs extract_lr_bc to glocal-align the Illumina SR adapter to each long read, reject out-of-range alignments, and report best edit distances used to build the cumulative distributions) — https://github.com/vpc-ccg/scTagger/
- **edlib** (Sequence-alignment library (v1.3.9) used by scTagger for adapter alignment / edit-distance computation) — https://github.com/Martinsos/edlib
- **Badread** (Long-read simulator available for generating simulated LR FASTQ inputs against which real-data adapter-alignment coverage is compared) — https://github.com/rrwick/Badread

## Examples

```
./scTagger.py extract_lr_bc -r real_trimmed.fastq.gz simulated.fastq.gz -o segments.tsv -p plot_dir/
```

## Evaluation signals

- Fraction of LRs with a successful adapter alignment exceeds 75% for the real (trimmed) dataset
- Fraction of LRs with a successful adapter alignment is approximately 92-93% for the simulated dataset
- Each output TSV row carries a read-id, a best edit distance, an alignment start position, and a 20 bp extracted segment
- Adapter alignments accepted by extract_lr_bc fall inside the automatically detected ranges around the mapping-location peaks (visible in the emitted alignment-location plot)
- Cumulative fraction curves for real vs simulated datasets match the ordering and shape of the Figure 5 distributions

## Limitations

- Adapter alignments falling outside the automatically detected ranges around mapping-location peaks are rejected, which caps achievable alignment coverage
- Simulated coverage (~92-93%) is an upper bound relative to real data (>75%); differences in error models (e.g., Badread chimeras, glitches, low-quality regions) will shift the distribution
- The comparison reflects scTagger's 10x Genomics-specific adapter (CTACACGACGCTCTTCCGATCT) and is not generalizable to platforms lacking a fixed sequence next to the barcode

## Evidence

- [readme] Badread is a long-read simulator tool that makes – you guessed it – bad reads! It can imitate many kinds of problems one might encounter in real long-read sets: [evidence span withheld — anchor status: external_doc]
