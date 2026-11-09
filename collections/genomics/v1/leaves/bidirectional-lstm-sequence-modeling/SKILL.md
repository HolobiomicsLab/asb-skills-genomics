---
name: bidirectional-lstm-sequence-modeling
description: Use when raw MinION .fast5 signal (not segmented events) must be translated
  into nucleotide sequences, and you need recurrent integration of local conv features
  into basecall probabilities without an event segmentation step.
when_to_use_negative:
- Input is already basecalled FASTQ/FASTA reads rather than raw .fast5 signal — the
  network operates on raw electrical signal only.
- You need the prior two-stage approach where raw data are segmented into k-mer signals
  before translation — this skill is explicitly segmentation-free, and segmentation
  suffers from variable DNA translocation speed and low signal-to-noise.
- You require the highest absolute accuracy on Human DNA and cannot tolerate the remaining
  accuracy gap (within 2% of Albacore on Human, and more computationally expensive
  than Albacore or BasecRAWller).
edam_operation: http://edamontology.org/operation_0262
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3473
tools:
- name: Chiron
  role: Provides the CNN+RNN+CTC basecaller whose bidirectional LSTM recurrent layers
    map convolved raw-signal features to basecall probabilities, with greedy and beam
    search decoders
  repo: https://github.com/haotianteng/Chiron
- name: TensorFlow
  role: Framework in which the BiLSTM layers, CTC loss, and decoders are implemented
    (TensorFlow 1.15 on Python 2.7)
- name: h5py
  role: Required for reading .fast5 raw signal files into the network
  repo: https://github.com/h5py/h5py
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giy037
    title: 'Chiron: translating nanopore raw signal directly into nucleotide sequence
      using deep learning'
  source_content_hash: sha256:7230a1377a2867c7cfc66b9339b92eeb0b1007027c74298f6ece8fc52d99bca4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haotianteng/Chiron
  tools:
  - Chiron
  - TensorFlow
  - h5py
  edam_operation: http://edamontology.org/operation_0262
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# bidirectional-lstm-sequence-modeling

## Summary

Stack bidirectional LSTM recurrent layers over convolutional features to map raw nanopore electrical signal to per-timestep base probabilities, feeding a CTC decoder for segmentation-free basecalling. This is the RNN core of Chiron's CNN+RNN+CTC architecture built in TensorFlow.

## When to use

Use when raw MinION .fast5 signal (not segmented events) must be translated into nucleotide sequences, and you need recurrent integration of local conv features into basecall probabilities without an event segmentation step.

## When NOT to use

- Input is already basecalled FASTQ/FASTA reads rather than raw .fast5 signal — the network operates on raw electrical signal only.
- You need the prior two-stage approach where raw data are segmented into k-mer signals before translation — this skill is explicitly segmentation-free, and segmentation suffers from variable DNA translocation speed and low signal-to-noise.
- You require the highest absolute accuracy on Human DNA and cannot tolerate the remaining accuracy gap (within 2% of Albacore on Human, and more computationally expensive than Albacore or BasecRAWller).

## Inputs

- Raw Oxford Nanopore MinION signal stored in .fast5 files (read via h5py)
- Trained Chiron model folder (e.g. chiron/model/DNA_default)
- Preset parameters (dna-pre: beam 30, segment_len 400, jump 390, batch_size 400)

## Outputs

- Per-timestep base probabilities/logits from the BiLSTM + softmax
- Basecalled FASTQ/FASTA files in the result folder of the output directory

## How to apply

Build the recurrent stack in TensorFlow 1.15 (Python 2.7) on top of a convolutional feature extractor: stack three bidirectional LSTM layers with 200 hidden units so the network integrates conv patterns in both directions of the raw input signal. Map the BiLSTM outputs to per-timestep character logits via a fully-connected layer and softmax over the nucleotide alphabet, then wire a CTC loss for training since no alignment between raw signal and bases exists. At inference, expose both a greedy decoder (--beam 0) and a beam search decoder with configurable beam width (default 30 in the dna-pre preset); beam search gives higher accuracy while greedy decoding is faster (1,652 vs 1,204 bp/s on GPU with beam_width=50). Feed raw signal in sliding windows (dna-pre preset: segment_len 400, jump 390, batch_size 400) through the trained model (e.g. DNA_default trained on R9.4 Lambda/E.coli data) to emit FASTQ/FASTA basecalls.

## Related tools

- **Chiron** (Provides the CNN+RNN+CTC basecaller whose bidirectional LSTM recurrent layers map convolved raw-signal features to basecall probabilities, with greedy and beam search decoders) — https://github.com/haotianteng/Chiron
- **TensorFlow** (Framework in which the BiLSTM layers, CTC loss, and decoders are implemented (TensorFlow 1.15 on Python 2.7))
- **h5py** (Required for reading .fast5 raw signal files into the network) — https://github.com/h5py/h5py

## Examples

```
chiron call -i <input_fast5_folder> -o <output_folder> -m chiron/model/DNA_default --preset dna-pre
```

## Evaluation signals

- The model definition constructs the TensorFlow computational graph without error and contains the BiLSTM, softmax, and CTC/decoder components.
- Decoding options behave as specified: greedy (--beam 0) runs and beam search with the dna-pre default beam width 30 yields higher accuracy at lower speed.
- Decoder throughput falls in expected ranges per README: greedy ~1,652 bp/s and beam search ~1,204 bp/s on GPU.
- Read-level accuracy on bacterial/viral test data is comparable to Albacore v2.0.1 (and best or second-best on all tested datasets, including Human despite no Human DNA in training).
- Batch size adjustments (default 400, e.g. -b 200 for limited RAM/GPU-RAM) complete without OOM while preserving output.

## Limitations

- Chiron is substantially more computationally expensive than Albacore and somewhat more expensive than BasecRAWller.
- Accuracy gap remains vs the proprietary Albacore on Human DNA (within 1% on bacterial but only within 2% on Human DNA).
- Larger sliding window or beam width improves accuracy but at the cost of increased memory and running time; larger batch sizes only give very slightly better results.
- Built with Python 2.7 and TensorFlow 1.15 — legacy environment constraints apply.

## Evidence

- [intro] Chiron has a novel architecture that couples a convolutional neural network (CNN) with an RNN and a connectionist temporal classification (CTC) decoder: "Chiron has a novel architecture that couples a convolutional neural network (CNN) with an RNN and a connectionist temporal classification (CTC) decoder [13]"
- [intro] DeepNano[12] introduced the idea of using a bidirectional recurrent neural network (RNN): "DeepNano[12] introduced the idea of using a bidirectional recurrent neural network (RNN)"
- [readme] Using a deep learning CNN+RNN+CTC structure to establish end-to-end basecalling for the nanopore sequencer.: "Using a deep learning CNN+RNN+CTC structure to establish end-to-end basecalling for the nanopore sequencer."
- [readme] Beam search decoder give a higher accuracy, and larger beam width can furthur improve the accuracy.: "Beam Seach decoder give a higher accuracy, and larger beam width can furthur improve the accuracy."
