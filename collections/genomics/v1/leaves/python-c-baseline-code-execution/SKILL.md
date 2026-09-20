---
name: python-c-baseline-code-execution
description: Use when a hardware-acceleration claim (e.g., HARU on Xilinx Kria) must
  be benchmarked against software-only signal mapping throughput on nanopore raw signal
  data such as BLOW5/FAST5 reads.
when_to_use_negative:
- The goal is the accelerated HARU path — this skill only covers the CPU/Python software
  baselines (sigfish-cpu and RUscripts-R9), not the FPGA-loaded sigfish-haru binary
- Input reads are already basecalled FASTQ to be mapped with base-domain tools like
  Minimap2 — this skill operates in the raw signal (squiggle) mapping domain via sDTW
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: RUscripts-R9
  role: Modified RUscripts Python pipeline executed end-to-end as the slow (~12.52
    reads/s) software-only baseline
  repo: https://github.com/beebdev/RUscripts-R9
- name: Sigfish-HARU
  role: Source of the sigfish-cpu binary, built CPU-only, run as the optimized C software
    baseline (162.29 reads/s on a 10-core i9 desktop)
  repo: https://github.com/beebdev/sigfish-haru
- name: HARU
  role: Hardware counterpart whose throughput the C baseline is measured to compare
    against
  repo: https://github.com/beebdev/HARU
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giad046
    title: Efficient real-time selective genome sequencing on resource-constrained
      devices
  source_content_hash: sha256:31787f28b0db3363ac5ce7334a6a190f7373de82ceb2c5e09b96c67ec53382b7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beebdev/RUscripts-R9
  tools:
  - HARU
  - RUscripts-R9
  - Sigfish-HARU
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# Python/C baseline code execution

## Summary

Executes the pure software baselines of the HARU paper — the modified RUscripts-R9 Python pipeline and the optimized C implementation (sigfish-cpu dtw) — to measure signal mapping throughput on the 1.382-million-read SARS-CoV-2 dataset, establishing the reference points (≈12.52 reads/s Python, 162.29 reads/s C on a 10-core i9 desktop) against which the FPGA-accelerated HARU is compared.

## When to use

Use when a hardware-acceleration claim (e.g., HARU on Xilinx Kria) must be benchmarked against software-only signal mapping throughput on nanopore raw signal data such as BLOW5/FAST5 reads.

## When NOT to use

- The goal is the accelerated HARU path — this skill only covers the CPU/Python software baselines (sigfish-cpu and RUscripts-R9), not the FPGA-loaded sigfish-haru binary
- Input reads are already basecalled FASTQ to be mapped with base-domain tools like Minimap2 — this skill operates in the raw signal (squiggle) mapping domain via sDTW

## Inputs

- SARS-CoV-2 raw signal dataset in BLOW5/FAST5 format (1.382 million reads, Zenodo DOI 10.5281/zenodo.7314838)
- Reference genome FASTA (MN908947.3)
- RUscripts-R9 repository (github.com/beebdev/RUscripts-R9)
- sigfish-haru repository cloned recursively (github.com/beebdev/sigfish-haru)

## Outputs

- throughput_baselines.csv containing measured reads/s for the Python baseline and the optimized C baseline
- output.paf signal-to-reference mapping file from sigfish-cpu dtw
- Wall-clock timings of both baseline runs

## How to apply

Obtain the 1.382M-read SARS-CoV-2 BLOW5 dataset (Zenodo DOI 10.5281/zenodo.7314838) and the MN908947.3 reference FASTA. Clone beebdev/RUscripts-R9 — the modified RUscripts supporting Python 3.6+, BLOW5 format, and ONT's R9.4 chemistry — and run its Python pipeline end-to-end against the reference, timing the run to compute reads/s. Then clone beebdev/sigfish-haru recursively, build the CPU-only binary without hardware acceleration (make without fpga=1), and run 'sigfish-cpu dtw -g <reference.fasta> -s <reads.blow5> > output.paf' with wall-clock timing. Record both measured throughputs in throughput_baselines.csv; the C baseline is the software-only reference for the HARU comparison, and the README itself provides a slower sigfish-cpu variant for demonstration of relative speed.

## Related tools

- **RUscripts-R9** (Modified RUscripts Python pipeline executed end-to-end as the slow (~12.52 reads/s) software-only baseline) — https://github.com/beebdev/RUscripts-R9
- **Sigfish-HARU** (Source of the sigfish-cpu binary, built CPU-only, run as the optimized C software baseline (162.29 reads/s on a 10-core i9 desktop)) — https://github.com/beebdev/sigfish-haru
- **HARU** (Hardware counterpart whose throughput the C baseline is measured to compare against) — https://github.com/beebdev/HARU

## Examples

```
./sigfish-cpu dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf
```

## Evaluation signals

- throughput_baselines.csv exists with positive reads/s values for both the Python and C baselines
- Measured Python baseline is order-of-magnitude consistent with the reported ~12.52 reads/s for RUscripts-R9
- Measured C baseline is order-of-magnitude consistent with ~162.29 reads/s on a 10-core desktop and ~432 reads/s on a 36-core Xeon server, with hardware-dependent deviations flagged for expert review
- sigfish-cpu produces a non-empty output.paf mapping file over the full read dataset against MN908947.3

## Limitations

- Baseline throughput is hardware-dependent: 162.29 reads/s was reported on a 10-core i9 desktop, while a 36-core Intel Xeon server reaches ~432 reads/s, so absolute numbers must be flagged for expert review when run on different machines
- The unoptimized RUscripts-R9 Python path is roughly 85x slower than HARU, so its runtime on the 1.382M-read dataset is very long
- Building the sigfish-cpu binary from sigfish-haru requires recursive clone and CPU-only compilation (make without fpga=1); the full HARU accelerator build needs proprietary AMD Xilinx tools (Vivado/PetaLinux 2021.1), which are not needed for the software baseline

## Evidence

- [results] The optimized C implementation of RUscripts achieves a signal mapping throughput of 162.29 reads/s when run on the desktop system with a 10-core i9 processor, serving as the software-only baseline against which HARU is compared.: [evidence span withheld — anchor status: not_found]
- [abstract] HARU is around 85x faster than the existing unoptimized multithreaded software.: "around 85× faster than the existing unoptimized multithreaded software"
- [discussion] The modified RUscripts to support Python 3.6+, BLOW5 format, and ONT's current nanopore chemistry R9.4.: [evidence span withheld — anchor status: not_found]
- [readme] If you wish, you can run the sigfish-cpu binary that does not use the hardware accelerator and see how slow it is.: "you can run the *sigfish-cpu* binary that does not use the hardware accelerator and see how slow it is"
