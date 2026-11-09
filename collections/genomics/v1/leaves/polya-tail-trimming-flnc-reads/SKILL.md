---
name: polya-tail-trimming-flnc-reads
description: Use when FLNC reads have already passed LIMA primer removal and isoseq3
  refine chimera filtering, and residual polyA tails must be detected and trimmed
  before mapping reads on the genome with minimap2 or uLTRA.
when_to_use_negative:
- Input is raw subread BAM or unrefined CCS reads — primer removal (LIMA) and chimeric
  read removal (isoseq3 refine) must happen first
- Input is already BAM-format FLNC reads that have not been converted to FASTA by
  bamtools convert
- Goal is polyA tail detection in aligned transcriptome data rather than trimming
  of unaligned FLNC reads
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3512
tools:
- name: TAMA
  role: TAMA cleanup (gstama_polyacleanup) detects and trims polyA tails from the
    selected FLNC reads
  repo: https://github.com/GenomeRIK/tama
- name: BAMTools
  role: BamTools convert converts the FLNC reads from BAM format into FASTA format
    as the input to polyA trimming
  repo: https://github.com/pezmaster31/bamtools
- name: nf-core/isoseq
  role: Pipeline that orchestrates the polyA trim step within the subread preprocessing
    workflow
  repo: https://github.com/nf-core/isoseq
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad150
    title: 'nf-core/isoseq: simple gene and isoform annotation with PacBio Iso-Seq
      long-read sequencing'
  source_content_hash: sha256:ed60f2631b21e5202422b07f8b3266c5d354fd511c96ef21dfab5f6b2832e8e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GenomeRIK/tama
  tools:
  - BAMTools
  - CCS
  - IsoSeq (isoseq3 refine)
  - LIMA
  - TAMA
  - nf-core/isoseq
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# polya-tail-trimming-flnc-reads

## Summary

Detect and trim polyA tails from selected FLNC reads in the nf-core/isoseq subread preprocessing stage using the TAMA cleanup module (gstama_polyacleanup), yielding chunked polyA-tail-free FLNC FASTA files ready for splice-aware genome alignment.

## When to use

Use when FLNC reads have already passed LIMA primer removal and isoseq3 refine chimera filtering, and residual polyA tails must be detected and trimmed before mapping reads on the genome with minimap2 or uLTRA.

## When NOT to use

- Input is raw subread BAM or unrefined CCS reads — primer removal (LIMA) and chimeric read removal (isoseq3 refine) must happen first
- Input is already BAM-format FLNC reads that have not been converted to FASTA by bamtools convert
- Goal is polyA tail detection in aligned transcriptome data rather than trimming of unaligned FLNC reads

## Inputs

- FLNC reads in FASTA format (primer-trimmed, chimera-filtered, converted from BAM via BAMTools convert)

## Outputs

- Chunked FLNC FASTA files with polyA tails removed (polyA-free FLNC reads)

## How to apply

Position this trimming after BAMTools conversion so the input is FLNC reads already converted from BAM into FASTA format by bamtools convert. Run TAMA cleanup (gstama_polyacleanup) on the selected FLNC FASTA reads to detect and trim their polyA tails, writing chunked polyA-tail-free FLNC FASTA files downstream. The rationale is that residual polyA tails can corrupt splice-aware alignment, so they are removed before the minimap2/uLTRA mapping step. Validate that the final chunked FASTA artifacts exist, are in FASTA format, and contain FLNC reads with primer pairs removed, chimeras discarded, and polyA tails removed per the pipeline's TAMA cleanup step.

## Related tools

- **TAMA** (TAMA cleanup (gstama_polyacleanup) detects and trims polyA tails from the selected FLNC reads) — https://github.com/GenomeRIK/tama
- **BAMTools** (BamTools convert converts the FLNC reads from BAM format into FASTA format as the input to polyA trimming) — https://github.com/pezmaster31/bamtools
- **nf-core/isoseq** (Pipeline that orchestrates the polyA trim step within the subread preprocessing workflow) — https://github.com/nf-core/isoseq

## Evaluation signals

- Final chunked FASTA artifacts exist and are in FASTA format
- Output reads are FLNC reads with primer pairs removed and chimeras discarded (upstream steps confirmed)
- FLNC reads no longer contain terminal polyA tails after gstama_polyacleanup
- Output chunked FASTA files are usable downstream by the minimap2/uLTRA genome mapping step

## Limitations

- No software version numbers are recorded for TAMA or other modules in the changelog section, creating a reproducibility gap for pinning tool versions
- Trimming operates only on reads surviving upstream LIMA and isoseq3 refine filtering; malformed inputs further downstream (e.g., a malformed GTF) affect other steps, not this one

## Evidence

- [intro] TAMA cleanup remove polyA tails from the selected reads.: "[GSTAMA_POLYACLEANUP](https://github.com/GenomeRIK/tama) TAMA cleanup remove polyA tails from the selected reads."
- [intro] Detect and trim polyA tails reads: "- [TAMA POLYA CLEAN UP](#tama-polya-clean-up) - Detect and trim polyA tails reads"
- [readme] This software was designed for processing Iso-Seq data and other long read transcriptome data.: [evidence span withheld — anchor status: external_doc]
