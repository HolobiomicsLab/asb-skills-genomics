---
name: raw-signal-sliding-window-segmentation
description: Use when input is raw nanopore fast5 electrical signal too long for direct
  end-to-end basecalling, and you need to split it into overlapping 300-sample slices,
  basecall each, and reassemble a full read.
when_to_use_negative:
- Input is already event-segmented or re-squiggled signal (e.g., Tombo re-squiggle
  output) — segmentation errors from variable translocation speed motivated end-to-end
  basecalling, so event tables are not the right input
- Reads are already basecalled to FASTQ and only consensus/assembly is needed (use
  Minimap2/Miniasm plus Racon polishing instead)
- Direct RNA signal with adapter/polyA tails — preprocess with BoostNano first, or
  use the RNA preset; raw DNA-style slicing alone will not trim those tails
edam_operation: http://edamontology.org/operation_3236
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: Chiron
  role: CNN+RNN+CTC basecaller that consumes the sliding-window slices and emits slice-level
    basecalls
  repo: https://github.com/haotianteng/chiron
- name: h5py
  role: Reads the raw electrical signal out of each fast5 file before windowing
  repo: https://github.com/h5py/h5py
- name: TensorFlow
  role: Backend required to run Chiron inference (tensorflow==1.15 or tensorflow-gpu==1.15)
  repo: https://github.com/haotianteng/chiron
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giy037
    title: 'Chiron: translating nanopore raw signal directly into nucleotide sequence
      using deep learning'
  source_content_hash: sha256:b7dad8c5f4a79125aa0e73c891174a17d099a747a410a320b4bac0730dd9ec96
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haotianteng/chiron
  tools:
  - Chiron
  - TensorFlow
  - h5py
  edam_operation: http://edamontology.org/operation_3236
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# raw-signal-sliding-window-segmentation

## Summary

Segments raw nanopore electrical signal (fast5) by sliding a fixed 300-sample window with a 30-sample step so the Chiron CNN+RNN+CTC basecaller can process long reads in overlapping slices that are later assembled into one consensus sequence.

## When to use

Use when input is raw nanopore fast5 electrical signal too long for direct end-to-end basecalling, and you need to split it into overlapping 300-sample slices, basecall each, and reassemble a full read.

## When NOT to use

- Input is already event-segmented or re-squiggled signal (e.g., Tombo re-squiggle output) — segmentation errors from variable translocation speed motivated end-to-end basecalling, so event tables are not the right input
- Reads are already basecalled to FASTQ and only consensus/assembly is needed (use Minimap2/Miniasm plus Racon polishing instead)
- Direct RNA signal with adapter/polyA tails — preprocess with BoostNano first, or use the RNA preset; raw DNA-style slicing alone will not trim those tails

## Inputs

- Oxford Nanopore MinION raw signal files in fast5 format (e.g., from the NA12878 human dataset or Lambda/E.coli runs)
- Trained Chiron model (DNA_default preset dna-pre, or RNA_default with rna-pre)

## Outputs

- Slice-level short basecalls (~10-20 bp per 300-sample window)
- One reassembled consensus sequence per input read, written to consensus_sequences.fasta (FASTA, A/C/G/T only)

## How to apply

Read each fast5 raw signal with h5py, then slide a 300-sample window with a step of 30 so consecutive windows overlap by 270 samples. Feed slices to the Chiron CNN+RNN+CTC basecaller in batches — Chiron's streaming mode in the paper used batch size 1,100, while the DNA default preset uses segment_len 400, jump 390, batch_size 400, beam 30. Because each window yields roughly 10-20 bp, the short slice-level basecalls are reassembled into one consensus sequence per input read by pairwise alignment between neighboring reads (overlap assembly), and written to a FASTA. Increasing the window size improves accuracy at the cost of memory and runtime, so choose the window/jump trade-off according to available GPU/CPU resources.

## Related tools

- **Chiron** (CNN+RNN+CTC basecaller that consumes the sliding-window slices and emits slice-level basecalls) — https://github.com/haotianteng/chiron
- **h5py** (Reads the raw electrical signal out of each fast5 file before windowing) — https://github.com/h5py/h5py
- **TensorFlow** (Backend required to run Chiron inference (tensorflow==1.15 or tensorflow-gpu==1.15)) — https://github.com/haotianteng/chiron

## Examples

```
python chiron/entry.py call -i <input_fast5_folder> -o <output_folder> -m chiron/model/DNA_default --preset dna-pre
```

## Evaluation signals

- consensus_sequences.fasta exists and parses as valid FASTA
- Every consensus sequence contains only A/C/G/T nucleotide characters
- Consensus lengths are consistent with the signal-to-base ratio implied by the 30-sample step (each 300-sample window yields ~10-20 bp)
- Output folders raw, result, segments, meta, and reference are created by chiron call
- Batch/window settings match the chosen preset (dna-pre: segment_len 400, jump 390, batch_size 400) or the paper's streaming configuration (window 300, step 30, batch 1,100)

## Limitations

- Increasing the sliding window size improves accuracy but raises memory and running time
- Chiron is substantially more computationally expensive than Albacore and somewhat more expensive than BasecRAWller
- Chiron is within ~1% accuracy of Albacore on bacterial DNA but only within ~2% on Human DNA

## Evidence

- [discussion] the raw signal was first sliced by a 300 length window, the window was slided by 30, and the sliced segments were fed into the basecaller with a batch size equal to 1,100.: "the raw signal was first sliced by a 300 length window, the window was slided by 30, and the sliced segments were fed into the basecaller with a batch size equal to 1,100."
- [discussion] the output short reads were simply assembled by a pair-wise alignment between neighboring reads, and the consensus sequence was output from this alignment.: "the output short reads were simply assembled by a pair-wise alignment between neighboring reads, and the consensus sequence was output from this alignment."
- [readme] DNA preset arguments value(dna-pre): 'start':0,'batch_size':400,'segment_len':400,'jump':390,'threads':0,'beam':30: "DNA preset arguments value(dna-pre): 'start':0,'batch_size':400,'segment_len':400,'jump':390,'threads':0,'beam':30"
- [discussion] Increasing the size of the sliding window will also improve accuracy but at the cost of increased memory and running time.: "Increasing the size of the sliding window will also improve accuracy but at the cost of increased memory and running time."
