---
name: ultra-genome-alignment-with-gtf
description: Use when mapping FLNC (full-length non-chimeric) Iso-Seq reads to a genome
  for which a GTF exon annotation exists, and accurate splice alignment around small
  exons is required.
when_to_use_negative:
- No GTF annotation is available for the reference genome — the pipeline falls back
  to minimap2 splice-aware alignment instead of uLTRA.
- The GTF is malformed — uLTRA produces spurious alignments in that case, so the annotation
  should be validated first.
- Reads are short genomic reads or non-transcriptomic data — uLTRA targets splice
  alignment of long transcriptomic reads.
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: uLTRA
  role: Splice-aligns FLNC reads onto the genome guided by the exon annotation database,
    particularly accurate for small exons
  repo: https://github.com/ksahlin/ultra
- name: minimap2
  role: Alternative splice-aware aligner used when no GTF annotation is available
    for the genome
  repo: https://github.com/lh3/minimap2
- name: nf-core/isoseq
  role: Nextflow pipeline that implements the conditional uLTRA/minimap2 mapping dispatch
    for FLNC reads
  repo: https://github.com/nf-core/isoseq
- name: BioPerl
  role: Removes spurious uLTRA alignments in the uLTRA branch of the pipeline
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad150
    title: 'nf-core/isoseq: simple gene and isoform annotation with PacBio Iso-Seq
      long-read sequencing'
  source_content_hash: sha256:4a7935f2e53be689c9319c3330b7572b468be05164309c6c38d7e4383c0f0221
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ksahlin/ultra
  tools:
  - BioPerl
  - minimap2
  - nf-core/isoseq
  - uLTRA
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# ultra-genome-alignment-with-gtf

## Summary

Aligns PacBio Iso-Seq FLNC reads onto a reference genome with uLTRA, a splice-aware aligner guided by exon annotations, as implemented in the nf-core/isoseq mapping step; uLTRA is chosen when a GTF annotation is available for the genome, and falls back to minimap2 when it is not.

## When to use

Use when mapping FLNC (full-length non-chimeric) Iso-Seq reads to a genome for which a GTF exon annotation exists, and accurate splice alignment around small exons is required.

## When NOT to use

- No GTF annotation is available for the reference genome — the pipeline falls back to minimap2 splice-aware alignment instead of uLTRA.
- The GTF is malformed — uLTRA produces spurious alignments in that case, so the annotation should be validated first.
- Reads are short genomic reads or non-transcriptomic data — uLTRA targets splice alignment of long transcriptomic reads.

## Inputs

- FLNC FASTA reads (post TAMA polyA clean-up, gunzipped for the uLTRA path)
- Reference genome FASTA
- Reference GTF exon annotation (triggers the uLTRA branch)

## Outputs

- Per-sample, per-chunk SAM alignment files (<sample>.chunk<X>.sam) under a uLTRA output directory (e.g. 06.2_ULTRA/)
- Downstream BioPerl-filtered alignments and samtools-sorted BAM files

## How to apply

Load the FLNC reads (output of the TAMA polyA clean-up step), the reference genome FASTA, and the GTF annotation. Because uLTRA path requires uncompressed input, decompress the FLNC FASTAs with GUNZIP first. Dispatch on annotation availability: if a GTF is present, invoke uLTRA in genome+annotation mode so the exon database can guide splice alignment and improve detection of small exons; otherwise invoke minimap2 in splice-aware mode instead. Run per sample and per chunk, writing <sample>.chunk<X>.sam alignment files to a branch-specific output directory (06.2_ULTRA/ in the nf-core/isoseq layout). In the full pipeline, spurious uLTRA alignments are subsequently filtered with BioPerl, and alignments are sorted and converted to BAM with samtools sort.

## Related tools

- **uLTRA** (Splice-aligns FLNC reads onto the genome guided by the exon annotation database, particularly accurate for small exons) — https://github.com/ksahlin/ultra
- **minimap2** (Alternative splice-aware aligner used when no GTF annotation is available for the genome) — https://github.com/lh3/minimap2
- **nf-core/isoseq** (Nextflow pipeline that implements the conditional uLTRA/minimap2 mapping dispatch for FLNC reads) — https://github.com/nf-core/isoseq
- **BioPerl** (Removes spurious uLTRA alignments in the uLTRA branch of the pipeline)

## Evaluation signals

- Both dispatch branches produce the expected <sample>.chunk<X>.sam alignment files under the uLTRA and minimap2 output directories (file_exists check on each branch's SAM outputs).
- Every expected SAM file is non-empty in the uLTRA output directory.
- FLNC FASTAs were successfully decompressed before the uLTRA run (GUNZIP step completed for the uLTRA path).
- Spurious alignments from the uLTRA run are removed by the downstream BioPerl filtering step.
- Alignments are subsequently sorted and converted to BAM by samtools sort without errors.

## Limitations

- uLTRA can produce spurious alignments when a malformed GTF is used; in earlier pipeline versions BioPerl removed these, and removing the bioperl module stops the pipeline when a malformed GTF is supplied.
- The uLTRA path requires FLNC FASTAs to be decompressed with GUNZIP before alignment.
- Without a GTF annotation for the genome, uLTRA cannot be used and the pipeline dispatches to minimap2 instead.
- No software version numbers are recorded for the tools/modules in the changelog, creating a reproducibility gap for pinning the uLTRA version.

## Evidence

- [readme] uLTRA as exon-annotation-guided splice aligner for long reads: [evidence span withheld — anchor status: external_doc]
- [readme] uLTRA accuracy on small exons: [evidence span withheld — anchor status: external_doc]
- [intro] Pipeline aligns FLNC reads on the genome with minimap2 or uLTRA: "[`MINIMAP2`](https://github.com/lh3/minimap2) or [`uLTRA`](https://github.com/ksahlin/ultra) aligns reads ont the genome."
- [intro] GUNZIP decompression is specific to the uLTRA path: "- [GUNZIP](#gunzip) - Decompress FLNC fastas (uLTRA path only)"
- [intro] BioPerl removes spurious uLTRA alignments: "- [BIOPERL](#bioperl) - Remove spurious alignments (uLTRA path only"
- [discussion] Malformed GTF causes spurious uLTRA alignments: "spurious alignments produced by uLTRA if a malformed GTF is used"
