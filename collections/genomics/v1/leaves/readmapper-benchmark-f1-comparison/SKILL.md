---
name: readmapper-benchmark-f1-comparison
description: Use when comparing raw-signal mapping tools (e.g., RawHash2 vs. RawHash
  vs. UNCALLED) on the same FAST5 datasets and you need precision/recall/F1 against
  a minimap2-derived ground truth.
when_to_use_negative:
- When evaluating basecalled-read mappers directly (minimap2 itself is the ground-truth
  generator here, not a benchmark subject).
- When no trusted basecalling/reference is available to produce minimap2 ground truth,
  since all F1 scores are relative to that standard.
- For overlap/assembly quality assessment (Rawsamble all-vs-all overlapping outputs
  are not scored by this precision/recall/F1 workflow).
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: RawHash2
  role: Raw-signal mapper under evaluation; indexes the reference genome into ref.ind
    and produces modified-PAF mapping outputs per dataset preset.
  repo: https://github.com/CMU-SAFARI/RawHash
- name: RawHash
  role: Baseline mapper (release v1.0) run on the same references and datasets to
    quantify RawHash2's F1 improvement.
  repo: https://github.com/CMU-SAFARI/RawHash
- name: minimap2
  role: Maps basecalled reads to the reference genome to generate the ground-truth
    PAF alignments used as the scoring standard.
  repo: https://github.com/lh3/minimap2
- name: UNCALLED
  role: Provides pafstats, used to score each tool PAF against the minimap2 ground
    truth and compute precision, recall, and F1; its PAF template is also followed
    by RawHash2.
  repo: https://github.com/skovaka/UNCALLED
- name: Sigmap
  role: Source of the R9.4 segmentation parameters and PAF reporting template referenced
    in RawHash2's mapping outputs.
  repo: https://github.com/haowenz/sigmap
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae478
    title: 'RawHash2: mapping raw nanopore signals using hash-based seeding and adaptive
      quantization'
  source_content_hash: sha256:ae4c1a45cdd21bc7681c7007b54880515d20aed991720649023d3067ca2250cb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMU-SAFARI/RawHash
  tools:
  - RawHash
  - RawHash2
  - Sigmap
  - UNCALLED
  - minimap2
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# readmapper-benchmark-f1-comparison

## Summary

Benchmark raw-signal nanopore read mappers by scoring their PAF outputs against minimap2-based ground-truth alignments using UNCALLED's pafstats to compute per-dataset precision, recall, and F1, as done to show RawHash2 improves F1 over RawHash by 10.57% on average (up to 20.25%).

## When to use

Use when comparing raw-signal mapping tools (e.g., RawHash2 vs. RawHash vs. UNCALLED) on the same FAST5 datasets and you need precision/recall/F1 against a minimap2-derived ground truth.

## When NOT to use

- When evaluating basecalled-read mappers directly (minimap2 itself is the ground-truth generator here, not a benchmark subject).
- When no trusted basecalling/reference is available to produce minimap2 ground truth, since all F1 scores are relative to that standard.
- For overlap/assembly quality assessment (Rawsamble all-vs-all overlapping outputs are not scored by this precision/recall/F1 workflow).

## Inputs

- FAST5 (or POD5/SLOW5/BLOW5) raw nanopore signal read files per test dataset
- Reference genome FASTA files (SARS-CoV-2, green algae, human NA12878, E. coli R10.4)
- Basecalled sequence reads for ground-truth generation
- Raw-signal k-mer pore models (e.g., legacy R9.4 450bps 6-mer template model, UNCALLED4 R10.4.1 k-mer model)

## Outputs

- Ground-truth PAF alignments from minimap2
- Per-tool modified-PAF mapping outputs for each dataset
- Per-dataset precision, recall, and F1 table for each mapper
- Mean and maximum RawHash2-over-RawHash F1 improvement values

## How to apply

First basecall the FAST5 reads from each test dataset and map them to the reference genome with minimap2 to produce ground-truth PAF alignments. Then build raw-signal indexes (e.g., rawhash2 ref.ind using the legacy R9.4 450bps 6-mer template model with 32 threads) and run each mapper with genome-appropriate presets: -x viral for SARS-CoV-2, -x sensitive for green algae, -x fast for human NA12878, and -x sensitive --r10 with the UNCALLED4 R10.4.1 k-mer model for the E. coli R10.4 set. Run RawHash v1 (release v1.0) on the same references and datasets for comparison. Finally, score every tool PAF against the minimap2 ground truth with UNCALLED's pafstats to obtain per-dataset precision, recall, and F1, aggregate into a table, and compute RawHash2's F1 delta over RawHash. The rationale is that minimap2 basecalled-sequence mapping provides a trusted reference standard, while pafstats measures how closely each raw-signal PAF (built on the UNCALLED/Sigmap PAF template) matches it.

## Related tools

- **RawHash2** (Raw-signal mapper under evaluation; indexes the reference genome into ref.ind and produces modified-PAF mapping outputs per dataset preset.) — https://github.com/CMU-SAFARI/RawHash
- **RawHash** (Baseline mapper (release v1.0) run on the same references and datasets to quantify RawHash2's F1 improvement.) — https://github.com/CMU-SAFARI/RawHash
- **minimap2** (Maps basecalled reads to the reference genome to generate the ground-truth PAF alignments used as the scoring standard.) — https://github.com/lh3/minimap2
- **UNCALLED** (Provides pafstats, used to score each tool PAF against the minimap2 ground truth and compute precision, recall, and F1; its PAF template is also followed by RawHash2.) — https://github.com/skovaka/UNCALLED
- **Sigmap** (Source of the R9.4 segmentation parameters and PAF reporting template referenced in RawHash2's mapping outputs.) — https://github.com/haowenz/sigmap

## Examples

```
minimap2 -ax map-ont ref.fa ont.fq.gz > aln.sam
```

## Evaluation signals

- Per-dataset precision, recall, and F1 values must be present for every dataset and every evaluated tool.
- Mean RawHash2-over-RawHash F1 delta should be in reasonable agreement with the reported 10.57% average improvement.
- Maximum RawHash2-over-RawHash F1 delta should be in reasonable agreement with the reported 20.25% maximum improvement.
- All tool outputs must be valid PAF files following the UNCALLED/Sigmap PAF template so pafstats can parse them.
- Ground truth must come from minimap2 mappings of basecalled reads on the identical reference genome used by the raw-signal mappers.

## Limitations

- UNCALLED can only be applied to legacy r9.4.1 data; R10.4.1 data requires different handling (RawHash2 adds R10.4 support).
- F1 scores are only as reliable as the minimap2 basecalled ground truth; results are relative to minimap2's mapping decisions, not absolute biological truth.
- Different datasets require different mapping presets (-x viral, -x sensitive, -x fast, -x sensitive --r10), so comparisons must keep presets genome-appropriate and consistent.

## Evidence

- [abstract] F1 accuracy improvement: "RawHash2 provides better F1 accuracy (on average by 10.57% and up to 20.25%)"
- [methods] PAF template: "We follow the similar PAF template used in [UNCALLED](https://github.com/skovaka/UNCALLED) and [Sigmap](https://github.com/haowenz/sigmap) to report the mapping information."
- [intro] raw signal mapping: "RawHash (and RawHash2) is a hash-based mechanism to map raw nanopore signals to a reference genome in real-time."
- [readme] minimap2 ground truth: [evidence span withheld — anchor status: external_doc]
- [readme] UNCALLED scope: [evidence span withheld — anchor status: external_doc]
