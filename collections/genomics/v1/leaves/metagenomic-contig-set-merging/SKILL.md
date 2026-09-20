---
name: metagenomic-contig-set-merging
description: Use when you have multiple FASTA files of assembled contigs from different
  assemblers (e.g., Velvet, SOAPdenovo, IDBA, Ray, CLC Bio) or related metagenome
  samples and want a single improved, more contiguous merged contig set.
when_to_use_negative:
- When you only have raw sequencing reads and no existing contig assemblies — MeGAMerge
  is a post-assembly process and requires already-assembled contigs as input
- When comparing total assembly size or N50/N90 as primary metrics — the article notes
  removal of small contigs impacts these metrics and they are 'not very useful for
  comparing metagenome assembly performance'
- When the input dataset is extremely large — overlap-based consensus tools' run-times
  increase exponentially with increasing input dataset size
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: MeGAMerge
  role: Perl wrapper that orchestrates dereplication, contig binning, Newbler reassembly
    of small contigs, and Minimus2 merging of pooled contigs
  repo: https://github.com/LANL-Bioinformatics/MeGAMerge
- name: Minimus2
  role: OLC-based assembler (AMOS toolbox, modified to allow ambiguous bases) used
    to merge Newbler output with original contigs >2 kb
- name: Newbler
  role: runAssembly reassembles contigs below the 2.0 kb cutoff with configurable
    minLen and cpu settings
- name: AMOS toolbox
  role: Source of Minimus2; requires MUMmer (nucmer, delta-filter, show-coords) compiled
    in 64-bit mode
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1038/srep06480
    title: Improved Assemblies Using a Source-Agnostic Pipeline for MetaGenomic Assembly
      by Merging (MeGAMerge) of Contigs
  source_content_hash: sha256:b7e4c7b6b160b923262cd2dff96c11abaacf3cfea78d038e55437f0b8fb1d092
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
  - MeGAMerge
  - Minimus2
  - Newbler
  - Ray
  - SOAPdenovo
  - Velvet
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# metagenomic-contig-set-merging

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

MeGAMerge is a post-assembly pipeline that pools contigs from multiple metagenomic assemblies (or multiple samples), dereplicates them, bins contigs at a 2.0 kb cutoff, reassembles smaller contigs with Newbler, and merges everything with Minimus2 to produce an improved, more contiguous contig set without data replication.

## When to use

Use when you have multiple FASTA files of assembled contigs from different assemblers (e.g., Velvet, SOAPdenovo, IDBA, Ray, CLC Bio) or related metagenome samples and want a single improved, more contiguous merged contig set.

## When NOT to use

- When you only have raw sequencing reads and no existing contig assemblies — MeGAMerge is a post-assembly process and requires already-assembled contigs as input
- When comparing total assembly size or N50/N90 as primary metrics — the article notes removal of small contigs impacts these metrics and they are 'not very useful for comparing metagenome assembly performance'
- When the input dataset is extremely large — overlap-based consensus tools' run-times increase exponentially with increasing input dataset size

## Inputs

- Any number of Multi-FASTA files of assembled contigs of any length (e.g., outputs from Velvet, SOAPdenovo, CLC Bio, IDBA, Ray, Newbler runs)
- Optionally, contig sets from related metagenome samples for cross-sample merging

## Outputs

- MergedContigs.fasta (per sample or per merge) in the output directory
- Per-assembly contiguity statistics (number of contigs, largest contig length, total bases in contigs >2 kb) for comparison, e.g., assembly_comparison_metrics.csv

## How to apply

Pool all input contig sets and run MeGAMerge's custom Perl dereplication step to remove exact duplicates, reverse complements, and subsequences. Bin contigs at the 2.0 kb cutoff: those below 2 kb are reassembled with Newbler (runAssembly, minLen 150, cpu 4), then the Newbler output is combined with the pool of contigs >2 kb and merged with Minimus2 (conserr 0.06, minIncludeLen 200, overlap 80, minID 98) to write MergedContigs.fasta. The rationale is that overlap-layout-consensus merging of contigs from diverse assembly runs increases contiguity: MeGAMerge outputs contain fewer, larger contigs with more predicted genes, and the largest MeGAMerge contigs contain more bases than an equal number of the largest contigs from any individual input set. Judge success by computing per-assembly metrics (number of contigs, largest contig length, total bases in contigs >2 kb) for every input assembly and the merged set, verifying the merged set reduces contig count while matching or exceeding the best input on large-contig metrics.

## Related tools

- **MeGAMerge** (Perl wrapper that orchestrates dereplication, contig binning, Newbler reassembly of small contigs, and Minimus2 merging of pooled contigs) — https://github.com/LANL-Bioinformatics/MeGAMerge
- **Minimus2** (OLC-based assembler (AMOS toolbox, modified to allow ambiguous bases) used to merge Newbler output with original contigs >2 kb)
- **Newbler** (runAssembly reassembles contigs below the 2.0 kb cutoff with configurable minLen and cpu settings)
- **AMOS toolbox** (Source of Minimus2; requires MUMmer (nucmer, delta-filter, show-coords) compiled in 64-bit mode)

## Examples

```
perl MeGAMerge-1.0.pl -cpu=4 -conserr=0.06 -minLen=150 -minIncludeLen=200 -o=MergedContigs.fasta output_dir velvet_contigs.fa soapdenovo_contigs.fa idba_contigs.fa
```

## Evaluation signals

- MergedContigs.fasta exists in the specified output directory
- The total number of contigs in the merged set is lower than the total number of dereplicated input contigs, with total assembled bases comparable to the best single-kmer input assembly
- The largest MeGAMerge contig exceeds the largest contig of every individual input assembly, and the largest MeGAMerge contigs contain more bases than an equal number of largest contigs from any individual input set
- Metrics file contains one row per sample/assembler combination with number of contigs, largest contig length, and total bases in contigs >2 kb
- For cross-sample merges, largest-contig size and bases in contigs >2 kb at least match (ideally exceed) the best within-sample merge

## Limitations

- Overlap-based consensus tools' run-times increase exponentially with increasing input dataset size, limiting scalability
- The merging process does introduce new chimeras, although MeGAMerge reduces the total number of chimeras input and removes contigs that could not be mapped to references
- Removal of small contigs impacts total assembly size and N50/N90 metrics, so these should not be used alone to judge improvement
- Better clustering, improved overlap-consensus assembly, and binning of contigs into overlapping groups were only 'currently being explored', not implemented in this version

## Evidence
