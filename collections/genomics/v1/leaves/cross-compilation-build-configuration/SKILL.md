---
name: cross-compilation-build-configuration
description: Use when building sigfish-haru or HARU binaries for the Kria KV260 (aarch64)
  target from a host machine, or when toggling between software-only and FPGA-accelerated
  builds.
when_to_use_negative:
- Running the prebuilt binary release directly on the Kria board — the Quickstart
  path avoids cross-compilation entirely
- Building for a non-Kria/non-aarch64 target platform, since the build flags (PROCESSOR=aarch64,
  Kria board preset) are platform-specific
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: Sigfish-HARU
  role: Proof-of-concept application built by cross-compilation that demonstrates
    HARU-accelerated squiggle mapping
  repo: https://github.com/beebdev/sigfish-haru
- name: HARU
  role: Verilog HDL accelerator core and user-space device driver sources built in
    Vivado and linked into sigfish-haru
  repo: https://github.com/beebdev/HARU
- name: Vivado 2022.2
  role: Xilinx tool used to synthesize the dtw_accel HDL core and generate the bitstream
    for the Kria KV260
- name: PetaLinux 2021.1
  role: Builds the custom embedded Linux image that the cross-compiled binaries run
    on
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giad046
    title: Efficient real-time selective genome sequencing on resource-constrained
      devices
  source_content_hash: sha256:c39ea2ca2f3401638a424a215467868450e31f661a52ea9e6a0185fe940d7776
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beebdev/sigfish-haru
  tools:
  - HARU
  - PetaLinux 2021.1
  - Sigfish-HARU
  - Vivado 2022.2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 16
---

# cross-compilation-build-configuration

## Summary

Builds the HARU/sigfish-haru software stack for the Kria KV260's Zynq Ultrascale+ MPSoC ARM cores, including CPU-only and FPGA-accelerated variants, using the Xilinx cross-compilation toolchain and version-aligned Vivado/PetaLinux tools.

## When to use

Use when building sigfish-haru or HARU binaries for the Kria KV260 (aarch64) target from a host machine, or when toggling between software-only and FPGA-accelerated builds.

## When NOT to use

- Running the prebuilt binary release directly on the Kria board — the Quickstart path avoids cross-compilation entirely
- Building for a non-Kria/non-aarch64 target platform, since the build flags (PROCESSOR=aarch64, Kria board preset) are platform-specific

## Inputs

- sigfish-haru source repository (cloned recursively with submodules)
- HARU repository HDL sources (Verilog files from hdl/src/, excluding the simulation subdirectory)
- Xilinx Vivado/Vitis 2021.1 toolchain
- device-tree-xlnx checked out at xlnx_rel_v2021.1
- device-tree-compiler (dtc) version 1.5+

## Outputs

- sigfish-cpu binary (software-only sDTW, built without fpga=1)
- sigfish-haru binary (FPGA-accelerated sDTW, built with fpga=1)
- Core accelerator bitstream (headerless .bin) for the Kria KV260

## How to apply

Clone the repositories with the recursive flag (`git clone --recursive https://github.com/beebdev/sigfish-haru`). Install Vitis 2021.1 (which provides the Xilinx cross-compilation toolchain) and checkout device-tree-xlnx at xlnx_rel_v2021.1 to keep tool versions aligned with the Kria platform. For the CPU-only software variant used in runtime-profiling baselines, build sigfish with `make PROCESSOR=aarch64` and WITHOUT the `fpga=1` flag; for the hardware-accelerated variant, include the `fpga=1` flag so the HARU sDTW accelerator driver is integrated. Building the core HDL accelerator itself requires Vivado 2021.1 targeting the Kria KV260 Vision AI Starter Kit with Verilog as the target language and a headerless bin bitstream (-bin_file) enabled, plus dtc version 1.5 or higher.

## Related tools

- **Sigfish-HARU** (Proof-of-concept application built by cross-compilation that demonstrates HARU-accelerated squiggle mapping) — https://github.com/beebdev/sigfish-haru
- **HARU** (Verilog HDL accelerator core and user-space device driver sources built in Vivado and linked into sigfish-haru) — https://github.com/beebdev/HARU
- **Vivado 2022.2** (Xilinx tool used to synthesize the dtw_accel HDL core and generate the bitstream for the Kria KV260)
- **PetaLinux 2021.1** (Builds the custom embedded Linux image that the cross-compiled binaries run on)

## Examples

```
git clone --recursive https://github.com/beebdev/sigfish-haru && cd sigfish-haru && make PROCESSOR=aarch64
```

## Evaluation signals

- CPU-only build completes with make PROCESSOR=aarch64 and produces a sigfish-cpu binary that runs the dtw command without the accelerator
- FPGA build with fpga=1 produces a sigfish-haru binary that successfully loads and uses the HARU sDTW accelerator
- Version alignment check: Vivado/Vitis, device-tree-xlnx checkout (xlnx_rel_v2021.1), and PetaLinux image are all from the 2021.1 toolchain generation
- Generated bitstream is a headerless .bin file (the -bin_file option was ticked in Vivado Bitstream settings)
- Built sigfish-haru binary runs the example command `./sigfish-haru dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf` and emits a PAF output

## Limitations

- Building the core accelerator 'is not intuitive and require proprietary software from AMD Xilinx' (Vivado/Vitis 2021.1)
- Developers with Xilinx tool versions lower than 2020.2 must upgrade to a platform supporting Kria
- dtc must be version 1.5 or higher or the device-tree build may fail

## Evidence

- [readme] The build steps tested and described below uses the 2021.1 version of Xilinx tools (Vivado and PetaLinux image). For developers with versions lower than 2020.2 you will need to update your tools to a platform supporting Kria.: "The build steps tested and described below uses the 2021.1 version of Xilinx tools (Vivado and PetaLinux image)."
- [readme] Sigfish-haru + driver (C, build with cross-compilation toolchain): "Sigfish-haru + driver (C, build with cross-compilation toolchain)"
- [readme] make sure to version to align with other tools: "device-tree-xlnx - make sure to checkout the version to align with other tools."
- [full_text] Run the sDTW squiggle-mapping command with wall-clock timing instrumentation: "./sigfish-haru dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf"
- [readme] Navigate to *Bitstream* and tick on *-bin_file* for headerless bitstream to be generated later.: "Navigate to *Bitstream* and tick on *-bin_file* for headerless bitstream to be generated later."
