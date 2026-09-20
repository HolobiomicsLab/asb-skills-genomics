---
name: minimap2-spliced-long-read-alignment
description: Use when long-read (Nanopore/PacBio) FASTQs from single-cell 10x Genomics
  protocols must be spliced-aligned to a reference genome before UMI assignment or
  isoform/gene matrix generation with Sicelore.
when_to_use_negative:
- Input reads are Illumina short reads — the article's strategy relies on minimap2's
  long-read spliced mode, not short-read splice aligners
- Reads have not been stranded/barcode-assigned in Step 1 — mapping unstranded or
  chimeric reads without prior scanfastq splitting undermines the forward-strand (-uf)
  assumption
- The analysis goal is short-read-free Visium-style quantification that does not require
  genome spliced alignment of long reads
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3170
tools:
- name: minimap2
  role: Spliced, forward-strand alignment of Nanopore reads/molecules to the reference
    genome (-ax splice -uf --sam-hit-only -t 20), with optional --junc-bed guidance
    from a Gencode-derived junctions.bed
  repo: https://github.com/lh3/minimap2
- name: samtools
  role: Sort and index the minimap2 output BAM so downstream Sicelore steps (UMI assignment,
    AddGeneNameTag, IsoformMatrix) can access it
  repo: https://github.com/samtools/samtools
- name: paftools.js
  role: Convert Gencode GTF exon coordinates into a BED-format splice junction file
    consumed by minimap2's --junc-bed option
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_004
  - task_003
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-17800-6
    title: High throughput error corrected Nanopore single cell transcriptome sequencing
  source_content_hash: sha256:4f58e244f25d023a09983f6b4840847e62e41de5a164c7503cfbcd743b824b37
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - Java
  - Sicelore-2.1.jar
  - minimap2
  - paftools.js
  - samtools
  - sicelore-2.1
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# minimap2-spliced-long-read-alignment

## Summary

Align stranded Nanopore long reads (barcode/UMI-annotated 'pass' reads or UMI consensus molecule FASTQs) to the reference genome in spliced, forward-strand mode with minimap2, optionally guided by a Gencode-derived splice-junction BED, then sort and index the resulting BAM with samtools for downstream Sicelore steps.

## When to use

Use when long-read (Nanopore/PacBio) FASTQs from single-cell 10x Genomics protocols must be spliced-aligned to a reference genome before UMI assignment or isoform/gene matrix generation with Sicelore.

## When NOT to use

- Input reads are Illumina short reads — the article's strategy relies on minimap2's long-read spliced mode, not short-read splice aligners
- Reads have not been stranded/barcode-assigned in Step 1 — mapping unstranded or chimeric reads without prior scanfastq splitting undermines the forward-strand (-uf) assumption
- The analysis goal is short-read-free Visium-style quantification that does not require genome spliced alignment of long reads

## Inputs

- Stranded 'pass' folder Nanopore FASTQ reads from NanoporeBC_UMI_finder.jar scanfastq, or concatenated molecule consensus FASTQs from Sicelore ComputeConsensus/DeduplicateMolecule
- Reference genome (e.g. hg38 or mouse Gencode vM18 genome) in FASTA with minimap2 index
- Splice-junction BED file (junctions.bed) derived from Gencode GTF exon coordinates via paftools.js

## Outputs

- Spliced alignment BAM of long reads/molecules to the reference genome
- Coordinate-sorted, indexed BAM (molecules.bam) ready for Sicelore UMI/gene-name tag transfer

## How to apply

First, derive a splice-junction BED file from the Gencode annotation (e.g. Gencode vM18 for mouse or gencode.v38.primary_assembly.annotation.gtf for human) using paftools.js, a companion script of minimap2, and supply it via minimap2's --junc-bed option. Map the 'pass' folder reads (or concatenated per-chromosome consensus molecule FASTQs from Sicelore ComputeConsensus/DeduplicateMolecule) with 'minimap2 -ax splice -uf --sam-hit-only -t 20', where -uf enforces forward-strand orientation of the already-stranded reads and --sam-hit-only suppresses unmapped records. Sort and index the output BAM with samtools so that downstream Sicelore steps (assignumis, AddGeneNameTag, AddBamMoleculeTags, IsoformMatrix) can query it. The spliced mode with annotation-guided junctions maximizes full-length transcript alignment accuracy, which is critical because ScNaUmi-seq uses these alignments to define splicing at single-cell level.

## Related tools

- **minimap2** (Spliced, forward-strand alignment of Nanopore reads/molecules to the reference genome (-ax splice -uf --sam-hit-only -t 20), with optional --junc-bed guidance from a Gencode-derived junctions.bed) — https://github.com/lh3/minimap2
- **samtools** (Sort and index the minimap2 output BAM so downstream Sicelore steps (UMI assignment, AddGeneNameTag, IsoformMatrix) can access it) — https://github.com/samtools/samtools
- **paftools.js** (Convert Gencode GTF exon coordinates into a BED-format splice junction file consumed by minimap2's --junc-bed option) — https://github.com/lh3/minimap2

## Examples

```
minimap2 -ax splice -uf --sam-hit-only -t 20 ref.fa pass.fastq.gz | samtools sort -o aligned.sorted.bam - && samtools index aligned.sorted.bam
```

## Evaluation signals

- The output BAM exists, is coordinate-sorted, and is indexed (.bai present) by samtools
- Aligned records retain read names carrying cell barcode (BC), UMI (U8) and read-number (RN) tags or are compatible with Sicelore AddBamMoleculeTags transfer
- Junctions.bed parses as valid BED (chromosome, start/end, strand) and its junctions are consistent with exon structure of the source Gencode GTF
- Alignment commands use the article's flags (-ax splice -uf --sam-hit-only -t 20) so reads map in forward orientation with only spliced, mapped hits reported
- Downstream Sicelore IsoformMatrix on the sorted/indexed BAM reproduces the expected gene/isoform metrics (e.g. ~63.6% UMI isoform assignment, ~6047 gene-UMIs and ~3795 isoform-UMIs per cell medians in the reproduction card)

## Limitations

- Reads without detected poly(A) and adapter in Step 1 land in the 'failed' folder and should not be used for this mapping step
- Chimeric cDNA reads must be split before mapping; unsplit chimeras can produce misleading spliced alignments
- Junction guidance depends on the quality/completeness of the Gencode annotation; junctions absent from the BED may not be recovered by the aligner

## Evidence

- [methods] Map the 'pass' folder reads against the reference genome with minimap2 (-ax splice -uf --sam-hit-only -t 20) and sort/index the BAM with samtools.: [evidence span withheld — anchor status: not_found]
- [other] The splice junction input for minimap2 is prepared by converting the Gencode vM18 GTF annotation into a BED file using paftools.js, a companion script of minimap2.: [evidence span withheld — anchor status: not_found]
- [other] Align consensus reads to the reference genome with minimap2 in spliced, forward-strand mode (minimap2 -ax splice -uf --sam-hit-only -t 20) and sort/index the resulting molecules.bam with samtools.: [evidence span withheld — anchor status: not_found]
- [readme] Step 2 - Mapping against reference genome: "Step 2 - [Mapping against reference genome](#mapping)"
