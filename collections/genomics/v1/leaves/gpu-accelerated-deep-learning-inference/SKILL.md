---
name: gpu-accelerated-deep-learning-inference
description: Use when you need to quantify deep-learning inference throughput on CPU
  vs Nvidia GPU hardware, e.g. measuring basecalling bp/s rates of a CNN+RNN+CTC model
  with greedy vs beam-search decoding.
when_to_use_negative:
- When the goal is accuracy benchmarking rather than speed — use read-level accuracy
  against a reference (e.g. Phage Lambda NC_001416.1) instead of timing runs
- When the basecaller is not open-source or not locally runnable (e.g. the proprietary
  ONT Albacore cloud/Metrichor pipeline), since timed local inference is not reproducible
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3474
tools:
- name: Chiron
  role: CNN+RNN+CTC basecaller whose call command is timed on CPU and GPU with greedy
    (--beam 0) and beam-search (--beam 50) decoders
  repo: https://github.com/haotianteng/Chiron
- name: TensorFlow
  role: Deep learning backend; tensorflow-gpu==1.15 enables GPU inference, tensorflow==1.15
    CPU-only
- name: h5py
  role: Dependency for reading .fast5 raw signal files during basecalling
  repo: https://github.com/h5py/h5py
- name: Albacore
  role: ONT basecaller run with default parameters on CPU as a comparison throughput
    baseline
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giy037
    title: 'Chiron: translating nanopore raw signal directly into nucleotide sequence
      using deep learning'
  source_content_hash: sha256:c9b82f80c5f98e57f550356999fc9f0a1f1a08e3968c0cd7c9d9d4ff1bfd6c43
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
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# gpu-accelerated-deep-learning-inference

## Summary

Benchmark and run deep learning inference (CNN+RNN+CTC basecalling) on CPU vs GPU hardware by timing Chiron's greedy and beam-search decoders to measure throughput in bp/s per device.

## When to use

Use when you need to quantify deep-learning inference throughput on CPU vs Nvidia GPU hardware, e.g. measuring basecalling bp/s rates of a CNN+RNN+CTC model with greedy vs beam-search decoding.

## When NOT to use

- When the goal is accuracy benchmarking rather than speed — use read-level accuracy against a reference (e.g. Phage Lambda NC_001416.1) instead of timing runs
- When the basecaller is not open-source or not locally runnable (e.g. the proprietary ONT Albacore cloud/Metrichor pipeline), since timed local inference is not reproducible

## Inputs

- 5 sample DNA .fast5 files (chiron/example_data/, courtesy of nanonet)
- Chiron DNA_default model folder
- tensorflow==1.15 or tensorflow-gpu==1.15 environment

## Outputs

- FASTQ/FASTA basecalling result files per fast5 input in the output folder's 'result' directory
- Device x decoder throughput table (bp/s), e.g. speed_table.csv with CPU greedy 21, CPU beam 17, GPU greedy 1652, GPU beam 1204

## How to apply

Install Chiron with TensorFlow 1.15 (or tensorflow-gpu==1.15 with CUDA and cuDNN for GPU runs) and use the 5 sample DNA fast5 files in chiron/example_data/ as a small reproducible test set. Run greedy decoding with 'chiron call -i <input_fast5_folder> -o <output_folder> --beam 0' and beam-search decoding with '--beam 50' on CPU, timing each run to compute bp/s. Repeat both runs on a GPU (e.g. Nvidia K80 or GTX 1080 Ti) using the DNA default model with the dna-pre preset (batch_size 400, segment_len 400, jump 390). Expect greedy to be faster than beam search on every device: the reported rates are CPU greedy 21 bp/s, CPU beam 17 bp/s, GPU greedy 1652 bp/s, GPU beam 1204 bp/s, and over 2,000 bp/s on desktop GPUs. Batch size can be tuned (default 400) — larger batches give slightly better results and faster inference speed but require more RAM/GPU-RAM.

## Related tools

- **Chiron** (CNN+RNN+CTC basecaller whose call command is timed on CPU and GPU with greedy (--beam 0) and beam-search (--beam 50) decoders) — https://github.com/haotianteng/Chiron
- **TensorFlow** (Deep learning backend; tensorflow-gpu==1.15 enables GPU inference, tensorflow==1.15 CPU-only)
- **h5py** (Dependency for reading .fast5 raw signal files during basecalling) — https://github.com/h5py/h5py
- **Albacore** (ONT basecaller run with default parameters on CPU as a comparison throughput baseline)

## Examples

```
chiron call -i <input_fast5_folder> -o <output_folder> -m chiron/model/DNA_default --preset dna-pre --beam 0
```

## Evaluation signals

- GPU greedy decoding rate should far exceed CPU rate (~1652 bp/s on K80 vs 21 bp/s CPU; 2657 bp/s on GTX 1080 Ti)
- Greedy decoder must be faster than beam-search decoder on the same device (e.g. 1652 vs 1204 bp/s on GPU; 21 vs 17 bp/s on CPU)
- Beam-search run must use beam_width=50 to reproduce the reported beam rate
- Run-to-run rates on identical hardware should fall within reasonable tolerance of the published table
- Output 'result' folder must contain fastq/fasta files named after each input fast5 file

## Limitations

- Chiron is substantially more computationally expensive than Albacore and somewhat more than BasecRAWller
- GPU throughput depends on hardware: reported rates differ between Nvidia K80 (1652 bp/s greedy) and GTX 1080 Ti (2657 bp/s)
- Larger batch sizes give slightly faster inference but increase RAM/GPU-RAM requirements; reduce batch size (e.g. -b 200) on limited memory
- Chiron was built with TensorFlow and python 2.7, requiring tensorflow(-gpu)==1.15

## Evidence

- [abstract] Chiron achieves basecalling speeds of more than 2,000 bases per second using desktop computer graphics processing units.: "Chiron achieves basecalling speeds of more than 2,000 bases per second using desktop computer graphics processing units."
- [methods] Beam search decoder / greedy decoder CLI: "Beam search decoder: chiron call -i <input> -o <output> --beam <beam_width>"
- [readme] README decoder speed table: "Greedy decoder give a faster decoding speed than the beam search decoder: | Device | Greedy decoder rate(bp/s) | Beam Search decoder rate(bp/s), beam_width=50 |"
- [discussion] Our model is substantially more computationally expensive than Albacore and somewhat more computationally expensive than BasecRAWller.: "Our model is substantially more computationally expensive than Albacore and somewhat more computationally expensive than BasecRAWller."
