---
name: paf-mapping-accuracy-scoring-precision-recall
description: Use when comparing raw nanopore signal read-mapping tools whose outputs
  are in PAF format and you need precision/recall/F1 against a sequence-based ground
  truth such as minimap2 alignments of basecalled reads.
when_to_use_negative:
- Inputs are basecalled sequence reads (FASTQ/FASTA) rather than raw-signal mapper
  PAF outputs — use standard sequence-alignment benchmarking instead
- No trusted ground-truth alignment exists for the dataset — pafstats requires a reference
  PAF to score against
- Evaluating overlap-based all-vs-all tasks (Rawsamble) where no reference genome
  ground truth is defined
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: UNCALLED
  role: Provides the pafstats utility that computes precision, recall, and F1 by comparing
    a mapper's PAF output against the ground-truth PAF
  repo: https://github.com/skovaka/UNCALLED
- name: minimap2
  role: Generates the ground-truth PAF alignments by mapping basecalled reads to the
    reference genome
  repo: https://github.com/lh3/minimap2
- name: RawHash2
  role: Tool under evaluation; produces modified-PAF mapping outputs from raw nanopore
    signals that are scored for precision/recall/F1
  repo: https://github.com/CMU-SAFARI/RawHash
- name: RawHash
  role: Baseline raw-signal mapper whose PAF outputs are scored for comparison against
    RawHash2
  repo: https://github.com/CMU-SAFARI/RawHash
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae478
    title: 'RawHash2: mapping raw nanopore signals using hash-based seeding and adaptive
      quantization'
  source_content_hash: sha256:3a077c6f13d1105b93f4fff8a652524d90dc00266e8b5e4a70c027c1bf2b6c04
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/skovaka/UNCALLED
  tools:
  - RawHash
  - RawHash2
  - UNCALLED
  - minimap2
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# paf-mapping-accuracy-scoring-precision-recall

## Summary

Scores raw-signal mapper PAF outputs (e.g., RawHash2, RawHash, UNCALLED) against minimap2-based ground-truth PAF alignments using UNCALLED's pafstats to compute per-dataset precision, recall, and F1, as done in the RawHash2 evaluation where RawHash2 improves F1 over RawHash by 10.57% on average (up to 20.25%).

## When to use

Use when comparing raw nanopore signal read-mapping tools whose outputs are in PAF format and you need precision/recall/F1 against a sequence-based ground truth such as minimap2 alignments of basecalled reads.

## When NOT to use

- Inputs are basecalled sequence reads (FASTQ/FASTA) rather than raw-signal mapper PAF outputs — use standard sequence-alignment benchmarking instead
- No trusted ground-truth alignment exists for the dataset — pafstats requires a reference PAF to score against
- Evaluating overlap-based all-vs-all tasks (Rawsamble) where no reference genome ground truth is defined

## Inputs

- Ground-truth PAF alignments produced by minimap2 on basecalled reads
- Modified-PAF mapping outputs from raw-signal mappers (RawHash2, RawHash)
- Basecalled FAST5 reads from test datasets (SARS-CoV-2, green algae, human NA12878, E. coli R10.4)

## Outputs

- Per-dataset precision, recall, and F1 values for each mapping tool
- Aggregated accuracy table with RawHash2-vs-RawHash F1 improvement percentages

## How to apply

First basecall the FAST5 reads of each test dataset and align them to the reference genome with minimap2 to produce ground-truth PAF alignments. Then map the same raw signals with each tool under test (e.g., RawHash2 with genome-appropriate presets such as -x viral for SARS-CoV-2, -x sensitive for green algae, -x fast for human NA12878, and -x sensitive --r10 with the UNCALLED4 R10.4.1 k-mer model for E. coli R10.4). Finally, score every tool PAF against the minimap2 ground truth with UNCALLED's pafstats, which computes precision, recall, and F1 per dataset; aggregate per-dataset scores into a table and compute deltas between mappers. The rationale is that minimap2 on basecalled sequences provides a high-quality reference standard against which raw-signal mapping sensitivity and precision can be quantified.

## Related tools

- **UNCALLED** (Provides the pafstats utility that computes precision, recall, and F1 by comparing a mapper's PAF output against the ground-truth PAF) — https://github.com/skovaka/UNCALLED
- **minimap2** (Generates the ground-truth PAF alignments by mapping basecalled reads to the reference genome) — https://github.com/lh3/minimap2
- **RawHash2** (Tool under evaluation; produces modified-PAF mapping outputs from raw nanopore signals that are scored for precision/recall/F1) — https://github.com/CMU-SAFARI/RawHash
- **RawHash** (Baseline raw-signal mapper whose PAF outputs are scored for comparison against RawHash2) — https://github.com/CMU-SAFARI/RawHash

## Evaluation signals

- Per-dataset precision, recall, and F1 values are present for every dataset and every mapper in the output table
- Mean RawHash2-over-RawHash F1 delta falls within reasonable agreement of the reported 10.57% average improvement
- Maximum RawHash2-over-RawHash F1 delta falls within reasonable agreement of the reported 20.25% maximum improvement
- All PAF outputs are in the same PAF template format so pafstats can parse them consistently
- Ground-truth PAFs cover all reads of each dataset (minimap2 on basecalled FAST5 reads against the same reference used for signal mapping)

## Limitations

- UNCALLED can only be applied to legacy r9.4.1 data, so pafstats-based scoring of UNCALLED-style outputs targets R9.4-era datasets; for R10.4.1, RawHash2 needs the --r10 flag and the UNCALLED4 k-mer model
- Scores depend on minimap2 as ground truth; disagreements may reflect minimap2's own alignment choices rather than mapper errors
- FAST5 reads compressed with VBZ compression require the ONT vbz_compression HDF5 plugin before basecalling/indexing

## Evidence

- [abstract] RawHash2 provides better F1 accuracy than RawHash: "RawHash2 provides better F1 accuracy (on average by 10.57% and up to 20.25%)"
- [other] pafstats scoring against minimap2 ground truth: [evidence span withheld — anchor status: not_found]
- [readme] RawHash raw-signal mapping definition: "RawHash (and RawHash2) is a hash-based mechanism to map raw nanopore signals to a reference genome in real-time."
- [readme] minimap2 PAF ground-truth generation: "Without any options, minimap2 takes a reference database and a query sequence
file as input and produce approximate mapping, without base-level alignment
(i.e. coordinates are only approximate and no"
