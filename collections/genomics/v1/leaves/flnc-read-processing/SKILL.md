---
name: flnc-read-processing
description: Use when the input consists of PacBio single-molecule subread files and
  the goal is to obtain clean full-length non-chimeric (FLNC) transcript reads, the
  upstream prerequisite for splice-aware genome mapping and isoform annotation.
when_to_use_negative:
- Input is short paired-end Illumina RNA-seq FASTQ data — use a short-read pipeline
  (e.g. nf-core/rnaseq) instead of subread-to-FLNC processing.
- Reads are already FLNC (post-LIMA/refine BAM) — skip CCS/LIMA/refine and proceed
  directly to FASTA conversion and polyA trimming.
- Input is already genome-aligned (sorted alignment BAM) — this skill operates before
  the ULTRA/MINIMAP2 mapping stage, not after it.
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: CCS
  role: Generates Circular Consensus Sequences (HiFi reads) from PacBio subreads,
    the first step toward FLNC reads
  repo: https://github.com/PacificBiosciences/ccs
- name: LIMA
  role: Cleans generated CCS by selecting sequences with valid pairs of primers and
    removing the primer sequences
  repo: https://github.com/pacificbiosciences/barcoding/
- name: IsoSeq (isoseq3 refine)
  role: Detects and discards chimeric reads to produce FLNC reads
  repo: https://github.com/PacificBiosciences/IsoSeq
- name: BAMTools
  role: BAMTOOLS CONVERT converts FLNC reads in BAM format into FASTA format for downstream
    trimming/alignment
  repo: https://github.com/pezmaster31/bamtools
- name: TAMA
  role: GSTAMA_POLYACLEANUP detects and trims polyA tails from the selected FLNC reads
  repo: https://github.com/GenomeRIK/tama
- name: nf-core/isoseq
  role: Nextflow pipeline orchestrating the CCS→LIMA→refine→convert→polyA-cleanup
    chain per sample/run
  repo: https://github.com/nf-core/isoseq
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad150
    title: 'nf-core/isoseq: simple gene and isoform annotation with PacBio Iso-Seq
      long-read sequencing'
  source_content_hash: sha256:bbcb2070d053c22ea14e946db030cc8426921ee88325a29a8f66f7ddd6a3ac11
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PacificBiosciences/ccs
  tools:
  - BAMTools
  - BioPerl
  - CCS
  - IsoSeq (isoseq3 refine)
  - LIMA
  - TAMA
  - nf-core/isoseq
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# flnc-read-processing

## Summary

Turns raw PacBio Iso-Seq subreads into full-length non-chimeric (FLNC) reads by generating Circular Consensus Sequences with CCS, removing primers with LIMA, discarding chimeras with isoseq3 refine, converting the FLNC BAM to FASTA with BAMTools, and trimming polyA tails with TAMA.

## When to use

Use when the input consists of PacBio single-molecule subread files and the goal is to obtain clean full-length non-chimeric (FLNC) transcript reads, the upstream prerequisite for splice-aware genome mapping and isoform annotation.

## When NOT to use

- Input is short paired-end Illumina RNA-seq FASTQ data — use a short-read pipeline (e.g. nf-core/rnaseq) instead of subread-to-FLNC processing.
- Reads are already FLNC (post-LIMA/refine BAM) — skip CCS/LIMA/refine and proceed directly to FASTA conversion and polyA trimming.
- Input is already genome-aligned (sorted alignment BAM) — this skill operates before the ULTRA/MINIMAP2 mapping stage, not after it.

## Inputs

- PacBio Iso-Seq subread files (BAM, e.g. bundled in the nf-core test dataset samplesheet)
- Primer sequences for valid primer-pair selection (used by LIMA)

## Outputs

- Circular Consensus Sequences (CCS) from subreads
- Primer-removed, valid-primer-pair CCS reads (LIMA output)
- Full-length non-chimeric (FLNC) reads in BAM format (isoseq3 refine output)
- FLNC reads in FASTA format (BAMTools CONVERT output)
- PolyA-trimmed FLNC FASTA reads (TAMA GSTAMA_POLYACLEANUP output)

## How to apply

First run CCS to generate a Circular Consensus Sequence (HiFi read) from each subread bundle. Next, LIMA cleans the generated CCS by selecting only sequences containing valid pairs of primers and removing the primer sequences themselves. IsoSeq refine (isoseq3) then detects and removes chimeric reads, yielding FLNC reads. Because downstream splice-aware aligners (minimap2 or uLTRA) take FASTA in this pipeline, BAMTOOLS CONVERT converts the FLNC BAM into FASTA, and TAMA cleanup (GSTAMA_POLYACLEANUP) detects and trims polyA tails from the selected reads. In the nf-core/isoseq implementation these run as chained Nextflow processes (CCS, LIMA, ISOSEQ REFINE, BAMTOOLS CONVERT, TAMA POLYA CLEAN UP) so each stage's output feeds the next stage.

## Related tools

- **CCS** (Generates Circular Consensus Sequences (HiFi reads) from PacBio subreads, the first step toward FLNC reads) — https://github.com/PacificBiosciences/ccs
- **LIMA** (Cleans generated CCS by selecting sequences with valid pairs of primers and removing the primer sequences) — https://github.com/pacificbiosciences/barcoding/
- **IsoSeq (isoseq3 refine)** (Detects and discards chimeric reads to produce FLNC reads) — https://github.com/PacificBiosciences/IsoSeq
- **BAMTools** (BAMTOOLS CONVERT converts FLNC reads in BAM format into FASTA format for downstream trimming/alignment) — https://github.com/pezmaster31/bamtools
- **TAMA** (GSTAMA_POLYACLEANUP detects and trims polyA tails from the selected FLNC reads) — https://github.com/GenomeRIK/tama
- **nf-core/isoseq** (Nextflow pipeline orchestrating the CCS→LIMA→refine→convert→polyA-cleanup chain per sample/run) — https://github.com/nf-core/isoseq

## Examples

```
nextflow run nf-core/isoseq -profile test
```

## Evaluation signals

- CCS step produces a circular consensus output (HiFi BAM) without failures from the subread input
- LIMA output contains only reads with valid primer pairs, with primer sequences removed from the reads
- isoseq3 refine output is a FLNC BAM containing no chimeric reads
- FLNC BAM converts to a non-empty FASTA file with matching read content (BAMTools CONVERT)
- TAMA cleanup output shows polyA tails detected and trimmed from the selected reads; downstream uLTRA path additionally requires GUNZIP-decompressed FLNC FASTAs

## Limitations

- No software version numbers are recorded for the tools/modules in this changelog section (reproducibility gap for pinning CCS/LIMA/IsoSeq versions).
- LIMA retains only sequences containing valid pairs of primers — reads lacking a valid primer pair are dropped.
- The downstream uLTRA alignment path is affected by upstream data quality: spurious alignments are produced when a malformed GTF is used, requiring a BioPerl cleanup step.

## Evidence

- [intro] CCS generates circular consensus from subreads: "[CCS](https://github.com/PacificBiosciences/ccs) generate a Circular Consensus Sequence from subreads."
- [intro] LIMA selects valid primer pairs and removes primers: "[LIMA](https://github.com/pacificbiosciences/barcoding/) clean generated CCS. It selects sequences containing valid pairs of primers and removed it."
- [intro] IsoSeq refine discards chimeric reads: "[ISOSEQ REFINE](https://github.com/PacificBiosciences/IsoSeq) discard chimeric reads."
- [intro] BAMTools converts BAM to FASTA: "[BAMTOOLS CONVERT](https://github.com/pezmaster31/bamtools) convert reads in BAM format into fasta format."
- [intro] TAMA trims polyA tails: "TAMA cleanup remove polyA tails from the selected reads."
- [readme] IsoSeq identifies transcripts from PacBio single-molecule data: [evidence span withheld — anchor status: not_found]
