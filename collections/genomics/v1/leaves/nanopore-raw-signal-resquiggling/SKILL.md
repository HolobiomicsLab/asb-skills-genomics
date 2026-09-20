---
name: nanopore-raw-signal-resquiggling
description: 'Use when preparing Nanopore DRS FAST5 reads for RedNano feature extraction:
  after guppy basecalling (--fast5_out) and before extracting k-mer raw signals, the
  signals must be resquiggled against a reference transcript FASTA.'
when_to_use_negative:
- Only a genome reference FASTA is available and no transcriptome FASTA can be built
  — tombo resquiggle requires the reference gene/transcript fasta, not a genome file
- The input is already a RedNano feature table (chrom/pos/k_mer/k_signals_rect/qual/mis/ins/dele
  rows) — resquiggling is upstream of feature extraction
- Working with Illumina or other non-nanopore short-read data — resquiggling only
  applies to Nanopore raw signal FAST5 files
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3524
tools:
- name: ont-tombo
  role: Performs the resquiggling of raw Nanopore signals to the reference transcript
    (v1.5.1, RawGenomeCorrected_001 corrected group)
  repo: https://github.com/Derryxu/RedNano
- name: ont-fast5-api
  role: Splits multi-read FAST5 files into single-read FAST5 files prior to basecalling
    and resquiggling (v4.0.2)
  repo: https://github.com/Derryxu/RedNano
- name: Guppy
  role: Basecalls the FAST5 reads with --fast5_out (v3.1.5, rna_r9.4.1_70bps_hac.cfg)
    to produce the Basecall_1D_001 group consumed by tombo
- name: RedNano
  role: Consumes the resquiggled FAST5 workspace via extract_features.py to combine
    raw-signal and basecalling-error features
  repo: https://github.com/Derryxu/RedNano
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae375
    title: RNA m6A detection using raw current signals and basecalling errors from
      Nanopore direct RNA sequencing reads
  source_content_hash: sha256:5515e82584cad1bf79ab07803f4a05eb244caf9ad94197a7cc96b9817c4c2063
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Derryxu/RedNano
  tools:
  - CUDA
  - Guppy
  - Python
  - RedNano
  - h5py
  - jvarkit
  - minimap2
  - ont-fast5-api
  - ont-tombo
  - samtools
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# nanopore-raw-signal-resquiggling

## Summary

Realign Nanopore direct RNA raw current signals to a reference transcript sequence using tombo resquiggle, producing event-level alignments (RawGenomeCorrected_001) that RedNano needs to extract combined raw-signal and basecalling-error features for m6A detection.

## When to use

Use when preparing Nanopore DRS FAST5 reads for RedNano feature extraction: after guppy basecalling (--fast5_out) and before extracting k-mer raw signals, the signals must be resquiggled against a reference transcript FASTA.

## When NOT to use

- Only a genome reference FASTA is available and no transcriptome FASTA can be built — tombo resquiggle requires the reference gene/transcript fasta, not a genome file
- The input is already a RedNano feature table (chrom/pos/k_mer/k_signals_rect/qual/mis/ins/dele rows) — resquiggling is upstream of feature extraction
- Working with Illumina or other non-nanopore short-read data — resquiggling only applies to Nanopore raw signal FAST5 files

## Inputs

- Single-read FAST5 files with guppy basecalls (Basecall_1D_001 group, produced with --fast5_out)
- Reference transcript FASTA (e.g. referance.transcript.fa), not a genome reference

## Outputs

- Resquiggled FAST5 files containing tombo event alignments in the RawGenomeCorrected_001 corrected group
- Downstream inputs for RedNano extract_features.py (-i fast5 workspace, --corrected_group RawGenomeCorrected_001)

## How to apply

First split multi-read FAST5 files into single-read files with ont_fast5_api (multi_to_single_fast5 -t 30) and basecall with guppy v3.1.5 using the rna_r9.4.1_70bps_hac.cfg configuration with --fast5_out so basecalled FAST5s are retained. Then run tombo v1.5.1 resquiggle on the guppy workspace with --basecall-group Basecall_1D_001, --fit-global-scale, --include-event-stdev, and --corrected-group RawGenomeCorrected_001, using multiple processes (e.g. --processes 30). Critically, the reference passed to resquiggle must be a transcript/gene FASTA, not a genome reference, because direct RNA reads are already transcript-derived. The resulting RawGenomeCorrected_001 corrected-group event alignments are later read by RedNano's scripts/extract_features.py via the --corrected_group argument to pair k-mer raw signals (k=5, signal length 65) with basecalling-error features from the sam2tsv-derived per-read TSVs.

## Related tools

- **ont-tombo** (Performs the resquiggling of raw Nanopore signals to the reference transcript (v1.5.1, RawGenomeCorrected_001 corrected group)) — https://github.com/Derryxu/RedNano
- **ont-fast5-api** (Splits multi-read FAST5 files into single-read FAST5 files prior to basecalling and resquiggling (v4.0.2)) — https://github.com/Derryxu/RedNano
- **Guppy** (Basecalls the FAST5 reads with --fast5_out (v3.1.5, rna_r9.4.1_70bps_hac.cfg) to produce the Basecall_1D_001 group consumed by tombo)
- **RedNano** (Consumes the resquiggled FAST5 workspace via extract_features.py to combine raw-signal and basecalling-error features) — https://github.com/Derryxu/RedNano

## Examples

```
tombo resquiggle --overwrite /path/to/fast5_guppy/workspace/ --basecall-group Basecall_1D_001 referance.transcript.fa --fit-global-scale --include-event-stdev --corrected-group RawGenomeCorrected_001 --processes 30
```

## Evaluation signals

- tombo resquiggle completes without errors on all single-read FAST5s and writes events into the RawGenomeCorrected_001 corrected group
- The resquiggle reference is a transcript FASTA; the command used --fit-global-scale and --include-event-stdev flags
- Subsequent extract_features.py runs with --corrected_group RawGenomeCorrected_001 produce a feature table with k_signals_rect, qual, mis, ins, dele columns
- The guppy workspace passed as -i contains basecalled FAST5s from the --fast5_out run (Basecall_1D_001 group present)

## Limitations

- tombo resquiggle must not be given a genome reference; only a reference gene/transcript fasta is valid
- Resquiggling depends on guppy v3.1.5 basecalls with --fast5_out; basecalls lacking the Basecall_1D_001 group in FAST5 format cannot be processed
- Multi-read FAST5 files must first be split with ont_fast5_api multi_to_single_fast5 before this step

## Evidence

- [readme] The tombo (v1.5.1) resquiggle referance.transcript.fa should not be genome file, it should be the referance gene fasta file.: "The tombo (v1.5.1) resquiggle referance.transcript.fa should not be genome file, it should be the referance gene fasta file."
- [readme] tombo resquiggle command with fit-global-scale: "tombo resquiggle --overwrite /path/to/fast5_guppy/workspace/ --basecall-group Basecall_1D_001 referance.transcript.fa --fit-global-scale --include-event-stdev --corrected-group RawGenomeCorrected_001"
- [abstract] RedNano is a deep-learning method that utilizes both raw signals and basecalling errors to detect m6A from Nanopore DRS reads: "Here, we propose RedNano, a deep-learning method designed to detect m6A from Nanopore DRS reads by utilizing both raw signals and basecalling errors."
- [readme] multi_to_single_fast5 splitting step: "When the downloaded Fast5 file(s) of raw nanopore sequencing reads contains multiple reads-id, use ont_fast5_api to split the multiple read Fast5 file(s) into single read Fast5 files."
