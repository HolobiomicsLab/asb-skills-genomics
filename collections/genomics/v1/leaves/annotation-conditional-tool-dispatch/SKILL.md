---
name: annotation-conditional-tool-dispatch
description: Decision support for choosing a splice-aware aligner for PacBio Iso-Seq
  FLNC reads based on whether a reference genome GTF annotation exists; trigger when
  building or reproducing the nf-core/isoseq mapping step.
when_to_use_negative:
- The analysis target is not FLNC long reads from PacBio Iso-Seq data being mapped
  onto a reference genome.
- A single aligner is mandated upstream (e.g. only minimap2 is permitted), so no annotation-dependent
  dispatch is needed.
- The reference GTF is known to be malformed and the bioperl spurious-alignment filter
  has been removed — the pipeline will stop instead of proceeding cleanly.
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: uLTRA
  role: Annotation-guided splice aligner used when a reference GTF is available; detects
    small exons via an exon-annotation database
  repo: https://github.com/ksahlin/ultra
- name: minimap2
  role: Fallback splice-aware aligner used when no GTF annotation exists for the genome
  repo: https://github.com/lh3/minimap2
- name: TAMA
  role: Upstream GSTAMA_POLYACLEANUP produces the FLNC reads fed into this mapping
    dispatch
  repo: https://github.com/GenomeRIK/tama
- name: BioPerl
  role: Removes spurious uLTRA alignments on the annotation-guided branch
  repo: https://bioperl.org/
- name: nf-core/isoseq
  role: Pipeline hosting the conditional ULTRA-or-MINIMAP2 mapping step
  repo: https://github.com/nf-core/isoseq
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad150
    title: 'nf-core/isoseq: simple gene and isoform annotation with PacBio Iso-Seq
      long-read sequencing'
  source_content_hash: sha256:6e9d78a4b2a643b867a94f40fe9b4e470df1bb96a5f228d5f96714c9c020378c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ksahlin/ultra
  tools:
  - BioPerl
  - TAMA
  - minimap2
  - nf-core/isoseq
  - uLTRA
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# annotation-conditional-tool-dispatch

## Summary

A conditional dispatch pattern from the nf-core/isoseq mapping step: align FLNC PacBio Iso-Seq reads with uLTRA when a reference GTF annotation is available, otherwise fall back to splice-aware minimap2. uLTRA exploits the annotation to detect small exons, while minimap2 keeps the pipeline runnable for unannotated genomes.

## When to use

Decision support for choosing a splice-aware aligner for PacBio Iso-Seq FLNC reads based on whether a reference genome GTF annotation exists; trigger when building or reproducing the nf-core/isoseq mapping step.

## When NOT to use

- The analysis target is not FLNC long reads from PacBio Iso-Seq data being mapped onto a reference genome.
- A single aligner is mandated upstream (e.g. only minimap2 is permitted), so no annotation-dependent dispatch is needed.
- The reference GTF is known to be malformed and the bioperl spurious-alignment filter has been removed — the pipeline will stop instead of proceeding cleanly.

## Inputs

- FLNC reads FASTA/FASTQ (post TAMA polyA clean-up, gunzipped for the uLTRA path)
- Reference genome FASTA
- Optional reference genome annotation in GTF format

## Outputs

- SAM alignment files per sample per chunk (<sample>.chunk<X>.sam) under 06.2_ULTRA/ (uLTRA branch) or 06_MINIMAP2/ (minimap2 branch)
- Sorted BAM alignments after SAMTOOLS SORT

## How to apply

Load the polyA-cleaned FLNC reads (output of the TAMA polyA clean-up step; decompressed with gunzip for the uLTRA path), the reference genome FASTA, and the optional reference GTF. Implement the dispatch condition on annotation availability: if a GTF is present, run uLTRA in genome+annotation mode so its annotation database guides splice alignment (especially accurate for small exons); otherwise invoke minimap2 in splice mode. Each tool chunks reads into per-sample, per-chunk alignments written as <sample>.chunk<X>.sam under branch-specific directories (06.2_ULTRA/ for uLTRA, 06_MINIMAP2/ for minimap2). Downstream of the uLTRA path, BioPerl removes spurious alignments — uLTRA produces spurious alignments if a malformed GTF is used — and both branches are followed by SAMTOOLS SORT to convert SAM to sorted BAM.

## Related tools

- **uLTRA** (Annotation-guided splice aligner used when a reference GTF is available; detects small exons via an exon-annotation database) — https://github.com/ksahlin/ultra
- **minimap2** (Fallback splice-aware aligner used when no GTF annotation exists for the genome) — https://github.com/lh3/minimap2
- **TAMA** (Upstream GSTAMA_POLYACLEANUP produces the FLNC reads fed into this mapping dispatch) — https://github.com/GenomeRIK/tama
- **BioPerl** (Removes spurious uLTRA alignments on the annotation-guided branch) — https://bioperl.org/
- **nf-core/isoseq** (Pipeline hosting the conditional ULTRA-or-MINIMAP2 mapping step) — https://github.com/nf-core/isoseq

## Examples

```
nextflow run nf-core/isoseq --input samplesheet.csv --genome_fa ref.fa --annotation genes.gtf   # omit --annotation to trigger the minimap2 fallback branch
```

## Evaluation signals

- Both dispatch branches exercised on a test genome produce the expected <sample>.chunk<X>.sam files.
- Every expected SAM file exists and is non-empty in both 06.2_ULTRA/ and 06_MINIMAP2/ (file_exists check per branch).
- The uLTRA branch is selected only when a GTF is supplied; minimap2 branch is selected when it is absent.
- SAM outputs from either branch convert cleanly to sorted BAM via SAMTOOLS SORT.
- On the uLTRA path, BioPerl filtering removes spurious alignments without deleting valid alignments.

## Limitations

- uLTRA produces spurious alignments when a malformed GTF is used; the BioPerl module must remain in place to stop the pipeline in that case.
- No software version numbers are recorded for the aligner modules, creating a reproducibility gap when pinning uLTRA or minimap2 versions.
- Later releases removed the 'uLTRA pipeline' process, the bioperl module, and the samtools sort module, so the dispatch layout is version-dependent.

## Evidence

- [other] uLTRA aligner leverages genome annotation to detect small exons and falls back to minimap2 when no annotation: [evidence span withheld — anchor status: not_found]
- [readme] uLTRA is a splice aligner guided by exon annotations, accurate on small exons: [evidence span withheld — anchor status: external_doc]
- [readme] minimap2 splice preset for long RNA reads: [evidence span withheld — anchor status: external_doc]
- [intro] both aligners align FLNC reads onto the genome: "[`MINIMAP2`](https://github.com/lh3/minimap2) or [`uLTRA`](https://github.com/ksahlin/ultra) aligns reads ont the genome."
- [discussion] uLTRA spurious alignments with malformed GTF: "spurious alignments produced by uLTRA if a malformed GTF is used"
