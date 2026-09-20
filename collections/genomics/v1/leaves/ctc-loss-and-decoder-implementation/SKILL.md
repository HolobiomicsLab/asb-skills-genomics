---
name: ctc-loss-and-decoder-implementation
description: Use when wiring a CNN+RNN basecaller to output DNA from raw nanopore
  signal, and you must attach a CTC loss for training and choose/configure a decoding
  strategy for inference.
when_to_use_negative:
- The basecalling pipeline still relies on event segmentation into k-mer signal segments
  — CTC is specifically for segmentation-free end-to-end basecalling
- You need the fastest possible decoding and accuracy loss from greedy decoding is
  unacceptable for your application
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3474
tools:
- name: Chiron
  role: Basecaller whose CTC decoder and beam/greedy decoding modes this skill implements
    and configures
  repo: github.com/haotianteng/Chiron
- name: TensorFlow
  role: Framework for building the CTC loss and beam search / greedy decoders in the
    model definition script
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giy037
    title: 'Chiron: translating nanopore raw signal directly into nucleotide sequence
      using deep learning'
  source_content_hash: sha256:384975160d6ebdf6fe0e3bcee9e836920603446ca6dbef3386a36c79141d83d9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Chiron
  - TensorFlow
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# ctc-loss-and-decoder-implementation

## Summary

Implement and configure the CTC loss and decoding layer of a segmentation-free basecaller like Chiron, offering a greedy decoder (beam 0) and a beam search decoder (default width 30 in the dna-pre preset) over per-timestep nucleotide logits.

## When to use

Use when wiring a CNN+RNN basecaller to output DNA from raw nanopore signal, and you must attach a CTC loss for training and choose/configure a decoding strategy for inference.

## When NOT to use

- The basecalling pipeline still relies on event segmentation into k-mer signal segments — CTC is specifically for segmentation-free end-to-end basecalling
- You need the fastest possible decoding and accuracy loss from greedy decoding is unacceptable for your application

## Inputs

- Per-timestep character logits from the CNN+RNN stack over the raw nanopore signal (sliding window: 300 signals, step 30)
- Reference nucleotide sequences for CTC loss during training
- FAST5 files containing raw MinION electrical signal

## Outputs

- Basecalled DNA sequence output as FASTQ/FASTA in the result folder
- Trained model with CTC loss attached to the softmax logits

## How to apply

Stack the CTC decoder on top of a fully-connected layer and softmax over the nucleotide alphabet that follows the five residual convolutional layers (256 channels, ReLU, batch normalization) and three bidirectional LSTM layers (200 hidden units). For training, compute CTC loss on the per-timestep logits against the reference sequence. For inference, expose two decoding modes: a greedy decoder selected with '--beam 0', which is faster, and a beam search decoder with '--beam <beam_width>' (default 30 per the dna-pre preset), which yields higher accuracy and can be increased for further gains. Validate that the graph constructs without error and that both decoder modes are callable from the entry point.

## Related tools

- **Chiron** (Basecaller whose CTC decoder and beam/greedy decoding modes this skill implements and configures) — github.com/haotianteng/Chiron
- **TensorFlow** (Framework for building the CTC loss and beam search / greedy decoders in the model definition script)

## Examples

```
chiron call -i <input> -o <output> --beam 30
```

## Evaluation signals

- model_definition.py contains CTC loss, softmax, and both greedy (beam 0) and beam search decoder components
- The computational graph constructs without error
- Beam search (--beam 30) produces higher read-level accuracy than greedy (--beam 0) on the same fast5 input
- Greedy decoding runs faster than beam search: e.g. 1652 bp/s vs 1204 bp/s on GPU with beam_width=50
- Output FASTQ/FASTA in the result folder matches the fast5 filenames it was basecalled from

## Limitations

- Larger beam width improves accuracy but increases decoding time; greedy decoding trades accuracy for speed
- Chiron is substantially more computationally expensive than Albacore and somewhat more expensive than BasecRAWller
- In the original Chiron, CTC decoding operates on preprocessed signal slices (segment_len 400, jump 390 for dna-pre), so window size trades accuracy against memory and running time

## Evidence

- [intro] Chiron has a novel architecture that couples a convolutional neural network (CNN) with an RNN and a connectionist temporal classification (CTC) decoder: "Chiron has a novel architecture that couples a convolutional neural network (CNN) with an RNN and a connectionist temporal classification (CTC) decoder"
- [methods] Beam search decoder: chiron call -i <input> -o <output> --beam <beam_width>: "Beam search decoder: chiron call -i <input> -o <output> --beam <beam_width>"
- [methods] Greedy decoder: chiron call -i <input> -o <output> --beam 0: "Greedy decoder: chiron call -i <input> -o <output> --beam 0"
- [readme] 'start':0,'batch_size':400,'segment_len':400,'jump':390,'threads':0,'beam':30: "'start':0,'batch_size':400,'segment_len':400,'jump':390,'threads':0,'beam':30"
- [readme] Beam Seach decoder give a higher accuracy, and larger beam width can furthur improve the accuracy. Greedy decoder give a faster decoding speed than the beam search decoder: "Beam Seach decoder give a higher accuracy, and larger beam width can furthur improve the accuracy. Greedy decoder give a faster decoding speed"
