---
name: exon-structure-transcript-assignment
description: Use when single-cell long-read (Nanopore) molecule consensus alignments
  must be assigned to annotated Gencode isoforms via strict exonic-structure matching
  to build isoform-level UMI count matrices per cell.
when_to_use_negative:
- You want novel isoform discovery or novel junction reconstruction — STRICT full-match
  assignment only recovers known Gencode transcripts (use Sicelore CollapseModel /
  step 7 instead).
- Input is raw Nanopore FASTQ rather than UMI-consensus molecules — barcode/UMI assignment,
  error correction and consensus generation must happen first (Steps 1–4b).
- Ambiguous multi-transcript assignments should be retained (e.g. gene-level-only
  quantification) — set AMBIGUOUS_ASSIGN accordingly or use a non-strict METHOD, as
  STRICT discards reads not fully matching one annotated isoform.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Sicelore-2.1.jar
  role: DeduplicateMolecule, AddGeneNameTag, AddBamMoleculeTags and IsoformMatrix
    execute the strict exon-structure isoform assignment and matrix generation
  repo: https://github.com/ucagenomix/sicelore-2.1
- name: minimap2
  role: Spliced, forward-strand alignment (-ax splice -uf) of UMI consensus reads
    to the reference genome
  repo: https://github.com/lh3/minimap2
- name: samtools
  role: Sorting and indexing molecules.bam between Sicelore steps
  repo: https://github.com/samtools/samtools
- name: spoa
  role: Partial-order-alignment consensus generation required for the molecule-consensus
    (Step 4b) branch of the workflow
  repo: https://github.com/rvaser/spoa
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-17800-6
    title: High throughput error corrected Nanopore single cell transcriptome sequencing
  source_content_hash: sha256:312f540039c91680efb8a0d1e9f1794d61da7f2abb7261cf689b536a4ac72c18
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# exon-structure-transcript-assignment

## Summary

Assigns single-cell Nanopore UMI molecules to known Gencode transcript isoforms by requiring a full exon-structure match between aligned consensus reads and annotated transcripts, then emits gene-, isoform- and junction-level UMI count matrices via Sicelore's IsoformMatrix.

## When to use

Use when single-cell long-read (Nanopore) molecule consensus alignments must be assigned to annotated Gencode isoforms via strict exonic-structure matching to build isoform-level UMI count matrices per cell.

## When NOT to use

- You want novel isoform discovery or novel junction reconstruction — STRICT full-match assignment only recovers known Gencode transcripts (use Sicelore CollapseModel / step 7 instead).
- Input is raw Nanopore FASTQ rather than UMI-consensus molecules — barcode/UMI assignment, error correction and consensus generation must happen first (Steps 1–4b).
- Ambiguous multi-transcript assignments should be retained (e.g. gene-level-only quantification) — set AMBIGUOUS_ASSIGN accordingly or use a non-strict METHOD, as STRICT discards reads not fully matching one annotated isoform.

## Inputs

- Per-chromosome molecule consensus FASTQs from the Sicelore ComputeConsensus step (option 4b)
- Reference genome FASTA (e.g. hg38/Gencode primary assembly)
- Gencode GTF annotation (e.g. gencode.v38.primary_assembly.annotation.gtf) converted to refFlat with gtfToGenePred
- Valid cell barcode list (ValidBarcodes.csv, e.g. from 10x 3M-february-2018.txt.gz whitelist)
- molecules.GE.tags.bam with GE (gene), U8 (UMI) and BC (cell barcode) tags

## Outputs

- _genematrix.txt — gene x cell UMI count matrix
- _isomatrix.txt — isoform x cell UMI count matrix
- _juncmatrix.txt — junction x cell count matrix
- _molinfos.txt — molecule-level assignment details
- Per-cell UMI yield metrics (reference medians: ~6047 gene-UMIs and ~3795 isoform-UMIs per cell)

## How to apply

First deduplicate per-chromosome molecule consensus FASTQs with Sicelore-2.1.jar DeduplicateMolecule for multi-copy genes, then align consensus reads to the reference genome with minimap2 in spliced forward-strand mode (minimap2 -ax splice -uf --sam-hit-only -t 20) and sort/index the molecules.bam with samtools. Annotate SAM records with gene names using Sicelore-2.1.jar AddGeneNameTag (TAG=GE, REFFLAT converted from the Gencode GTF with gtfToGenePred, ALLOW_MULTI_GENE_READS=true, USE_STRAND_INFO=true) and transfer BC/U8/RN tags from read names with AddBamMoleculeTags. Run Sicelore-2.1.jar IsoformMatrix with METHOD=STRICT, DELTA=2 (allowed margin of added/lacking bases at exon boundaries), MAXCLIP=150 (reads with >150 nt clipping at either end are discarded as chimeric) and AMBIGUOUS_ASSIGN=false so only full exonic-structure matches count. The STRICT setting is the decision point: it requires a complete match to one annotated Gencode transcript, which bounds the assignment yield (~63.6% of UMIs in the reference study) but guarantees isoform correctness.

## Related tools

- **Sicelore-2.1.jar** (DeduplicateMolecule, AddGeneNameTag, AddBamMoleculeTags and IsoformMatrix execute the strict exon-structure isoform assignment and matrix generation) — https://github.com/ucagenomix/sicelore-2.1
- **minimap2** (Spliced, forward-strand alignment (-ax splice -uf) of UMI consensus reads to the reference genome) — https://github.com/lh3/minimap2
- **samtools** (Sorting and indexing molecules.bam between Sicelore steps) — https://github.com/samtools/samtools
- **spoa** (Partial-order-alignment consensus generation required for the molecule-consensus (Step 4b) branch of the workflow) — https://github.com/rvaser/spoa

## Examples

```
java -jar Sicelore-2.1.jar IsoformMatrix INPUT=molecules.GE.tags.bam GENETAG=GE UMITAG=U8 CELLTAG=BC REFFLAT=refFlat CSV=ValidBarcodes.csv DELTA=2 MAXCLIP=150 METHOD=STRICT AMBIGUOUS_ASSIGN=false
```

## Evaluation signals

- Fraction of UMIs assigned to a known Gencode transcript matches the reported ~63.6%
- Number of fully supported transcripts is ~33,002
- Per-cell UMI medians are ~6047 gene-UMIs and ~3795 isoform-UMIs
- All four output files (_genematrix.txt, _isomatrix.txt, _juncmatrix.txt, _molinfos.txt) are present and structurally valid
- Matrix dimensions match the ValidBarcodes.csv cell list; cells with fewer than MINUMI UMIs were excluded by SelectValidCellBarcode

## Limitations

- STRICT full-exon-structure matching discards transcripts not exactly matching Gencode models, capping isoform assignment yield (~63.6% of UMIs in the reference dataset).
- Reads with extensive clipping (>150 nt by default MAXCLIP) are discarded, removing potential chimeric cDNA but also some genuine reads.
- DELTA=2 permits a small margin of added/lacking sequence at exon boundaries, so near-boundary indel errors can still pass as matches; multi-gene reads must be explicitly allowed (ALLOW_MULTI_GENE_READS=true) or they are dropped.

## Evidence

- [methods] DELTA exon junction margin filter: "authorizing a DELTA (default 2) bases margin of added or lacking sequences at exon boundaries"
- [methods] MAXCLIP chimeric read filter: "SAM records with extensive non-matching sequences at either end, hard- or soft clipping are discarded (MAXCLIP parameter, defaults to > 150 nt )"
- [readme] Sicelore purpose: "SiCeLoRe (Single Cell Long Read) is a suite of tools dedicated to cell or spatial barcode and unique molecular identifier (UMI) assignment of highly multiplexed single cell"
- [abstract] ScNaUmi-seq capability: "ScNaUmi-seq allows defining splicing and SNVs (RNA editing) at a single cell level"
