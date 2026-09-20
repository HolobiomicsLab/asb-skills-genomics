---
name: cnn-rnn-ctc-neural-basecalling
description: Use when raw nanopore fast5 electrical signals must be translated directly
  into nucleotide sequence without event segmentation, or when an open-source, trainable
  alternative to ONT's proprietary Albacore/Metrichor basecallers is needed.
when_to_use_negative:
- Input is already segmented/event-tabulated signal (e.g.ONT event data) — prior two-stage
  basecallers handle that and Chiron's raw-signal slicing is unnecessary
- Direct-RNA fast5 files with adapter and polyA stall still present — call BoostNano
  first, since Chiron's RNA model expects preprocessed signal
- Only FASTQ/FASTA reads are available with no raw signal — there is nothing to basecall;
  use read-level analysis skills instead
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: Chiron
  role: CNN+RNN+CTC end-to-end basecaller that translates fast5 raw signal into nucleotide
    sequence; invoked via 'chiron call' with DNA/RNA presets and beam/greedy decoders
  repo: https://github.com/haotianteng/Chiron
- name: h5py
  role: Python library used to read the raw electrical signal out of each fast5 HDF5
    file before slicing
  repo: https://github.com/h5py/h5py
- name: TensorFlow
  role: Deep learning framework in which Chiron's CNN+RNN+CTC model is implemented
    (install tensorflow==1.15 or tensorflow-gpu==1.15)
- name: BoostNano
  role: Optional preprocessing tool that segments and removes adapter and polyA tails
    from raw RNA signal before Chiron RNA basecalling
  repo: https://github.com/haotianteng/BoostNano
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giy037
    title: 'Chiron: translating nanopore raw signal directly into nucleotide sequence
      using deep learning'
  source_content_hash: sha256:7b0c762a399041b2ecdeb023b74a0d85e4b67f73498e0643e09c16c7265d7af7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haotianteng/Chiron
  tools:
  - BoostNano
  - Chiron
  - TensorFlow
  - h5py
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# cnn-rnn-ctc-neural-basecalling

## Summary

End-to-end basecalling of Oxford Nanopore raw electrical signal using a coupled CNN+RNN+CTC neural network (Chiron), which bypasses the error-prone signal segmentation step by slicing raw fast5 signals, basecalling each window, and assembling overlapping slice basecalls into a consensus read sequence.

## When to use

Use when raw nanopore fast5 electrical signals must be translated directly into nucleotide sequence without event segmentation, or when an open-source, trainable alternative to ONT's proprietary Albacore/Metrichor basecallers is needed.

## When NOT to use

- Input is already segmented/event-tabulated signal (e.g.ONT event data) — prior two-stage basecallers handle that and Chiron's raw-signal slicing is unnecessary
- Direct-RNA fast5 files with adapter and polyA stall still present — call BoostNano first, since Chiron's RNA model expects preprocessed signal
- Only FASTQ/FASTA reads are available with no raw signal — there is nothing to basecall; use read-level analysis skills instead

## Inputs

- Raw nanopore signal files in .fast5 (HDF5) format, e.g. MinION R9.4 runs (DNA or direct-RNA)
- Pre-trained Chiron model folder (DNA_default or RNA_default) or a user-trained model
- Optional: reference genome (e.g. Phage Lambda NC_001416.1, E. coli) for accuracy benchmarking

## Outputs

- Per-fast5 basecalled FASTA/FASTQ files in the result folder
- Reassembled per-read consensus sequences in consensus_sequences.fasta (FASTA, A/C/G/T only)
- Segment, raw, and metadata subfolders produced by chiron call

## How to apply

Read the raw electrical signal from each fast5 file with h5py, then slice the signal with a 300-sample sliding window shifted by 30 samples so consecutive windows overlap by 270 samples. Feed the slices into the Chiron CNN+RNN+CTC basecaller in batches — the paper's streaming configuration used batch size 1,100, while the README's dna-pre preset uses segment_len 400, jump 390, batch_size 400, and beam width 30; reduce batch size if GPU/RAM is limited. Choose a decoder: greedy decoding (--beam 0) is faster (~1652 bp/s on GPU) while beam search (e.g. beam 30 or 50) gives higher accuracy (~1204 bp/s). For direct RNA, preprocess fast5 files with BoostNano to segment out the adapter and polyA tail before calling with the RNA_default model and --mode rna. Reassemble the per-slice short basecalls by pairwise alignment between neighboring reads and output a consensus sequence per input read to FASTA; note that larger sliding windows improve accuracy at the cost of memory and running time.

## Related tools

- **Chiron** (CNN+RNN+CTC end-to-end basecaller that translates fast5 raw signal into nucleotide sequence; invoked via 'chiron call' with DNA/RNA presets and beam/greedy decoders) — https://github.com/haotianteng/Chiron
- **h5py** (Python library used to read the raw electrical signal out of each fast5 HDF5 file before slicing) — https://github.com/h5py/h5py
- **TensorFlow** (Deep learning framework in which Chiron's CNN+RNN+CTC model is implemented (install tensorflow==1.15 or tensorflow-gpu==1.15))
- **BoostNano** (Optional preprocessing tool that segments and removes adapter and polyA tails from raw RNA signal before Chiron RNA basecalling) — https://github.com/haotianteng/BoostNano

## Examples

```
python chiron/entry.py call -i chiron/example_folder/ -o <output_folder> -m chiron/model/DNA_default --preset dna-pre
```

## Evaluation signals

- Output FASTA parses as valid FASTA and every consensus sequence contains only A/C/G/T characters
- Consensus length is consistent with the signal-to-base ratio implied by the 30-sample window step
- Read-level accuracy is within ~1% of Albacore v2.0.1 on bacterial/viral DNA and within ~2% on Human DNA (NA12878)
- Decoder rates fall in expected ranges (GPU: ~1652 bp/s greedy, ~1204 bp/s at beam width 50; CPU: ~21 bp/s greedy)
- Total basecalling throughput exceeds 2,000 bases per second on desktop GPUs for the full model

## Limitations

- High computational cost: substantially more expensive than Albacore and somewhat more than BasecRAWller
- Increasing sliding window size improves accuracy but increases memory and running time
- Accuracy gap of up to 2% versus Albacore remains on Human DNA; more training on broader species spectra expected to help
- Further accuracy gains depend on better pairwise alignment of overlapping slice reads and consensus calling
- RNA basecalling requires BoostNano preprocessing (adapter/polyA removal); only Python < 3.9 tested for BoostNano, and TF 1.15 required

## Evidence

- [discussion] the raw signal was first sliced by a 300 length window, the window was slided by 30, and the sliced segments were fed into the basecaller with a batch size equal to 1,100.: "the raw signal was first sliced by a 300 length window, the window was slided by 30, and the sliced segments were fed into the basecaller with a batch size equal to 1,100."
- [discussion] the output short reads were simply assembled by a pair-wise alignment between neighboring reads, and the consensus sequence was output from this alignment.: "the output short reads were simply assembled by a pair-wise alignment between neighboring reads, and the consensus sequence was output from this alignment."
- [readme] 'start':0,'batch_size':400,'segment_len':400,'jump':390,'threads':0,'beam':30: "'start':0,'batch_size':400,'segment_len':400,'jump':390,'threads':0,'beam':30"
- [intro] Chiron has a novel architecture that couples a convolutional neural network (CNN) with an RNN and a connectionist temporal classification (CTC) decoder: "Chiron has a novel architecture that couples a convolutional neural network (CNN) with an RNN and a connectionist temporal classification (CTC) decoder"
- [discussion] Increasing the size of the sliding window will also improve accuracy but at the cost of increased memory and running time.: "Increasing the size of the sliding window will also improve accuracy but at the cost of increased memory and running time."
