---
name: consensus-sequence-assembly
description: Use when you have raw .fast5 nanopore signals that were sliced with a
  fixed window (e.g. 300 samples, step 30, or Chiron's DNA preset segment_len 400
  / jump 390) and need the overlapping slice-level basecalls merged into one full-length
  read sequence.
when_to_use_negative:
- Input is already a set of full-length basecalled reads needing reference alignment
  or de novo assembly rather than within-read slice merging — the paper instead aligns
  basecalled reads to a reference with graphmap or assembles drafts with Minimap2/Miniasm
  plus Racon polishing.
- Input is direct-RNA signal with adapter/polyA tails that still require segmentation
  and trimming (use BoostNano preprocessing, not consensus assembly).
- The goal is to recover per-slice basecalls rather than a merged per-read sequence
  — Chiron already writes per-fast5 fastq/fasta results in the output 'result' folder
  without assembly.
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Chiron
  role: CNN+RNN+CTC basecaller that consumes the sliding-window signal slices and
    produces the short overlapping basecalled segments to be assembled; also the source
    of the DNA_default preset (segment_len 400, jump 390, batch_size 400)
  repo: https://github.com/haotianteng/chiron
- name: h5py
  role: Python library used to read the raw electrical signal from each fast5 file
    before windowing and basecalling
  repo: https://github.com/h5py/h5py
- name: BoostNano
  role: Optional preprocessing (Chiron project tool) that segments and removes adapter
    and polyA tails from raw signal before basecalling, chiefly for direct-RNA input
  repo: https://github.com/haotianteng/BoostNano
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giy037
    title: 'Chiron: translating nanopore raw signal directly into nucleotide sequence
      using deep learning'
  source_content_hash: sha256:aa52a38bfe260ec842af62d1e506ee105bdb6b22a181c4348430256d824da8a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haotianteng/chiron
  tools:
  - BoostNano
  - Chiron
  - h5py
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# consensus-sequence-assembly

## Summary

Reassembles the short (~10-20 bp) basecalled slices produced by Chiron's 300-sample sliding-window basecaller into a single consensus sequence per raw nanopore read, using pairwise alignment between neighboring slices as described in the Chiron paper's streaming discussion.

## When to use

Use when you have raw .fast5 nanopore signals that were sliced with a fixed window (e.g. 300 samples, step 30, or Chiron's DNA preset segment_len 400 / jump 390) and need the overlapping slice-level basecalls merged into one full-length read sequence.

## When NOT to use

- Input is already a set of full-length basecalled reads needing reference alignment or de novo assembly rather than within-read slice merging — the paper instead aligns basecalled reads to a reference with graphmap or assembles drafts with Minimap2/Miniasm plus Racon polishing.
- Input is direct-RNA signal with adapter/polyA tails that still require segmentation and trimming (use BoostNano preprocessing, not consensus assembly).
- The goal is to recover per-slice basecalls rather than a merged per-read sequence — Chiron already writes per-fast5 fastq/fasta results in the output 'result' folder without assembly.

## Inputs

- Raw nanopore electrical signal files in .fast5 format (read via h5py)
- Slice-level basecalls produced by the Chiron CNN+RNN+CTC basecaller (short ~10-20 bp segments per window)

## Outputs

- One reassembled consensus sequence per input read, written to consensus_sequences.fasta in FASTA format

## How to apply

Read each raw signal from its fast5 file with h5py and slice it with a fixed 300-sample window slid by 30 samples, so consecutive windows overlap by 270 samples; feed slices into the Chiron CNN+RNN+CTC basecaller (streaming configuration used batch size 1,100, while the DNA preset uses segment_len 400, jump 390, batch_size 400, beam 30). Because neighboring slices overlap, each genomic position is covered by multiple short basecalled segments, so assemble the output short reads by pairwise alignment between neighboring reads and emit the consensus sequence from that alignment, as the paper does to recover the full read in real time. Write each reassembled sequence to a FASTA file (e.g. consensus_sequences.fasta). Choose the window/step trade-off deliberately: a larger sliding window improves accuracy but at the cost of increased memory and running time, and better alignment of the overlapping slices is an explicit route to accuracy gains.

## Related tools

- **Chiron** (CNN+RNN+CTC basecaller that consumes the sliding-window signal slices and produces the short overlapping basecalled segments to be assembled; also the source of the DNA_default preset (segment_len 400, jump 390, batch_size 400)) — https://github.com/haotianteng/chiron
- **h5py** (Python library used to read the raw electrical signal from each fast5 file before windowing and basecalling) — https://github.com/h5py/h5py
- **BoostNano** (Optional preprocessing (Chiron project tool) that segments and removes adapter and polyA tails from raw signal before basecalling, chiefly for direct-RNA input) — https://github.com/haotianteng/BoostNano

## Examples

```
python chiron/entry.py call -i <input_fast5_folder> -o <output_folder> -m chiron/model/DNA_default --preset dna-pre
```

## Evaluation signals

- consensus_sequences.fasta exists and parses as valid FASTA
- Every consensus sequence contains only A/C/G/T nucleotide characters
- Consensus sequence lengths are consistent with the signal-to-base ratio implied by the 30-sample sliding step (roughly 10-20 bp per 300-sample window)
- Per-read consensus output corresponds one-to-one with an input fast5 file (result files share the fast5 file name)

## Limitations

- Increasing the sliding window size improves accuracy but at the cost of increased memory and running time
- Further accuracy gains depend on better alignment of the overlapping slices and consensus calling — the paper notes the simple pairwise alignment between neighboring reads is a key improvement avenue
- The streaming/slicing configuration (300-sample window, step 30, batch size 1,100) differs from Chiron's DNA default preset (segment_len 400, jump 390, batch_size 400), so parameters must be matched to the intended setup

## Evidence

- [discussion] the raw signal was first sliced by a 300 length window, the window was slided by 30, and the sliced segments were fed into the basecaller with a batch size equal to 1,100.: "the raw signal was first sliced by a 300 length window, the window was slided by 30, and the sliced segments were fed into the basecaller with a batch size equal to 1,100."
- [discussion] the output short reads were simply assembled by a pair-wise alignment between neighboring reads, and the consensus sequence was output from this alignment.: "the output short reads were simply assembled by a pair-wise alignment between neighboring reads, and the consensus sequence was output from this alignment."
- [readme] 'start':0,'batch_size':400,'segment_len':400,'jump':390,'threads':0,'beam':30: "'start':0,'batch_size':400,'segment_len':400,'jump':390,'threads':0,'beam':30"
- [discussion] Increasing the size of the sliding window will also improve accuracy but at the cost of increased memory and running time.: "Increasing the size of the sliding window will also improve accuracy but at the cost of increased memory and running time."
- [discussion] Chiron has the potential to stream these input raw signal "slices" into output sequence data: "Chiron has the potential to stream these input raw signal "slices" into output sequence data"
