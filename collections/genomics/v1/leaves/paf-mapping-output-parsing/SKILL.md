---
name: paf-mapping-output-parsing
description: Use when raw-signal mapper outputs are in the UNCALLED-style modified
  PAF format and you need to extract mapped read positions, throughput (mapped bases/sec),
  or bases/chunks processed before the mapping decision.
when_to_use_negative:
- The mapping output is a standard 12-column minimap2 PAF without the UNCALLED-style
  mapping-decision tags — the bases/chunks-parsed fields will not exist
- The goal is base-level alignment quality (CIGAR/identity), which the raw-signal
  mappers' PAF output does not provide without a downstream signal-alignment step
  (DTW as in RawAlign)
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: RawHash2
  role: Produces the mapping.paf outputs being parsed; supports FAST5, POD5, SLOW5,
    and BLOW5 inputs and the R10.4 flow cell
  repo: https://github.com/CMU-SAFARI/RawHash
- name: UNCALLED
  role: Defines the modified PAF output template that RawHash2 follows; its PAF outputs
    are parsed for cross-tool throughput comparison
  repo: https://github.com/skovaka/UNCALLED
- name: Sigmap
  role: Comparison mapper whose PAF-format mapping outputs are parsed for throughput
    and accuracy benchmarking
  repo: https://github.com/haowenz/sigmap
- name: Minimap2
  role: Produces ground-truth sequence-space alignments used as reference when scoring
    PAF mappings for accuracy/F1
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_004
  - task_005
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae478
    title: 'RawHash2: mapping raw nanopore signals using hash-based seeding and adaptive
      quantization'
  source_content_hash: sha256:16ed0c98cf5bc20f4dcf5fc2b56eeb193e0ebdec3c9deef774e392f3114e255e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMU-SAFARI/RawHash
  tools:
  - Minimap2
  - RawHash
  - RawHash2
  - Sigmap
  - UNCALLED
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# paf-mapping-output-parsing

## Summary

Parse and interpret PAF-format mapping outputs produced by raw nanopore signal mappers (RawHash2, RawHash, UNCALLED, Sigmap), which follow the UNCALLED-style modified PAF template that records mapping decisions and early-termination information.

## When to use

Use when raw-signal mapper outputs are in the UNCALLED-style modified PAF format and you need to extract mapped read positions, throughput (mapped bases/sec), or bases/chunks processed before the mapping decision.

## When NOT to use

- The mapping output is a standard 12-column minimap2 PAF without the UNCALLED-style mapping-decision tags — the bases/chunks-parsed fields will not exist
- The goal is base-level alignment quality (CIGAR/identity), which the raw-signal mappers' PAF output does not provide without a downstream signal-alignment step (DTW as in RawAlign)

## Inputs

- RawHash2/RawHash/UNCALLED/Sigmap mapping output files (mapping.paf) in the UNCALLED-style modified PAF format
- R9.4.1 and R10.4.1 FAST5 raw signal datasets (e.g., d1_sars-cov-2_r94, d6_ecoli_r104)
- minimap2 ground-truth alignments in the same modified PAF format (for accuracy/F1 computation)
- Wall-clock runtime measurements per tool/dataset run

## Outputs

- throughput.tsv with mapped bases/sec/thread per genome/tool combination and RawHash2 speedup ratios
- bases_chunks_summary.csv with mean bases and chunks processed before the mapping decision, per tool and dataset
- Mapping accuracy/F1 scores from PAF-vs-minimap2-ground-truth comparison

## How to apply

RawHash2 and the comparison tools emit mapping results in a PAF file that follows the UNCALLED PAF template, so parsing must account for the extra mapping-decision fields rather than only the 12 standard PAF columns. For throughput benchmarks (e.g., R9.4.1 FAST5 datasets like d1_sars-cov-2_r94), parse each tool's mapping.paf and compute mapped bases per second per thread, then form RawHash2 speedup ratios over UNCALLED, Sigmap, and RawHash. For early-termination analysis (Table 2), recover from every mapped read the read-prefix length in bases and the number of signal chunks processed before RawHash/RawHash2 stopped mapping, group by tool and dataset (e.g., d3_yeast_r94, d5_human_na12878_r94), and compute the RawHash/RawHash2 reduction ratio (~1.9x expected). For accuracy evaluation, compare the PAF mappings against minimap2 ground-truth alignments reported in the same modified PAF format to compute F1/accuracy. Use pandas or equivalent tabular parsing to aggregate per-dataset and average metrics into TSV/CSV tables.

## Related tools

- **RawHash2** (Produces the mapping.paf outputs being parsed; supports FAST5, POD5, SLOW5, and BLOW5 inputs and the R10.4 flow cell) — https://github.com/CMU-SAFARI/RawHash
- **UNCALLED** (Defines the modified PAF output template that RawHash2 follows; its PAF outputs are parsed for cross-tool throughput comparison) — https://github.com/skovaka/UNCALLED
- **Sigmap** (Comparison mapper whose PAF-format mapping outputs are parsed for throughput and accuracy benchmarking) — https://github.com/haowenz/sigmap
- **Minimap2** (Produces ground-truth sequence-space alignments used as reference when scoring PAF mappings for accuracy/F1) — https://github.com/lh3/minimap2

## Examples

```
rawhash2 -t 32 ref.ind test/data/d1_sars-cov-2_r94/fast5_files > mapping.paf
```

## Evaluation signals

- Average RawHash2 speedup ratios over UNCALLED, Sigmap, and RawHash reproduce the reported ~26.5x, 19.2x, and 4.0x values within tolerance
- Computed average sequencing-length reduction (RawHash bases / RawHash2 bases) falls within a tolerance band around 1.9x
- Every parsed PAF line yields a valid (query, reference, mapping-decision) record; no rows dropped by the parser
- Chained seeds from the parsed output reproduce a valid mapping entry in the generated PAF-format output
- Per-dataset mean bases/chunks values computed per group (e.g., d1_sars-cov-2_r94 vs d5_human_na12878_r94) are internally consistent with per-read sums

## Limitations

- UNCALLED can only be applied to legacy r9.4.1 data, so cross-tool PAF parsing for R10.4 comparisons is limited to RawHash2
- FAST5 files compressed with VBZ compression from Nanopore require the HDF5 plugin for reading, which affects downstream parsing pipelines
- The PAF is approximate mapping without base-level alignment (no CIGAR), so accuracy must be judged against ground truth alignments rather than alignment identities

## Evidence

- [methods] We follow the similar PAF template used in [UNCALLED] and [Sigmap] to report the mapping information.: "We follow the similar PAF template used in [UNCALLED](https://github.com/skovaka/UNCALLED) and [Sigmap](https://github.com/haowenz/sigmap) to report the mapping information."
- [readme] RawHash2 is a hash-based mechanism to map raw nanopore signals to a reference genome in real-time.: "RawHash (and RawHash2) is a hash-based mechanism to map raw nanopore signals to a reference genome in real-time."
- [other] parse the mapping-decision record to recover the length of the read prefix (in bases) and the number of signal chunks: [evidence span withheld — anchor status: not_found]
- [methods] RawHash can be used to map reads from FAST5, POD5, SLOW5, or BLOW5 files to a reference genome in sequence format.: "RawHash can be used to map reads from **FAST5, POD5, SLOW5, or BLOW5** files to a reference genome in sequence format."
