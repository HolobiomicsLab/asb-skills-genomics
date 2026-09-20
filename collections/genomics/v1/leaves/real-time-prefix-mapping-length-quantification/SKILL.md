---
name: real-time-prefix-mapping-length-quantification
description: Use when comparing early-termination behavior of real-time raw signal
  mappers (RawHash/RawHash2) on FAST5 test datasets and you need per-dataset mean
  bases/chunks processed before the mapping decision from PAF outputs.
when_to_use_negative:
- PAF outputs come from base-level mappers (e.g., minimap2) that lack the UNCALLED-template
  mapping-decision records with prefix lengths and chunk counts
- The analysis goal is F1 mapping accuracy or throughput comparison rather than early-termination
  sequencing length before the mapping decision
- Signals were not analyzed in real time / early termination was disabled, so no prefix-based
  mapping decision exists in the output
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: RawHash
  role: Baseline raw signal mapper whose early-termination prefix lengths and chunk
    counts are quantified from its PAF output
  repo: https://github.com/CMU-SAFARI/RawHash
- name: RawHash2
  role: Improved mapper whose reduced average sequencing length (~1.9× less than RawHash)
    is the quantity being measured
  repo: https://github.com/CMU-SAFARI/RawHash
- name: UNCALLED
  role: Source of the modified PAF output template used to report mapping information
    (mapping-decision records)
  repo: https://github.com/skovaka/UNCALLED
- name: Sigmap
  role: Also cited as a template source for reporting mapping information in the PAF
    output
  repo: https://github.com/haowenz/sigmap
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae478
    title: 'RawHash2: mapping raw nanopore signals using hash-based seeding and adaptive
      quantization'
  source_content_hash: sha256:05ea31fada07c5361af3bf321fa032ebfdc079b29fbb2d4b0140c57d21eb0073
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMU-SAFARI/RawHash
  tools:
  - RawHash
  - RawHash2
  - Sigmap
  - UNCALLED
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# Real-time prefix mapping length quantification (RawHash vs RawHash2)

## Summary

Quantifies how much signal each raw nanopore signal mapper processes before halting — bases and chunks of the read prefix consumed prior to the mapping decision — by parsing modified PAF mapping outputs (UNCALLED-style template) from RawHash and RawHash2. Used to show that RawHash2 reduces average sequencing length before the mapping decision by ~1.9× over RawHash.

## When to use

Use when comparing early-termination behavior of real-time raw signal mappers (RawHash/RawHash2) on FAST5 test datasets and you need per-dataset mean bases/chunks processed before the mapping decision from PAF outputs.

## When NOT to use

- PAF outputs come from base-level mappers (e.g., minimap2) that lack the UNCALLED-template mapping-decision records with prefix lengths and chunk counts
- The analysis goal is F1 mapping accuracy or throughput comparison rather than early-termination sequencing length before the mapping decision
- Signals were not analyzed in real time / early termination was disabled, so no prefix-based mapping decision exists in the output

## Inputs

- RawHash and RawHash2 PAF mapping outputs (mapping.paf) in the modified PAF format following the UNCALLED template, containing mapping-decision records
- Test FAST5 raw nanopore signal datasets (e.g., d1_sars-cov-2_r94, d3_yeast_r94, d4_green_algae_r94, d5_human_na12878_r94, d6_ecoli_r104)

## Outputs

- bases_chunks_summary.csv with mean bases processed and mean chunks processed per tool and dataset
- Per-dataset and overall RawHash2-vs-RawHash sequencing length reduction ratios

## How to apply

Load the PAF mapping outputs previously produced by RawHash and RawHash2 (mapping.paf in the modified PAF format following the UNCALLED template) for each test FAST5 dataset. For every mapped read, parse the mapping-decision record to recover the read prefix length in bases and the number of signal chunks processed, exploiting early termination (mapping stops once the read prefix can be mapped to the reference). Group per-read values by tool (RawHash vs RawHash2) and dataset (e.g., d1_sars-cov-2_r94, d3_yeast_r94, d4_green_algae_r94, d5_human_na12878_r94, d6_ecoli_r104) and compute mean bases and mean chunks per group with pandas. Compute the sequencing length reduction ratio (RawHash average bases / RawHash2 average bases) per dataset and overall. Validate that the overall reduction falls within a tolerance band around the reported ~1.9× (Section 3.4), which is attributed mainly to RawHash2's improved mapping accuracy enabling quicker mapping decisions, and write bases_chunks_summary.csv.

## Related tools

- **RawHash** (Baseline raw signal mapper whose early-termination prefix lengths and chunk counts are quantified from its PAF output) — https://github.com/CMU-SAFARI/RawHash
- **RawHash2** (Improved mapper whose reduced average sequencing length (~1.9× less than RawHash) is the quantity being measured) — https://github.com/CMU-SAFARI/RawHash
- **UNCALLED** (Source of the modified PAF output template used to report mapping information (mapping-decision records)) — https://github.com/skovaka/UNCALLED
- **Sigmap** (Also cited as a template source for reporting mapping information in the PAF output) — https://github.com/haowenz/sigmap

## Examples

```
import pandas as pd; df = pd.read_csv('mapping.paf', sep='\t', comment='#'); df.groupby(['tool','dataset'])[['bases_processed','chunks_processed']].mean().assign(ratio=lambda d: d.loc['RawHash','bases_processed']/d.loc['RawHash2','bases_processed']).to_csv('bases_chunks_summary.csv')
```

## Evaluation signals

- Overall RawHash2/RawHash average-bases reduction ratio falls within a tolerance band around 1.9, matching the value reported in Section 3.4
- Every mapped read yields a parseable mapping-decision record with non-negative prefix length (bases) and chunk count; zero reads missing these fields
- Per-dataset means are finite and ordered consistently with the paper's claim (RawHash2 bases processed < RawHash bases processed) across evaluated datasets
- bases_chunks_summary.csv contains one row per (tool, dataset) pair covering all five evaluated datasets
- Bases and chunks columns agree directionally (fewer chunks should accompany fewer bases for the same dataset)

## Limitations

- Only mapped reads are included; unmapped reads that exhaust the full signal provide no early-decision prefix lengths and may skew the comparison
- The measurement depends on the modified PAF (UNCALLED template) convention; other PAF conventions will not carry the mapping-decision record
- Reduction ratio is an aggregate; per-dataset ratios may deviate from the overall ~1.9× average

## Evidence

- [methods] PAF template follows UNCALLED/Sigmap convention: "We follow the similar PAF template used in [UNCALLED](https://github.com/skovaka/UNCALLED)"
- [methods] PAF reporting template: "and [Sigmap](https://github.com/haowenz/sigmap) to report the mapping information."
- [readme] RawHash is a real-time raw nanopore signal mapper: "RawHash (and RawHash2) is a hash-based mechanism to map raw nanopore signals to a reference genome in real-time."
- [readme] UNCALLED enables real-time targeted sequencing via raw signal mapping: [evidence span withheld — anchor status: external_doc]
