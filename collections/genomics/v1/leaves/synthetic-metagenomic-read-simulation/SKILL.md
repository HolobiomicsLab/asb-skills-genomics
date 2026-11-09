---
name: synthetic-metagenomic-read-simulation
description: Use when a controlled benchmark with known reference genomes is required
  to quantify how a merge/assembly pipeline changes contig counts, unmapped contigs,
  reference genome coverage, and chimeras.
when_to_use_negative:
- The sample is real (not synthetically derived) metagenomic data with no reference
  genomes, so unmapped-contig and reference-coverage validation is impossible
- The goal is to compare read-level QC protocols rather than post-assembly merging,
  since MeGAMerge is a post-assembly process operating on already-assembled contigs
- The input is a single assembly whose contigs should not be merged; use the -single_genome=1
  mode only when merging long reads/single-genome contigs, not multi-assembler pooling
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0196
tools:
- name: MetaSim
  role: Generates synthetic shotgun reads using a customized Illumina error model
    with in-house per-base quality assignments
- name: SOAPdenovo
  role: De novo assembler of trimmed simulated reads for individual Illumina assemblies
    across Kmers 51-71
- name: Velvet
  role: De novo assembler of trimmed simulated reads for individual Illumina assemblies
- name: IDBA
  role: De novo assembler of trimmed simulated reads tested for several samples
- name: Ray
  role: De novo assembler of trimmed simulated reads tested for several samples
- name: Newbler
  role: Reassembles contigs smaller than the 2.0 kb cutoff within the MeGAMerge pipeline
- name: MeGAMerge
  role: Perl wrapper that dereplicates, bins at 2.0 kb, reassembles small contigs
    with Newbler, and merges with Minimus2
  repo: https://github.com/LANL-Bioinformatics/MeGAMerge
- name: Minimus2
  role: OLC-based merger from the AMOS toolbox used to MeGAMerge Newbler output with
    contigs >2.0 kb
  repo: https://github.com/LANL-Bioinformatics/MeGAMerge
- name: BWA
  role: Maps simulated reads back to final MeGAMerged contigs for validation statistics
- name: SAMtools
  role: Parses the BWA BAM to count unmapped contigs
- name: Nucmer
  role: Aligns contigs to the 1,000 reference genomes to obtain reference genome coverage
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/srep06480
    title: Improved Assemblies Using a Source-Agnostic Pipeline for MetaGenomic Assembly
      by Merging (MeGAMerge) of Contigs
  source_content_hash: sha256:f00220323bf17c9c37e596e468878f023f44356a421d4e806a8e20c499db543a
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
  - BWA
  - IDBA
  - MeGAMerge
  - MetaSim
  - Minimus2
  - Newbler
  - Nucmer
  - Ray
  - SAMtools
  - SOAPdenovo
  - Velvet
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# Synthetic metagenomic read simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates a ground-truth synthetic metagenome by simulating shotgun reads (MetaSim, customized Illumina error model with in-house per-base quality assignments) from 1,000 known reference genomes, then trims, assembles per-Kmer (51–71), merges contigs with MeGAMerge, and validates via read mapping (BWA/SAMtools) and reference alignment (Nucmer).

## When to use

Use when a controlled benchmark with known reference genomes is required to quantify how a merge/assembly pipeline changes contig counts, unmapped contigs, reference genome coverage, and chimeras.

## When NOT to use

- The sample is real (not synthetically derived) metagenomic data with no reference genomes, so unmapped-contig and reference-coverage validation is impossible
- The goal is to compare read-level QC protocols rather than post-assembly merging, since MeGAMerge is a post-assembly process operating on already-assembled contigs
- The input is a single assembly whose contigs should not be merged; use the -single_genome=1 mode only when merging long reads/single-genome contigs, not multi-assembler pooling

## Inputs

- 1,000 randomly selected reference genomes (Supplementary Table S8)
- FASTQ simulated shotgun reads from MetaSim (volume equivalent to one Illumina HiSeq 2000 lane)
- Multi-FASTA contig files from SOAPdenovo, Velvet, IDBA, Ray (Kmers 51–71), and optionally CLC Bio

## Outputs

- MergedContigs.fasta (final MeGAMerged contig set)
- BAM of reads mapped to final contigs (BWA) with unmapped-contig counts (SAMtools)
- Reference genome coverage per genome from Nucmer contig-to-reference alignments
- Chimera tallies before merging, after merging, and carried over from dereplicated inputs
- validation_summary.csv (totals plus 1,000 per-genome coverage rows)

## How to apply

Simulate shotgun reads with MetaSim using a customized Illumina error model with in-house per-base quality assignments for the 1,000 randomly selected genomes (Supplementary Table S8), producing read volume equivalent to a single Illumina HiSeq 2000 lane. Trim reads with a sliding-window protocol at minimum quality score 2, then de novo assemble independently with SOAPdenovo, Velvet, IDBA, and Ray across Kmers 51–71. Pool all contigs, dereplicate with the custom Perl script (remove exact duplicates, reverse complements, subsequences), bin at the 2.0 kb cutoff, reassemble smaller contigs with Newbler (-minLen=150, -minIncludeLen=200, -conserr=0.06), and merge via Minimus2 in MeGAMerge writing the final set with -o (default MergedContigs.fasta). Map reads back with BWA and parse the BAM with SAMtools to count unmapped contigs, align contigs to references with Nucmer for coverage, and flag contigs not contiguously aligning to a single reference genome as chimeric; record everything in validation_summary.csv.

## Related tools

- **MetaSim** (Generates synthetic shotgun reads using a customized Illumina error model with in-house per-base quality assignments)
- **SOAPdenovo** (De novo assembler of trimmed simulated reads for individual Illumina assemblies across Kmers 51-71)
- **Velvet** (De novo assembler of trimmed simulated reads for individual Illumina assemblies)
- **IDBA** (De novo assembler of trimmed simulated reads tested for several samples)
- **Ray** (De novo assembler of trimmed simulated reads tested for several samples)
- **Newbler** (Reassembles contigs smaller than the 2.0 kb cutoff within the MeGAMerge pipeline)
- **MeGAMerge** (Perl wrapper that dereplicates, bins at 2.0 kb, reassembles small contigs with Newbler, and merges with Minimus2) — https://github.com/LANL-Bioinformatics/MeGAMerge
- **Minimus2** (OLC-based merger from the AMOS toolbox used to MeGAMerge Newbler output with contigs >2.0 kb) — https://github.com/LANL-Bioinformatics/MeGAMerge
- **BWA** (Maps simulated reads back to final MeGAMerged contigs for validation statistics)
- **SAMtools** (Parses the BWA BAM to count unmapped contigs)
- **Nucmer** (Aligns contigs to the 1,000 reference genomes to obtain reference genome coverage)

## Evaluation signals

- Merged contig count drops below 3.7 million from 11.7 million dereplicated input contigs
- Unmapped contigs decrease from 318 (inputs) to 114 (merged set)
- Reference genome coverage is maintained: 80.26% (inputs) vs 80.25% (merged)
- Total chimeras drop from 10,087 to 9,225, with 6,940 carried over from the dereplicated inputs
- validation_summary.csv contains value_in_range totals and exactly 1,000 per-genome coverage rows (row_count_equals)

## Limitations

- Overlap-based consensus tools' run-times increase exponentially with increasing input dataset size, limiting scalability of the Minimus2/Newbler merge
- The merging process introduces new chimeras even though MeGAMerge reduces the total number of chimeras input
- Removing very small contigs (<2 kb) impacts total assembly size and N50/N90 metrics, which are themselves not very useful for comparing metagenome assembly performance

## Evidence
