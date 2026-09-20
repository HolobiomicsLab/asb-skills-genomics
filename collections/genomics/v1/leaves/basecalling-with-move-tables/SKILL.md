---
name: basecalling-with-move-tables
description: Use when ONT POD5 or FAST5 signal files must be basecalled with retained
  move tables so downstream signal-based tools (e.g. DeepMod2) can anchor ionic current
  signals to basecalls for methylation calling.
when_to_use_negative:
- The input BAM already contains move tables (proceed directly to minimap2 alignment
  or DeepMod2 detect instead of re-basecalling).
- You only need sequence-level analysis (no signal-based 5mC calling), so move-table
  retention adds unnecessary pipeline complexity.
- The signal files come from a chemistry for which no matching Guppy/Dorado model
  exists.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: Dorado
  role: Basecaller used with --emit-moves (and chemistry-matched model) to produce
    a BAM with move tables from POD5/FAST5 inputs
  repo: https://github.com/nanoporetech/dorado
- name: Guppy
  role: Alternative ONT basecaller run with --bam_out --moves_out and a kit-matched
    config to emit move tables in the output BAM
- name: samtools
  role: Converts the basecalled BAM to FASTQ while retaining move-table tags (-T "*"
    or -T mv,ts) prior to alignment; cat combines pass/fail shards
- name: minimap2
  role: Aligns reads to GRCh38 with -ax map-ont -y so move tables survive into the
    aligned BAM for reference anchoring
- name: DeepMod2
  role: Downstream consumer that requires move-table-bearing BAMs to align signals
    to basecalls for BiLSTM 5mC prediction
  repo: https://github.com/WGLab/DeepMod2
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-45778-y
    title: A signal processing and deep learning framework for methylation detection
      using Oxford Nanopore sequencing
  source_content_hash: sha256:be06fcfcfe1aaec5b889602724120e85e0347786eaa83419e97fe33e3d810d93
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nanoporetech/dorado
  tools:
  - DeepMod2
  - Dorado
  - Guppy
  - minimap2
  - modkit
  - samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# basecalling-with-move-tables

## Summary

Basecall Oxford Nanopore POD5/FAST5 signal files with Dorado (--emit-moves) or Guppy (--moves_out) so the resulting BAM carries per-read move tables, which DeepMod2 requires to align ionic-current signals to basecalls for 5mC methylation detection.

## When to use

Use when ONT POD5 or FAST5 signal files must be basecalled with retained move tables so downstream signal-based tools (e.g. DeepMod2) can anchor ionic current signals to basecalls for methylation calling.

## When NOT to use

- The input BAM already contains move tables (proceed directly to minimap2 alignment or DeepMod2 detect instead of re-basecalling).
- You only need sequence-level analysis (no signal-based 5mC calling), so move-table retention adds unnecessary pipeline complexity.
- The signal files come from a chemistry for which no matching Guppy/Dorado model exists.

## Inputs

- POD5 signal files (R10.4.1 flowcell datasets)
- FAST5 signal files (R9.4.1 flowcell datasets)
- Chemistry-matched basecaller model or config (e.g. dna_r10.4.1_e8.2_400bps_hac@v4.3.0 / dna_r10.4.1_e8.2_400bps_5khz_hac.cfg)
- Reference genome FASTA (e.g. GRCh38 analysis set) for optional aligned output

## Outputs

- Basecalled BAM file carrying per-read move tables (mv tag)
- Aligned BAM (aligned.bam) per sample/flowcell preserving move tables for reference-anchored methylation calling

## How to apply

Basecall each POD5/FAST5 dataset with Dorado using --emit-moves and a chemistry-matched model (e.g. dna_r10.4.1_e8.2_400bps_hac@v4.3.0 for R10.4.1), or with Guppy using --bam_out --moves_out and the matching config (e.g. dna_r10.4.1_e8.2_400bps_5khz_hac.cfg); the move tables are what allow signal alignment with the basecalls. If Guppy emits pass/fail shard BAMs, concatenate them with samtools cat before proceeding. Align the basecalled reads to the reference genome (e.g. GRCh38 analysis set) while retaining move tables by piping samtools fastq with the tag-retention flag (-T "*" or -T mv,ts) into minimap2 -ax map-ont with -y, producing an aligned.bam per sample/flowcell. Alignment is recommended because it enables reference-anchored signal handling and estimation of per-site methylation levels in the downstream DeepMod2 detect step.

## Related tools

- **Dorado** (Basecaller used with --emit-moves (and chemistry-matched model) to produce a BAM with move tables from POD5/FAST5 inputs) — https://github.com/nanoporetech/dorado
- **Guppy** (Alternative ONT basecaller run with --bam_out --moves_out and a kit-matched config to emit move tables in the output BAM)
- **samtools** (Converts the basecalled BAM to FASTQ while retaining move-table tags (-T "*" or -T mv,ts) prior to alignment; cat combines pass/fail shards)
- **minimap2** (Aligns reads to GRCh38 with -ax map-ont -y so move tables survive into the aligned BAM for reference anchoring)
- **DeepMod2** (Downstream consumer that requires move-table-bearing BAMs to align signals to basecalls for BiLSTM 5mC prediction) — https://github.com/WGLab/DeepMod2

## Examples

```
dorado basecaller dna_r10.4.1_e8.2_400bps_hac@v4.3.0 pod5s/ --emit-moves > basecall.bam
```

## Evaluation signals

- Each read in the output BAM carries the move table tag, and tags survive alignment because samtools fastq -T and minimap2 -y were used.
- One valid, sorted/indexable aligned BAM is produced per sample/flowcell (Guppy pass/fail shards first merged with samtools cat).
- DeepMod2 detect runs successfully in reference-anchored mode on the aligned BAM plus signal files, which is only possible when move tables are present.
- Benchmarking downstream methylation calls yields per-read F1 within the paper-reported ranges: 95.7–97.1% for R9.4.1 Ashkenazim trio and 97.5–98.2% for R10.4.1 (comparable to Guppy's 97.9–98.5%).

## Limitations

- Move tables only approximately demarcate signal boundaries between consecutive bases, which can produce uninformative signal summary statistics for some bases.
- The basecaller model/config must match the flowcell chemistry (R9.4.1 vs R10.4.1) and kit, or the move tables and basecalls will be invalid for signal anchoring.
- Guppy output may be split into pass/fail shard BAMs that must be concatenated before alignment.

## Evidence

- [results] It requires POD5/FAST5 files to be basecalled using Guppy or Dorado with move table output to allow signal alignment with the basecalls: "It requires POD5/FAST5 files to be basecalled using Guppy or Dorado with move table output to allow signal alignment with the basecalls"
- [results] The BAM file is recommended to be aligned to a reference genome to allow reference anchoring of signals and estimation of per-site methylation levels: "The BAM file is recommended to be aligned to a reference genome to allow reference anchoring of signals and estimation of per-site methylation levels"
- [readme] Basecall your FAST5/POD5 files with Dorado (using `--emit-moves`) or Guppy (using `--bam_out --moves_out`) parameters to get a BAM file with move tables: [evidence span withheld — anchor status: external_doc]
- [readme] samtools fastq basecall.bam -T mv,ts | minimap2 -ax map-ont ref.fa - -y -t NUM_THREADS |samtools view -o aligned.bam: [evidence span withheld — anchor status: external_doc]
