---
name: minimap2-splice-alignment-without-annotation
description: Use when mapping cleaned FLNC reads onto a genome that lacks a reference
  GTF annotation, or when deciding which aligner branch nf-core/isoseq should dispatch
  to based on annotation availability.
when_to_use_negative:
- A reference GTF annotation is available for the genome — nf-core/isoseq dispatches
  to uLTRA instead, since it uses the annotation to detect small exons.
- The reads are genomic (not transcriptomic/spliced) PacBio reads — minimap2 genomic
  presets such as map-pb or map-hifi are the appropriate mode, not -ax splice.
- A downstream step expects annotation-guided junction detection or annotated-junction
  biasing (e.g. minimap2's --junc-bed), since this branch runs without any GTF.
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: minimap2
  role: Splice-aware fallback aligner that maps FLNC reads onto the genome when no
    reference GTF is available
  repo: https://github.com/lh3/minimap2
- name: uLTRA
  role: Annotation-guided alternative aligner (the other dispatch branch), chosen
    when a GTF is present
  repo: https://github.com/ksahlin/ultra
- name: nf-core/isoseq
  role: Nextflow pipeline hosting the conditional aligner dispatch and the 06_MINIMAP2
    output branch
  repo: https://github.com/nf-core/isoseq
- name: Nextflow
  role: Workflow engine implementing the per-sample, per-chunk parallel dispatch of
    the alignment branches
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad150
    title: 'nf-core/isoseq: simple gene and isoform annotation with PacBio Iso-Seq
      long-read sequencing'
  source_content_hash: sha256:7df368719e75cb53458f2e457aaf8012b15a5211b0339afb8f9425b95d5d8087
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - Nextflow
  - minimap2
  - nf-core/isoseq
  - uLTRA
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# minimap2-splice-alignment-without-annotation

## Summary

Splice-aware mapping of cleaned PacBio Iso-Seq FLNC reads onto a reference genome with minimap2, used as the fallback branch of nf-core/isoseq when no reference GTF annotation is available for the genome.

## When to use

Use when mapping cleaned FLNC reads onto a genome that lacks a reference GTF annotation, or when deciding which aligner branch nf-core/isoseq should dispatch to based on annotation availability.

## When NOT to use

- A reference GTF annotation is available for the genome — nf-core/isoseq dispatches to uLTRA instead, since it uses the annotation to detect small exons.
- The reads are genomic (not transcriptomic/spliced) PacBio reads — minimap2 genomic presets such as map-pb or map-hifi are the appropriate mode, not -ax splice.
- A downstream step expects annotation-guided junction detection or annotated-junction biasing (e.g. minimap2's --junc-bed), since this branch runs without any GTF.

## Inputs

- PolyA-cleaned FLNC reads (FASTA, output of the TAMA polyA clean-up step)
- Reference genome FASTA
- Samplesheet defining samples and per-sample chunking

## Outputs

- Per-sample, per-chunk splice-aware alignments: 06_MINIMAP2/<sample>.chunk<X>.sam

## How to apply

In the nf-core/isoseq mapping step, implement a dispatch condition on annotation availability: if no GTF annotation exists for the target genome, run minimap2 in splice-aware mode (`-ax splice`) on the polyA-cleaned FLNC reads (produced by the TAMA cleanup step) against the reference genome FASTA. This is the fallback to uLTRA, which is preferred when an annotation is present because it is exon-annotation-guided and particularly accurate for small exons. Inputs are read per sample and per chunk, and each chunk is written as an independent SAM alignment file under the 06_MINIMAP2/ output directory. Because no annotation guides junction detection, splice junctions are inferred from the read alignments alone, so alignment quality depends on minimap2's splice preset rather than annotated junctions.

## Related tools

- **minimap2** (Splice-aware fallback aligner that maps FLNC reads onto the genome when no reference GTF is available) — https://github.com/lh3/minimap2
- **uLTRA** (Annotation-guided alternative aligner (the other dispatch branch), chosen when a GTF is present) — https://github.com/ksahlin/ultra
- **nf-core/isoseq** (Nextflow pipeline hosting the conditional aligner dispatch and the 06_MINIMAP2 output branch) — https://github.com/nf-core/isoseq
- **Nextflow** (Workflow engine implementing the per-sample, per-chunk parallel dispatch of the alignment branches)

## Examples

```
minimap2 -ax splice ref.fa flnc_reads.fa > 06_MINIMAP2/sample1.chunk1.sam
```

## Evaluation signals

- Every expected 06_MINIMAP2/<sample>.chunk<X>.sam file exists and is non-empty for all samples and chunks in the samplesheet.
- The SAM records contain splice-aware CIGAR strings with N operations (intron gaps), confirming the splice preset was used rather than a contiguous genomic preset.
- Each SAM file references only contigs present in the supplied reference genome FASTA.
- Running the same inputs with a valid GTF switches the pipeline to the uLTRA branch (06.2_ULTRA/), confirming the annotation-availability condition governs dispatch.
- Downstream sorting/merging of the chunk SAMs succeeds without malformed alignment records.

## Limitations

- Without an annotation, junction detection relies solely on minimap2's splice inference; accuracy gains from annotation guidance (e.g. small-exon detection via uLTRA) are lost.
- The uLTRA branch can emit spurious alignments when a malformed GTF is used, so annotation quality affects which branch is trustworthy.
- Tool/module changes in the pipeline history (e.g. removal of the samtools sort module and the uLTRA pipeline process) mean output expectations can shift between nf-core/isoseq versions.

## Evidence

- [intro] The mapping step offers uLTRA, which uses the reference genome annotation to detect small exons, and falls back to minimap2 when no annotation is available for the genome.: [evidence span withheld — anchor status: not_found]
- [readme] minimap2 splice preset for spliced long reads, strand unknown: [evidence span withheld — anchor status: external_doc]
- [readme] uLTRA aligns long transcriptomic reads using exon annotations: [evidence span withheld — anchor status: external_doc]
- [readme] splice-aware alignment of PacBio Iso-Seq reads against a reference genome: "splice-aware alignment of PacBio Iso-Seq or Nanopore cDNA or Direct RNA reads
against a reference genome"
