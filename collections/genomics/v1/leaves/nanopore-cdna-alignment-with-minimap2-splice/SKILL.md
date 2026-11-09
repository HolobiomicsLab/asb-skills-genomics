---
name: nanopore-cdna-alignment-with-minimap2-splice
description: Use when analyzing 10x Oxford Nanopore long-read single-cell cDNA (or
  PacBio with an appropriate index) and reads must be aligned to a genome for downstream
  isoform reconstruction and gene quantification.
when_to_use_negative:
- The input is short-read data (e.g. Cell Ranger/short-read UMI quantification), which
  should instead serve as the comparison baseline, not be aligned with minimap2 splice.
- No genome reference directory is available and only transcript-level quantification
  is needed — scywalker alignment is genome-based via minimap2_splice.
- The dataset's organelle chromosomes are not specified at reference-building time;
  the isoquant-based code can hang or crash on the very different organelle data.
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: minimap2
  role: Default aligner in scywalker (minimap2_splice, splice preset) used to align
    full-length ONT cDNA reads to the genome
  repo: https://github.com/derijkp/scywalker
- name: scywalker
  role: End-to-end workflow invoking minimap2_splice alignment as part of cellbarcode
    assignment, isoform reconstruction, and per-cell quantification
  repo: https://github.com/derijkp/scywalker
- name: IsoQuant
  role: Downstream isoform and gene reconstruction/quantification applied to the splice-aware
    alignments
  repo: https://github.com/ablab/IsoQuant
- name: genomecomb
  role: Implementation framework providing reference directories, indexes, and distributed
    execution around the alignment step
  repo: https://github.com/derijkp/genomecomb
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae549
    title: 'Scywalker: scalable end-to-end data analysis workflow for long-read single-cell
      transcriptome sequencing'
  source_content_hash: sha256:578a5fcc4833e9a2d1a4521908723d023f4c990c29239fa84637f2f6788ca617
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/derijkp/scywalker
  tools:
  - IsoQuant
  - genomecomb
  - minimap2
  - scywalker
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# nanopore-cdna-alignment-with-minimap2-splice

## Summary

Align full-length Oxford Nanopore single-cell cDNA reads to a reference genome using minimap2 with the splice preset, the default aligner inside scywalker's end-to-end pipeline, to support isoform/gene reconstruction and per-cell quantification. Splice-aware alignment preserves exon boundaries, which is essential for IsoQuant-based transcript discovery downstream.

## When to use

Use when analyzing 10x Oxford Nanopore long-read single-cell cDNA (or PacBio with an appropriate index) and reads must be aligned to a genome for downstream isoform reconstruction and gene quantification.

## When NOT to use

- The input is short-read data (e.g. Cell Ranger/short-read UMI quantification), which should instead serve as the comparison baseline, not be aligned with minimap2 splice.
- No genome reference directory is available and only transcript-level quantification is needed — scywalker alignment is genome-based via minimap2_splice.
- The dataset's organelle chromosomes are not specified at reference-building time; the isoquant-based code can hang or crash on the very different organelle data.

## Inputs

- Oxford Nanopore full-length single-cell cDNA FASTQ files (10x 3'/5' protocol, in a sample/project fastq directory)
- Reference genome FASTA and GTF transcript annotation (or genomecomb reference directory, e.g. refdb_hg38)
- Matching minimap2 splice index for the reference genome

## Outputs

- Aligned long-read BAM/alignment files positioned on the genome with splice-aware mapping
- Downstream bulk and per-cell gene and isoform count matrices (count files with _raw and emptydrops _filtered variants) produced from the alignments

## How to apply

Build the reference directory with scywalker_makerefdir from a genome fasta and gtf (or a genomecomb refdb such as refdb_hg38), making sure to specify organelle chromosomes (e.g. chrM, chrPt) so the organelle-specific code path is used, because the isoquant-based code often hangs or crashes on organelle data. The default aligner in scywalker is minimap2_splice (minimap2 with the splice preset); when downloading a genomecomb reference, also install the matching minimap2 index files (e.g. refdb_hg38-minimap2-0.110.0.tar.gz). Alignment runs as part of the single scywalker command, which distributes work over chromosomes/regions and cores (-d), then feeds alignments to IsoQuant-based isoform reconstruction and per-cell UMI counting. For PacBio data, create the appropriate minimap2 index by passing -pacbioindex 1 to scywalker_makerefdir. Multimapping gene reads are weighed (a read mapping to N genes gives each gene 1/N).

## Related tools

- **minimap2** (Default aligner in scywalker (minimap2_splice, splice preset) used to align full-length ONT cDNA reads to the genome) — https://github.com/derijkp/scywalker
- **scywalker** (End-to-end workflow invoking minimap2_splice alignment as part of cellbarcode assignment, isoform reconstruction, and per-cell quantification) — https://github.com/derijkp/scywalker
- **IsoQuant** (Downstream isoform and gene reconstruction/quantification applied to the splice-aware alignments) — https://github.com/ablab/IsoQuant
- **genomecomb** (Implementation framework providing reference directories, indexes, and distributed execution around the alignment step) — https://github.com/derijkp/genomecomb

## Examples

```
scywalker -v 1 -d 24 -refdir refdb_hg38 -sc_expectedcells 183 -threads 6 test10x
```

## Evaluation signals

- Benchmark on scmixology2: cell-level Pearson correlation of UMI counts against Cell Ranger reaches R = 0.942 (vs 0.65 for BLAZE-FLAMES and 0.734 for wf-single-cell).
- Gene-per-cell correlation against short reads reaches R = 0.89 on the scmixology2 dataset (Supplementary Fig S3).
- spliced alignments respect exon boundaries: IsoQuant-based reconstruction succeeds without hanging, indicating organelle chromosomes were handled by the dedicated code path.
- Multimapping handling is verifiable: a read mapping to N genes contributes 1/N count to each gene in gene counts.
- Runtimes on the fixed 24-core scmixology2 run are ~3h37 for the scywalker path (vs 5h41 for wf-single-cell).

## Limitations

- The isoquant-based code often hangs or crashes on the very different organelle data, so organelle chromosomes (e.g. chrM, chrPt) must be specified in scywalker_makerefdir to enable the organelle-specific algorithm.
- scywalker_makerefdir only creates an ONT minimap2 index by default; analyzing PacBio data requires an additional index (-pacbioindex 1).
- Default chromosome grouping relies on '_' in alt/unplaced chromosome names (e.g. chr1_KI270706v1_random); genomes lacking this pattern require the -groupchromosomes option.

## Evidence

- [methods] The default aligner is minimap2_splice (minimap2 with the splice preset): "The default aligner is minimap2_splice (minimap2 with the splice preset)"
- [readme] organelle-specific code path needed because isoquant crashes: "The isoquant based code often hangs or crashes on the very different organelle data."
- [readme] organelles must be specified at reference building: "If there are organelles in the genome sequence, it is important to specify them so the organelle specific algorithm can be used"
- [readme] PacBio index option: "the appropriate index can be added by giving the 
option '-pacbioindex 1' to scywalker_makerefdir"
- [results] multimapping read weighting in gene counting: "multimapping reads are weighed (if maps to N genes -> each gets 1/N count)"
