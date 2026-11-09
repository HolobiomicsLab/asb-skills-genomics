---
name: read-until-alignment-profiling
description: Use when benchmarking software signal-alignment (sDTW) runtimes against
  reference length, or when needing quantitative evidence that CPU squiggle mapping
  becomes a computational bottleneck for Read Until and justifies FPGA acceleration.
when_to_use_negative:
- The goal is to measure the HARU FPGA-accelerated sDTW itself rather than the software
  baseline — this skill deliberately times sigfish-cpu only
- Input reads are already basecalled and alignment is in base space (e.g., Minimap2);
  this skill profiles raw-signal-domain sDTW squiggle mapping
- Only a single reference length is available, since power-law exponent fitting requires
  multiple increasing reference lengths
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Sigfish-HARU
  role: Provides the sigfish-cpu binary used to time single-thread CPU sDTW alignments
    at each reference length
  repo: https://github.com/beebdev/sigfish-haru
- name: HARU
  role: FPGA sDTW accelerator whose performance advantage over the CPU baseline grows
    with reference length; source of the comparison context
  repo: https://github.com/beebdev/HARU
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giad046
    title: Efficient real-time selective genome sequencing on resource-constrained
      devices
  source_content_hash: sha256:240d18208485ca52207bbf80345eb0178f59c6fb5aa0cd82d533f9aa420ed7f7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beebdev/sigfish-haru
  tools:
  - HARU
  - Sigfish-HARU
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# read-until-alignment-profiling

## Summary

Profile the single-thread CPU execution time of the sigfish sDTW (subsequence dynamic time warping) squiggle-mapping step as reference sequence length increases, establishing the empirical superlinear (quadratic O(MN)) scaling that motivates hardware acceleration with HARU.

## When to use

Use when benchmarking software signal-alignment (sDTW) runtimes against reference length, or when needing quantitative evidence that CPU squiggle mapping becomes a computational bottleneck for Read Until and justifies FPGA acceleration.

## When NOT to use

- The goal is to measure the HARU FPGA-accelerated sDTW itself rather than the software baseline — this skill deliberately times sigfish-cpu only
- Input reads are already basecalled and alignment is in base space (e.g., Minimap2); this skill profiles raw-signal-domain sDTW squiggle mapping
- Only a single reference length is available, since power-law exponent fitting requires multiple increasing reference lengths

## Inputs

- Reference FASTA (e.g., nCoV-2019.reference.fasta or hg38 RFC1 region) from which subregions of increasing length (1 kb to 295 kb) are extracted
- Fixed query read file in BLOW5 format (reads_0_0.blow5)
- sigfish-cpu binary built from the sigfish-haru repository with acceleration disabled

## Outputs

- Runtime-versus-reference-length CSV containing measured single-thread execution times
- Fitted power-law model runtime = c * N^k with empirical complexity exponent k

## How to apply

Build the sigfish-cpu binary from the sigfish-haru repository on aarch64 with hardware acceleration disabled (make PROCESSOR=aarch64 without fpga=1) so alignments run purely on CPU. Extract subregions of increasing length (1 kb to 295 kb) from a reference FASTA such as nCoV-2019.reference.fasta or an hg38 RFC1 region to serve as progressively longer alignment targets. For each reference length, run ./sigfish-cpu dtw with the subregion supplied via -g and the fixed query reads_0_0.blow5 supplied via -s, timing single-thread execution per alignment. Record runtime versus reference length into a CSV and fit a power-law model runtime = c * N^k to estimate the empirical complexity exponent. The rationale is that the CPU-versus-FPGA gap widens with reference length, so measured CPU scaling quantifies the software bottleneck that HARU's FPGA accelerator removes.

## Related tools

- **Sigfish-HARU** (Provides the sigfish-cpu binary used to time single-thread CPU sDTW alignments at each reference length) — https://github.com/beebdev/sigfish-haru
- **HARU** (FPGA sDTW accelerator whose performance advantage over the CPU baseline grows with reference length; source of the comparison context) — https://github.com/beebdev/HARU

## Examples

```
./sigfish-cpu dtw -g reference_subregion_295kb.fasta -s test_data/reads_0_0.blow5 > output.paf
```

## Evaluation signals

- The runtime-versus-reference-length CSV exists and contains measurements for each extracted subregion length
- Runtimes increase monotonically with reference length
- The fitted power-law exponent k falls within [1.8, 2.2], confirming quadratic O(MN) software behavior
- The CPU curve runs without hardware acceleration (no fpga=1 flag), verified by the sigfish-cpu binary being used
- The CPU-versus-FPGA runtime gap widens as reference length increases, matching the expected HARU speedup trend

## Limitations

- Timings are single-thread and platform-specific (aarch64 Kria KV260 ARM cores); absolute runtimes do not transfer to other CPUs
- Building the full accelerator requires proprietary AMD Xilinx tools (Vivado 2021.1/2022.2, PetaLinux 2021.1), though the CPU-only profiling path needs only sigfish-cpu compilation
- The power-law fit assumes clean scaling; cache effects or per-alignment overhead at very short reference lengths may distort the exponent

## Evidence

- [other] Single-thread CPU sDTW query time grows superlinearly (toward O(MN)) with reference length: [evidence span withheld — anchor status: not_found]
- [readme] sigfish-cpu dtw invocation: "./sigfish-cpu dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf"
- [readme] sigfish-cpu described as slow software baseline: "If you wish, you can run the *sigfish-cpu* binary that does not use the hardware accelerator and see how slow it is."
- [discussion] sigfish-haru repository purpose: "Project name: Sigfish-HARUDescription: Source code that demonstrates the proof-of-concept integration of HARU accelerator for squiggle mapping"
