---
name: genomic-region-based-job-distribution
description: Use when running scywalker's align-and-reconstruct stages on whole genomes
  where splitting work over chromosomes or sub-chromosomal regions is needed to parallelise
  and bound memory.
when_to_use_negative:
- Small or test references with no alt/unplaced contigs (e.g. the scywalker_test chromosome-17
  dataset built with scywalker_makerefdir -organelles '') — default grouping already
  suffices and region splitting is unnecessary
- 'Organelle chromosomes (e.g. chrM, chrPt): they must be excluded from the IsoQuant-based
  region processing via the refdir organelles configuration rather than distributed
  like standard chromosomes'
- Data that is not coordinate-anchored genomic read data (e.g. already demultiplexed
  count tables) — region distribution applies to alignment/transcript reconstruction,
  not downstream counting
edam_operation: http://edamontology.org/operation_3840
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: scywalker
  role: End-to-end workflow whose align-and-reconstruct stages are distributed over
    genomic regions (distrreg default g5000000)
  repo: https://github.com/derijkp/scywalker
- name: scywalker_makerefdir
  role: Builds the reference directory and controls chromosome grouping via -groupchromosomes
    for region distribution
  repo: https://github.com/derijkp/scywalker
- name: minimap2
  role: Default aligner (minimap2_splice, splice preset) run per distributed genomic
    region
- name: IsoQuant
  role: Per-region transcript reconstruction backend; runs only on non-organelle regions
  repo: https://github.com/ablab/IsoQuant
- name: genomecomb
  role: Framework in which scywalker and its region-distribution logic are implemented
  repo: https://github.com/derijkp/genomecomb
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae549
    title: 'Scywalker: scalable end-to-end data analysis workflow for long-read single-cell
      transcriptome sequencing'
  source_content_hash: sha256:786df33a2d5567a70040e48c9e6f9f599ab1e6bc236733fc3ea9180b11e42ecb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/derijkp/scywalker
  tools:
  - IsoQuant
  - genomecomb
  - minimap2
  - scywalker
  - scywalker_makerefdir
  edam_operation: http://edamontology.org/operation_3840
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# genomic-region-based-job-distribution

## Summary

scywalker distributes alignment (minimap2_splice) and IsoQuant-based transcript reconstruction over genomic regions (~5 Mb target, default distrreg g5000000) and groups small alt/unplaced chromosomes, cutting peak memory and enabling server/cluster parallelisation.

## When to use

Use when running scywalker's align-and-reconstruct stages on whole genomes where splitting work over chromosomes or sub-chromosomal regions is needed to parallelise and bound memory.

## When NOT to use

- Small or test references with no alt/unplaced contigs (e.g. the scywalker_test chromosome-17 dataset built with scywalker_makerefdir -organelles '') — default grouping already suffices and region splitting is unnecessary
- Organelle chromosomes (e.g. chrM, chrPt): they must be excluded from the IsoQuant-based region processing via the refdir organelles configuration rather than distributed like standard chromosomes
- Data that is not coordinate-anchored genomic read data (e.g. already demultiplexed count tables) — region distribution applies to alignment/transcript reconstruction, not downstream counting

## Inputs

- Reference directory (refdir) built by scywalker_makerefdir from a genome multifasta (e.g. genome.fa) and a transcripts GTF (e.g. genes.gtf)
- scywalker project or sample directory containing a fastq/ subdirectory with the input FASTQ files
- Matching minimap2 indexes for the refdir (ONT by default; -pacbioindex 1 for PacBio)
- Optional -groupchromosomes regular-expression list for non-standard chromosome naming

## Outputs

- Region-partitioned alignment and transcript reconstruction jobs (default distrreg g5000000, ~5 Mb targets)
- Merged transcript models carrying unique location-based transcript IDs across region boundaries
- Adapted read_assignment table (ambiguity, gambiguity, umicount, covered_pct, polyA, completeness) and per-droplet isoform/gene count tables (_raw and _filtered)

## How to apply

Prepare the reference with scywalker_makerefdir; by default alt and unplaced chromosomes (usually small, often carrying a "_") are grouped together, and for genomes where this pattern fails, pass the -groupchromosomes option with a list of regular expressions that name each group. During analysis, alignment with minimap2_splice (the scywalker default, splice preset) and transcript reconstruction with IsoQuant are distributed over regions with default distrreg g5000000 (~5 Mb target), with splits only placed in gene-free regions so a split never cuts a transcript. Organelle chromosomes must be declared via the refdir organelles configuration and handled separately, since the isoquant based code often hangs or crashes on the very different organelle data. Finally, per-region transcript models are merged using unique location-based transcript IDs so downstream read assignment and counting operate on a single coherent model set.

## Related tools

- **scywalker** (End-to-end workflow whose align-and-reconstruct stages are distributed over genomic regions (distrreg default g5000000)) — https://github.com/derijkp/scywalker
- **scywalker_makerefdir** (Builds the reference directory and controls chromosome grouping via -groupchromosomes for region distribution) — https://github.com/derijkp/scywalker
- **minimap2** (Default aligner (minimap2_splice, splice preset) run per distributed genomic region)
- **IsoQuant** (Per-region transcript reconstruction backend; runs only on non-organelle regions) — https://github.com/ablab/IsoQuant
- **genomecomb** (Framework in which scywalker and its region-distribution logic are implemented) — https://github.com/derijkp/genomecomb

## Examples

```
scywalker -v 1 -d 8 -dmaxmem 64G -refdir g17 -sc_expectedcells 183 -cellmarkerfile markers_chr17.tsv -threads 6 test10x
```

## Evaluation signals

- Region splits fall only in gene-free stretches (default 250kbase gene-free regions) so no transcript model is severed across two regions
- Default distrreg g5000000 yields ~5 Mb target regions; per-region memory use stays within the requested -dmaxmem (e.g. 64G)
- Alt/unplaced chromosomes are grouped under the expected prefix (e.g. chr1_KI270706v1_random under chr1_) or per the supplied -groupchromosomes regex list
- Merged transcript models carry unique location-based transcript IDs with no duplicates from per-region reconstruction
- Organelle chromosomes are absent from the IsoQuant-based per-region outputs (handled via refdir organelles configuration)

## Limitations

- The default grouping of alt/unplaced chromosomes relies on names containing "_" (e.g. chr1_KI270706v1_random); genomes without this pattern need an explicit -groupchromosomes list
- The isoquant based code often hangs or crashes on very different organelle data, so organelle chromosomes must be configured separately in the refdir
- Splits are constrained to gene-free regions, which can limit parallelism for gene-dense genomes

## Evidence

- [readme] When distributing over chromosomes (or regions), by default alt and unplaced chromosomes (usually small) are grouped together.: "When distributing over chromosomes (or regions), by default alt and
unplaced chromosomes (usually small) are grouped together."
- [readme] The scywalker_makerefdir command supports -groupchromosomes to specify a different way of grouping via regular expressions.: "the scywalker_makerefdir
command supports the option `-groupchromosomes` to specify a different way
of grouping: Each chromosome is matched to the given list of regular
expressions"
- [readme] The isoquant based code often hangs or crashes on organelle data, requiring the organelle-specific algorithm.: "If there are organelles in the genome sequence, it is important to specify them so the organelle
specific algorithm can be used: The isoquant based code often hangs or crashes on the
very different"
- [methods] The default aligner is minimap2_splice (minimap2 with the splice preset).: "The default aligner is minimap2_splice (minimap2 with the splice preset)"
