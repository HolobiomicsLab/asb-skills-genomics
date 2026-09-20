---
name: assembly-metrics-comparison-across-assemblers
description: Use when multiple FASTQ metagenomic samples (e.g., HMP or Horizon oil-spill
  Illumina reads) are being assembled and you need to decide whether a merged contig
  set outperforms individual assembler/kmer runs on contig count, contiguity, and
  assembled bases.
when_to_use_negative:
- Total assembly size or N50/N90 is the primary metric of interest — the paper states
  these are 'not very useful for comparing metagenome assembly performance' because
  small-contig removal affects them
- Input is already a single merged or curated contig set with no individual assemblies
  to compare against
- The input dataset is so large that overlap-based consensus merging is impractical
  — Minimus2/Newbler run-times increase exponentially with increasing input dataset
  size
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: Velvet
  role: Individual Illumina de novo assembler run at small and large kmer ranges to
    produce input contig sets
- name: SOAPdenovo
  role: Individual Illumina de novo assembler run at small and large kmer ranges to
    produce input contig sets
- name: CLC Bio de novo assembler
  role: Commercial assembler used for individual Illumina assemblies compared against
    MeGAMerge
- name: IDBA
  role: Additional assembler tested for several samples to widen the comparison
- name: Ray
  role: Additional assembler tested for several samples to widen the comparison
- name: MeGAMerge
  role: Post-assembly pipeline that pools, dereplicates, bins, and merges contig sets
    from all assemblers
  repo: https://github.com/LANL-Bioinformatics/MeGAMerge
- name: Newbler
  role: Reassembles contigs below the 2.0 kb cutoff during MeGAMerge
- name: Minimus2
  role: OLC-based merging of Newbler output with >2.0 kb contigs to produce MergedContigs.fasta
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/srep06480
    title: Improved Assemblies Using a Source-Agnostic Pipeline for MetaGenomic Assembly
      by Merging (MeGAMerge) of Contigs
  source_content_hash: sha256:71e7b53195d79db53ceab61ebe052ee8d83394b54ea3d945f3c0a3133ff87e5c
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
  - CLC Bio de novo assembler
  - IDBA
  - MeGAMerge
  - Minimus2
  - Newbler
  - Ray
  - SOAPdenovo
  - Velvet
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.85
n_claims: 20
---

# assembly-metrics-comparison-across-assemblers

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark de novo metagenomic assemblies by running multiple assemblers (Velvet, SOAPdenovo, CLC Bio, IDBA, Ray) at small (21-31) and large (85-105) kmer ranges on the same trimmed reads, optionally pooling contigs through MeGAMerge, and comparing number of contigs, largest contig, and total bases in contigs >2 kb per assembly.

## When to use

Use when multiple FASTQ metagenomic samples (e.g., HMP or Horizon oil-spill Illumina reads) are being assembled and you need to decide whether a merged contig set outperforms individual assembler/kmer runs on contig count, contiguity, and assembled bases.

## When NOT to use

- Total assembly size or N50/N90 is the primary metric of interest — the paper states these are 'not very useful for comparing metagenome assembly performance' because small-contig removal affects them
- Input is already a single merged or curated contig set with no individual assemblies to compare against
- The input dataset is so large that overlap-based consensus merging is impractical — Minimus2/Newbler run-times increase exponentially with increasing input dataset size

## Inputs

- Trimmed Illumina FASTQ reads from metagenomic samples (e.g., HMP SRS022071, Horizon oil-spill reads)
- Multi-FASTA contig files from individual assembler/kmer runs (Velvet, SOAPdenovo, CLC Bio, IDBA, Ray)
- MeGAMerge runtime parameters (overlap, minID, conserr, minLen, minIncludeLen, cpu)

## Outputs

- MergedContigs.fasta per sample from MeGAMerge
- Per-assembly metrics table (number of contigs, largest contig length, total bases in contigs >2 kb) with one row per sample/assembler/kmer combination

## How to apply

Trim reads with a sliding-window protocol at a minimum quality score of 2, then run Velvet, SOAPdenovo, and the CLC Bio de novo assembler independently on each sample at small (21-31) and large (85-105) kmer ranges, adding IDBA and Ray runs where feasible. Pool all resulting contig sets and run MeGAMerge: dereplicate with the custom Perl script (removing exact duplicates, reverse complements, and subsequences), bin contigs at the 2.0 kb cutoff, reassemble contigs below 2 kb with Newbler (minLen 150, cpu 4), and merge with Minimus2 (conserr 0.06, minIncludeLen 200) to output MergedContigs.fasta. Compute per-assembly metrics — number of contigs, largest contig length, and total bases in contigs >2 kb — since the paper notes total assembly size and N50/N90 are not very useful for comparing metagenome assembly performance. Judge success by whether the MeGAMerged set produces fewer, larger contigs with total assembled bases comparable to the best single kmer assembly, and whether its largest contig exceeds the largest contig of every individual input assembly.

## Related tools

- **Velvet** (Individual Illumina de novo assembler run at small and large kmer ranges to produce input contig sets)
- **SOAPdenovo** (Individual Illumina de novo assembler run at small and large kmer ranges to produce input contig sets)
- **CLC Bio de novo assembler** (Commercial assembler used for individual Illumina assemblies compared against MeGAMerge)
- **IDBA** (Additional assembler tested for several samples to widen the comparison)
- **Ray** (Additional assembler tested for several samples to widen the comparison)
- **MeGAMerge** (Post-assembly pipeline that pools, dereplicates, bins, and merges contig sets from all assemblers) — https://github.com/LANL-Bioinformatics/MeGAMerge
- **Newbler** (Reassembles contigs below the 2.0 kb cutoff during MeGAMerge)
- **Minimus2** (OLC-based merging of Newbler output with >2.0 kb contigs to produce MergedContigs.fasta)

## Examples

```
MeGAMerge-1.0.pl -conserr=0.06 -minLen=150 -minIncludeLen=200 -cpu=4 merged_output velvet_contigs.fa soapdenovo_contigs.fa clc_contigs.fa
```

## Evaluation signals

- The metrics table contains one row per sample/assembler/kmer combination and for the MeGAMerged set
- For each sample, the largest MeGAMerge contig exceeds the largest contig of every individual input assembly
- The MeGAMerged set has fewer total contigs than any single input assembly, with larger contigs in the >2 kb bin
- Total assembled bases in the MeGAMerged set are comparable to the best single kmer assembly (no large loss of assembled sequence)
- Dereplication removed exact duplicates, reverse complements, and subsequences before binning (pool size shrinks after the Perl dereplication step)

## Limitations

- Overlap-based consensus tools' run-times increase exponentially with increasing input dataset size, limiting scalability to very large samples
- The merging process introduces new chimeras, even though MeGAMerge reduces the total number of chimeras input and removes many unmappable (incorrect) contigs
- Total assembly size and N50/N90 metrics are affected by removal of small contigs (<2 kb) and are described as not very useful for comparing metagenome assembly performance
- Improvements such as better clustering, improved overlap consensus assembly, and binning of contigs into overlapping groups were only 'currently being explored', not implemented

## Evidence

- [discussion] Dereplication of contigs is performed using a custom implemented Perl script, to remove exact duplicates, reverse compliments, and subsequences: "Dereplication of contigs is performed using a custom implemented Perl script, to remove exact duplicates, reverse compliments, and subsequences"
- [discussion] It is always the case that the largest MeGAMerge contigs contain more bases than an equal number of largest contigs from any individual input data set: "It is always the case that the largest MeGAMerge contigs contain more bases than an equal number of largest contigs from any individual input data set"
- [methods] conserr default 0.06: "-conserr=NN            Maximum conservation error for minimus2 (default 0.06)"
- [discussion] Reads were trimmed using a sliding window based trimming protocol, with a minimum quality score of 2 to remove data of poor quality: "Reads were trimmed using a sliding window based trimming protocol, with a minimum quality score of 2 to remove data of poor quality"
- [discussion] Minimus2 is a tool from the AMOS toolbox: "the sequences generated by Newbler and the original contigs greater than 2.0 kb in length are MeGAMerged using Minimus2, a tool from the AMOS toolbox1718"
- [readme] Usage line from README: "MeGAMerge-1.0.pl [options] output_directory <list of fastas>"
