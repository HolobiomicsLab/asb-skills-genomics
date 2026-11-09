---
name: single-cell-isoform-quantification-matrix-generation
description: Use when long-read single-cell Nanopore data with 10x cell barcodes and
  UMIs must be quantified against known transcript annotations to produce per-cell
  isoform and gene count matrices.
when_to_use_negative:
- Input is raw Nanopore FASTQ without cell barcode/UMI annotation or prior UMI-guided
  consensus generation — run Sicelore Steps 1–4b (NanoporeBC_UMI_finder, mapping,
  UMI assignment, ComputeConsensus) first.
- The goal is novel isoform discovery from unannotated splicing; STRICT IsoformMatrix
  assignment only counts UMIs matching known Gencode transcript structures.
- Data are short-read scRNA-seq (e.g. Illumina-only 10x data) without full-length
  long reads.
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Sicelore-2.1.jar
  role: DeduplicateMolecule, AddGeneNameTag, AddBamMoleculeTags and IsoformMatrix
    assign UMIs to Gencode isoforms and emit count matrices
  repo: https://github.com/ucagenomix/sicelore-2.1
- name: minimap2
  role: spliced forward-strand alignment of molecule consensus reads to the reference
    genome
  repo: https://github.com/lh3/minimap2
- name: samtools
  role: sort and index the molecules.bam produced by minimap2
  repo: https://github.com/samtools/samtools
- name: spoa
  role: partial-order alignment used by the ComputeConsensus step generating the UMI
    consensus sequences fed into this skill
  repo: https://github.com/rvaser/spoa
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-17800-6
    title: High throughput error corrected Nanopore single cell transcriptome sequencing
  source_content_hash: sha256:23d2147d552179d9db3bbe34affd0fc082767993cffc8d6e230d0640cb00b397
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ucagenomix/sicelore-2.1
  tools:
  - Java
  - Sicelore-2.1.jar
  - minimap2
  - samtools
  - spoa
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# single-cell-isoform-quantification-matrix-generation

## Summary

Generates gene-, isoform- and junction-level UMI count matrices per cell by requiring a full exonic-structure match between UMI consensus reads and annotated Gencode transcripts, using Sicelore-2.1 IsoformMatrix on molecule BAMs.

## When to use

Use when long-read single-cell Nanopore data with 10x cell barcodes and UMIs must be quantified against known transcript annotations to produce per-cell isoform and gene count matrices.

## When NOT to use

- Input is raw Nanopore FASTQ without cell barcode/UMI annotation or prior UMI-guided consensus generation — run Sicelore Steps 1–4b (NanoporeBC_UMI_finder, mapping, UMI assignment, ComputeConsensus) first.
- The goal is novel isoform discovery from unannotated splicing; STRICT IsoformMatrix assignment only counts UMIs matching known Gencode transcript structures.
- Data are short-read scRNA-seq (e.g. Illumina-only 10x data) without full-length long reads.

## Inputs

- Per-chromosome molecule consensus FASTQs from the Sicelore ComputeConsensus step
- Gencode GTF annotation (e.g. gencode.v38.primary_assembly.annotation.gtf) converted to refFlat with gtfToGenePred
- Valid cell barcode whitelist (ValidBarcodes.csv, e.g. 3M-february-2018.txt.gz from Cellranger)
- Reference genome (e.g. hg38/Gencode v38 primary assembly)

## Outputs

- _genematrix.txt (per-cell gene-level UMI count matrix)
- _isomatrix.txt (per-cell isoform-level UMI count matrix)
- _juncmatrix.txt (per-cell junction count matrix)
- _molinfos.txt (molecule information records)
- molecules.GE.tags.bam (molecule BAM with GE, BC, U8, RN tags)

## How to apply

After UMI-guided error correction, concatenate per-chromosome molecule consensus FASTQs and deduplicate molecules from multi-copy genes with Sicelore-2.1.jar DeduplicateMolecule. Align consensus reads to the reference genome with minimap2 in spliced forward-strand mode (minimap2 -ax splice -uf --sam-hit-only -t 20), sort/index with samtools, then add gene names via Sicelore-2.1.jar AddGeneNameTag (TAG=GE, REFFLAT derived from the Gencode GTF with gtfToGenePred, ALLOW_MULTI_GENE_READS=true, USE_STRAND_INFO=true) and transfer BC/U8/RN tags with AddBamMoleculeTags. Run Sicelore-2.1.jar IsoformMatrix with GENETAG=GE, UMITAG=U8, CELLTAG=BC, DELTA=2, MAXCLIP=150, METHOD=STRICT, AMBIGUOUS_ASSIGN=false to assign UMIs to isoforms by full exonic-structure match, authorizing a 2-base margin at exon boundaries and discarding reads with >150 nt clipping. The STRICT method emits _genematrix.txt, _isomatrix.txt, _juncmatrix.txt and _molinfos.txt.

## Related tools

- **Sicelore-2.1.jar** (DeduplicateMolecule, AddGeneNameTag, AddBamMoleculeTags and IsoformMatrix assign UMIs to Gencode isoforms and emit count matrices) — https://github.com/ucagenomix/sicelore-2.1
- **minimap2** (spliced forward-strand alignment of molecule consensus reads to the reference genome) — https://github.com/lh3/minimap2
- **samtools** (sort and index the molecules.bam produced by minimap2) — https://github.com/samtools/samtools
- **spoa** (partial-order alignment used by the ComputeConsensus step generating the UMI consensus sequences fed into this skill) — https://github.com/rvaser/spoa

## Examples

```
java -jar Sicelore-2.1.jar IsoformMatrix INPUT=molecules.GE.tags.bam GENETAG=GE UMITAG=U8 CELLTAG=BC REFFLAT=refFlat CSV=ValidBarcodes.csv DELTA=2 MAXCLIP=150 METHOD=STRICT
```

## Evaluation signals

- Fraction of UMIs assigned to a known Gencode transcript isoform is ~63.6% with full exonic-structure matching (METHOD=STRICT).
- Number of fully supported transcripts is ~33,002.
- Per-cell medians are ~6047 gene-UMIs and ~3795 isoform-UMIs.
- _genematrix.txt, _isomatrix.txt, _juncmatrix.txt and _molinfos.txt are present with the expected matrix structure.
- No SAM records with hard/soft clipping exceeding MAXCLIP=150 nt contribute to assignments.

## Limitations

- Only UMIs whose consensus exon structure fully matches an annotated Gencode transcript are assigned; isoforms not in the annotation are missed unless novel isoform discovery (Step 7, CollapseModel with MINEVIDENCE) is run separately.
- SAM records with extensive non-matching ends (>150 nt clipping) are discarded, dropping potential chimeric-read signal.
- Ambiguous isoform assignment is disabled (AMBIGUOUS_ASSIGN=false), so multi-isoform molecules are excluded from the isoform matrix.
- The DELTA=2 margin allows small exon-boundary errors but is a fixed tolerance; larger Nanopore indel errors at junctions cause misassignment.

## Evidence

- [methods] authorizing a DELTA (default 2) bases margin of added or lacking sequences at exon boundaries: "authorizing a DELTA (default 2) bases margin of added or lacking sequences at exon boundaries"
- [methods] MAXCLIP chimeric read filter: "SAM records with extensive non-matching sequences at either end, hard- or soft clipping are discarded (MAXCLIP parameter, defaults to > 150 nt )"
- [readme] IsoformMatrix emits count matrices: "Step 4 - Generate cell/spatialBC x Gene-/Isoform-/Junction-level matrices"
- [methods] Gencode annotation source: "wget https://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_38/gencode.v38.primary_assembly.annotation.gtf.gz"
