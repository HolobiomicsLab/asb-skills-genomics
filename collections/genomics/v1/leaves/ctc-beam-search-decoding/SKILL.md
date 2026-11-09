---
name: ctc-beam-search-decoding
description: Use when basecalling raw fast5 nanopore signal with Chiron and you must
  choose between greedy CTC decoding for throughput and beam-search decoding for maximum
  read-level accuracy.
when_to_use_negative:
- Input is already basecalled FASTQ/FASTA — decoding is only applied to raw signal
  fast5 files.
- Direct-RNA data without prior preprocessing — use BoostNano to trim adapter and
  polyA tails before RNA basecalling.
- Throughput-constrained production runs where the ~2% accuracy gain of beam search
  over greedy does not justify the slower decoding speed.
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Chiron
  role: CNN+RNN+CTC basecaller whose call command implements both greedy (--beam 0)
    and beam-search (--beam <width>) CTC decoding of raw signal
  repo: https://github.com/haotianteng/Chiron
- name: TensorFlow
  role: Backend (tensorflow or tensorflow-gpu 1.15) executing the CTC decoder on CPU
    or GPU
- name: h5py
  role: Reads raw signal from .fast5 HDF5 files before decoding
  repo: https://github.com/h5py/h5py
- name: Albacore
  role: Comparison basecaller used to benchmark greedy vs beam-search accuracy and
    speed against Chiron's decoding
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giy037
    title: 'Chiron: translating nanopore raw signal directly into nucleotide sequence
      using deep learning'
  source_content_hash: sha256:721ca03ce271b8a14171eefaf1e213eb4a1630a9df47ee68a54d7917e0468901
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haotianteng/Chiron
  tools:
  - Albacore
  - BasecRAWller
  - Chiron
  - Metrichor
  - TensorFlow
  - h5py
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# ctc-beam-search-decoding

## Summary

Decodes Chiron's CTC output layer from raw nanopore signal into nucleotide sequence, choosing between a greedy decoder (--beam 0) and a beam-search decoder (--beam <width>) that trades decoding speed for higher basecalling accuracy.

## When to use

Use when basecalling raw fast5 nanopore signal with Chiron and you must choose between greedy CTC decoding for throughput and beam-search decoding for maximum read-level accuracy.

## When NOT to use

- Input is already basecalled FASTQ/FASTA — decoding is only applied to raw signal fast5 files.
- Direct-RNA data without prior preprocessing — use BoostNano to trim adapter and polyA tails before RNA basecalling.
- Throughput-constrained production runs where the ~2% accuracy gain of beam search over greedy does not justify the slower decoding speed.

## Inputs

- Folder of ONT MinION raw signal .fast5 files (DNA or direct-RNA)
- A trained Chiron model folder (e.g. chiron/model/DNA_default or RNA_default)
- Preset selection (dna-pre or rna-pre) defining segment_len, jump, and batch_size

## Outputs

- Basecalled FASTQ/FASTA files (one per input fast5) in <output_folder>/result/
- Measured decoding throughput in bp/s per device x decoder combination (if timed)

## How to apply

Run Chiron's call command on a folder of fast5 files using the DNA_default model with the dna-pre preset (batch_size 400, segment_len 400, jump 390), which sets a default beam width of 30. To decode with beam search, pass --beam <beam_width> (e.g. --beam 30 for accuracy benchmarks, --beam 50 for the speed table); to decode with the greedy CTC decoder, pass --beam 0, which is faster but slightly less accurate. Choose greedy when throughput dominates (e.g. ~1,652 bp/s on an Nvidia K80 GPU vs 1,204 bp/s with beam search; 21 vs 17 bp/s on CPU) and beam search when read-level identity rate matters — the paper's accuracy comparisons used both decoders. Larger beam widths can further improve accuracy at additional computational cost, and the output is written as fastq/fasta files in the result folder of the output directory.

## Related tools

- **Chiron** (CNN+RNN+CTC basecaller whose call command implements both greedy (--beam 0) and beam-search (--beam <width>) CTC decoding of raw signal) — https://github.com/haotianteng/Chiron
- **TensorFlow** (Backend (tensorflow or tensorflow-gpu 1.15) executing the CTC decoder on CPU or GPU)
- **h5py** (Reads raw signal from .fast5 HDF5 files before decoding) — https://github.com/h5py/h5py
- **Albacore** (Comparison basecaller used to benchmark greedy vs beam-search accuracy and speed against Chiron's decoding)

## Examples

```
python chiron/entry.py call -i chiron/example_data/ -o <output_folder> -m chiron/model/DNA_default --preset dna-pre --beam 30
```

## Evaluation signals

- Greedy decoding (beam 0) is faster than beam search on the same device: reported rates are 21 vs 17 bp/s on CPU and 1,652 vs 1,204 bp/s on an Nvidia K80 GPU, and 2,657 bp/s on a GTX 1080 Ti.
- Beam-search decoding should yield equal or higher per-read identity rate than greedy on the same read set; benchmark runs used beam width 30 for accuracy tables and 50 for the speed table.
- Output folder contains result/ with one fastq/fasta per input fast5, and basecalled reads align to the reference (e.g. NC_001416.1 via graphmap) with identity within ~0.01 of Albacore v2.0.1 on bacterial/viral datasets.
- Command exits without CTC decode errors using the dna-pre preset values batch_size 400, segment_len 400, jump 390; reduce batch size if GPU RAM is limited.

## Limitations

- Beam search decoding is slower than greedy; larger beam widths further improve accuracy at added runtime and memory cost.
- Chiron basecalling is substantially more computationally expensive than Albacore and somewhat more expensive than BasecRAWller.
- On Human DNA (NA12878), accuracy remains up to ~2% below Albacore, since the DNA_default model was trained only on a mix of Lambda and E. coli R9.4 data.
- Chiron is built with TensorFlow 1.15 and Python 2.7-era tooling; GPU decoding requires CUDA and cuDNN.

## Evidence

- [methods] Beam search decoder: chiron call -i <input> -o <output> --beam <beam_width>: "Beam search decoder: chiron call -i <input> -o <output> --beam <beam_width>"
- [methods] Greedy decoder: chiron call -i <input> -o <output> --beam 0: "Greedy decoder: chiron call -i <input> -o <output> --beam 0"
- [readme] Beam Seach decoder give a higher accuracy, and larger beam width can furthur improve the accuracy. Greedy decoder give a faster decoding speed than the beam search decoder: "Beam Seach decoder give a higher accuracy, and larger beam width can furthur improve the accuracy."
- [intro] Chiron has a novel architecture that couples a convolutional neural network (CNN) with an RNN and a connectionist temporal classification (CTC) decoder: "Chiron has a novel architecture that couples a convolutional neural network (CNN) with an RNN and a connectionist temporal classification (CTC) decoder"
- [readme] 'start':0,'batch_size':400,'segment_len':400,'jump':390,'threads':0,'beam':30: "'start':0,'batch_size':400,'segment_len':400,'jump':390,'threads':0,'beam':30"
