---
name: dorado-move-table-basecalling
description: Use when raw ONT signal files (POD5/FAST5) must be converted to a BAM
  with move tables as the front-end of a signal-based methylation-calling workflow
  such as DeepMod2.
when_to_use_negative:
- Downstream caller does not use raw signal (e.g. pure sequence-based tools) — move
  tables are only needed to align ionic current to basecalls
- Input is already a basecalled BAM with move-table tags (mv,ts); go directly to alignment
  or methylation calling
- Flowcell chemistry has no compatible Dorado/Guppy model (DeepMod2 provides models
  only for R10.4.1 and R9.4.1 flowcells)
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: Dorado
  role: Primary basecaller run with --emit-moves to produce a BAM with move tables
    from POD5/FAST5 input
  repo: https://github.com/nanoporetech/dorado
- name: Guppy
  role: Alternative basecaller invoked with --bam_out --moves_out to achieve the same
    move-table-bearing BAM
- name: minimap2
  role: Aligns basecalled reads to the reference with -ax map-ont while preserving
    move-table tags through the samtools fastq -T mv,ts handoff
- name: samtools
  role: Converts the move-table BAM to FASTQ with -T mv,ts and view output for minimap2,
    keeping tags intact
- name: DeepMod2
  role: Downstream consumer that requires the move tables to align signal with basecalls
    for BiLSTM/Transformer methylation inference
  repo: https://github.com/WGLab/DeepMod2
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-45778-y
    title: A signal processing and deep learning framework for methylation detection
      using Oxford Nanopore sequencing
  source_content_hash: sha256:6b7dff24b41f8bc2428fb302676a89abb6543acd5d572832e8dc872ce862ae78
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nanoporetech/dorado
  tools:
  - DeepMod2
  - Dorado
  - Guppy
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# dorado-move-table-basecalling

## Summary

Basecall Oxford Nanopore POD5/FAST5 signal files with Dorado (or Guppy) while emitting move tables, so DeepMod2 can align ionic-current signals to basecalls for 5mCpG methylation detection on R9.4.1 and R10.4.1 flowcells.

## When to use

Use when raw ONT signal files (POD5/FAST5) must be converted to a BAM with move tables as the front-end of a signal-based methylation-calling workflow such as DeepMod2.

## When NOT to use

- Downstream caller does not use raw signal (e.g. pure sequence-based tools) — move tables are only needed to align ionic current to basecalls
- Input is already a basecalled BAM with move-table tags (mv,ts); go directly to alignment or methylation calling
- Flowcell chemistry has no compatible Dorado/Guppy model (DeepMod2 provides models only for R10.4.1 and R9.4.1 flowcells)

## Inputs

- POD5 signal files (e.g. ONT POD5 sample dataset from github.com/WGLab/DeepMod2/releases/tag/v0.3.0)
- FAST5 signal files
- Dorado simplex basecalling model appropriate to the flowcell chemistry (e.g. dna_r10.4.1_e8.2_400bps_hac@v4.3.0)
- Reference genome FASTA (e.g. GRCh38.fa) for downstream alignment

## Outputs

- Unaligned BAM with move tables (MM/ML tags come later from the methylation caller)
- Aligned BAM retaining mv and ts tags after minimap2 -ax map-ont

## How to apply

Run Dorado basecaller with the kit-matched model (e.g. dna_r10.4.1_e8.2_400bps_hac@v4.3.0 for R10.4.1) and the --emit-moves flag to embed move tables in the output BAM; with Guppy, use --bam_out --moves_out instead. Move tables are mandatory downstream because DeepMod2 needs them to align raw signal with basecalls before extracting 19 signal summary statistics per base in a 21-bp window centered at each CpG. Retain move tags through alignment by piping through samtools fastq -T mv,ts into minimap2 -ax map-ont against the reference (e.g. GRCh38) with -y to preserve tags. Aligning the BAM is recommended rather than optional because reference anchoring enables per-site methylation frequencies in addition to per-read calls.

## Related tools

- **Dorado** (Primary basecaller run with --emit-moves to produce a BAM with move tables from POD5/FAST5 input) — https://github.com/nanoporetech/dorado
- **Guppy** (Alternative basecaller invoked with --bam_out --moves_out to achieve the same move-table-bearing BAM)
- **minimap2** (Aligns basecalled reads to the reference with -ax map-ont while preserving move-table tags through the samtools fastq -T mv,ts handoff)
- **samtools** (Converts the move-table BAM to FASTQ with -T mv,ts and view output for minimap2, keeping tags intact)
- **DeepMod2** (Downstream consumer that requires the move tables to align signal with basecalls for BiLSTM/Transformer methylation inference) — https://github.com/WGLab/DeepMod2

## Examples

```
dorado basecaller dna_r10.4.1_e8.2_400bps_hac@v4.3.0 pod5s/ --emit-moves > basecall.bam
```

## Evaluation signals

- Output BAM contains SAM records with the mv (move table) and ts tags present on every read
- minimap2 alignment completes without dropping the mv tag; verify with samtools view on a few reads that tags persist post-alignment
- Basecalling model name matches the flowcell chemistry in the POD5 dataset (e.g. R10.4.1_e8.2_400bps_hac for R10.4.1 data)
- Downstream DeepMod2 detect runs without complaining about missing move tables or signal-to-basecall alignment
- Alignment rate against the reference genome is reasonable for the sample, confirming reads were basecalled and mapped correctly before methylation calling

## Limitations

- Move tables provide only approximate signal-to-basecall alignment; they do not always accurately demarcate boundaries between signals of consecutive bases, which can yield uninformative signal summary statistics for some CpG windows
- Dorado/Guppy models must match the kit; DeepMod2 supports only R10.4.1 and R9.4.1 flowcell data
- Dorado is optimized for Nvidia GPUs and Apple Silicon; on CPU-only hardware it will be slow (motivating DeepMod2 model pruning for the inference step, not basecalling)

## Evidence

- [results] It requires POD5/FAST5 files to be basecalled using Guppy or Dorado with move table output to allow signal alignment with the basecalls: "It requires POD5/FAST5 files to be basecalled using Guppy or Dorado with move table output to allow signal alignment with the basecalls"
- [results] The BAM file is recommended to be aligned to a reference genome to allow reference anchoring of signals and estimation of per-site methylation levels: "The BAM file is recommended to be aligned to a reference genome to allow reference anchoring of signals and estimation of per-site methylation levels"
- [results] the move table does not always accurately demarcate boundaries between signals associated with consecutive bases: "the move table does not always accurately demarcate boundaries between signals associated with consecutive bases, which can lead to uninformative signal summary statistics"
- [readme] Basecall your FAST5/POD5 files with Dorado (using `--emit-moves`) or Guppy (using `--bam_out --moves_out`) parameters to get a BAM file with move tables: [evidence span withheld — anchor status: external_doc]
- [readme] samtools fastq basecall.bam -T mv,ts | minimap2 -ax map-ont ref.fa - -y -t NUM_THREADS: [evidence span withheld — anchor status: external_doc]
