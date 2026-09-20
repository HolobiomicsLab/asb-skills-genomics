---
name: software-throughput-benchmarking
description: Use when you need a CPU-only software baseline throughput (reads/s) for
  raw-signal squiggle mapping to quantify how much a hardware accelerator like HARU
  improves on software-only Read Until alignment.
when_to_use_negative:
- You are evaluating the HARU hardware-accelerated path itself — this skill covers
  only the software-only (sigfish-cpu / RUscripts-R9 Python) baselines, not sigfish-haru
  runs on the Kria FPGA.
- Your dataset is not raw nanopore signal in BLOW5/FAST5 with R9.4 chemistry — RUscripts-R9
  was specifically modified for BLOW5 format and ONT's R9.4 chemistry.
- You need base-level alignment (e.g., Minimap2 on basecalled FASTQ) rather than raw-signal-domain
  squiggle mapping via sDTW.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: RUscripts-R9
  role: Python software baseline whose end-to-end Read Until mapping throughput is
    timed (~12.52 reads/s on a 10-core i9)
  repo: https://github.com/beebdev/RUscripts-R9
- name: Sigfish-HARU
  role: Provides the optimized CPU-only sigfish-cpu binary used as the fast C software
    baseline for sDTW squiggle mapping
  repo: https://github.com/beebdev/sigfish-haru
- name: HARU
  role: Target accelerator against which the measured software baselines are compared;
    not executed in the software-only benchmark itself
  repo: https://github.com/beebdev/HARU
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giad046
    title: Efficient real-time selective genome sequencing on resource-constrained
      devices
  source_content_hash: sha256:f22a9d4b6cef572c1c6482ea01eb913d0f4f1fcd2a84aaa2adcc1f990899ce76
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beebdev/RUscripts-R9
  tools:
  - HARU
  - RUscripts-R9
  - Sigfish-HARU
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# software-throughput-benchmarking

## Summary

Measures signal mapping throughput (reads/s) of pure software baselines — the RUscripts-R9 Python pipeline and the optimized sigfish-cpu sDTW binary — over the 1.382M-read SARS-CoV-2 dataset, establishing the CPU-only reference points against which the HARU FPGA accelerator is compared.

## When to use

Use when you need a CPU-only software baseline throughput (reads/s) for raw-signal squiggle mapping to quantify how much a hardware accelerator like HARU improves on software-only Read Until alignment.

## When NOT to use

- You are evaluating the HARU hardware-accelerated path itself — this skill covers only the software-only (sigfish-cpu / RUscripts-R9 Python) baselines, not sigfish-haru runs on the Kria FPGA.
- Your dataset is not raw nanopore signal in BLOW5/FAST5 with R9.4 chemistry — RUscripts-R9 was specifically modified for BLOW5 format and ONT's R9.4 chemistry.
- You need base-level alignment (e.g., Minimap2 on basecalled FASTQ) rather than raw-signal-domain squiggle mapping via sDTW.

## Inputs

- 1.382M-read SARS-CoV-2 raw nanopore signal dataset in BLOW5/FAST5 format (Zenodo DOI 10.5281/zenodo.7314838)
- SARS-CoV-2 reference genome FASTA (MN908947.3 / nCoV-2019.reference.fasta)
- RUscripts-R9 repository (Python 3.6+, BLOW5, R9.4 chemistry)
- sigfish-haru repository built CPU-only (sigfish-cpu binary)

## Outputs

- throughput_baselines.csv with measured reads/s for the Python and C software baselines
- output.paf mapping file from each sigfish-cpu dtw run
- wall-clock timing measurements per baseline configuration

## How to apply

Obtain the 1.382-million-read SARS-CoV-2 BLOW5/FAST5 dataset (Zenodo DOI 10.5281/zenodo.7314838) and the MN908947.3 reference genome FASTA, then clone github.com/beebdev/RUscripts-R9 (the modified RUscripts supporting Python 3.6+, BLOW5 format, and R9.4 chemistry) and build github.com/beebdev/sigfish-haru recursively as a CPU-only binary (make without fpga=1). Run the RUscripts-R9 Python pipeline end-to-end over the full read set against the reference and time it; the article reports ~12.52 reads/s for this Python baseline on a 10-core i9 desktop. Run the optimized C baseline as 'sigfish-cpu dtw -g <reference.fasta> -s <reads.blow5> > output.paf' and time it; the reported optimized C throughput is 162.29 reads/s on the 10-core desktop and ~432 reads/s on a 36-core Intel Xeon server. Aggregate measured reads/s into throughput_baselines.csv, flagging hardware-dependent deviations for expert review since throughput is strongly hardware-dependent.

## Related tools

- **RUscripts-R9** (Python software baseline whose end-to-end Read Until mapping throughput is timed (~12.52 reads/s on a 10-core i9)) — https://github.com/beebdev/RUscripts-R9
- **Sigfish-HARU** (Provides the optimized CPU-only sigfish-cpu binary used as the fast C software baseline for sDTW squiggle mapping) — https://github.com/beebdev/sigfish-haru
- **HARU** (Target accelerator against which the measured software baselines are compared; not executed in the software-only benchmark itself) — https://github.com/beebdev/HARU

## Examples

```
./sigfish-cpu dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf
```

## Evaluation signals

- throughput_baselines.csv exists with positive reads/s values for both the Python and C baselines
- Optimized C baseline throughput is order-of-magnitude consistent with the reported 162.29 reads/s on a 10-core desktop (and ~432 reads/s on a 36-core Xeon server), with hardware deviations flagged
- Python baseline throughput is order-of-magnitude consistent with the reported ~12.52 reads/s for RUscripts-R9
- sigfish-cpu dtw run emits a non-empty output.paf against the nCoV-2019/MN908947.3 reference
- Timing is computed over the full 1.382M-read dataset, not a subset, so throughput denominators match the reported comparison

## Limitations

- Software throughput is strongly hardware-dependent: the same C baseline gives 162.29 reads/s on a 10-core i9 desktop but ~432 reads/s on a 36-core Xeon server, so deviations require expert review.
- The optimized C implementation already includes multithreading optimizations; comparing against unoptimized software inflates apparent accelerator gains (HARU is ~85x faster than unoptimized but ~2.5x faster than the optimized 36-core version).
- The CPU-only build must not enable hardware acceleration (make without fpga=1); the sigfish-cpu path does not use the FPGA accelerator.

## Evidence

- [abstract] HARU comparison baseline: "HARU on a Xilinx FPGA embedded with a 4-core ARM processor is around 2.5× faster than a highly optimized multithreaded software version"
- [readme] CPU baseline invocation: "./sigfish-cpu dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf"
- [methods] Example dtw mapping run: "./sigfish-haru dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf"
- [discussion] RUscripts-R9 definition: [evidence span withheld — anchor status: not_found]
- [other] CPU-only build: [evidence span withheld — anchor status: not_found]
