---
name: decoding-rate-measurement-and-tabulation
description: Use when you need to quantify how fast Chiron (or a comparable CTC-decoder
  basecaller) processes nanopore fast5 raw signal, and to compare greedy vs beam-search
  decoding throughput across CPU and GPU devices.
when_to_use_negative:
- The goal is basecalling accuracy evaluation (read-level identity vs a reference)
  rather than throughput/speed measurement — this skill times decoding only.
- The basecaller lacks a CTC decoder option and cannot be run in both greedy and beam-search
  modes, so the decoder-speed comparison is not meaningful.
- Only a very small input (a single short read) is available, where timing noise will
  swamp the measured rate.
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: Chiron
  role: The basecaller whose greedy (--beam 0) and beam-search (--beam 50) CTC decoders
    are timed to measure bp/s throughput
  repo: https://github.com/haotianteng/chiron
- name: TensorFlow
  role: Backend (tensorflow==1.15 CPU or tensorflow-gpu==1.15) that executes the CNN+RNN
    model and enables device (CPU vs GPU) benchmarking
  repo: https://www.tensorflow.org/
- name: h5py
  role: Reads the input .fast5 nanopore raw signal files during decoding
  repo: https://github.com/h5py/h5py
- name: Albacore
  role: Optional ONT basecaller (v1.1.1/v2.0.1, default parameters) run on CPU to
    obtain a comparison CPU throughput rate
- name: BasecRAWller
  role: Optional RNN basecaller (version 0.1, default parameters) run on CPU for comparison
    throughput rates
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giy037
    title: 'Chiron: translating nanopore raw signal directly into nucleotide sequence
      using deep learning'
  source_content_hash: sha256:19461f106b88cd59a4679fd558c80ecac755c4e6132e977abd55c09686a59d22
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haotianteng/chiron
  tools:
  - Albacore
  - BasecRAWller
  - Chiron
  - TensorFlow
  - h5py
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# decoding-rate-measurement-and-tabulation

## Summary

Benchmark the basecalling throughput (bp/s) of Chiron's greedy vs beam-search CTC decoders on CPU and GPU hardware, then compile the device × decoder rates into a table mirroring the paper's reported speeds.

## When to use

Use when you need to quantify how fast Chiron (or a comparable CTC-decoder basecaller) processes nanopore fast5 raw signal, and to compare greedy vs beam-search decoding throughput across CPU and GPU devices.

## When NOT to use

- The goal is basecalling accuracy evaluation (read-level identity vs a reference) rather than throughput/speed measurement — this skill times decoding only.
- The basecaller lacks a CTC decoder option and cannot be run in both greedy and beam-search modes, so the decoder-speed comparison is not meaningful.
- Only a very small input (a single short read) is available, where timing noise will swamp the measured rate.

## Inputs

- DNA fast5 raw signal files (e.g. the 5 sample files in chiron/example_data/)
- Trained Chiron DNA_default model folder
- CPU and/or GPU (e.g. Nvidia K80, GTX 1080 Ti) execution environment with TensorFlow 1.15

## Outputs

- Per-run wall-clock timings converted to decoding rates in bp/s
- speed_table.csv with device × decoder rows (e.g. CPU greedy 21, CPU beam 17, GPU greedy 1652, GPU beam 1204 bp/s)
- Basecalled FASTQ/FASTA result files from chiron call runs

## How to apply

Install Chiron with tensorflow==1.15 (CPU) or tensorflow-gpu==1.15 (GPU) plus h5py, and use the 5 sample DNA fast5 files in chiron/example_data/ (courtesy of nanonet) as the input. Run greedy decoding with 'chiron call -i <input_fast5_folder> -o <output_folder> --beam 0' and beam-search decoding with '--beam 50', timing each run to compute the bp/s rate (total bases basecalled / wall-clock time). Use the dna-pre preset parameters (batch_size 400, segment_len 400, jump 390) and the DNA_default model trained on R9.4 Lambda/E.coli data; note the README reports CPU rates of 21 bp/s (greedy) and 17 bp/s (beam width 50), and GPU rates of 1652 bp/s (greedy) and 1204 bp/s (beam). Optionally run Albacore (v1.1.1) and BasecRAWller (v0.1) with default parameters on CPU to obtain comparison throughput rates. Compile the measured device × decoder rates into speed_table.csv and validate against the reported values within run-to-run tolerance, confirming greedy decoding is faster than beam search on each device.

## Related tools

- **Chiron** (The basecaller whose greedy (--beam 0) and beam-search (--beam 50) CTC decoders are timed to measure bp/s throughput) — https://github.com/haotianteng/chiron
- **TensorFlow** (Backend (tensorflow==1.15 CPU or tensorflow-gpu==1.15) that executes the CNN+RNN model and enables device (CPU vs GPU) benchmarking) — https://www.tensorflow.org/
- **h5py** (Reads the input .fast5 nanopore raw signal files during decoding) — https://github.com/h5py/h5py
- **Albacore** (Optional ONT basecaller (v1.1.1/v2.0.1, default parameters) run on CPU to obtain a comparison CPU throughput rate)
- **BasecRAWller** (Optional RNN basecaller (version 0.1, default parameters) run on CPU for comparison throughput rates)

## Examples

```
chiron call -i chiron/example_data/ -o output/ --beam 0 --preset dna-pre && chiron call -i chiron/example_data/ -o output_beam/ --beam 50 --preset dna-pre
```

## Evaluation signals

- Reproduced CPU rates fall near 21 bp/s (greedy) and 17 bp/s (beam width 50) within reasonable run-to-run tolerance
- Reproduced GPU rates fall near 1652 bp/s (greedy) and 1204 bp/s (beam) within reasonable tolerance
- Greedy decoding is faster than beam-search decoding on each device tested
- speed_table.csv mirrors the structure of the reported table (device × decoder rows with bp/s rates)
- Timing methodology is consistent: same input fast5 files, dna-pre preset (batch_size 400, segment_len 400, jump 390), and timing convention across all runs

## Limitations

- Chiron is substantially more computationally expensive than Albacore and somewhat more than BasecRAWller, so CPU-only throughput is very low (tens of bp/s) relative to GPU rates.
- Increasing the size of the sliding window improves accuracy but increases memory and running time, so throughput results are configuration-dependent.
- Batch size affects inference speed — larger batch sizes give (very) slightly faster inference, so batch_size must be held fixed for comparable timings.

## Evidence

- [readme] Greedy decoder give a faster decoding speed than the beam search decoder: "Greedy decoder give a faster decoding speed than the beam search decoder"
- [readme] decoder rate table in README: "Beam Seach decoder give a higher accuracy, and larger beam width can furthur improve the accuracy."
- [discussion] computationally expensive relative to other basecallers: "Our model is substantially more computationally expensive than Albacore and somewhat more computationally expensive than BasecRAWller."
- [abstract] GPU speed claim in abstract: "Chiron achieves basecalling speeds of more than 2,000 bases per second using desktop computer graphics processing units."
- [methods] Beam search decoder CLI usage: "Beam search decoder: chiron call -i <input> -o <output> --beam <beam_width>"
