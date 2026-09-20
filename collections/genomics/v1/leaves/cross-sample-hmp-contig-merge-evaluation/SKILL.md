---
name: cross-sample-hmp-contig-merge-evaluation
description: Use when you have independently assembled contig FASTA files (any number
  of input Multi-FASTA sets) from related metagenome samples and want to merge them
  via OLC-based assembly into one improved contig set and evaluate contiguity gains.
when_to_use_negative:
- The input is raw reads rather than assembled contig FASTA files — MeGAMerge is a
  post-assembly process expecting Multi-FASTA contig sets, not FASTQ reads
- The samples are unrelated or from very different environments, so contigs are unlikely
  to share the ≥98% identity overlaps (default -minID) needed for OLC-based merging
- The combined dereplicated contig pool is very large — overlap-based consensus tools'
  run-times increase exponentially with increasing input dataset size
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: MeGAMerge
  role: Perl wrapper that accepts any number of contig FASTA files, dereplicates them,
    and orchestrates Newbler and Minimus2 to produce the merged contig set
  repo: https://github.com/LANL-Bioinformatics/MeGAMerge
- name: Newbler
  role: Assembles the smaller (<2 kb) dereplicated contigs within the MeGAMerge pipeline,
    run with -cpu=4
- name: Minimus2
  role: Performs the final OLC-based merge of Newbler output with the original >2
    kb contig pool in the MeGAMerge pipeline
- name: AMOS toolbox
  role: Provides Minimus2, which has been modified to allow ambiguous bases to be
    included in the assembly
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/srep06480
    title: Improved Assemblies Using a Source-Agnostic Pipeline for MetaGenomic Assembly
      by Merging (MeGAMerge) of Contigs
  source_content_hash: sha256:225463a864718446943197ce9a3c7398464e8a526634f0e190ba6ba46299cdcd
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
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# cross-sample-hmp-contig-merge-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Apply MeGAMerge to pool and OLC-merge independently assembled contig sets from multiple related metagenome samples (e.g., HMP SRA read sets) into a single improved contig set, then compare cross-sample versus within-sample contiguity statistics to confirm the merge yields larger contigs and more bases in large contigs without data replication.

## When to use

Use when you have independently assembled contig FASTA files (any number of input Multi-FASTA sets) from related metagenome samples and want to merge them via OLC-based assembly into one improved contig set and evaluate contiguity gains.

## When NOT to use

- The input is raw reads rather than assembled contig FASTA files — MeGAMerge is a post-assembly process expecting Multi-FASTA contig sets, not FASTQ reads
- The samples are unrelated or from very different environments, so contigs are unlikely to share the ≥98% identity overlaps (default -minID) needed for OLC-based merging
- The combined dereplicated contig pool is very large — overlap-based consensus tools' run-times increase exponentially with increasing input dataset size

## Inputs

- Two or more independently assembled contig sets in Multi-FASTA format (one per related metagenome sample)
- Raw sequencing reads (e.g., HMP SRA) for each sample if re-assembly from scratch is required
- MeGAMerge Perl wrapper with Newbler (runAssembly), Minimus2 (AMOS toolbox), and MUMmer installed in $PATH or via -bindir

## Outputs

- MergedContigs.fasta — the single merged, improved contig set from the cross-sample MeGAMerge run
- assembly_comparison_metrics.csv — per-merge contiguity comparison (largest contig length, total bases in contigs >2 kb) for cross-sample and within-sample merges

## How to apply

Retrieve publicly available reads (e.g., HMP SRA) for the related samples, assemble each independently keeping contigs of at least 150 bases (MeGAMerge -minLen default) and excluding contigs under 200 bases from the minimus stage (-minIncludeLen default 200). Pool the contig sets and dereplicate with MeGAMerge's custom Perl dereplication step, which removes exact duplicates, reverse complements, and subsequences. Run MeGAMerge with Newbler (4 CPUs, -cpu=4) on the smaller contigs and Minimus2 (maximum conservation error 0.06, -conserr default) on the >2 kb pool (contigs binned at a 2.0 kb size cutoff), directing output to MergedContigs.fasta via -o. Recompute contiguity statistics — largest contig length and total bases in contigs >2 kb — for the cross-sample merge and each within-sample merge, writing the per-merge comparison to assembly_comparison_metrics.csv; judge success by the cross-sample merge matching or exceeding the best within-sample merge on both metrics.

## Related tools

- **MeGAMerge** (Perl wrapper that accepts any number of contig FASTA files, dereplicates them, and orchestrates Newbler and Minimus2 to produce the merged contig set) — https://github.com/LANL-Bioinformatics/MeGAMerge
- **Newbler** (Assembles the smaller (<2 kb) dereplicated contigs within the MeGAMerge pipeline, run with -cpu=4)
- **Minimus2** (Performs the final OLC-based merge of Newbler output with the original >2 kb contig pool in the MeGAMerge pipeline)
- **AMOS toolbox** (Provides Minimus2, which has been modified to allow ambiguous bases to be included in the assembly)

## Examples

```
perl MeGAMerge-1.0.pl -cpu=4 -conserr=0.06 -o=MergedContigs.fasta merged_output/ sampleA_contigs.fasta sampleB_contigs.fasta
```

## Evaluation signals

- MergedContigs.fasta exists in the MeGAMerge output directory after the run completes
- assembly_comparison_metrics.csv exists and contains one row per merge (cross-sample plus each within-sample merge) with largest-contig length and bases in contigs >2 kb
- The cross-sample merged contig set is no smaller in contiguity than the best within-sample merge: largest contig length and bases in contigs >2 kb should at least match, ideally exceed, within-sample values
- Dereplication removed exact duplicates, reverse complements, and subsequences, so no contig in the merged pool is a subsequence of another input contig
- The largest MeGAMerge contigs contain more total bases than an equal number of the largest contigs from any individual input contig set

## Limitations

- Overlap-based consensus tools' run-times increase exponentially with increasing input dataset size, limiting scalability to very large pooled contig sets
- The merging process introduces new chimeras, even though MeGAMerge reduces the total number of chimeras input into the process
- Removal of small contigs (<2 kb from the merge, <200 from minimus inclusion) impacts total assembly size and N50/N90 metrics, which are described as not very useful for comparing metagenome assembly performance

## Evidence
