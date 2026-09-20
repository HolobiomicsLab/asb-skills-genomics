---
name: long-read-isoform-assignment-quality-verification
description: Use when molecule consensus BAMs from Nanopore scRNA-seq must be assigned
  to annotated Gencode isoforms and the resulting isoform-level UMI count matrices
  need verification against reported assignment rates and per-cell medians.
when_to_use_negative:
- Input is raw Nanopore FASTQ rather than UMI-consensus molecules — the full-match
  isoform assignment requires consensus sequences per UMI (Step 4b), not raw noisy
  reads
- The goal is novel isoform discovery or unannotated transcript models — IsoformMatrix
  STRICT mode only assigns UMIs whose exon structure fully matches annotated Gencode
  transcripts
- Working with short-read 10x data alone, which lacks full-length transcript information
  needed for isoform-level assignment
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Sicelore-2.1.jar
  role: DeduplicateMolecule, AddGeneNameTag, AddBamMoleculeTags and IsoformMatrix
    (METHOD=STRICT) perform the molecule deduplication, gene annotation and strict
    isoform assignment producing the count matrices
  repo: https://github.com/ucagenomix/sicelore-2.1
- name: minimap2
  role: Aligns UMI consensus reads to the reference genome in spliced, forward-strand
    mode (-ax splice -uf) to produce molecules.bam for isoform assignment
  repo: https://github.com/lh3/minimap2
- name: samtools
  role: Sorts and indexes molecules.bam and the tag-annotated BAMs between Sicelore
    steps
  repo: https://github.com/samtools/samtools
- name: Java
  role: Runtime required to execute Sicelore-2.1.jar tools (Java >= 13)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-17800-6
    title: High throughput error corrected Nanopore single cell transcriptome sequencing
  source_content_hash: sha256:b53a9b283e91a6f644cc8ad03196f2b7fa3747d6281e62727048e0897ee30baf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ucagenomix/sicelore-2.1
  tools:
  - Java
  - Sicelore-2.1.jar
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# long-read-isoform-assignment-quality-verification

## Summary

Verifies that UMI consensus molecules from Nanopore long-read single-cell data are correctly assigned to known Gencode transcript isoforms by full exonic-structure matching, using Sicelore's IsoformMatrix with STRICT method, and validates the resulting gene-/isoform-level count matrices against expected assignment rates and per-cell yields.

## When to use

Use when molecule consensus BAMs from Nanopore scRNA-seq must be assigned to annotated Gencode isoforms and the resulting isoform-level UMI count matrices need verification against reported assignment rates and per-cell medians.

## When NOT to use

- Input is raw Nanopore FASTQ rather than UMI-consensus molecules — the full-match isoform assignment requires consensus sequences per UMI (Step 4b), not raw noisy reads
- The goal is novel isoform discovery or unannotated transcript models — IsoformMatrix STRICT mode only assigns UMIs whose exon structure fully matches annotated Gencode transcripts
- Working with short-read 10x data alone, which lacks full-length transcript information needed for isoform-level assignment

## Inputs

- per-chromosome molecule consensus FASTQs from the Sicelore ComputeConsensus step
- Gencode GTF annotation (e.g. gencode.v38.primary_assembly.annotation.gtf) converted to refFlat via gtfToGenePred
- reference genome (e.g. hg38) for minimap2 spliced alignment
- ValidBarcodes.csv of validated cell barcodes
- 10x Genomics barcode whitelist (e.g. 3M-february-2018.txt.gz)

## Outputs

- molecules.GE.tags.bam with gene-name (GE), cell barcode (BC), UMI (U8) and read-number (RN) tags
- _genematrix.txt gene-level UMI count matrix per cell
- _isomatrix.txt isoform-level UMI count matrix per cell
- _juncmatrix.txt junction count matrix per cell
- _molinfos.txt molecule-level assignment information
- isoform assignment rate (~63.6% of UMIs assigned to a known Gencode transcript)

## How to apply

Deduplicate molecules from concatenated per-chromosome consensus FASTQs with Sicelore-2.1.jar DeduplicateMolecule, then align consensus reads to the reference genome with minimap2 in spliced, forward-strand mode (minimap2 -ax splice -uf --sam-hit-only -t 20) and sort/index the molecules.bam with samtools. Annotate molecules with gene names using Sicelore-2.1.jar AddGeneNameTag (TAG=GE, REFFLAT generated from the Gencode GTF via gtfToGenePred, ALLOW_MULTI_GENE_READS=true, USE_STRAND_INFO=true), then transfer BC/U8/RN tags from read names into the BAM with AddBamMoleculeTags. Run IsoformMatrix with GENETAG=GE, UMITAG=U8, CELLTAG=BC, DELTA=2, MAXCLIP=150, METHOD=STRICT, AMBIGUOUS_ASSIGN=false and MAPQV0=false so UMIs are assigned only when the UMI consensus exon structure fully matches an annotated transcript, with a DELTA=2 base margin allowed at exon boundaries and chimeric reads with >150 nt clipping discarded. Validate outputs by confirming the fraction of assigned UMIs, the number of fully supported transcripts, and per-cell medians of gene- and isoform-UMIs match expected values, and check file presence/structure of the emitted matrix and metrics files.

## Related tools

- **Sicelore-2.1.jar** (DeduplicateMolecule, AddGeneNameTag, AddBamMoleculeTags and IsoformMatrix (METHOD=STRICT) perform the molecule deduplication, gene annotation and strict isoform assignment producing the count matrices) — https://github.com/ucagenomix/sicelore-2.1
- **minimap2** (Aligns UMI consensus reads to the reference genome in spliced, forward-strand mode (-ax splice -uf) to produce molecules.bam for isoform assignment) — https://github.com/lh3/minimap2
- **samtools** (Sorts and indexes molecules.bam and the tag-annotated BAMs between Sicelore steps) — https://github.com/samtools/samtools
- **Java** (Runtime required to execute Sicelore-2.1.jar tools (Java >= 13))

## Examples

```
java -jar Sicelore-2.1.jar IsoformMatrix INPUT=molecules.GE.tags.bam GENETAG=GE UMITAG=U8 CELLTAG=BC REFFLAT=refFlat CSV=ValidBarcodes.csv DELTA=2 MAXCLIP=150 METHOD=STRICT AMBIGUOUS_ASSIGN=false MAPQV0=false
```

## Evaluation signals

- The fraction of UMIs assigned to a known Gencode transcript matches the reported ~63.6%
- The number of fully supported transcripts matches ~33,002
- Per-cell medians are ~6047 gene-UMIs and ~3795 isoform-UMIs
- The _genematrix.txt, _isomatrix.txt, _juncmatrix.txt and _molinfos.txt files are present with correct structure in the output directory
- Assigned molecules respect the STRICT filters: DELTA=2 exon-boundary margin, MAXCLIP=150 clipping cutoff, and AMBIGUOUS_ASSIGN=false (no ambiguous isoform assignments)

## Limitations

- Only UMIs whose consensus exon structures fully match annotated transcripts are assigned (~63.6%), so novel isoforms and unannotated transcripts are excluded under METHOD=STRICT
- SAM records with extensive non-matching sequences at either end (hard- or soft-clipping > MAXCLIP, default 150 nt) are discarded as chimeric
- A DELTA (default 2) base margin of added or lacking sequences at exon boundaries is tolerated, which may blur assignments at imprecise splice junctions

## Evidence

- [results] Using the Sicelore IsoformMatrix method with full exonic-structure matching, 63.6% of UMIs were assigned to a known Gencode transcript isoform: [evidence span withheld — anchor status: not_found]
- [methods] IsoformMatrix with DELTA exon junction margin: "authorizing a DELTA (default 2) bases margin of added or lacking sequences at exon boundaries"
- [methods] MAXCLIP chimeric read filter: "SAM records with extensive non-matching sequences at either end, hard- or soft clipping are discarded (MAXCLIP parameter, defaults to > 150 nt )"
- [readme] IsoformMatrix step 4b option in README: "Step 4 - Generate cell/spatialBC x Gene-/Isoform-/Junction-level matrices"
