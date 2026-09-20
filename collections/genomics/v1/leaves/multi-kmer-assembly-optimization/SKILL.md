---
name: multi-kmer-assembly-optimization
description: Use when Illumina metagenomic reads (e.g., HMP or Horizon oil-spill samples)
  have been quality-trimmed and the goal is an improved assembly beyond what any single
  assembler/kmer run achieves, by merging multiple individual assemblies.
when_to_use_negative:
- You have a single small dataset where one assembler/kmer run already meets contiguity
  needs — overlap-based consensus tools' run-times increase exponentially with input
  dataset size
- 'You need an unchimeric assembly: the merging process introduces new chimeras even
  though MeGAMerge reduces total input chimeras'
- Total assembly size or N50/N90 are the primary metrics of interest — the removal
  of small contigs affects these, and they are described as not very useful for comparing
  metagenome assembly performance
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: MeGAMerge
  role: Perl wrapper that pools, dereplicates, and merges contigs from multiple assemblies
    via OLC assembly
  repo: https://github.com/LANL-Bioinformatics/MeGAMerge
- name: Newbler
  role: Reassembles contigs below the 2.0 kb cutoff (runAssembly, minLen 150, cpu
    4)
- name: Minimus2
  role: Merges Newbler output with >2.0 kb contigs (conserr 0.06, minIncludeLen 200);
    part of AMOS toolbox
- name: Velvet
  role: Individual Illumina assembler run at small and large kmer ranges as input
    to the merge
- name: SOAPdenovo
  role: Individual Illumina assembler run at small and large kmer ranges as input
    to the merge
- name: CLC Bio de novo assembler
  role: Individual Illumina assembler run at small and large kmer ranges as input
    to the merge
- name: IDBA
  role: Additional assembler tested for several samples to supply input contigs
- name: Ray
  role: Additional assembler tested for several samples to supply input contigs
- name: AMOS toolbox
  role: Provides Minimus2; Minimus2 was modified to allow ambiguous bases in the assembly
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/srep06480
    title: Improved Assemblies Using a Source-Agnostic Pipeline for MetaGenomic Assembly
      by Merging (MeGAMerge) of Contigs
  source_content_hash: sha256:ac840675ce9df3274309bc7175167fd76b3723a76f9b91fddeb7bf15af866b16
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
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# multi-kmer-assembly-optimization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Optimizes metagenomic de novo assembly by running multiple assemblers (Velvet, SOAPdenovo, CLC Bio, IDBA, Ray) at both small (21-31) and large (85-105) kmer ranges, then merging all contig sets with MeGAMerge — a post-assembly OLC-based pipeline using Newbler and Minimus2 — to yield fewer, larger, more contiguous contigs.

## When to use

Use when Illumina metagenomic reads (e.g., HMP or Horizon oil-spill samples) have been quality-trimmed and the goal is an improved assembly beyond what any single assembler/kmer run achieves, by merging multiple individual assemblies.

## When NOT to use

- You have a single small dataset where one assembler/kmer run already meets contiguity needs — overlap-based consensus tools' run-times increase exponentially with input dataset size
- You need an unchimeric assembly: the merging process introduces new chimeras even though MeGAMerge reduces total input chimeras
- Total assembly size or N50/N90 are the primary metrics of interest — the removal of small contigs affects these, and they are described as not very useful for comparing metagenome assembly performance

## Inputs

- Quality-trimmed Illumina metagenomic reads (FASTQ), e.g. HMP SRS022071 or Horizon oil-spill samples
- Individual assembly contig sets (Multi-FASTA) from Velvet, SOAPdenovo, CLC Bio, IDBA, and/or Ray across small (21-31) and large (85-105) kmer ranges

## Outputs

- MergedContigs.fasta — the final MeGAMerged contig set per sample
- Per-assembly metrics: number of contigs, largest contig length, and total bases in contigs >2 kb for each assembler/kmer run and the merged set

## How to apply

Trim reads with a sliding-window protocol at a minimum quality score of 2, then run Velvet, SOAPdenovo, and the CLC Bio de novo assembler independently at small (21-31) and large (85-105) kmer values, optionally adding IDBA and Ray runs. Pool all resulting contig FASTA files and dereplicate them with MeGAMerge's custom Perl script, which removes exact duplicates, reverse complements, and subsequences. Bin contigs at the 2.0 kb cutoff: contigs <2 kb are reassembled with Newbler (minLen 150, cpu 4), and the Newbler output plus the >2.0 kb contigs are merged with Minimus2 (conserr 0.06, minIncludeLen 200) to produce MergedContigs.fasta. Rationale is that different kmers and assemblers capture different regions, so OLC-based merging of the pooled assemblies outperforms any single run; larger kmers and longer reads can dramatically affect the merged contigs.

## Related tools

- **MeGAMerge** (Perl wrapper that pools, dereplicates, and merges contigs from multiple assemblies via OLC assembly) — https://github.com/LANL-Bioinformatics/MeGAMerge
- **Newbler** (Reassembles contigs below the 2.0 kb cutoff (runAssembly, minLen 150, cpu 4))
- **Minimus2** (Merges Newbler output with >2.0 kb contigs (conserr 0.06, minIncludeLen 200); part of AMOS toolbox)
- **Velvet** (Individual Illumina assembler run at small and large kmer ranges as input to the merge)
- **SOAPdenovo** (Individual Illumina assembler run at small and large kmer ranges as input to the merge)
- **CLC Bio de novo assembler** (Individual Illumina assembler run at small and large kmer ranges as input to the merge)
- **IDBA** (Additional assembler tested for several samples to supply input contigs)
- **Ray** (Additional assembler tested for several samples to supply input contigs)
- **AMOS toolbox** (Provides Minimus2; Minimus2 was modified to allow ambiguous bases in the assembly)

## Examples

```
MeGAMerge-1.0.pl -conserr=0.06 -minLen=150 -minIncludeLen=200 -cpu=4 outdir velvet_k21.fa velvet_k31.fa soapdenovo_k85.fa clc_k105.fa idba.fa ray.fa
```

## Evaluation signals

- The largest MeGAMerge contig exceeds the largest contig of every individual input assembly for each sample
- The merged set contains fewer total contigs than the dereplicated input pool while total assembled bases are comparable to the best single-kmer assembly
- Total bases in large (>2 kb) contigs show greater contiguity in the MeGAMerged set than in any individual assembler/kmer run
- Reads map back to the final contigs (e.g., BWA mapping parsed with SAMtools) and reference genome coverage is maintained (Nucmer alignment) on simulated/reference data
- The output_directory contains MergedContigs.fasta as declared by the -o option (default name)

## Limitations

- Overlap-based consensus tools' run-times increase exponentially with increasing input dataset size
- While MeGAMerge reduces the total number of input chimeras, the merging process introduces new chimeras
- Removal of small contigs (<2 kb) impacts total assembly size and N50/N90 metrics
- MUMmer must be compiled in 64-bit mode for larger genome projects, and AMOS must be configured to locate nucmer, delta-filter, and show-coords

## Evidence
