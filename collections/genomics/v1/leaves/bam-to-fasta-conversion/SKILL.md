---
name: bam-to-fasta-conversion
description: Use when a FLNC read set produced by PacBio isoseq3 refine exists only
  as a BAM file and downstream steps (e.g. TAMA gstama_polyacleanup) require FASTA
  input.
when_to_use_negative:
- The reads are already in FASTA/FASTQ format and the next step (e.g. TAMA cleanup
  or an aligner) can consume them directly.
- You need to preserve BAM-specific per-read tags or quality information — conversion
  to FASTA discards everything except the sequence records.
- You are working with short-read BAM alignments that need sorting or SAM conversion
  rather than sequence export to FASTA.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: BAMTools
  role: Provides the bamtools convert command used to convert reads in BAM format
    into FASTA format for downstream TAMA cleanup
  repo: https://github.com/pezmaster31/bamtools
- name: nf-core/isoseq
  role: Pipeline that places the BAM-to-FASTA conversion step between isoseq3 refine
    and TAMA polyA cleanup
  repo: https://github.com/nf-core/isoseq
- name: IsoSeq (isoseq3 refine)
  role: Produces the FLNC BAM input that this conversion step consumes
  repo: https://github.com/PacificBiosciences/IsoSeq
- name: TAMA
  role: Downstream consumer (gstama_polyacleanup) requiring the FASTA output to detect
    and trim polyA tails
  repo: https://github.com/GenomeRIK/tama
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad150
    title: 'nf-core/isoseq: simple gene and isoform annotation with PacBio Iso-Seq
      long-read sequencing'
  source_content_hash: sha256:fe851bc0fbfdeabe2cbee91965303dfd25f4bdef27efa2e6d0268833654d3d7c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pezmaster31/bamtools
  tools:
  - BAMTools
  - CCS
  - IsoSeq (isoseq3 refine)
  - LIMA
  - TAMA
  - nf-core/isoseq
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# bam-to-fasta-conversion

## Summary

Converts PacBio FLNC read files from BAM format into FASTA format using BAMTools convert, the intermediate step in nf-core/isoseq between isoseq3 refine (chimera removal) and TAMA cleanup (polyA trimming), because downstream TAMA cleanup consumes FASTA input.

## When to use

Use when a FLNC read set produced by PacBio isoseq3 refine exists only as a BAM file and downstream steps (e.g. TAMA gstama_polyacleanup) require FASTA input.

## When NOT to use

- The reads are already in FASTA/FASTQ format and the next step (e.g. TAMA cleanup or an aligner) can consume them directly.
- You need to preserve BAM-specific per-read tags or quality information — conversion to FASTA discards everything except the sequence records.
- You are working with short-read BAM alignments that need sorting or SAM conversion rather than sequence export to FASTA.

## Inputs

- FLNC read file in BAM format (output of PacBio isoseq3 refine in nf-core/isoseq)

## Outputs

- FLNC read file in FASTA format (chunked FASTA artifacts consumed by TAMA polyA cleanup)

## How to apply

In the nf-core/isoseq workflow, run bamtools convert immediately after isoseq3 refine, using the primer-trimmed, chimera-discarded FLNC BAM as input. The conversion extracts the read sequences into FASTA records so that the subsequent TAMA cleanup (gstama_polyacleanup) step can detect and trim polyA tails and write chunked polyA-tail-free FLNC FASTA files. No filtering is performed at this step — it is a pure format transformation; all filtering (valid primer pairs via LIMA, chimera removal via isoseq3 refine, polyA trimming via TAMA) happens in the surrounding steps. Because the BAM contains aligned/unaligned read data with tags, verify afterwards that the output is plain FASTA rather than an intermediate format.

## Related tools

- **BAMTools** (Provides the bamtools convert command used to convert reads in BAM format into FASTA format for downstream TAMA cleanup) — https://github.com/pezmaster31/bamtools
- **nf-core/isoseq** (Pipeline that places the BAM-to-FASTA conversion step between isoseq3 refine and TAMA polyA cleanup) — https://github.com/nf-core/isoseq
- **IsoSeq (isoseq3 refine)** (Produces the FLNC BAM input that this conversion step consumes) — https://github.com/PacificBiosciences/IsoSeq
- **TAMA** (Downstream consumer (gstama_polyacleanup) requiring the FASTA output to detect and trim polyA tails) — https://github.com/GenomeRIK/tama

## Evaluation signals

- The final chunked FASTA artifacts exist in FASTA format (not BAM), as required by the pipeline's TAMA cleanup step.
- The number of FASTA records matches the number of reads in the input FLNC BAM — conversion must be lossless with respect to reads (no filtering).
- The FASTA output contains FLNC reads whose primer pairs are valid and chimeras were already discarded, since conversion occurs only after isoseq3 refine.
- Downstream gstama_polyacleanup runs successfully on the FASTA and produces polyA-tail-free FLNC FASTA files.

## Limitations

- No software version numbers are recorded for the tools/modules, including BAMTools, creating a reproducibility gap for pinning tool versions.
- Conversion to FASTA discards BAM-native per-read metadata (quality values, tags), so this step cannot be used if read-level annotations must be retained.
- The bamtools README in the provided context is truncated, so command-level parameter details are not documented here.

## Evidence

- [intro] BamTools convert converts reads in BAM format into fasta format: "[BAMTOOLS CONVERT](https://github.com/pezmaster31/bamtools) convert reads in BAM format into fasta format."
- [intro] Pipeline lists BAMTOOLS CONVERT as the step converting bam file into fasta file: "- [BAMTOOLS CONVERT](#bamtools-convert) - Convert bam file into fasta file"
- [readme] BamTools is an end-user toolkit for handling BAM files: "BamTools provides both a programmer's API and an end-user's toolkit for handling
BAM files."
- [intro] Workflow places conversion before TAMA polyA cleanup on FLNC reads: "- [TAMA POLYA CLEAN UP](#tama-polya-clean-up) - Detect and trim polyA tails reads"
