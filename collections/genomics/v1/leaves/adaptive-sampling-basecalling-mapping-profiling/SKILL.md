---
name: adaptive-sampling-basecalling-mapping-profiling
description: Use when benchmarking a signal-level RNA classification method against
  basecalling-plus-mapping adaptive sampling, or when building a read-until reject
  decision that requires mapping reads to a curated reference (e.g., protein_coding
  mRNA transcripts).
when_to_use_negative:
- When a direct raw-signal classifier (RISER) is available, since the paper shows
  it is faster than GPU-accelerated basecalling and mapping whether run on GPU or
  CPU
- When the sequencing kit or chemistry differs from R9.4.1 RNA002 without an available
  basecaller model configuration for the new chemistry
- When post-sequencing bioinformatic filtering of already-sequenced reads (not real-time
  rejection) is the goal
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: Dorado
  role: GPU basecalling server used in the adaptive sampling pipeline (RNA R9.4.1
    fast configuration) to convert raw DRS signals to sequence before mapping
- name: minimap2 (v2.17)
  role: Maps basecalled reads against the GENCODE v34 protein_coding transcriptome
    with '-ax map-ont' to produce the adaptive sampling reject decision
- name: BoostNano
  role: Pre-trims the sequencing adapter and poly(A) tail from raw fast5 signals so
    adaptive sampling and RISER receive comparable inputs
  repo: https://github.com/haotianteng/BoostNano
- name: nanoporetech/read_until_api
  role: Provides the mechanism to obtain read data from MinKNOW in real time and return
    unblock/reject calls during adaptive sampling
  repo: nanoporetech/read_until_api
- name: samtools (v1.10)
  role: Post-mapping filtering of mappings (option -F 2324) to retain only high-confidence
    alignments
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-48673-8
    title: Biochemical-free enrichment or depletion of RNA classes in real-time during
      direct RNA sequencing with RISER
  source_content_hash: sha256:68c54ba04d8a089fcead65f2118eb3a03c95f1ee2acc578788238afb7a2d2261
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haotianteng/BoostNano
  tools:
  - BoostNano
  - Dorado
  - PyTorch
  - RISER
  - minimap2
  - minimap2 (v2.17)
  - nanoporetech/read_until_api
  - samtools (v1.10)
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# adaptive-sampling-basecalling-mapping-profiling

## Summary

Implements and benchmarks ONT's sequence-based adaptive sampling reject decision pipeline — real-time basecalling (e.g., Dorado) followed by read mapping (minimap2) against a filtered reference transcriptome — as the computational baseline against which direct-signal classifiers like RISER are compared for speed and classification behavior.

## When to use

Use when benchmarking a signal-level RNA classification method against basecalling-plus-mapping adaptive sampling, or when building a read-until reject decision that requires mapping reads to a curated reference (e.g., protein_coding mRNA transcripts).

## When NOT to use

- When a direct raw-signal classifier (RISER) is available, since the paper shows it is faster than GPU-accelerated basecalling and mapping whether run on GPU or CPU
- When the sequencing kit or chemistry differs from R9.4.1 RNA002 without an available basecaller model configuration for the new chemistry
- When post-sequencing bioinformatic filtering of already-sequenced reads (not real-time rejection) is the goal

## Inputs

- Raw nanopore DRS fast5 signals (RNA002/R9.4.1), pre-trimmed with BoostNano
- Dorado basecalling server with the RNA R9.4.1 fast configuration
- Reference transcriptome: GENCODE v34 (GRCh38.p13) filtered to protein_coding transcripts
- ReadUntil API connection to a MinION Mk1B / MinKNOW server

## Outputs

- Per-read classification decision (mRNA vs non-mRNA) from mapping coordinates
- Per-read decision latency measurements (e.g., ~5.8 s per signal on GPU)
- Timing summary table (e.g., riser_vs_as_timing.csv) and speed-comparison plots (e.g., speed_comparison.png)
- Mapped BAM/SAM from minimap2 '-ax map-ont --secondary=no', optionally filtered with samtools -F 2324

## How to apply

First pre-trim the sequencing adapter and variable-length poly(A) tail from raw DRS fast5 signals with BoostNano so both adaptive sampling and the competing method operate on comparable trimmed input. Run adaptive sampling by streaming each signal to a basecalling server (e.g., Dorado with the RNA R9.4.1 fast configuration) and mapping the resulting reads to a filtered reference (GENCODE v34, GRCh38.p13, restricted to protein_coding transcripts) using minimap2 with '-ax map-ont', issuing the reject/unblock decision once a signal maps to the target class. Record per-read wall-clock time from signal receipt to mapping-based decision. For fair comparison, fix input length (e.g., trim all signals to 6.2 s) and repeat the benchmark on a single GPU and a single CPU for both pipelines. Compare per-read and summary classification times and cumulative fraction of signals classified versus time against the direct-signal classifier's timing.

## Related tools

- **Dorado** (GPU basecalling server used in the adaptive sampling pipeline (RNA R9.4.1 fast configuration) to convert raw DRS signals to sequence before mapping)
- **minimap2 (v2.17)** (Maps basecalled reads against the GENCODE v34 protein_coding transcriptome with '-ax map-ont' to produce the adaptive sampling reject decision)
- **BoostNano** (Pre-trims the sequencing adapter and poly(A) tail from raw fast5 signals so adaptive sampling and RISER receive comparable inputs) — https://github.com/haotianteng/BoostNano
- **nanoporetech/read_until_api** (Provides the mechanism to obtain read data from MinKNOW in real time and return unblock/reject calls during adaptive sampling) — nanoporetech/read_until_api
- **samtools (v1.10)** (Post-mapping filtering of mappings (option -F 2324) to retain only high-confidence alignments)

## Examples

```
minimap2 -ax map-ont --secondary=no gencode_v34_protein_coding.fa basecalled_reads.fastq > mapped.sam
```

## Evaluation signals

- Per-signal adaptive sampling decision time should be longer than RISER's classification time on both GPU and CPU (paper: AS averaged ~5.8 s per 6.2 s signal on GPU)
- Expected speed ratios when compared to RISER: ~44× slower on GPU vs GPU, ~371× slower when RISER runs on CPU, and CPU-RISER ~16× faster than GPU-AS
- All minimap2 alignments should use '-ax map-ont --secondary=no' and unmapped/secondary/supplementary/reverse alignments removed via samtools -F 2324
- All input signals must be equal length (6.2 s) and adapter/poly(A)-trimmed by BoostNano before timing comparisons
- Cumulative fraction of signals classified versus time should show basecalling-plus-mapping trailing direct-signal classification

## Limitations

- Adaptive sampling requires basecalling plus mapping, making it more computationally intensive and unable to run on CPU alone, unlike direct-signal classification
- Mapping requires an explicit predefined reference target list; the paper notes no prior approach supports enrichment/depletion 'without requiring the explicit definition of specific targets'
- In playback mode, rejected signals are only clipped rather than physically ejected, so effects must be inferred from read-length distributions

## Evidence

- [abstract] RISER is faster than GPU-accelerated basecalling and mapping on both GPU and CPU: "Furthermore, using a GPU or a CPU, RISER is faster than GPU-accelerated basecalling and mapping"
- [methods] minimap2 mapping parameters: "mapped to the GENCODE reference transcriptome (release 34, assembly GRCh38.p13) with minimap233 (v2.17) using options -ax map-ont --secondary=no"
- [methods] BoostNano pre-trimming rationale: "the sequencing adapter and poly(A) tail were removed from the raw nanopore signals using BoostNano32"
- [readme] read_until_api README description: [evidence span withheld — anchor status: external_doc]
- [intro] direct signal classification advantage: "unlike real-time basecalling, RISER can be run with just a CPU"
