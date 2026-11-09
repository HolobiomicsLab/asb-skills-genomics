---
name: bed12-annotation-format-validation
description: Use when the nf-core/isoseq pipeline has completed TAMA collapse and
  TAMA merge and you must confirm each per-sample merged annotation file is a valid
  12-column BED12 before treating it as the pipeline's final result.
when_to_use_negative:
- Input is still raw SAM/BAM alignment output from uLTRA or minimap2 (pre-collapse)
  rather than a collapsed/merged BED annotation
- The target annotation is a GTF/GFF transcript file rather than the pipeline's BED12
  output
- You need biological correctness of the annotation (isoform accuracy), not structural
  format compliance — this skill only checks BED12 schema
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: nf-core/isoseq
  role: Pipeline whose merged annotation output (10_GSTAMA_MERGE/<sample>.bed) is
    validated
  repo: https://github.com/nf-core/isoseq
- name: TAMA
  role: Produces the chunk-level collapsed BED12 (GSTAMA collapse) and the merged
    per-sample BED12 (GSTAMA merge) that this skill validates
  repo: https://github.com/GenomeRIK/tama
- name: samtools
  role: Sorts SAM alignments into BAM before collapse; its sorted output feeds the
    BED12-producing steps
- name: uLTRA
  role: Alternative splice-aware aligner whose alignment output is converted to the
    collapsed BED12 annotations
  repo: https://github.com/ksahlin/ultra
- name: minimap2
  role: Alternative splice-aware aligner whose alignment output is converted to the
    collapsed BED12 annotations
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad150
    title: 'nf-core/isoseq: simple gene and isoform annotation with PacBio Iso-Seq
      long-read sequencing'
  source_content_hash: sha256:3d2520fd3340065cffb7f0df0144d4ee608dd721f2433ded0d9c23c15266a12b
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
  - TAMA
  - minimap2
  - nf-core/isoseq
  - samtools
  - uLTRA
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# bed12-annotation-format-validation

## Summary

Validates that the final per-sample merged genome annotation produced by the nf-core/isoseq pipeline (TAMA merge output in 10_GSTAMA_MERGE/<sample>.bed) is a well-formed, non-empty BED12 file with intact block/exon structure. This is the acceptance check that confirms the end-to-end Iso-Seq workflow (CCS through collapse/merge) delivered its main output.

## When to use

Use when the nf-core/isoseq pipeline has completed TAMA collapse and TAMA merge and you must confirm each per-sample merged annotation file is a valid 12-column BED12 before treating it as the pipeline's final result.

## When NOT to use

- Input is still raw SAM/BAM alignment output from uLTRA or minimap2 (pre-collapse) rather than a collapsed/merged BED annotation
- The target annotation is a GTF/GFF transcript file rather than the pipeline's BED12 output
- You need biological correctness of the annotation (isoform accuracy), not structural format compliance — this skill only checks BED12 schema

## Inputs

- Merged per-sample BED12 annotation files produced by TAMA merge at 10_GSTAMA_MERGE/<sample>.bed
- Pipeline samplesheet listing the samples/runs expected to have outputs

## Outputs

- Pass/fail verdict per sample confirming 10_GSTAMA_MERGE/<sample>.bed exists, is non-empty, and has exactly 12 tab-delimited columns with intact block/thick-start/thick-end fields
- Optionally, a list of malformed rows or missing/malformed sample files

## How to apply

After the pipeline's post-processing stage (samtools sort of the uLTRA/minimap2 SAM alignments, TAMA collapse into chunk-level <sample>_collapsed.bed models, and TAMA merge into one annotation per sample), locate the merged files at 10_GSTAMA_MERGE/<sample>.bed. For every sample in the samplesheet, verify the file exists, is non-empty, and parses as BED12: exactly 12 tab-delimited columns including chrom, chromStart, chromEnd, name, score, strand, thickStart, thickEnd, itemRgb, blockCount, blockSizes and blockStarts, with block fields internally consistent (blockSizes/blockStarts sums matching the exon span). The rationale is that TAMA merge is the terminal step converting collapsed transcript models into a common non-redundant annotation, so structural corruption or truncation there silently invalidates the whole run's deliverable. Fail the validation if any column count deviates, if block structure is malformed, or if any expected sample's file is missing or zero-length.

## Related tools

- **nf-core/isoseq** (Pipeline whose merged annotation output (10_GSTAMA_MERGE/<sample>.bed) is validated) — https://github.com/nf-core/isoseq
- **TAMA** (Produces the chunk-level collapsed BED12 (GSTAMA collapse) and the merged per-sample BED12 (GSTAMA merge) that this skill validates) — https://github.com/GenomeRIK/tama
- **samtools** (Sorts SAM alignments into BAM before collapse; its sorted output feeds the BED12-producing steps)
- **uLTRA** (Alternative splice-aware aligner whose alignment output is converted to the collapsed BED12 annotations) — https://github.com/ksahlin/ultra
- **minimap2** (Alternative splice-aware aligner whose alignment output is converted to the collapsed BED12 annotations) — https://github.com/lh3/minimap2

## Examples

```
awk -F'\t' 'NF!=12 {print FILENAME": bad BED12 at line "NR; exit 1}' 10_GSTAMA_MERGE/<sample>.bed && echo "<sample>.bed: valid BED12"
```

## Evaluation signals

- Each expected 10_GSTAMA_MERGE/<sample>.bed file exists and is non-empty (file size > 0 bytes)
- Every data line has exactly 12 tab-delimited columns (BED12 schema conformance)
- blockSizes and blockStarts lists have blockCount entries and their spans are consistent with the chromStart/chromEnd interval
- thickStart/thickEnd fields lie within the [chromStart, chromEnd] range as required by BED12
- One merged BED file is produced per sample in the samplesheet (completeness across parallelized per-sample processing)

## Limitations

- Structural validation cannot detect biologically incorrect isoform models — a schema-valid BED12 may still contain wrong transcript structures
- The pipeline's own documentation notes spurious uLTRA alignments arise when a malformed GTF is used; malformed reference annotations can propagate bad transcript models into a format-valid BED12
- Changelog entries note that the bioperl module and samtools sort module have been removed in later versions, so intermediate file expectations (sorted SAM inputs) may differ across pipeline versions

## Evidence

- [intro] SAMTOOLS SORT sorts alignment and converts SAM to BAM: "- [SAMTOOLS SORT](#samtools-sort) - Sort alignment and convert sam file into bam file"
- [readme] TAMA README scope for Iso-Seq long-read transcriptome data: [evidence span withheld — anchor status: external_doc]
- [abstract] Pipeline is for automatic read processing and genome annotation: "a pipeline for automatic read processing and genome annotation"
