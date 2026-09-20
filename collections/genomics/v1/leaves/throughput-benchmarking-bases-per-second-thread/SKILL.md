---
name: throughput-benchmarking-bases-per-second-thread
description: Use when comparing wall-clock throughput of raw signal mapping tools
  on R9.4.1 FAST5 datasets and you need per-thread mapped bases/second and speedup
  ratios over a baseline mapper.
when_to_use_negative:
- Evaluating mapping accuracy (F1 score) rather than speed — the accuracy comparison
  is a separate measurement and should not be inferred from throughput runs.
- Mapping R10.4 flow cell data — UNCALLED can only be applied to legacy r9.4.1 data,
  so cross-version throughput comparisons would be invalid.
- When the goal is de novo overlap generation (Rawsamble all-vs-all overlapping) rather
  than reference-guided signal mapping throughput.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: RawHash2
  role: Test mapper whose throughput is measured; also used to index reference genomes
    with the R9.4.1 pore model
  repo: https://github.com/CMU-SAFARI/RawHash
- name: RawHash
  role: Baseline mapper (v1.0) run with default parameters for speedup comparison
  repo: https://github.com/CMU-SAFARI/RawHash
- name: UNCALLED
  role: Baseline raw signal mapper run with default parameters for speedup comparison
  repo: https://github.com/skovaka/UNCALLED
- name: Sigmap
  role: Baseline streaming signal mapper run with default parameters for speedup comparison
  repo: https://github.com/haowenz/sigmap
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae478
    title: 'RawHash2: mapping raw nanopore signals using hash-based seeding and adaptive
      quantization'
  source_content_hash: sha256:4f34ae85c9efa9d2ec306a5ca17ed35ce5b8a7303255228b854622297e7c5027
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMU-SAFARI/RawHash
  tools:
  - RawHash
  - RawHash2
  - Sigmap
  - UNCALLED
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# throughput-benchmarking-bases-per-second-thread

## Summary

Benchmark raw nanopore signal mappers (RawHash2 vs RawHash, UNCALLED, Sigmap) by timing each tool on identical FAST5 datasets and reference genomes, then computing mapped bases per second per thread from PAF outputs to derive speedup ratios.

## When to use

Use when comparing wall-clock throughput of raw signal mapping tools on R9.4.1 FAST5 datasets and you need per-thread mapped bases/second and speedup ratios over a baseline mapper.

## When NOT to use

- Evaluating mapping accuracy (F1 score) rather than speed — the accuracy comparison is a separate measurement and should not be inferred from throughput runs.
- Mapping R10.4 flow cell data — UNCALLED can only be applied to legacy r9.4.1 data, so cross-version throughput comparisons would be invalid.
- When the goal is de novo overlap generation (Rawsamble all-vs-all overlapping) rather than reference-guided signal mapping throughput.

## Inputs

- R9.4.1 raw nanopore signal FAST5 datasets (e.g., test/data/d1_sars-cov-2_r94/fast5_files)
- Reference genome FASTA files (e.g., ref.fasta)
- ONT pore model file for indexing (legacy_r9.4_180mv_450bps_6mer/template_median68pA.model)
- Thread count parameter (e.g., 32 threads) applied uniformly to all runs

## Outputs

- PAF mapping output per genome/tool combination (e.g., mapping.paf)
- Throughput table with mapped bases per second per thread and speedup ratios (throughput.tsv)

## How to apply

Index each reference genome with rawhash2 using the R9.4.1 pore model (extern/kmer_models/legacy/legacy_r9.4_180mv_450bps_6mer/template_median68pA.model) and 32 threads to produce ref.ind. Run RawHash2 with default parameters on each FAST5 dataset (e.g., rawhash2 -t 32 ref.ind test/data/d1_sars-cov-2_r94/fast5_files > mapping.paf), applying the genome-appropriate preset (viral, sensitive for <500M bases, fast for >500M bases) where required, while running RawHash, UNCALLED, and Sigmap with their default parameters on the same datasets and references. Record wall-clock runtime for each genome/tool run, parse each PAF output, and compute mapped bases per second per thread for every genome/tool combination. Finally compute RawHash2 speedup ratios over RawHash, UNCALLED, and Sigmap per genome and on average, and save the table (e.g., throughput.tsv). Rationale: holding reference, input FAST5 files, and thread count constant isolates mapper efficiency; the expected result is RawHash2 providing average speedups of 26.5x over UNCALLED, 19.2x over Sigmap, and 4.0x over RawHash.

## Related tools

- **RawHash2** (Test mapper whose throughput is measured; also used to index reference genomes with the R9.4.1 pore model) — https://github.com/CMU-SAFARI/RawHash
- **RawHash** (Baseline mapper (v1.0) run with default parameters for speedup comparison) — https://github.com/CMU-SAFARI/RawHash
- **UNCALLED** (Baseline raw signal mapper run with default parameters for speedup comparison) — https://github.com/skovaka/UNCALLED
- **Sigmap** (Baseline streaming signal mapper run with default parameters for speedup comparison) — https://github.com/haowenz/sigmap

## Examples

```
rawhash2 -d ref.ind -p extern/kmer_models/legacy/legacy_r9.4_180mv_450bps_6mer/template_median68pA.model -t 32 ref.fasta && rawhash2 -t 32 ref.ind test/data/d1_sars-cov-2_r94/fast5_files > mapping.paf
```

## Evaluation signals

- Computed average speedup ratios reproduce the reported values: 26.5x vs UNCALLED, 19.2x vs Sigmap, 4.0x vs RawHash (within reasonable tolerance).
- Per-genome RawHash2 speedup vs RawHash falls within the reported range (on average 4.0x, up to 9.9x).
- Every genome/tool combination was run on the same FAST5 inputs, reference, and thread count (32), so throughput values are directly comparable.
- Each PAF output parses cleanly and yields a nonzero mapped-bases count per second per thread; PAF format follows the UNCALLED/Sigmap template.
- Wall-clock runtimes are recorded for every run before ratio computation, so throughput is not estimated post hoc.

## Limitations

- UNCALLED can only be applied to legacy r9.4.1 data, so the benchmark is restricted to R9.4.1 signals and cannot cover R10.4 flow cell versions.
- RawHash2's minimizer seeding mode is around 3X faster but slightly less accurate than non-minimizer mode, so throughput figures depend on the chosen mode/preset.
- Presets are genome-size dependent (viral, sensitive for <500M bases, fast for >500M bases); using the wrong preset changes measured throughput.
- RawHash2 can index on-the-fly during mapping, but the benchmark workflow explicitly pre-generates ref.ind, so timing excludes index construction.

## Evidence

- [abstract] Abstract throughput claim: "better throughput (on average by 4.0× and up to 9.9×) than RawHash"
- [readme] Methods indexing workflow: "Below is an example that generates an index file `ref.ind` for the reference genome `ref.fasta` using a certain k-mer model located under `extern` and `32` threads."
- [readme] Readme mapping example: "rawhash2 -t 32 ref.ind test/data/contamination/fast5_files/Min*.fast5 test/data/d1_sars-cov-2_r94/fast5_files > mapping.paf"
- [methods] PAF format alignment with baselines: "We follow the similar PAF template used in [UNCALLED](https://github.com/skovaka/UNCALLED) and [Sigmap](https://github.com/haowenz/sigmap) to report the mapping information."
- [readme] UNCALLED scope restriction: [evidence span withheld — anchor status: external_doc]
