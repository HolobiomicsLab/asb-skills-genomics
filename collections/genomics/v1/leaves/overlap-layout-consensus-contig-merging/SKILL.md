---
name: overlap-layout-consensus-contig-merging
description: Use when you have pooled Multi-FASTA contig files from several assemblies
  (e.g. SOAPdenovo, Velvet, IDBA, Ray across Kmers 51-71) and need one improved contig
  set without re-sequencing data replication.
when_to_use_negative:
- Input is already a single, final contig set with no alternative assemblies to merge
  — dereplication and OLC merging would add little and may introduce new chimeras
- 'The dataset is very large: overlap-based consensus tools'' run-times increase exponentially
  with increasing input dataset size'
- The input is raw reads or FASTQ data rather than already-assembled contig FASTAs
  — MeGAMerge is a post-assembly process
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: MeGAMerge
  role: Perl wrapper that dereplicates pooled contigs, bins at 2.0 kb, invokes Newbler
    and Minimus2, and exposes overlap parameters (overlap length, % identity, conserr)
    at runtime
  repo: https://github.com/LANL-Bioinformatics/MeGAMerge
- name: Newbler
  role: runAssembly reassembles the <2.0 kb small-contig bin with -minLen=150, -minIncludeLen=200,
    -conserr=0.06
- name: Minimus2
  role: OLC-based merging of Newbler output with the >2.0 kb contig pool, at minimum
    overlap 80 bp / 98% identity
- name: AMOS toolbox
  role: Provides Minimus2; modified to allow ambiguous bases in the assembly; requires
    MUMmer (nucmer, delta-filter, show-coords) in 64-bit mode
- name: MUMmer
  role: Required dependency of AMOS/Minimus2; must be compiled in 64-bit mode for
    larger genome projects
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1038/srep06480
    title: Improved Assemblies Using a Source-Agnostic Pipeline for MetaGenomic Assembly
      by Merging (MeGAMerge) of Contigs
  source_content_hash: sha256:05e04fe818659ec8d2fcaffaa6e100ba9ea4f5b1d29401014ef6b4e317fd0631
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: cc-by-nc-sa
    url: https://github.com/LANL-Bioinformatics/MeGAMerge
  repo_url: https://github.com/LANL-Bioinformatics/MeGAMerge
  tools:
  - AMOS toolbox
  - IDBA
  - MUMmer
  - MeGAMerge
  - MetaSim
  - Minimus2
  - Newbler
  - Ray
  - SOAPdenovo
  - Velvet
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# overlap-layout-consensus-contig-merging

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Merges contigs from multiple independent metagenomic assemblies (different assemblers and/or Kmers) into an improved, non-replicated contig set via MeGAMerge, a Perl wrapper that dereplicates inputs, bins contigs at a 2.0 kb cutoff, reassembles small contigs with Newbler, and merges everything with Minimus2 (OLC-based, AMOS toolbox).

## When to use

Use when you have pooled Multi-FASTA contig files from several assemblies (e.g. SOAPdenovo, Velvet, IDBA, Ray across Kmers 51-71) and need one improved contig set without re-sequencing data replication.

## When NOT to use

- Input is already a single, final contig set with no alternative assemblies to merge — dereplication and OLC merging would add little and may introduce new chimeras
- The dataset is very large: overlap-based consensus tools' run-times increase exponentially with increasing input dataset size
- The input is raw reads or FASTQ data rather than already-assembled contig FASTAs — MeGAMerge is a post-assembly process

## Inputs

- Any number of Multi-FASTA files of assembled contigs of any length (from different assemblers and/or Kmer values)
- Optionally, single-genome assembled contigs (run with -single_genome=1)

## Outputs

- MergedContigs.fasta (final improved merged contig set written to the output directory)
- Newbler and Minimus2 intermediate assembly outputs in the output directory

## How to apply

Pool all input contig FASTA files and dereplicate them with the custom MeGAMerge Perl step, removing exact duplicates, reverse complements, and subsequences. Bin dereplicated contigs at the 2.0 kb size cutoff; assemble the small-contig bin with Newbler runAssembly (-minLen=150, -minIncludeLen=200, -conserr=0.06, -cpu=4) and combine its output with the >2.0 kb pool. Merge the combined sequences with Minimus2 from the AMOS toolbox using a minimum overlap of 80 bp at 98% identity, locating MUMmer and AMOS executables via -bindir and writing the final set with -o (default MergedContigs.fasta). Use -single_genome=1 for single-genome inputs so one Newbler pass runs with fewer excluded contigs. The rationale is that OLC merging of source-agnostic contigs produces longer contigs and more predicted genes without data replication, as validated by read mapping (BWA/SAMtools), Nucmer reference-coverage alignment, gene prediction, and similarity searches.

## Related tools

- **MeGAMerge** (Perl wrapper that dereplicates pooled contigs, bins at 2.0 kb, invokes Newbler and Minimus2, and exposes overlap parameters (overlap length, % identity, conserr) at runtime) — https://github.com/LANL-Bioinformatics/MeGAMerge
- **Newbler** (runAssembly reassembles the <2.0 kb small-contig bin with -minLen=150, -minIncludeLen=200, -conserr=0.06)
- **Minimus2** (OLC-based merging of Newbler output with the >2.0 kb contig pool, at minimum overlap 80 bp / 98% identity)
- **AMOS toolbox** (Provides Minimus2; modified to allow ambiguous bases in the assembly; requires MUMmer (nucmer, delta-filter, show-coords) in 64-bit mode)
- **MUMmer** (Required dependency of AMOS/Minimus2; must be compiled in 64-bit mode for larger genome projects)

## Examples

```
perl MeGAMerge-1.0.pl -bindir=/usr/local/amos -newblerdir=/usr/local/454/bin -o=MergedContigs.fasta merged_output assemblers/*.contigs.fa
```

## Evaluation signals

- On synthetic 1,000-genome data, merged contigs should drop from ~11.7M dereplicated inputs to under 3.7M contigs with reference coverage maintained (80.26% -> 80.25%) and unmapped contigs reduced (318 -> 114)
- The final output file MergedContigs.fasta exists in the output directory and is a valid FASTA of contigs
- The largest MeGAMerge contigs contain more bases than an equal number of largest contigs from any individual input assembly
- Chimera tally should decrease overall (10,087 -> 9,225 in the synthetic benchmark, with 6,940 carried over), though new chimeras introduced by merging are expected
- Runtime parameters echo defaults when unspecified: -overlap=80, -minID=98, -conserr=0.06, -minLen=150, -minIncludeLen=200, -cpu=4

## Limitations

- Overlap-based consensus tools' run-times increase exponentially with increasing input dataset size
- The merging process introduces new chimeras, even though the total number of input chimeras is reduced
- Removal of small contigs (<2 kb) impacts total assembly size and N50/N90 metrics, which are themselves not very useful for comparing metagenome assembly performance
- MUMmer must be compiled in 64-bit mode for larger genome projects; AMOS must find nucmer, delta-filter, and show-coords at configure time (NUCMER/DELTAFILTER/SHOWCOORDS variables)
- Improvements such as better clustering, improved overlap consensus assembly, and binning of contigs into overlapping groups were only 'currently being explored', not implemented

## Evidence
