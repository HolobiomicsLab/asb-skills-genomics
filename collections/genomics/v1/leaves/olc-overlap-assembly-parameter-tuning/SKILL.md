---
name: olc-overlap-assembly-parameter-tuning
description: Use when you have two or more Multi-FASTA files of assembled contigs
  (from different samples, assemblers, or platforms) and want to merge them into a
  single improved contig set via OLC assembly.
when_to_use_negative:
- Input is raw reads rather than already-assembled contigs — MeGAMerge is a post-assembly
  process and expects FASTA contig files, not FASTQ reads
- Only a single assembly exists and no merging is desired — a single assembler run
  is the appropriate choice instead
- Input dataset is extremely large — overlap-based consensus tools' run-times increase
  exponentially with increasing input dataset size
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: MeGAMerge
  role: Perl wrapper that pools, dereplicates, bins, and OLC-merges input contig FASTAs,
    exposing all overlap parameters at runtime
  repo: https://github.com/LANL-Bioinformatics/MeGAMerge
- name: Newbler
  role: runAssembly is invoked on the pool of contigs smaller than the 2.0 kb cutoff
- name: Minimus2
  role: OLC merger (AMOS toolbox component) applied to Newbler output combined with
    the >2.0 kb contig pool; modified to allow ambiguous bases
- name: AMOS toolbox
  role: Provides Minimus2; requires MUMmer executables (nucmer, delta-filter, show-coords)
    compiled in 64-bit mode
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/srep06480
    title: Improved Assemblies Using a Source-Agnostic Pipeline for MetaGenomic Assembly
      by Merging (MeGAMerge) of Contigs
  source_content_hash: sha256:6891bf428c215dc19f8376551db1635c0f49f8cc4edd741a31ce0ac016d99f07
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
  - MeGAMerge
  - Minimus2
  - Newbler
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# olc-overlap-assembly-parameter-tuning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Configure and run overlap-layout-consensus (OLC) based merging of pre-assembled metagenomic contigs with MeGAMerge, tuning runtime overlap parameters (overlap, minID, conserr, minLen, minIncludeLen) to produce an improved merged contig set without data replication.

## When to use

Use when you have two or more Multi-FASTA files of assembled contigs (from different samples, assemblers, or platforms) and want to merge them into a single improved contig set via OLC assembly.

## When NOT to use

- Input is raw reads rather than already-assembled contigs — MeGAMerge is a post-assembly process and expects FASTA contig files, not FASTQ reads
- Only a single assembly exists and no merging is desired — a single assembler run is the appropriate choice instead
- Input dataset is extremely large — overlap-based consensus tools' run-times increase exponentially with increasing input dataset size

## Inputs

- Any number of Multi-FASTA files containing assembled contigs of any length (e.g. contig sets from multiple related metagenome samples or from different assemblers)
- Optional runtime parameter choices: overlap, minID, conserr, cpu, minLen, minIncludeLen, single_genome flag

## Outputs

- MergedContigs.fasta (or user-named file via -o): a single improved contig set produced by OLC-based merging
- Contiguity comparison statistics (largest contig length, total bases in contigs >2 kb) for the merged set versus each input assembly

## How to apply

Assemble each input sample independently first, retaining contigs of at least 150 bases (MeGAMerge -minLen default) and excluding contigs under 200 bases from the Minimus2 stage (-minIncludeLen default 200). Pool and dereplicate the contig sets with MeGAMerge's custom Perl dereplication step, which removes exact duplicates, reverse complements, and subsequences. Bin contigs at a 2.0 kb cutoff: smaller contigs are assembled with Newbler (runAssembly, e.g. -cpu=4), and the Newbler output is combined with the >2 kb pool and merged using Minimus2 from the AMOS toolbox (requires MUMmer compiled in 64-bit mode). Set the maximum conservation error for Minimus2 via -conserr (default 0.06) and minimum overlap length/identity via -overlap (default 80) and -minID (default 98); all overlap parameters are user-declarable at runtime. Direct the final merged contigs to an output file (default MergedContigs.fasta) with -o, then judge success by comparing contiguity statistics — largest contig length, total bases in contigs >2 kb, predicted gene counts — against the individual input assemblies.

## Related tools

- **MeGAMerge** (Perl wrapper that pools, dereplicates, bins, and OLC-merges input contig FASTAs, exposing all overlap parameters at runtime) — https://github.com/LANL-Bioinformatics/MeGAMerge
- **Newbler** (runAssembly is invoked on the pool of contigs smaller than the 2.0 kb cutoff)
- **Minimus2** (OLC merger (AMOS toolbox component) applied to Newbler output combined with the >2.0 kb contig pool; modified to allow ambiguous bases)
- **AMOS toolbox** (Provides Minimus2; requires MUMmer executables (nucmer, delta-filter, show-coords) compiled in 64-bit mode)

## Examples

```
MeGAMerge-1.0.pl -conserr=0.06 -cpu=4 -o MergedContigs.fasta output_directory sampleA_contigs.fasta sampleB_contigs.fasta
```

## Evaluation signals

- MergedContigs.fasta (or -o target) exists in the output directory and is a valid Multi-FASTA file
- The merged set's largest contig length and total bases in contigs >2 kb at least match, ideally exceed, the best individual input assembly
- Largest MeGAMerge contigs contain more bases than an equal number of largest contigs from any individual input data set
- Number of predicted genes on merged contigs is increased relative to individual assemblies, with no replication of data
- Total number of dereplicated input contigs is reduced after merging while reference genome coverage is maintained

## Limitations

- Overlap-based consensus tools' run-times increase exponentially with increasing input dataset size, limiting scalability
- The merging process introduces new chimeras, although MeGAMerge reduces the total number of chimeras input into the process
- Removal of small contigs (<2 kb, <200 bp from Minimus stage) impacts total assembly size and N50/N90 metrics, which are not very useful for comparing metagenome assembly performance
- Better clustering, improved overlap consensus-based assembly, and binning of contigs into overlapping groups were only 'currently being explored', not implemented at publication time

## Evidence
