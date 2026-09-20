---
name: basecaller-throughput-benchmarking-across-hardware
description: Use when comparing basecalling speed of nanopore signal-to-sequence tools
  across hardware, or when choosing a decoder (--beam setting) and device (CPU vs
  GPU) to meet a sequencing throughput target.
when_to_use_negative:
- The goal is basecalling accuracy (read-level identity) rather than speed — use accuracy
  benchmarking against Albacore v2.0.1/DeepNano instead of throughput timing
- Input reads are not raw nanopore signal (.fast5); e.g. they are already basecalled
  FASTQ/FASTA, so there is nothing to decode or time
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: Chiron
  role: Basecaller under test; provides greedy (`--beam 0`) and beam-search (`--beam
    <width>`) CTC decoding whose per-device rates are benchmarked
  repo: https://github.com/haotianteng/Chiron
- name: TensorFlow
  role: Deep learning backend; install tensorflow==1.15 for CPU runs and tensorflow-gpu==1.15
    with CUDA/cuDNN for GPU throughput runs
- name: h5py
  role: Reads the .fast5 raw-signal files fed into the timed basecalling runs
  repo: https://github.com/h5py/h5py
- name: Albacore
  role: Proprietary ONT basecaller run with default parameters on CPU to obtain a
    comparison throughput rate
- name: BasecRAWller
  role: RNN-based basecaller (version 0.1) run with default parameters on CPU as a
    comparative throughput baseline
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giy037
    title: 'Chiron: translating nanopore raw signal directly into nucleotide sequence
      using deep learning'
  source_content_hash: sha256:0c8f32af3e48f9464ded8175527619d5d2e2a201262325971ffc0ef75bab7396
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haotianteng/Chiron
  tools:
  - Albacore
  - BasecRAWller
  - Chiron
  - TensorFlow
  - h5py
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# basecaller-throughput-benchmarking-across-hardware

## Summary

A benchmarking skill for measuring basecaller inference throughput (bp/s) across decoding algorithms (greedy vs beam-search CTC) and compute devices (CPU vs Nvidia GPUs), as characterized for the Chiron CNN+RNN+CTC nanopore basecaller.

## When to use

Use when comparing basecalling speed of nanopore signal-to-sequence tools across hardware, or when choosing a decoder (--beam setting) and device (CPU vs GPU) to meet a sequencing throughput target.

## When NOT to use

- The goal is basecalling accuracy (read-level identity) rather than speed — use accuracy benchmarking against Albacore v2.0.1/DeepNano instead of throughput timing
- Input reads are not raw nanopore signal (.fast5); e.g. they are already basecalled FASTQ/FASTA, so there is nothing to decode or time

## Inputs

- Folder of Oxford Nanopore raw signal .fast5 files (e.g. chiron/example_data/, 5 sample DNA fast5 courtesy of nanonet)
- Chiron DNA_default model with dna-pre preset (batch_size 400, segment_len 400, jump 390)
- Hardware definitions: CPU, and CUDA/cuDNN-equipped Nvidia GPUs (K80, GTX 1080 Ti)

## Outputs

- Per-run timing measurements yielding throughput in bases per second (bp/s) for each device x decoder combination
- speed_table.csv — a device x decoder rate table (CPU greedy 21, CPU beam 17, GPU greedy 1652, GPU beam 1204 bp/s)

## How to apply

Install Chiron with `pip install chiron` plus TensorFlow 1.15 (CPU or tensorflow-gpu), which also pulls in h5py for reading .fast5 files. Run basecalling on a fixed input set — e.g. the 5 sample DNA .fast5 files in chiron/example_data/ — using the DNA_default model with the dna-pre preset (batch_size 400, segment_len 400, jump 390). Time each run twice: once with the greedy decoder (`--beam 0`) and once with beam search (`--beam 50`, note the dna-pre preset default beam is 30), then compute bp/s as basecalled bases divided by wall-clock time. Repeat on GPU hardware (e.g. Nvidia K80, GTX 1080 Ti) with tensorflow-gpu==1.15; optionally run Albacore and BasecRAWller with default parameters on CPU for comparison. Compile device x decoder rates into a table mirroring the reported values (CPU: 21 bp/s greedy, 17 bp/s beam width=50; GPU: 1652 bp/s greedy on K80, 1204 bp/s beam, 2657 bp/s greedy on GTX 1080 Ti) and check that greedy decoding is faster than beam search on each device.

## Related tools

- **Chiron** (Basecaller under test; provides greedy (`--beam 0`) and beam-search (`--beam <width>`) CTC decoding whose per-device rates are benchmarked) — https://github.com/haotianteng/Chiron
- **TensorFlow** (Deep learning backend; install tensorflow==1.15 for CPU runs and tensorflow-gpu==1.15 with CUDA/cuDNN for GPU throughput runs)
- **h5py** (Reads the .fast5 raw-signal files fed into the timed basecalling runs) — https://github.com/h5py/h5py
- **Albacore** (Proprietary ONT basecaller run with default parameters on CPU to obtain a comparison throughput rate)
- **BasecRAWller** (RNN-based basecaller (version 0.1) run with default parameters on CPU as a comparative throughput baseline)

## Examples

```
chiron call -i chiron/example_data/ -o out_gpu_greedy -m chiron/model/DNA_default --preset dna-pre --beam 0
```

## Evaluation signals

- Reproduced CPU rates match the reported 21 bp/s (greedy) and 17 bp/s (beam width=50) within reasonable run-to-run tolerance
- Reproduced GPU greedy rate is in the 1,652 bp/s (K80) to 2,657 bp/s (GTX 1080 Ti) range, and beam-search rate is 1,204 bp/s on K80
- Invariant holds on every device: greedy decoding rate > beam-search decoding rate
- Basecalled output FASTQ/FASTA files appear in the result folder of the output directory for all 5 input fast5 files, confirming the timed runs completed
- Each rate is computed from a fixed input set and timing is taken under identical preset parameters (batch_size 400, segment_len 400, jump 390) per device

## Limitations

- GPU benchmarking requires NVIDIA hardware with CUDA toolkit and cuDNN installed, and the pinned tensorflow-gpu==1.15; other TensorFlow versions are not validated
- Beam width trades speed for accuracy — larger beam widths further improve accuracy but reduce bp/s, so rates are only comparable at matched beam settings
- Chiron is substantially more computationally expensive than Albacore and somewhat more expensive than BasecRAWller, so CPU-only throughput is very low (tens of bp/s)
- Timing is sensitive to batch size — larger batch sizes give faster inference speed; reduce batch (default 400) if GPU RAM is limited, which changes measured rates

## Evidence

- [readme] Greedy decoder give a faster decoding speed than the beam search decoder: "Beam Seach decoder give a higher accuracy, and larger beam width can furthur improve the accuracy.
Greedy decoder give a faster decoding speed than the beam search decoder"
- [readme] Device decoder-rate table: "| CPU | 21 | 17 |
| GPU | 1652 | 1204 |"
- [abstract] Chiron achieves basecalling speeds of more than 2,000 bases per second using desktop computer graphics processing units.: "Chiron achieves basecalling speeds of more than 2,000 bases per second using desktop computer graphics processing units."
- [readme] Preset parameters for dna-pre: "**DNA preset arguments value(dna-pre):**
'start':0,'batch_size':400,'segment_len':400,'jump':390,'threads':0,'beam':30"
- [discussion] Comparison basecallers run with defaults: "All basecalling with Albacore (v1.1.1 and v2.0.1) and BasecRAWller [13] (version 0.1) was done with default parameters."
