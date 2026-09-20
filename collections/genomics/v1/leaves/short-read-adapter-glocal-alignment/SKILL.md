---
name: short-read-adapter-glocal-alignment
description: Use when long-read FASTQ reads from a split hybrid LR-SR scRNA-seq library
  must have their 10x barcode-adjacent segment located and extracted, via adapter
  alignment with edit-distance scoring.
when_to_use_negative:
- The long-read library template lacks a fixed sequence adjacent to the cellular barcode
  — non-10x platform support is only possible if the LR template includes some fixed
  sequence next to the barcode
- The goal is matching barcodes between SR and LR reads (this skill only extracts
  LR segments; matching is the separate match_trie step)
- Extracting short-read barcodes from a Cell Ranger BAM — use the extract_sr_bc step
  instead
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_0080
tools:
- name: scTagger
  role: Implements the extract_lr_bc step that performs the adapter glocal alignment
    and downstream segment extraction
  repo: https://github.com/vpc-ccg/scTagger/
- name: edlib
  role: Sequence alignment library used by scTagger to glocal-align the short-read
    adapter to each long read and compute the best edit distance
  repo: https://github.com/Martinsos/edlib
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2022.104530
    title: Fast and accurate matching of cellular barcodes across short-reads and
      long-reads of single-cell RNA-seq experiments
  source_content_hash: sha256:d10505006c6b3e72b1ca42d46ee4b290173c38a0707ba37304e1984a58c2bf5d
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
  - edlib
  - scTagger
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# short-read-adapter-glocal-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Locates the cellular barcode segment within each long read by glocal-aligning the fixed Illumina short-read adapter sequence (default CTACACGACGCTCTTCCGATCT) to every long read with edlib, then extracting the bases immediately downstream of the adapter alignment. This is the extract_lr_bc stage of scTagger that produces candidate barcode segments for downstream trie matching.

## When to use

Use when long-read FASTQ reads from a split hybrid LR-SR scRNA-seq library must have their 10x barcode-adjacent segment located and extracted, via adapter alignment with edit-distance scoring.

## When NOT to use

- The long-read library template lacks a fixed sequence adjacent to the cellular barcode — non-10x platform support is only possible if the LR template includes some fixed sequence next to the barcode
- The goal is matching barcodes between SR and LR reads (this skill only extracts LR segments; matching is the separate match_trie step)
- Extracting short-read barcodes from a Cell Ranger BAM — use the extract_sr_bc step instead

## Inputs

- Long-read FASTQ files (one or more, space-separated paths; gzipped inputs auto-detected by .gz suffix)
- Optional: manually specified ranges (-g) of where the short-read adapter should be found on the long reads
- Optional: custom short-read adapter sequence (-sa)

## Outputs

- A TSV file with four columns: read-id, best edit distance with the short-read adapter, starting point of the long read matching the adapter, and the extracted long-read segment (default 20bp after adapter)
- Optional plot of optimal adapter alignment locations on the long reads (-p)

## How to apply

Run scTagger's extract_lr_bc step: glocal-align the short-read adapter sequence (default CTACACGACGCTCTTCCGATCT, overridable with -sa) to each long read using edlib (v1.3.9), recording the best edit distance per read. Adapter alignment loci on the forward and reverse strands form peaks (visualized with the optional -p plot); automatically detect the ranges around these peaks and reject any adapter alignments falling outside them, unless ranges are manually fixed with -g. For each successful alignment, extract --num-bp-afte bases (default 20) starting at the end of the adapter alignment — this segment is where the probability of seeing a barcode is highest. Gzipped FASTQ inputs are auto-detected by the .gz suffix, and -t parallelizes processing across threads.

## Related tools

- **scTagger** (Implements the extract_lr_bc step that performs the adapter glocal alignment and downstream segment extraction) — https://github.com/vpc-ccg/scTagger/
- **edlib** (Sequence alignment library used by scTagger to glocal-align the short-read adapter to each long read and compute the best edit distance) — https://github.com/Martinsos/edlib

## Examples

```
./scTagger.py extract_lr_bc -r "path/to/long/read/fastq" -o "path/to/output/file" -p "path/to/output/plots"
```

## Evaluation signals

- Output TSV exists and contains exactly the four required columns: read-id, best edit distance with the short-read adapter, adapter alignment starting point, and the extracted long-read segment
- Adapter alignments that passed the peak-range filter fall within the automatically detected (or -g specified) ranges on the forward/reverse strands
- Extracted segments are --num-bp-afte (default 20) bases long, starting at the end of the adapter alignment
- With -p enabled, the alignment-location plot shows clear peaks on both forward and reverse strands, consistent with a barcode-adjacent adapter position
- Downstream extract_sr_bc_from_lr / match_trie steps accept the TSV segments and yield cellular barcode matches at edit distance e ≤ 2

## Limitations

- Adapter alignments outside the automatically detected peak ranges are rejected, so genuinely valid alignments in low-coverage regions may be lost (mitigatable only by manually fixing ranges with -g)
- This stage is specific to 10x Genomics scRNA-seq templates; extension to other platforms requires a fixed sequence next to the barcode in the LR template
- Downstream barcode matching is limited to default maximum edit distance e=2 because the vast majority of LRs with >2 edits have no unique barcode match

## Evidence
