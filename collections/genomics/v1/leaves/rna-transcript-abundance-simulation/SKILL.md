---
name: rna-transcript-abundance-simulation
description: Use when simulating ONT direct-RNA (RNA004) nanopore signals from a transcriptome
  where per-transcript abundance and truncation profiles must be honored, e.g. reproducing
  HG002-RNA004 experiments.
when_to_use_negative:
- Input is DNA genomic data — use the ont_r1041_dna_5khz preset workflow instead of
  the RNA004 preset and transcript profile
- Simulating from already-basecalled reads rather than a transcriptome reference —
  use Read mode without the abundance TSV
- The analysis does not require transcript abundance/truncation fidelity and uniform
  read sampling is acceptable — omit --trans-profile rather than supplying a fabricated
  profile
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: NanoSimFormer
  role: Generates direct-RNA RNA004 POD5 signals from a transcriptome using --trans-profile
    abundance/truncation TSV and the ont_rna004_4khz preset
  repo: https://github.com/BioinfoSZU/NanoSimFormer
- name: Dorado
  role: Basecalls the simulated RNA signals into FASTQ/BAM when --basecall is enabled
  repo: github.com/nanoporetech/dorado
- name: Docker image chobits323/nano-sim:v1.1
  role: Recommended execution environment containing pre-trained model files for the
    simulation
  repo: https://github.com/BioinfoSZU/NanoSimFormer
- name: minimap2
  role: Aligns basecalled RNA reads to the transcriptome for accuracy and abundance-correlation
    assessment
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btag402
    title: 'NanoSimFormer: an end-to-end transformer-based nanopore signal simulator
      with basecaller guidance'
  source_content_hash: sha256:f352fee86eb54a4834f4699e034684ccc07379c300e8f8f4d7a98ab767bb8804
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioinfoSZU/NanoSimFormer
  tools:
  - Docker
  - Docker image chobits323/nano-sim:v1.1
  - Dorado
  - NanoSimFormer
  - Squigulator
  - minimap2
  - seq2squiggle
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# rna-transcript-abundance-simulation

## Summary

Simulate direct-RNA nanopore signals (RNA004) with transcript-specific abundance and truncation using NanoSimFormer's --trans-profile 3-column TSV and the ont_rna004_4khz preset, enabling abundance-faithful synthetic transcriptomic signal data.

## When to use

Use when simulating ONT direct-RNA (RNA004) nanopore signals from a transcriptome where per-transcript abundance and truncation profiles must be honored, e.g. reproducing HG002-RNA004 experiments.

## When NOT to use

- Input is DNA genomic data — use the ont_r1041_dna_5khz preset workflow instead of the RNA004 preset and transcript profile
- Simulating from already-basecalled reads rather than a transcriptome reference — use Read mode without the abundance TSV
- The analysis does not require transcript abundance/truncation fidelity and uniform read sampling is acceptable — omit --trans-profile rather than supplying a fabricated profile

## Inputs

- Transcriptome reference FASTA (e.g. HG002 transcripts)
- 3-column transcript profile TSV specifying transcript abundance and truncation
- NanoSimFormer ont_rna004_4khz preset (model checkpoints from Zenodo 10.5281/zenodo.20364515 if not using Docker)

## Outputs

- Simulated POD5 signal file of direct-RNA reads
- Basecalled FASTQ (or BAM with --emit-bam) from Dorado via --basecall
- Optional sampled transcript FASTA via --sample-output
- Per-transcript abundance comparison after minimap2 alignment

## How to apply

Run NanoSimFormer in Reference mode against a transcriptome FASTA with --preset ont_rna004_4khz, supplying a 3-column TSV via --trans-profile that encodes transcript IDs with abundance and truncation values so simulated read counts mirror experimental abundance. Enable --basecall so Dorado produces a basecalled FASTQ (or --emit-bam for BAM) alongside the simulated POD5, then align reads to the transcriptome with minimap2 to assess accuracy and abundance correlation. The article reports NanoSimFormer-derived reads maintain high correlations with experimental abundance, so judge success by comparing simulated per-transcript read counts against the input profile and verifying basecalling accuracy/Q-scores are consistent with real RNA004 baselines.

## Related tools

- **NanoSimFormer** (Generates direct-RNA RNA004 POD5 signals from a transcriptome using --trans-profile abundance/truncation TSV and the ont_rna004_4khz preset) — https://github.com/BioinfoSZU/NanoSimFormer
- **Dorado** (Basecalls the simulated RNA signals into FASTQ/BAM when --basecall is enabled) — github.com/nanoporetech/dorado
- **Docker image chobits323/nano-sim:v1.1** (Recommended execution environment containing pre-trained model files for the simulation) — https://github.com/BioinfoSZU/NanoSimFormer
- **minimap2** (Aligns basecalled RNA reads to the transcriptome for accuracy and abundance-correlation assessment)

## Examples

```
python -m nano_signal_simulator simulate --input transcripts.fasta --output out_rna --mode Reference --preset ont_rna004_4khz --trans-profile transcript_profile.tsv --basecall --gpu 0
```

## Evaluation signals

- Simulated per-transcript read counts correlate with the abundance column of the input 3-column trans-profile TSV (article claims high correlations with experimental abundance)
- Dorado basecalled FASTQ is produced with median basecalling accuracy exceeding 99% and Q-scores above 22.8, as reported for ONT direct RNA sequencing
- Output directory contains valid POD5 signal files plus basecalled FASTQ (or BAM with --emit-bam)
- minimap2 alignment of basecalled reads to the transcriptome yields per-read alignments with plausibly high identity for RNA004 reads
- Read length distribution and truncation behavior match the truncation column of the trans-profile TSV

## Limitations

- Requires an NVIDIA GPU with CUDA compute capability >= 8.x and NVIDIA driver >= 525; tested on Ubuntu 22.04 only
- Model checkpoints must be downloaded from Zenodo (10.5281/zenodo.20364515) unless using the Docker image, which bundles them
- Supports only the RNA004 direct-RNA chemistry preset (ont_rna004_4khz); other RNA chemistries require adapting the model per the extensibility guide

## Evidence

- [intro] It supports both ONT DNA (R10.4.1) and direct-RNA (RNA004) signal simulation, enabling users to generate synthetic POD5 files from references or existing basecalled reads.: "It supports both ONT DNA (R10.4.1) and direct-RNA (RNA004) signal simulation, enabling users to generate synthetic POD5 files from references"
- [readme] --trans-profile option description in the simulate subcommand help.: "--trans-profile TRANS_PROFILE
                        3-column TSV file for simulating transcripts with specific abundance and truncation (default: None)"
- [abstract] Article abstract claim on RNA performance and abundance correlation.: "NanoSimFormer achieves a median basecalling accuracy exceeding 99% and Q-scores above 22.8 for Oxford Nanopore Technologies’ latest DNA R10.4.1 and direct RNA sequencing"
- [methods] Methods note on --basecall behavior.: [evidence span withheld — anchor status: not_found]
- [readme] Docker image containing pre-trained model files.: "**Note:** The Docker image already contains all required pre-trained model files."
