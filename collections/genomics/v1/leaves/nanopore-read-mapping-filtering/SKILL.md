---
name: nanopore-read-mapping-filtering
description: Use when raw nanopore DRS fast5 signals need transcript-level assignment
  — e.g. before curating training/testing sets for RISER, or before computing per-transcript
  read/nucleotide counts for enrichment or depletion benchmarks.
when_to_use_negative:
- Data are cDNA nanopore reads or short-read RNA-seq rather than direct RNA sequencing
  signals with adapter and poly(A) structure
- The goal is real-time classification during a live run — RISER classifies raw signals
  directly without basecalling or a reference, so this offline mapping chain is unnecessary
- A reference transcriptome is unavailable and de novo or signal-level analysis is
  intended (RISER works without basecalling or a reference)
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: BoostNano
  role: Segments and trims the sequencing adapter and variable-length poly(A) tail
    from raw nanopore signals prior to basecalling, replacing the raw signal in the
    fast5 so a basecaller can run directly on processed files
  repo: https://github.com/haotianteng/BoostNano
- name: Guppy (v4.0.14)
  role: Basecalls the trimmed fast5 files using --flowcell FLO-MIN106 --kit SQK-RNA002
    to produce read sequences from raw signals
- name: minimap2 (v2.17)
  role: Maps basecalled reads to the GENCODE reference transcriptome with '-ax map-ont
    --secondary=no', generating transcript-level alignments
- name: samtools (v1.10)
  role: Filters mapped reads with option -F 2324 to remove reverse-strand, secondary,
    supplementary, and unmapped alignments, retaining only high-confidence mappings
- name: RISER
  role: 'Downstream consumer of the filtered alignments: mapped reads are used to
    curate training/testing sets for the RISER model and to compute per-transcript
    percent-change statistics'
  repo: https://github.com/comprna/riser
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-48673-8
    title: Biochemical-free enrichment or depletion of RNA classes in real-time during
      direct RNA sequencing with RISER
  source_content_hash: sha256:7a0426af43d8d9aab89f268233cd5a37839e3da9c548051d0dadf31879bbface
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haotianteng/BoostNano
  tools:
  - BoostNano
  - Guppy (v4.0.14)
  - RISER
  - minimap2 (v2.17)
  - samtools (v1.10)
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# nanopore-read-mapping-filtering

## Summary

A standard nanopore direct RNA sequencing (DRS) analysis chain: trim adapter/poly(A) from raw signals with BoostNano, basecall with Guppy, map to the GENCODE reference transcriptome with minimap2, and retain only high-confidence alignments with samtools. This produces reliable per-read transcript assignments needed for downstream quantification and RISER label curation.

## When to use

Use when raw nanopore DRS fast5 signals need transcript-level assignment — e.g. before curating training/testing sets for RISER, or before computing per-transcript read/nucleotide counts for enrichment or depletion benchmarks.

## When NOT to use

- Data are cDNA nanopore reads or short-read RNA-seq rather than direct RNA sequencing signals with adapter and poly(A) structure
- The goal is real-time classification during a live run — RISER classifies raw signals directly without basecalling or a reference, so this offline mapping chain is unnecessary
- A reference transcriptome is unavailable and de novo or signal-level analysis is intended (RISER works without basecalling or a reference)

## Inputs

- Raw nanopore DRS fast5 signal files (FLO-MIN106 flowcell, SQK-RNA002 kit)
- GENCODE reference transcriptome FASTA/index (release 34 or 43, assembly GRCh38.p13)

## Outputs

- Adapter/poly(A)-trimmed fast5 signals (BoostNano replaces the Signal slot, preserving old signal in Signal_Old)
- Basecalled read sequences (Guppy output)
- High-confidence transcriptome BAM alignments (reverse-strand, secondary, supplementary, and unmapped reads removed via -F 2324)
- Per-read transcript assignments suitable for read/nucleotide counting per transcript

## How to apply

First preprocess raw fast5 signals with BoostNano to segment and trim the sequencing adapter and variable-length poly(A) tail from the start of each signal. Basecall the trimmed fast5 files with Guppy (v4.0.14) using the options --flowcell FLO-MIN106 --kit SQK-RNA002 for RNA002 data. Map the resulting reads to the GENCODE reference transcriptome (GRCh38.p13; the RISER model used release 34, while the whole-blood analysis used release 43) with minimap2 (v2.17) using '-ax map-ont --secondary=no' to suppress secondary alignments. Finally, remove alignments to the reverse strand, secondary or supplementary mappings, and unmapped reads using samtools (v1.10) with option -F 2324 so that only high-confidence forward-strand transcript alignments are retained for downstream read counting or model curation.

## Related tools

- **BoostNano** (Segments and trims the sequencing adapter and variable-length poly(A) tail from raw nanopore signals prior to basecalling, replacing the raw signal in the fast5 so a basecaller can run directly on processed files) — https://github.com/haotianteng/BoostNano
- **Guppy (v4.0.14)** (Basecalls the trimmed fast5 files using --flowcell FLO-MIN106 --kit SQK-RNA002 to produce read sequences from raw signals)
- **minimap2 (v2.17)** (Maps basecalled reads to the GENCODE reference transcriptome with '-ax map-ont --secondary=no', generating transcript-level alignments)
- **samtools (v1.10)** (Filters mapped reads with option -F 2324 to remove reverse-strand, secondary, supplementary, and unmapped alignments, retaining only high-confidence mappings)
- **RISER** (Downstream consumer of the filtered alignments: mapped reads are used to curate training/testing sets for the RISER model and to compute per-transcript percent-change statistics) — https://github.com/comprna/riser

## Examples

```
python boostnano_eval.py -i INPUT_FAST5_FOLDER -o OUTPUT_FOLDER -m MODEL_FOLDER --replace; guppy_basecaller --flowcell FLO-MIN106 --kit SQK-RNA002 -i OUTPUT_FOLDER -s bc_out; minimap2 -ax map-ont --secondary=no gencode.fa reads.fq > aln.sam; samtools view -b -F 2324 aln.sam > filtered.bam
```

## Evaluation signals

- minimap2 invoked with '--secondary=no' so no secondary alignments appear in the BAM
- samtools filter flag equals 2324, i.e. BAM retains only forward-strand primary mapped reads
- BoostNano output fast5 files contain trimmed signals in the Signal slot with original signal preserved in Signal_Old
- Alignment rate to the GENCODE transcriptome is consistent with high-confidence DRS reads (most reads map to transcriptome references on the forward strand)
- Downstream per-transcript counts computed from the filtered BAM reproduce expected biological signals (e.g. globin transcripts dominating control whole-blood runs)

## Limitations

- minimap2 with --secondary=no suppresses secondary alignments, so multimapping reads yield no assignment and may be lost from counts
- The -F 2324 filter removes reverse-strand alignments entirely; transcripts detected only on the reverse strand are excluded by design
- RISER's live-run signal trimming uses an efficient adapter/poly(A) trimming strategy rather than BoostNano, so offline and live preprocessing differ; BoostNano additionally requires Python < 3.9
- Filter steps vary by dataset context: model development used GENCODE release 34 while the whole-blood globin analysis used release 43, so results are reference-version dependent

## Evidence

- [methods] Fast5 files were basecalled with Guppy (v4.0.14) using options –flowcell FLO-MIN106 –kit SQK-RNA002: "Fast5 files were basecalled with Guppy (v4.0.14) using options –flowcell FLO-MIN106 –kit SQK-RNA002"
- [methods] mapped to the GENCODE reference transcriptome (release 34, assembly GRCh38.p13) with minimap233 (v2.17) using options -ax map-ont --secondary=no: "mapped to the GENCODE reference transcriptome (release 34, assembly GRCh38.p13) with minimap233 (v2.17) using options -ax map-ont --secondary=no"
- [methods] unmapped reads were removed using samtools34 (v1.10) with option -F 2324: "unmapped reads were removed using samtools34 (v1.10) with option -F 2324"
- [methods] To retain only high-confidence mappings, alignments to the reverse strand, secondary or supplementary mappings and unmapped reads were removed using samtools: "To retain only high-confidence mappings, alignments to the reverse strand, secondary or supplementary mappings and unmapped reads were removed using samtools"
- [readme] This will save the segmented siganl in to the original Signal Slot in the fast5 files, and copy the old signal into the Signal_Old slot, so a basecaller can directly run on the processed fast5 files.: [evidence span withheld — anchor status: external_doc]
