---
name: cross-tool-speedup-ratio-comparison
description: Use when benchmarking raw signal mappers head-to-head (RawHash2 vs RawHash/UNCALLED/Sigmap)
  on R9.4.1 FAST5 data and you need speedup ratios (26.5x/19.2x/4.0x) or sequencing-length
  reduction (~1.9x).
when_to_use_negative:
- The goal is accuracy (F1) benchmarking rather than speed — that requires a separate
  accuracy evaluation protocol, not throughput ratio computation
- Input data is R10.4 or POD5/SLOW5/BLOW5 — the reported speedup ratios (26.5x/19.2x/4.0x)
  were established for R9.4.1 FAST5 data with default parameters, and UNCALLED can
  only be applied to legacy R9.4.1 data
- The task is all-vs-all overlap generation with Rawsamble (no reference genome) rather
  than reference-guided mapping throughput
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: RawHash2
  role: Primary mapper whose throughput and early-termination length are measured
    and compared against baselines; indexed with the R9.4.1 pore model on 32 threads
  repo: https://github.com/CMU-SAFARI/RawHash
- name: RawHash
  role: Baseline mapper (v1 release) whose throughput and bases/chunks-before-mapping-decision
    are used as the denominator of the speedup and ~1.9x reduction ratios
  repo: https://github.com/CMU-SAFARI/RawHash
- name: UNCALLED
  role: Baseline mapper run with default parameters; also defines the PAF output template
    used to parse mapping-decision records
  repo: https://github.com/skovaka/UNCALLED
- name: Sigmap
  role: Baseline streaming mapper run with default parameters on the same FAST5 datasets
    to compute the 19.2x average speedup
  repo: https://github.com/haowenz/sigmap
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae478
    title: 'RawHash2: mapping raw nanopore signals using hash-based seeding and adaptive
      quantization'
  source_content_hash: sha256:629ceba6e763997c54170b828af6d21d1093e49fdf7cdfd74280debf8bc41620
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMU-SAFARI/RawHash
  tools:
  - RawHash
  - RawHash2
  - Sigmap
  - UNCALLED
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# cross-tool-speedup-ratio-comparison

## Summary

A benchmarking skill for computing normalized speedup ratios of raw nanopore signal mappers: it runs RawHash2 against RawHash, UNCALLED, and Sigmap on identical R9.4.1 FAST5 datasets, parses each tool's PAF output, and derives mapped-bases-per-second-per-thread throughput ratios plus early-termination sequencing-length reduction.

## When to use

Use when benchmarking raw signal mappers head-to-head (RawHash2 vs RawHash/UNCALLED/Sigmap) on R9.4.1 FAST5 data and you need speedup ratios (26.5x/19.2x/4.0x) or sequencing-length reduction (~1.9x).

## When NOT to use

- The goal is accuracy (F1) benchmarking rather than speed — that requires a separate accuracy evaluation protocol, not throughput ratio computation
- Input data is R10.4 or POD5/SLOW5/BLOW5 — the reported speedup ratios (26.5x/19.2x/4.0x) were established for R9.4.1 FAST5 data with default parameters, and UNCALLED can only be applied to legacy R9.4.1 data
- The task is all-vs-all overlap generation with Rawsamble (no reference genome) rather than reference-guided mapping throughput

## Inputs

- R9.4.1 FAST5 raw signal datasets (e.g., d1_sars-cov-2_r94, d3_yeast_r94, d4_green_algae_r94, d5_human_na12878_r94, d6_ecoli_r104)
- Reference genome FASTA files for each dataset
- R9.4.1 6mer pore model file (legacy_r9.4_180mv_450bps_6mer/template_median68pA.model)
- PAF mapping outputs from RawHash2, RawHash, UNCALLED, and Sigmap (UNCALLED-style PAF template)
- Wall-clock runtimes recorded for each tool/genome run

## Outputs

- throughput.tsv: mapped bases per second per thread per genome/tool plus RawHash2 speedup ratios over RawHash, UNCALLED, and Sigmap
- bases_chunks_summary.csv: mean bases and mean signal chunks processed before the mapping decision per tool per dataset, with the RawHash/RawHash2 reduction ratio (~1.9x)

## How to apply

Index each reference genome with rawhash2 using the R9.4.1 pore model (extern/kmer_models/legacy/legacy_r9.4_180mv_450bps_6mer/template_median68pA.model) and 32 threads to produce ref.ind, then run rawhash2 with default parameters on each FAST5 dataset applying the genome-appropriate preset (viral, sensitive for <500M bases, fast for >500M bases). Run RawHash, UNCALLED, and Sigmap with their default parameters on the same FAST5 datasets and references, recording wall-clock runtime per run. Parse each tool's PAF output — which follows the PAF template used in UNCALLED and Sigmap — and compute mapped bases per second per thread for every genome/tool combination, then average the RawHash2-over-baseline ratios and validate them against the reported Figure 2 values (avg 26.5x, 19.2x, 4.0x). For the length-reduction analysis, parse the mapping-decision records in the RawHash/RawHash2 PAFs to recover the read prefix in bases and the number of signal chunks processed before early termination stops the mapping, group by tool and dataset (e.g., d1_sars-cov-2_r94, d3_yeast_r94, d4_green_algae_r94, d5_human_na12878_r94, d6_ecoli_r104), and compute the RawHash/RawHash2 mean-bases ratio, which should fall in a tolerance band around 1.9x.

## Related tools

- **RawHash2** (Primary mapper whose throughput and early-termination length are measured and compared against baselines; indexed with the R9.4.1 pore model on 32 threads) — https://github.com/CMU-SAFARI/RawHash
- **RawHash** (Baseline mapper (v1 release) whose throughput and bases/chunks-before-mapping-decision are used as the denominator of the speedup and ~1.9x reduction ratios) — https://github.com/CMU-SAFARI/RawHash
- **UNCALLED** (Baseline mapper run with default parameters; also defines the PAF output template used to parse mapping-decision records) — https://github.com/skovaka/UNCALLED
- **Sigmap** (Baseline streaming mapper run with default parameters on the same FAST5 datasets to compute the 19.2x average speedup) — https://github.com/haowenz/sigmap

## Examples

```
rawhash2 -t 32 ref.ind test/data/d1_sars-cov-2_r94/fast5_files > mapping.paf
```

## Evaluation signals

- Computed average speedup ratios reproduce the reported Figure 2 values (avg 26.5x vs UNCALLED, 19.2x vs Sigmap, 4.0x vs RawHash, up to 9.9x vs RawHash) within a reasonable tolerance
- RawHash/RawHash2 average bases-processed ratio falls in a tolerance band around 1.9x, matching the Section 3.4 value
- Per-genome ratios are computed for every genome/tool combination (not just aggregate averages) before averaging
- PAF outputs parse cleanly under the UNCALLED/Sigmap PAF template, recovering per-read mapped prefix length in bases and chunk count
- All tools were run on identical FAST5 inputs with default parameters, and rawhash2 indexing used the R9.4.1 6mer pore model with 32 threads

## Limitations

- UNCALLED only applies to legacy R9.4.1 data, so cross-tool throughput ratios are restricted to R9.4.1 FAST5 inputs
- Throughput depends on RawHash2's mode choice: the minimizer seeding mode is around 3X faster but slightly less accurate than non-minimizer mode, and genome-size-dependent presets (sensitive <500M bases, fast >500M bases) must be matched or ratios will be skewed
- Mapping stops once a read prefix maps to the reference (early termination), so throughput is measured as mapped bases per second, not total bases sequenced per second

## Evidence

- [abstract] better throughput (on average by 4.0× and up to 9.9×) than RawHash: [evidence span withheld — anchor status: not_found]
- [methods] RawHash can be used to map reads from **FAST5, POD5, SLOW5, or BLOW5** files to a reference genome in sequence format.: "RawHash can be used to map reads from **FAST5, POD5, SLOW5, or BLOW5** files to a reference genome in sequence format."
- [readme] We follow the similar PAF template used in [UNCALLED](https://github.com/skovaka/UNCALLED) and [Sigmap](https://github.com/haowenz/sigmap) to report the mapping information.: "We follow the similar PAF template used in [UNCALLED](https://github.com/skovaka/UNCALLED) and [Sigmap](https://github.com/haowenz/sigmap) to report the mapping information."
- [readme] When the prefix of reads can be mapped to a reference genome, RawHash will stop mapping and provide the mapping information in PAF format.: "When the prefix of reads can be mapped to a reference genome, RawHash will stop mapping and provide the mapping information in PAF format."
