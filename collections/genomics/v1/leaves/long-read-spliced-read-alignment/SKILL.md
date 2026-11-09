---
name: long-read-spliced-read-alignment
description: Use when starting from cellbarcoded long-read FASTQ (10x ONT, or PacBio
  with appropriate options) and a scywalker/genomecomb reference directory, needing
  spliced genome alignments before isoform reconstruction and per-cell counting.
when_to_use_negative:
- Input is matched short-read data rather than long reads — scywalker is designed
  for long-read data without the need for matching short reads
- Data are Illumina-style short reads or the goal is small-RNA/unaligned fragment
  analysis rather than spliced cDNA alignment
- No reference directory exists and genome/transcripts.gtf cannot be provided — alignment
  requires a prebuilt refdir with minimap2 indexes
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
tools:
- name: minimap2
  role: Default aligner used in spliced mode (minimap2_splice) to map long reads to
    the genome; minimap2_splicesmall available experimentally for small-exon detection
  repo: https://github.com/derijkp/scywalker
- name: scywalker
  role: End-to-end pipeline that invokes the spliced alignment on barcoded FASTQ within
    region-distributed analysis
  repo: https://github.com/derijkp/scywalker
- name: scywalker_makerefdir
  role: Builds the reference directory with ONT (default) or PacBio (-pacbioindex
    1) minimap2 indexes from fasta + gtf
  repo: https://github.com/derijkp/scywalker
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae549
    title: 'Scywalker: scalable end-to-end data analysis workflow for long-read single-cell
      transcriptome sequencing'
  source_content_hash: sha256:dd812a7036477ee823fd5c28e2f34c39845e8476f79e15bfdab5d9181f5e1fba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/derijkp/scywalker
  tools:
  - genomecomb
  - minimap2
  - scywalker
  - scywalker_makerefdir
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# long-read-spliced-read-alignment

## Summary

Aligns droplet-barcoded long-read (Oxford Nanopore or PacBio) FASTQ reads to a reference genome in spliced mode with minimap2 as the first analysis step of scywalker, producing the alignments that downstream IsoQuant transcript reconstruction and counting consume.

## When to use

Use when starting from cellbarcoded long-read FASTQ (10x ONT, or PacBio with appropriate options) and a scywalker/genomecomb reference directory, needing spliced genome alignments before isoform reconstruction and per-cell counting.

## When NOT to use

- Input is matched short-read data rather than long reads — scywalker is designed for long-read data without the need for matching short reads
- Data are Illumina-style short reads or the goal is small-RNA/unaligned fragment analysis rather than spliced cDNA alignment
- No reference directory exists and genome/transcripts.gtf cannot be provided — alignment requires a prebuilt refdir with minimap2 indexes

## Inputs

- Cellbarcoded long-read FASTQ files in a sample directory fastq/ subdirectory (or project directory with samplesheet)
- scywalker reference directory (refdir) built by scywalker_makerefdir from genome fasta + transcripts.gtf, with minimap2 index
- Optional 10x barcode whitelist (e.g. 737K-august-2016.txt.gz)

## Outputs

- Barcoded spliced alignments of long reads to the reference genome (BAM consumed by IsoQuant reconstruction)
- Downstream read_assignment table and per-droplet isoform/gene count tables derived from the alignments

## How to apply

Build a reference directory with scywalker_makerefdir from a genome fasta and transcripts.gtf, which creates the ONT minimap2 index by default (add -pacbioindex 1 for PacBio data); specify organelle chromosomes so they are excluded, since IsoQuant-based code hangs on organelle data. Run scywalker on the sample/project directory with the default aligner minimap2_splice (minimap2 with the splice preset); use minimap2_splicesmall only experimentally for small-exon detection. Distribute alignment and transcript reconstruction over genomic regions with the default distrreg setting g5000000 (~5 Mb regions, splits only in 250kbase gene-free regions), using multiple threads/cores (e.g. -d 8, -threads 6, -dmaxmem 64G). The resulting barcoded alignments feed the region-wise IsoQuant transcript reconstruction, merging, read assignment, and per-droplet gene/isoform count tables.

## Related tools

- **minimap2** (Default aligner used in spliced mode (minimap2_splice) to map long reads to the genome; minimap2_splicesmall available experimentally for small-exon detection) — https://github.com/derijkp/scywalker
- **scywalker** (End-to-end pipeline that invokes the spliced alignment on barcoded FASTQ within region-distributed analysis) — https://github.com/derijkp/scywalker
- **scywalker_makerefdir** (Builds the reference directory with ONT (default) or PacBio (-pacbioindex 1) minimap2 indexes from fasta + gtf) — https://github.com/derijkp/scywalker

## Examples

```
scywalker -v 1 -d 8 -dmaxmem 64G -refdir g17 -threads 6 -sc_expectedcells 183 test10x
```

## Evaluation signals

- Confirm alignments derive from minimap2_splice (the scywalker default) rather than an unintended preset
- Check the refdir contains the matching minimap2 indexes (ONT default; PacBio only if -pacbioindex 1 was set)
- Confirm organelle chromosomes (e.g. chrM chrPt) were excluded from the region-distributed alignment/reconstruction, since IsoQuant hangs on organelle data
- Verify downstream outputs exist: read_assignment table with ambiguity, gambiguity, umicount, covered_pct, polyA, completeness fields
- On the limited scywalker_test chr17 dataset, confirm the run completes with -sc_expectedcells 183 and produces both _raw and _filtered count files

## Limitations

- IsoQuant-based code often hangs or crashes on organelle data, so organelle chromosomes must be flagged at refdir creation and excluded
- PacBio analysis requires the additional PacBio minimap2 index (-pacbioindex 1); by default only the ONT index is built
- Chromosome grouping of alt/unplaced contigs relies on names containing '_' (e.g. chr1_KI270706v1_random); use -groupchromosomes for genomes where this pattern fails
- minimap2_splicesmall is for experimental small-exon detection only, not the default

## Evidence

- [methods] Default aligner is minimap2 splice preset: "The default aligner is minimap2_splice (minimap2 with the splice preset)"
- [intro] Scywalker end-to-end from fastqs: [evidence span withheld — anchor status: not_found]
- [readme] makerefdir builds reference from fasta and gtf: "You can use the included command scywalker_makerefdir to make a reference
directory starting from a fasta and a gtf transcript file."
- [readme] PacBio index option: "the appropriate index can be added by giving the 
option '-pacbioindex 1' to scywalker_makerefdir"
- [readme] IsoQuant hangs on organelle data: "The isoquant based code often hangs or crashes on the
very different organelle data."
- [readme] Test run invocation with threads and memory: "scywalker -v 1 -d 8 \
	-dmaxmem 64G \
	-refdir g17 \
	-sc_expectedcells 183"
