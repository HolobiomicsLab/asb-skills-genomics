---
name: sam-alignment-output-validation
description: Use when reconstructing or testing the nf-core/isoseq mapping step and
  you must confirm that each aligner dispatch branch (uLTRA with a GTF, minimap2 without)
  emitted the expected non-empty <sample>.chunk<X>.sam files.
when_to_use_negative:
- The pipeline has already moved past alignment (e.g. after transcript collapsing/merging),
  where SAM artifacts from the mapping step are no longer the current deliverable.
- The dispatch condition being tested does not involve uLTRA vs minimap2 (e.g. validating
  CCS or LIMA primer-removal outputs).
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: uLTRA
  role: Aligns FLNC reads to the genome guided by a GTF annotation when one is available;
    its SAM outputs (06.2_ULTRA/) are validated by this skill
  repo: https://github.com/ksahlin/ultra
- name: minimap2
  role: Splice-aware fallback aligner used when no GTF annotation is available; its
    SAM outputs (06_MINIMAP2/) are validated by this skill
  repo: https://github.com/lh3/minimap2
- name: nf-core/isoseq
  role: Pipeline whose conditional mapping step (ULTRA or MINIMAP2) produces the SAM
    files under validation
  repo: https://github.com/nf-core/isoseq
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad150
    title: 'nf-core/isoseq: simple gene and isoform annotation with PacBio Iso-Seq
      long-read sequencing'
  source_content_hash: sha256:02ddea788f346cf747366a37753770d2814f0b968d3f76674f17d59f2948939d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ksahlin/ultra
  tools:
  - minimap2
  - nf-core/isoseq
  - uLTRA
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# SAM alignment output validation

## Summary

Validates that the FLNC-to-genome mapping step of nf-core/isoseq produced the expected per-sample, per-chunk SAM alignment files under both dispatch branches (06.2_ULTRA/ for uLTRA, 06_MINIMAP2/ for minimap2), confirming the conditional aligner dispatch ran correctly.

## When to use

Use when reconstructing or testing the nf-core/isoseq mapping step and you must confirm that each aligner dispatch branch (uLTRA with a GTF, minimap2 without) emitted the expected non-empty <sample>.chunk<X>.sam files.

## When NOT to use

- The pipeline has already moved past alignment (e.g. after transcript collapsing/merging), where SAM artifacts from the mapping step are no longer the current deliverable.
- The dispatch condition being tested does not involve uLTRA vs minimap2 (e.g. validating CCS or LIMA primer-removal outputs).

## Inputs

- <sample>.chunk<X>.sam files under 06.2_ULTRA/ (uLTRA branch)
- <sample>.chunk<X>.sam files under 06_MINIMAP2/ (minimap2 branch)
- Expected sample/chunk list derived from the samplesheet

## Outputs

- Pass/fail validation report per branch (uLTRA and minimap2)
- List of missing or empty SAM files, if any

## How to apply

After running the mapping step on FLNC reads (output of TAMA polyA clean-up, gunzip-decompressed for the uLTRA path) against a test reference genome, check both branch output directories. When a reference GTF annotation is supplied, uLTRA aligns in genome+annotation mode and writes 06.2_ULTRA/<sample>.chunk<X>.sam; when no annotation exists, minimap2 runs in splice-aware mode and writes 06_MINIMAP2/<sample>.chunk<X>.sam. For each expected sample and chunk, assert the SAM file exists and is non-empty (file_exists check per branch). Exercising both branches on the same test genome verifies the dispatch condition on annotation availability is correctly implemented.

## Related tools

- **uLTRA** (Aligns FLNC reads to the genome guided by a GTF annotation when one is available; its SAM outputs (06.2_ULTRA/) are validated by this skill) — https://github.com/ksahlin/ultra
- **minimap2** (Splice-aware fallback aligner used when no GTF annotation is available; its SAM outputs (06_MINIMAP2/) are validated by this skill) — https://github.com/lh3/minimap2
- **nf-core/isoseq** (Pipeline whose conditional mapping step (ULTRA or MINIMAP2) produces the SAM files under validation) — https://github.com/nf-core/isoseq

## Examples

```
test -s results/06.2_ULTRA/sample1.chunk1.sam && test -s results/06_MINIMAP2/sample1.chunk1.sam && echo 'both aligner branches OK'
```

## Evaluation signals

- file_exists check passes for every expected <sample>.chunk<X>.sam in 06.2_ULTRA/
- file_exists check passes for every expected <sample>.chunk<X>.sam in 06_MINIMAP2/
- Each SAM file is non-empty (non-zero size and contains alignment records)
- Both dispatch branches were exercised on the test genome, so outputs exist in both directories
- The set of validated SAM files matches the per-sample, per-chunk expectations from the input samplesheet

## Limitations

- uLTRA can produce spurious alignments when a malformed GTF is used, so non-empty SAM output alone does not prove alignment correctness; a malformed GTF should stop the pipeline.
- The samtools sort module was removed from the workflow, so downstream BAM sorting can no longer be relied on as an additional validation step after mapping.
- Software version numbers for the aligners are not recorded in the changelog, limiting reproducibility of the validated runs.

## Evidence

- [other] The mapping step offers uLTRA, which uses the reference genome annotation to detect small exons, and falls back to minimap2 when no annotation is available for the genome.: [evidence span withheld — anchor status: not_found]
- [readme] uLTRA is a tool for splice alignment of long transcriptomic reads to a genome, guided by a database of exon annotations.: [evidence span withheld — anchor status: external_doc]
- [other] Validation: acceptance is met when both dispatch branches produce the expected <sample>.chunk<X>.sam alignment files under 06.2_ULTRA/ and 06_MINIMAP2/.: [evidence span withheld — anchor status: not_found]
- [discussion] spurious alignments produced by uLTRA if a malformed GTF is used: "spurious alignments produced by uLTRA if a malformed GTF is used"
- [intro] [`MINIMAP2`](https://github.com/lh3/minimap2) or [`uLTRA`](https://github.com/ksahlin/ultra) aligns reads ont the genome.: "[`MINIMAP2`](https://github.com/lh3/minimap2) or [`uLTRA`](https://github.com/ksahlin/ultra) aligns reads ont the genome."
