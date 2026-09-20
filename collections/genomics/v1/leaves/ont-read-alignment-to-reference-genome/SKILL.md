---
name: ont-read-alignment-to-reference-genome
description: Use when POD5/FAST5 reads have been basecalled with move tables and an
  aligned BAM is needed for reference-anchored signal-to-base alignment (e.g. DeepMod2
  5mC calling) or per-site methylation frequencies.
when_to_use_negative:
- Reads are already provided as an aligned BAM (e.g. Guppy/Dorado run with a reference
  supplied directly) — skip re-alignment and go straight to methylation detection
- Only de novo assembly or alignment-free analyses are planned and no reference genome
  is available
- Downstream methylation calling will run in unaligned/read-level-only mode, where
  the reference anchor is not used
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: minimap2
  role: Aligns ONT basecalled reads to the reference genome (-ax map-ont, -y) while
    preserving move-table tags needed for signal anchoring
- name: samtools
  role: Converts basecalled BAM to FASTQ retaining mv/ts tags (-T mv,ts), concatenates
    pass/fail shards (samtools cat), and sorts/indexes the aligned BAM (samtools sort
    --write-index)
- name: Dorado
  role: Basecaller that emits the initial BAM with move tables (--emit-moves) prior
    to alignment
  repo: nanoporetech/dorado
- name: Guppy
  role: Alternative basecaller producing BAMs with move tables via --bam_out --moves_out
    prior to alignment
- name: DeepMod2
  role: 'Downstream consumer of the aligned BAM: uses the reference to anchor signals
    and add MM/ML methylation tags'
  repo: github.com/WGLab/DeepMod2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-45778-y
    title: A signal processing and deep learning framework for methylation detection
      using Oxford Nanopore sequencing
  source_content_hash: sha256:ae290d815163e9c93beef64c7bf3ef511cd7147834911f67086b6749f8721966
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
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# ONT read alignment to reference genome

## Summary

Align Dorado/Guppy-basecalled Oxford Nanopore reads to a reference genome (e.g. GRCh38) with minimap2 while preserving move-table tags, producing aligned BAMs that enable reference-anchored methylation detection with DeepMod2.

## When to use

Use when POD5/FAST5 reads have been basecalled with move tables and an aligned BAM is needed for reference-anchored signal-to-base alignment (e.g. DeepMod2 5mC calling) or per-site methylation frequencies.

## When NOT to use

- Reads are already provided as an aligned BAM (e.g. Guppy/Dorado run with a reference supplied directly) — skip re-alignment and go straight to methylation detection
- Only de novo assembly or alignment-free analyses are planned and no reference genome is available
- Downstream methylation calling will run in unaligned/read-level-only mode, where the reference anchor is not used

## Inputs

- Basecalled BAM with move tables (from Dorado --emit-moves or Guppy --bam_out --moves_out), optionally split pass/fail shards
- Reference genome FASTA in analysis set (e.g. GRCh38.fa) with .fai index

## Outputs

- Coordinate-sorted, indexed aligned BAM per sample/flowcell retaining mv/ts tags (e.g. aligned.bam)
- Alignment suitable as --bam input for reference-anchored DeepMod2 detect runs

## How to apply

Basecall FAST5/POD5 with Dorado using --emit-moves (e.g. model dna_r10.4.1_e8.2_400bps_hac@v4.3.0) or Guppy with --bam_out --moves_out so the BAM carries the move tables needed for signal-to-basecall alignment. Convert the basecalled BAM back to FASTQ with samtools fastq, explicitly passing the mv (and ts) tags with -T, and pipe into minimap2 with -ax map-ont and -y so move-table tags are preserved through the alignment; write the result to aligned.bam (combine Guppy pass/fail shard BAMs with samtools cat if needed). Aligning to a reference genome (e.g. GRCh38 analysis-set FASTA with .fai index) allows reference anchoring of signals and estimation of per-site methylation levels downstream, so supply the reference FASTA to DeepMod2 after sorting and indexing (samtools sort --write-index).

## Related tools

- **minimap2** (Aligns ONT basecalled reads to the reference genome (-ax map-ont, -y) while preserving move-table tags needed for signal anchoring)
- **samtools** (Converts basecalled BAM to FASTQ retaining mv/ts tags (-T mv,ts), concatenates pass/fail shards (samtools cat), and sorts/indexes the aligned BAM (samtools sort --write-index))
- **Dorado** (Basecaller that emits the initial BAM with move tables (--emit-moves) prior to alignment) — nanoporetech/dorado
- **Guppy** (Alternative basecaller producing BAMs with move tables via --bam_out --moves_out prior to alignment)
- **DeepMod2** (Downstream consumer of the aligned BAM: uses the reference to anchor signals and add MM/ML methylation tags) — github.com/WGLab/DeepMod2

## Examples

```
samtools fastq basecall.bam -T mv,ts | minimap2 -ax map-ont ref.fa - -y -t 8 | samtools view -o aligned.bam
```

## Evaluation signals

- aligned.bam is a valid, coordinate-sorted, indexable BAM (samtools quickcheck / samtools index succeeds)
- mv (move table) and ts tags are retained on reads in the aligned BAM (inspect with samtools view and -T mv,ts was used on samtools fastq; minimap2 run with -y)
- Read count in the aligned BAM matches the basecalled BAM (accounting for secondary/supplementary records from minimap2)
- Feeding the aligned BAM plus reference FASTA to DeepMod2 detect produces reference-anchored per-site BED outputs and MM/ML-tagged output BAM
- Per-read methylation F1 on benchmarks falls in the paper-reported ~95% range (95.7–97.1% for the R9.4.1 Ashkenazim trio)

## Limitations

- Move tables provide only approximate signal-to-basecall alignment and do not always accurately demarcate boundaries between signals of consecutive bases, which can lead to uninformative signal summary statistics at some CpGs
- Alignment is required only for reference anchoring and per-site frequencies; without it DeepMod2 cannot compute per-site methylation levels
- Feature matrices are excluded when the read sequence has <0.75 percent identity with the aligned reference in the 21-bp window centered at the CpG, so poorly aligned reads lose methylation calls

## Evidence

- [readme] Align basecalled reads to a reference genome while retaining the move tables: [evidence span withheld — anchor status: external_doc]
- [results] Alignment enables reference anchoring and per-site methylation: "The BAM file is recommended to be aligned to a reference genome to allow reference anchoring of signals and estimation of per-site methylation levels"
- [results] Move-table output is a prerequisite for signal alignment: "It requires POD5/FAST5 files to be basecalled using Guppy or Dorado with move table output to allow signal alignment with the basecalls"
- [results] Move tables are approximate and can yield uninformative statistics: "the move table does not always accurately demarcate boundaries between signals associated with consecutive bases, which can lead to uninformative signal summary statistics"
- [methods] Reference-anchored DeepMod2 invocation uses the aligned BAM: [evidence span withheld — anchor status: external_doc]
