---
name: pacbio-hifi-alignment-index-generation
description: Use when preparing a scywalker/genomecomb reference directory (via scywalker_makerefdir
  or a downloaded genomecomb refdb) for PacBio HiFi single-cell FASTQ input; without
  this step the reference only carries the default ONT minimap2 index.
when_to_use_negative:
- Input is Oxford Nanopore (10x) single-cell data — the ONT minimap2 index is built
  by default and '-pacbioindex 1' is unnecessary
- A reference directory that already contains a valid PacBio minimap2 index is being
  reused — regenerating the index is redundant
- Running the chromosome-17-limited scywalker test dataset with the example command,
  where scywalker_makerefdir is invoked without '-pacbioindex 1' because the example
  targets nanopore-style test data
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3308
tools:
- name: scywalker
  role: Provides the scywalker_makerefdir command that builds the reference directory
    and its minimap2 indexes; '-pacbioindex 1' adds the PacBio index
  repo: https://github.com/derijkp/scywalker
- name: genomecomb
  role: Underlying package in which scywalker is implemented; genomecomb reference
    directories with matching minimap2 indexes can be downloaded and reused as refdirs
  repo: https://github.com/derijkp/genomecomb/
- name: minimap2
  role: Aligner whose splice preset (minimap2_splice, used with '-preset pacbio')
    consumes the generated PacBio index during read alignment
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae549
    title: 'Scywalker: scalable end-to-end data analysis workflow for long-read single-cell
      transcriptome sequencing'
  source_content_hash: sha256:ee2d173f390a927c483801887ea8e820157e4890de0c294ab04234927900139b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/derijkp/scywalker
  tools:
  - genomecomb
  - minimap2
  - scywalker
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# pacbio-hifi-alignment-index-generation

## Summary

When analyzing PacBio HiFi single-cell data with scywalker, the minimap2 reference directory must explicitly include a PacBio index, because scywalker_makerefdir builds only the ONT minimap2 index by default. Adding '-pacbioindex 1' ensures reads align correctly with minimap2_splice under the '-preset pacbio' run.

## When to use

Use when preparing a scywalker/genomecomb reference directory (via scywalker_makerefdir or a downloaded genomecomb refdb) for PacBio HiFi single-cell FASTQ input; without this step the reference only carries the default ONT minimap2 index.

## When NOT to use

- Input is Oxford Nanopore (10x) single-cell data — the ONT minimap2 index is built by default and '-pacbioindex 1' is unnecessary
- A reference directory that already contains a valid PacBio minimap2 index is being reused — regenerating the index is redundant
- Running the chromosome-17-limited scywalker test dataset with the example command, where scywalker_makerefdir is invoked without '-pacbioindex 1' because the example targets nanopore-style test data

## Inputs

- Genome sequence multifasta file (e.g. hg38 genome FASTA, plain or compressed .gz/.zst)
- GTF transcript annotation file (or a genomecomb gene tsv file)
- Organelle chromosome list (e.g. 'chrM chrPt') if organelles are present in the genome sequence
- Optionally, a prebuilt genomecomb reference directory (e.g. refdb_hg38-0.110.0.tar.gz) with its matching minimap2 index archive

## Outputs

- scywalker/genomecomb reference directory (refdir) containing the genome, annotation, and minimap2 indexes including the PacBio index

## How to apply

Build or extend the reference directory with scywalker_makerefdir from a genome FASTA and GTF (e.g. hg38), specifying any organelle chromosomes so the organelle-specific algorithm is used, and add '-pacbioindex 1' so the PacBio-appropriate minimap2 index is created alongside the default ONT index. Alternatively, download a prebuilt genomecomb reference (e.g. refdb_hg38-0.110.0.tar.gz) and also fetch the matching minimap2 index archives, ensuring a PacBio index is present. Later, the scywalker run must use '-preset pacbio' so alignment proceeds with minimap2_splice against this index. The rationale is that the pipeline defaults to Oxford Nanopore data, so skipping this step leaves PacBio reads without a compatible index and breaks downstream isoform/gene reconstruction via IsoQuant.

## Related tools

- **scywalker** (Provides the scywalker_makerefdir command that builds the reference directory and its minimap2 indexes; '-pacbioindex 1' adds the PacBio index) — https://github.com/derijkp/scywalker
- **genomecomb** (Underlying package in which scywalker is implemented; genomecomb reference directories with matching minimap2 indexes can be downloaded and reused as refdirs) — https://github.com/derijkp/genomecomb/
- **minimap2** (Aligner whose splice preset (minimap2_splice, used with '-preset pacbio') consumes the generated PacBio index during read alignment)

## Examples

```
scywalker_makerefdir -organelles chrM -pacbioindex 1 hg38ref hg38.fa genes.gtf
```

## Evaluation signals

- The reference directory contains both the default ONT minimap2 index and the additional PacBio index after running scywalker_makerefdir with '-pacbioindex 1'
- Downstream scywalker runs on PacBio FASTQs with '-preset pacbio' align reads successfully (no missing-index failure) using minimap2_splice
- When reusing a genomecomb refdb (e.g. refdb_hg38-0.110.0), both the reference archive and the matching minimap2 index archive (e.g. refdb_hg38-minimap2-0.110.0.tar.gz) are installed and version-matched
- Organelle chromosomes (e.g. chrM) were specified at refdir creation so IsoQuant-based code does not hang or crash on organelle data
- PacBio vs short-read quantification remains highly correlated (cell-level R = 0.986, per-cell gene-count R = 0.886 on HG002), indicating the alignment index supported correct quantification

## Limitations

- By default scywalker_makerefdir only creates an ONT minimap2 index; PacBio data cannot be aligned correctly unless '-pacbioindex 1' is given
- When using downloaded genomecomb references, the matching minimap2 indexes must be downloaded and installed separately or alignment cannot proceed
- Organelles present in the genome sequence must be explicitly specified; the IsoQuant-based code often hangs or crashes on the very different organelle data
- The default chromosome grouping relies on '_' in names of alt/unplaced chromosomes; for genomes where this pattern fails, '-groupchromosomes' with regular expressions is needed

## Evidence

- [readme] By default scywalker_makerefdir will only create an ont minimap2 index for the genome.: "By default scywalker_makerefdir will only create an ont minimap2 index for the genome."
- [methods] PacBio minimap2 index creation via -pacbioindex 1: "the appropriate index can be added by giving the 
option '-pacbioindex 1' to scywalker_makerefdir"
- [readme] be sure to also download and install the matching minimap2 indexes: "When downloadinga reference, be sure to also download and install the matching minimap2 indexes, e.g."
- [intro] scywalker can analyse PacBio data with the appropriate options: "scywalker is a package designed to analyse single cell (10x) Oxford nanopore long read data (without the need for matching short read data), but it can also analyse PacBio data with the appropriate"
