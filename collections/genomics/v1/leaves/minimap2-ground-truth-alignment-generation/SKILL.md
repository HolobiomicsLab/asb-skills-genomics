---
name: minimap2-ground-truth-alignment-generation
description: 'Use when a raw-signal mapping benchmark needs reference alignments:
  basecalled FAST5/POD5 reads must be mapped to a reference genome with minimap2 to
  produce ground-truth PAFs for computing precision/recall/F1 of tool outputs.'
when_to_use_negative:
- When the goal is de novo assembly of raw signals without a reference genome (use
  Rawsamble all-vs-all overlapping instead)
- When no basecalled sequences are available and basecalling cannot be performed,
  since minimap2 requires sequence (FASTA/FASTQ) input
- When the comparison target is throughput or indexing speed rather than mapping accuracy,
  which does not require ground-truth PAFs
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: minimap2
  role: Aligns basecalled reads to the reference genome to produce the ground-truth
    PAF alignments used as the scoring reference
  repo: https://github.com/lh3/minimap2
- name: UNCALLED
  role: Provides the pafstats utility that scores each mapper PAF against the minimap2
    ground truth to compute precision, recall, and F1
  repo: https://github.com/skovaka/UNCALLED
- name: RawHash2
  role: Produces the modified-PAF raw-signal mapping outputs that are evaluated against
    the minimap2 ground truth
  repo: https://github.com/CMU-SAFARI/RawHash
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae478
    title: 'RawHash2: mapping raw nanopore signals using hash-based seeding and adaptive
      quantization'
  source_content_hash: sha256:a0b1b469f8e0f331cbd9444b775438ab6c689d39e45585bdd66728b72a4c2607
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - RawHash
  - RawHash2
  - UNCALLED
  - minimap2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# minimap2-ground-truth-alignment-generation

## Summary

Generates ground-truth PAF alignments by basecalling nanopore reads and mapping them to the reference genome with minimap2, so that raw-signal mapper outputs (RawHash2, RawHash, UNCALLED) can be scored for precision, recall, and F1 using UNCALLED's pafstats.

## When to use

Use when a raw-signal mapping benchmark needs reference alignments: basecalled FAST5/POD5 reads must be mapped to a reference genome with minimap2 to produce ground-truth PAFs for computing precision/recall/F1 of tool outputs.

## When NOT to use

- When the goal is de novo assembly of raw signals without a reference genome (use Rawsamble all-vs-all overlapping instead)
- When no basecalled sequences are available and basecalling cannot be performed, since minimap2 requires sequence (FASTA/FASTQ) input
- When the comparison target is throughput or indexing speed rather than mapping accuracy, which does not require ground-truth PAFs

## Inputs

- FAST5 (or POD5/SLOW5/BLOW5) raw nanopore signal files per test dataset
- Reference genome in FASTA format (e.g., SARS-CoV-2, green algae, human NA12878, E. coli R10.4)
- Basecalled read sequences (FASTQ/FASTA) derived from the FAST5 files
- Mapping outputs in PAF format from each raw-signal mapper being evaluated

## Outputs

- Ground-truth PAF alignment file per dataset (minimap2 mappings of basecalled reads to the reference genome)
- Per-dataset precision, recall, and F1 scores for each evaluated mapper, computed by UNCALLED pafstats

## How to apply

Basecall the FAST5 reads from each test dataset and align the resulting sequences to the reference genome with minimap2 (e.g., the map-ont preset for Oxford Nanopore genomic reads), writing the alignments in PAF format; these PAFs serve as the ground truth. Run each raw-signal mapper (RawHash2 with genome-appropriate presets such as -x viral, -x sensitive, or -x fast, and RawHash v1.0) on the same datasets to produce modified-PAF mapping outputs. Score every tool PAF against the minimap2 ground-truth PAF with UNCALLED's pafstats to obtain per-dataset precision, recall, and F1. The rationale is that minimap2 operates on basecalled sequences and is treated as the accurate reference standard, while raw-signal mappers are judged by how well they agree with it.

## Related tools

- **minimap2** (Aligns basecalled reads to the reference genome to produce the ground-truth PAF alignments used as the scoring reference) — https://github.com/lh3/minimap2
- **UNCALLED** (Provides the pafstats utility that scores each mapper PAF against the minimap2 ground truth to compute precision, recall, and F1) — https://github.com/skovaka/UNCALLED
- **RawHash2** (Produces the modified-PAF raw-signal mapping outputs that are evaluated against the minimap2 ground truth) — https://github.com/CMU-SAFARI/RawHash

## Examples

```
minimap2 -ax map-ont ref.fa basecalled_reads.fq > ground_truth.paf
```

## Evaluation signals

- Ground-truth PAF exists for every test dataset and contains alignments for basecalled reads against the matching reference genome
- Every tool PAF yields non-empty precision, recall, and F1 values from pafstats for each dataset
- Reproduced mean RawHash2-over-RawHash F1 improvement falls within reasonable agreement of the reported 10.57% average
- Reproduced maximum F1 improvement falls within reasonable agreement of the reported 20.25%
- Ground-truth and tool PAFs use the same reference genome and PAF template conventions (following the UNCALLED/Sigmap PAF template)

## Limitations

- Ground truth is minimap2-based, so scores measure agreement with minimap2 rather than absolute biological truth
- UNCALLED applies only to legacy r9.4.1 data; for r10.4.1 data the pipeline must rely on other tools (RawHash2 supports R10.4 with the UNCALLED4 R10.4.1 k-mer model)
- FAST5 reads compressed with VBZ require the ONT VBZ compression HDF5 plugin to be readable

## Evidence

- [methods] minimap2 as accuracy reference: "We follow the similar PAF template used in [UNCALLED](https://github.com/skovaka/UNCALLED)"
- [abstract] mapping accuracy evaluated via F1: "RawHash2 provides better F1 accuracy (on average by 10.57% and up to 20.25%)"
- [readme] minimap2 role and usage: "Minimap2 is a versatile sequence alignment program that aligns DNA or mRNA
sequences against a large reference database."
- [readme] UNCALLED as raw-signal mapper: [evidence span withheld — anchor status: external_doc]
