---
name: long-read-sequencing-pipeline-execution
description: Use when PacBio Iso-Seq subread BAM inputs must be converted into full-length
  transcript isoform annotations (e.g., a merged BED12 genome annotation) via the
  reproducible nf-core/isoseq workflow rather than ad-hoc per-tool scripts.
when_to_use_negative:
- Input data is not PacBio Iso-Seq long-read sequencing (e.g., short-read RNA-seq)
  — nf-core/rnaseq or other pipelines are the appropriate choice.
- The goal is transcriptome assembly of full-length transcripts rather than genome
  annotation/isoform identification — Iso-Seq identifies full-length transcripts and
  isoforms, removing the need for complex analysis such as transcriptome assembly.
- Only a single downstream analysis step is needed (e.g., only polyA trimming) — invoking
  the full pipeline is unnecessary when a single tool (e.g., TAMA cleanup) suffices.
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: nf-core/isoseq
  role: The Nextflow pipeline orchestrating all stages from subread preprocessing
    through merged genome annotation
  repo: https://github.com/nf-core/isoseq
- name: Nextflow
  role: Workflow engine the pipeline is built on; launched with the 'test' profile
    to execute the pipeline
  repo: https://github.com/nf-core/isoseq
- name: CCS
  role: Generates Circular Consensus Sequences from subreads
  repo: https://github.com/PacificBiosciences/ccs
- name: LIMA
  role: Cleans generated CCS by selecting sequences containing valid pairs of primers
    and removing the primers
  repo: https://github.com/pacificbiosciences/barcoding
- name: IsoSeq (isoseq3 refine)
  role: Detects and removes chimeric reads to produce FLNC reads
  repo: https://github.com/PacificBiosciences/IsoSeq
- name: BAMTools
  role: BAMTOOLS CONVERT converts the FLNC BAM file into FASTA format
  repo: https://github.com/pezmaster31/bamtools
- name: TAMA
  role: GSTAMA_POLYACLEANUP removes polyA tails from selected FLNC reads; GSTAMA merge
    collapses and merges sorted alignments into the final annotation
  repo: https://github.com/GenomeRIK/tama
- name: minimap2
  role: Splice-aware aligner mapping FLNC reads onto the genome
  repo: https://github.com/lh3/minimap2
- name: uLTRA
  role: Alternative splice-aware aligner for FLNC reads; spurious alignments are then
    removed with BioPerl
  repo: https://github.com/ksahlin/ultra
- name: BioPerl
  role: Removes spurious uLTRA alignments with gaps (N) in CIGAR strings (uLTRA path
    only)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad150
    title: 'nf-core/isoseq: simple gene and isoform annotation with PacBio Iso-Seq
      long-read sequencing'
  source_content_hash: sha256:333a8e02c75a4c2be85464e470e32cc70abe434daac071933e8523c6100c912d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nf-core/isoseq
  tools:
  - BAMTools
  - BioPerl
  - CCS
  - IsoSeq (isoseq3 refine)
  - LIMA
  - Nextflow
  - TAMA
  - minimap2
  - nf-core/isoseq
  - uLTRA
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# long-read-sequencing-pipeline-execution

## Summary

Executes the nf-core/isoseq Nextflow pipeline end to end on PacBio Iso-Seq long-read data — from subread preprocessing (CCS, LIMA, isoseq3 refine) through FLNC mapping (minimap2/uLTRA) to alignment post-processing (TAMA collapse/merge) — producing per-sample merged genome annotation files.

## When to use

Use when PacBio Iso-Seq subread BAM inputs must be converted into full-length transcript isoform annotations (e.g., a merged BED12 genome annotation) via the reproducible nf-core/isoseq workflow rather than ad-hoc per-tool scripts.

## When NOT to use

- Input data is not PacBio Iso-Seq long-read sequencing (e.g., short-read RNA-seq) — nf-core/rnaseq or other pipelines are the appropriate choice.
- The goal is transcriptome assembly of full-length transcripts rather than genome annotation/isoform identification — Iso-Seq identifies full-length transcripts and isoforms, removing the need for complex analysis such as transcriptome assembly.
- Only a single downstream analysis step is needed (e.g., only polyA trimming) — invoking the full pipeline is unnecessary when a single tool (e.g., TAMA cleanup) suffices.

## Inputs

- PacBio subread BAM files (nf-core test dataset bundled with the 'test' profile)
- Reference genome FASTA
- Samplesheet listing samples or sequencing runs (all samples read and parallelized per sample)
- Primer sequences for LIMA primer-pair selection

## Outputs

- CCS BAM files (Circular Consensus Sequences per sample)
- FLNC reads (BAM, and FASTA after BAMTOOLS CONVERT and polyA trimming)
- Sorted FLNC-to-genome alignments (BAM) from minimap2 or uLTRA
- Per-sample merged genome annotation in BED12 format at 10_GSTAMA_MERGE/<sample>.bed (TAMA Merge output)

## How to apply

Launch nf-core/isoseq with Nextflow using the 'test' profile to validate the run on the bundled nf-core test dataset (subreads plus reference genome) before switching to real data. The pipeline processes data in stages: CCS generates Circular Consensus Sequences from subreads; LIMA selects sequences containing valid pairs of primers and removes them; ISOSEQ REFINE (isoseq3) discards chimeric reads to yield FLNC reads; BAMTOOLS CONVERT converts the FLNC BAM into FASTA; TAMA (GSTAMA_POLYACLEANUP) detects and trims polyA tails; then minimap2 or uLTRA maps the FLNC reads onto the genome (with BIOPERL removing spurious uLTRA alignments caused by gaps (N) in CIGAR strings, only on the uLTRA path). Finally, sorted alignments are collapsed and merged with TAMA (GSTAMA merge), producing the merged per-sample genome annotation at 10_GSTAMA_MERGE/<sample>.bed. Multiple samples or sequencing runs can be analysed at the same time from a samplesheet, with computation parallelized per sample.

## Related tools

- **nf-core/isoseq** (The Nextflow pipeline orchestrating all stages from subread preprocessing through merged genome annotation) — https://github.com/nf-core/isoseq
- **Nextflow** (Workflow engine the pipeline is built on; launched with the 'test' profile to execute the pipeline) — https://github.com/nf-core/isoseq
- **CCS** (Generates Circular Consensus Sequences from subreads) — https://github.com/PacificBiosciences/ccs
- **LIMA** (Cleans generated CCS by selecting sequences containing valid pairs of primers and removing the primers) — https://github.com/pacificbiosciences/barcoding
- **IsoSeq (isoseq3 refine)** (Detects and removes chimeric reads to produce FLNC reads) — https://github.com/PacificBiosciences/IsoSeq
- **BAMTools** (BAMTOOLS CONVERT converts the FLNC BAM file into FASTA format) — https://github.com/pezmaster31/bamtools
- **TAMA** (GSTAMA_POLYACLEANUP removes polyA tails from selected FLNC reads; GSTAMA merge collapses and merges sorted alignments into the final annotation) — https://github.com/GenomeRIK/tama
- **minimap2** (Splice-aware aligner mapping FLNC reads onto the genome) — https://github.com/lh3/minimap2
- **uLTRA** (Alternative splice-aware aligner for FLNC reads; spurious alignments are then removed with BioPerl) — https://github.com/ksahlin/ultra
- **BioPerl** (Removes spurious uLTRA alignments with gaps (N) in CIGAR strings (uLTRA path only))

## Examples

```
nextflow run nf-core/isoseq -profile test
```

## Evaluation signals

- Each 10_GSTAMA_MERGE/<sample>.bed file exists and is non-empty for every sample in the samplesheet.
- Each merged annotation file conforms to BED12 format: exactly 12 tab-delimited columns including block and thick-start/thick-end fields, as documented in the pipeline's Output documentation.
- Intermediate artifacts appear in order: CCS BAMs after the CCS step, primer-cleaned CCS after LIMA, FLNC reads after ISOSEQ REFINE, and trimmed FASTA after GSTAMA_POLYACLEANUP.
- Alignment BAMs are sorted (SAMTOOLS SORT converts SAM to sorted BAM), and the uLTRA path shows spurious alignments removed by the BioPerl cleanup step.
- The pipeline completes without stopping — note it will stop in case of a malformed GTF, so a clean exit indicates input annotation files were well-formed.

## Limitations

- uLTRA can produce spurious alignments when a malformed GTF is used; the BioPerl module addresses this, and removing the bioperl module causes the pipeline to stop in case of a malformed GTF.
- No software version numbers are recorded for tools/modules in the changelog section, creating a reproducibility gap for pinning tool versions.
- Some modules were removed in later releases (samtools sort, bioperl, uLTRA pipeline process), so behavior varies by pipeline version.
- The GUNZIP, ULTRA/MINIMAP2 BIOPERL cleanup steps are uLTRA-path-only, so the two aligner choices lead to different intermediate steps.

## Evidence

- [abstract] nf-core/isoseq is a pipeline for automatic read processing and genome annotation from Iso-Seq data: "Here, we present nf-core/isoseq, a pipeline for automatic read processing and genome annotation."
- [intro] CCS step: "[CCS](https://github.com/PacificBiosciences/ccs) generate a Circular Consensus Sequence from subreads."
- [intro] LIMA primer removal: "It selects sequences containing valid pairs of primers and removed it."
- [intro] Alignment step: "[`MINIMAP2`](https://github.com/lh3/minimap2) or [`uLTRA`](https://github.com/ksahlin/ultra) aligns reads ont the genome."
- [discussion] uLTRA malformed GTF limitation: "spurious alignments produced by uLTRA if a malformed GTF is used"
- [intro] TAMA polyA cleanup: "TAMA cleanup remove polyA tails from the selected reads."
- [intro] BioPerl cleanup rationale: "Some CIGAR string sometimes with a gap (N)."
