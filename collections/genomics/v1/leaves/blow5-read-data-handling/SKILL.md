---
name: blow5-read-data-handling
description: Use when the analysis consumes Oxford Nanopore raw signal reads in BLOW5
  (or FAST5) format and requires squiggle-to-reference mapping with sigfish-haru or
  RUscripts-R9 against a reference genome FASTA.
when_to_use_negative:
- The data is already basecalled (FASTQ/BAM) and the workflow only needs sequence-domain
  mapping with Minimap2 rather than raw-signal sDTW mapping
- The reads are in FAST5 and the pipeline explicitly requires BLOW5; convert first
  via sigfish tooling rather than feeding FAST5 directly to these commands
- A selective-sequencing Read Until run on the Kria board is intended but the HARU
  accelerator firmware has not been loaded (xmutil loadapp haru-dtw-firmware); a CPU-only
  baseline run will not measure hardware acceleration
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: Sigfish-HARU
  role: Example application that demonstrates squiggle mapping of BLOW5 reads with
    the HARU sDTW hardware accelerator; invoked as sigfish-haru dtw
  repo: https://github.com/beebdev/sigfish-haru
- name: HARU
  role: sDTW FPGA accelerator (Verilog core and user-space driver) loaded onto the
    Kria KV260 to accelerate signal mapping of BLOW5 reads
  repo: https://github.com/beebdev/HARU
- name: RUscripts-R9
  role: Modified RUscripts Python pipeline supporting Python 3.6+, BLOW5 format, and
    R9.4 chemistry for software-only signal mapping baselines
  repo: https://github.com/beebdev/RUscripts-R9
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giad046
    title: Efficient real-time selective genome sequencing on resource-constrained
      devices
  source_content_hash: sha256:e1949175b9dc1ac5ef7b430fded093fe88904dc0bf8e9bad16798c30d162120d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beebdev/sigfish-haru
  tools:
  - HARU
  - RUscripts-R9
  - Sigfish-HARU
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# blow5-read-data-handling

## Summary

Acquire, prepare, and stream BLOW5-format nanopore raw signal reads (e.g., the 1.382-million-read SARS-CoV-2 dataset from Zenodo DOI 10.5281/zenodo.7314838) into sDTW squiggle-mapping pipelines such as sigfish-haru/sigfish-cpu against a reference FASTA.

## When to use

Use when the analysis consumes Oxford Nanopore raw signal reads in BLOW5 (or FAST5) format and requires squiggle-to-reference mapping with sigfish-haru or RUscripts-R9 against a reference genome FASTA.

## When NOT to use

- The data is already basecalled (FASTQ/BAM) and the workflow only needs sequence-domain mapping with Minimap2 rather than raw-signal sDTW mapping
- The reads are in FAST5 and the pipeline explicitly requires BLOW5; convert first via sigfish tooling rather than feeding FAST5 directly to these commands
- A selective-sequencing Read Until run on the Kria board is intended but the HARU accelerator firmware has not been loaded (xmutil loadapp haru-dtw-firmware); a CPU-only baseline run will not measure hardware acceleration

## Inputs

- BLOW5 raw signal read file (e.g., reads_0_0.blow5; full SARS-CoV-2 dataset of 1.382M reads from Zenodo DOI 10.5281/zenodo.7314838)
- Reference genome FASTA (e.g., nCoV-2019.reference.fasta / MN908947.3)

## Outputs

- PAF mapping file (output.paf) of squiggle-to-reference alignments
- Measured throughput (reads/s) and, optionally, per-stage runtime breakdown CSV (e.g., throughput_baselines.csv, runtime_breakdown.csv)

## How to apply

Obtain the raw signal dataset in BLOW5 format (e.g., reads_0_0.blow5 from the SARS-CoV-2 dataset on Zenodo, DOI 10.5281/zenodo.7314838) and the matching reference genome FASTA (nCoV-2019.reference.fasta / MN908947.3). Pass both directly to the sigfish dtw subcommand using the -g flag for the reference and -s flag for the BLOW5 reads, writing PAF mappings to stdout. For CPU-only baseline timing runs, build sigfish without the fpga=1 flag and use the sigfish-cpu binary; for hardware-accelerated runs, first load the HARU sDTW accelerator onto the Kria FPGA with xmutil loadapp haru-dtw-firmware. Time the end-to-end run over the full read set to compute reads/s throughput, and verify output.paf contains positive mapping entries consistent with expected baselines (~162 reads/s for optimized C on a 10-core desktop).

## Related tools

- **Sigfish-HARU** (Example application that demonstrates squiggle mapping of BLOW5 reads with the HARU sDTW hardware accelerator; invoked as sigfish-haru dtw) — https://github.com/beebdev/sigfish-haru
- **HARU** (sDTW FPGA accelerator (Verilog core and user-space driver) loaded onto the Kria KV260 to accelerate signal mapping of BLOW5 reads) — https://github.com/beebdev/HARU
- **RUscripts-R9** (Modified RUscripts Python pipeline supporting Python 3.6+, BLOW5 format, and R9.4 chemistry for software-only signal mapping baselines) — https://github.com/beebdev/RUscripts-R9

## Examples

```
./sigfish-haru dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf
```

## Evaluation signals

- output.paf exists and contains positive mapping entries after running sigfish-haru dtw on the example test_data
- Measured CPU throughput is order-of-magnitude consistent with reported baselines (~12.52 reads/s RUscripts-R9 Python; ~162 reads/s optimized C on a 10-core i9 desktop; ~432 reads/s on a 36-core Xeon server)
- sDTW stage accounts for >98% of total software-only runtime in the per-stage breakdown, consistent with the Fig. 6 report
- xmutil listapps confirms the haru-dtw-firmware accelerator loaded successfully before hardware-accelerated runs

## Limitations

- Throughput baselines are hardware-dependent (10-core i9 desktop vs. 36-core Xeon server) and deviations must be flagged for expert review
- Building the accelerator requires proprietary AMD Xilinx tools (Vivado/Vitis 2021.1, PetaLinux 2021.1) and is described as not intuitive
- SARS-CoV-2 dataset hosting/accession is not stated in the article discussion section; rely on the Zenodo DOI for retrieval

## Evidence

- [methods] sigfish-haru dtw on example data: "./sigfish-haru dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf"
- [readme] BLOW5 example data included in binary package: "Run the included *sigfish-haru* software binary that uses the hardware accelerator. The binary package includes example data."
- [discussion] RUscripts-R9 supports BLOW5 and R9.4: [evidence span withheld — anchor status: not_found]
- [readme] sigfish-haru is a fork of sigfish for squiggle mapping: [evidence span withheld — anchor status: external_doc]
