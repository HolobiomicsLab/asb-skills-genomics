---
name: metagenomic-de-novo-assembly
description: Use when metagenomic Illumina (or 454) reads are available, multiple
  de novo assemblies (e.g. different kmer values or assemblers) have been generated,
  and improved contiguity (larger contigs, more genes) is needed beyond any single
  assembler run.
when_to_use_negative:
- When only a single assembler and kmer choice is available — there is nothing to
  dereplicate and merge, so the OLC merging step cannot improve contiguity
- When the input dataset is extremely large, since overlap-based consensus tools'
  run-times increase exponentially with increasing input dataset size
- When the analysis goal is a gene/feature table or taxonomy profile rather than an
  improved contig set — assembly metrics like N50/N90 and total assembly size are
  not very useful for comparing metagenome assembly performance in that context
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: MeGAMerge
  role: Perl wrapper that orchestrates dereplication, Newbler reassembly of small
    contigs, and Minimus2 merging of pooled contigs
  repo: https://github.com/LANL-Bioinformatics/MeGAMerge
- name: Newbler
  role: Reassembles contigs below the 2.0 kb bin from the pooled dereplicated contig
    set (runAssembly)
- name: Minimus2
  role: OLC-based merging of the Newbler output with contigs >2.0 kb to produce MergedContigs.fasta
- name: AMOS toolbox
  role: Provides Minimus2, modified to allow ambiguous bases in the assembly
- name: Velvet
  role: Individual Illumina de novo assembler contributing contigs to the merge pool
- name: SOAPdenovo
  role: Individual Illumina de novo assembler contributing contigs to the merge pool
- name: CLC Bio de novo assembler
  role: Individual Illumina de novo assembler contributing contigs to the merge pool
- name: IDBA
  role: Additional assembler tested for several samples to enlarge the merge pool
- name: Ray
  role: Additional assembler tested for several samples to enlarge the merge pool
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/srep06480
    title: Improved Assemblies Using a Source-Agnostic Pipeline for MetaGenomic Assembly
      by Merging (MeGAMerge) of Contigs
  source_content_hash: sha256:74ffa363173cf8c8335bb53cbb95a22338f0c12de18dfc369edabb144be89ebd
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
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# metagenomic-de-novo-assembly

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A post-assembly metagenomic assembly improvement strategy (MeGAMerge) that pools contigs from multiple de novo assemblers (Velvet, SOAPdenovo, CLC Bio, IDBA, Ray) and merges them via overlap-layout-consensus to produce fewer, larger contigs with more predicted genes than any single assembly.

## When to use

Use when metagenomic Illumina (or 454) reads are available, multiple de novo assemblies (e.g. different kmer values or assemblers) have been generated, and improved contiguity (larger contigs, more genes) is needed beyond any single assembler run.

## When NOT to use

- When only a single assembler and kmer choice is available — there is nothing to dereplicate and merge, so the OLC merging step cannot improve contiguity
- When the input dataset is extremely large, since overlap-based consensus tools' run-times increase exponentially with increasing input dataset size
- When the analysis goal is a gene/feature table or taxonomy profile rather than an improved contig set — assembly metrics like N50/N90 and total assembly size are not very useful for comparing metagenome assembly performance in that context

## Inputs

- FASTQ read files from metagenomic samples (e.g. HMP shotgun samples such as SRS022071, Horizon oil spill Illumina reads, 454 reads)
- Multiple Multi-FASTA contig files from individual assembler runs (Velvet, SOAPdenovo, CLC Bio, IDBA, Ray)
- Assembled 454 reads (optional, for Newbler)

## Outputs

- MergedContigs.fasta — final improved contig set per sample
- Per-assembly comparison metrics: number of contigs, largest contig length, total bases in contigs >2 kb
- Read-mapping statistics (BWA-aligned BAM parsed with SAMtools) and reference genome coverage via Nucmer alignment

## How to apply

Independently assemble trimmed reads (sliding-window quality trimming, minimum quality score 2) with multiple assemblers such as Velvet, SOAPdenovo, and the CLC Bio de novo assembler at small (21-31) and large (85-105) kmer ranges, optionally adding IDBA and Ray runs. Pool all resulting Multi-FASTA contig sets and dereplicate them with the custom Perl script to remove exact duplicates, reverse complements, and subsequences. Bin contigs at the 2.0 kb cutoff: contigs >2 kb are carried forward directly, while smaller contigs are reassembled with Newbler (minLen 150, cpu 4) and the Newbler output is combined with the >2 kb pool. Merge everything with Minimus2 from the AMOS toolbox (conserr 0.06, minIncludeLen 200, default overlap 80, minID 98) to output MergedContigs.fasta. The rationale is that overlap-based consensus merging across assemblies captures longer contigs than any individual kmer choice while removing poorly supported small contigs.

## Related tools

- **MeGAMerge** (Perl wrapper that orchestrates dereplication, Newbler reassembly of small contigs, and Minimus2 merging of pooled contigs) — https://github.com/LANL-Bioinformatics/MeGAMerge
- **Newbler** (Reassembles contigs below the 2.0 kb bin from the pooled dereplicated contig set (runAssembly))
- **Minimus2** (OLC-based merging of the Newbler output with contigs >2.0 kb to produce MergedContigs.fasta)
- **AMOS toolbox** (Provides Minimus2, modified to allow ambiguous bases in the assembly)
- **Velvet** (Individual Illumina de novo assembler contributing contigs to the merge pool)
- **SOAPdenovo** (Individual Illumina de novo assembler contributing contigs to the merge pool)
- **CLC Bio de novo assembler** (Individual Illumina de novo assembler contributing contigs to the merge pool)
- **IDBA** (Additional assembler tested for several samples to enlarge the merge pool)
- **Ray** (Additional assembler tested for several samples to enlarge the merge pool)

## Examples

```
MeGAMerge-1.0.pl -minLen=150 -minIncludeLen=200 -conserr=0.06 -cpu=4 merged_out velvet_contigs.fa soapdenovo_contigs.fa clc_contigs.fa
```

## Evaluation signals

- The largest MeGAMerge contig exceeds the largest contig of every individual input assembly for each sample
- The largest MeGAMerge contigs contain more bases than an equal number of largest contigs from any individual input data set
- Total assembled bases in contigs >2 kb are comparable to the best single kmer assembly while contig number is reduced
- The metrics CSV contains one row per sample/assembler combination and gene prediction (e.g. Prodigal on contigs >2 kb) yields more predicted genes for the merged set
- Read-mapping validation with BWA/SAMtools and reference coverage via Nucmer confirms MeGAMerge removes incorrect contigs while maintaining reference genome coverage

## Limitations

- Overlap-based consensus tools' run-times increase exponentially with increasing input dataset size, limiting scalability
- The merging process introduces new chimeras, though MeGAMerge reduces the total number of chimeras input
- Removal of very small contigs (<2 kb) impacts total assembly size and N50/N90 metrics
- Improvements such as better clustering and binning of contigs into overlapping groups were only 'currently being explored', not yet implemented
- Requires Newbler (runAssembly), AMOS/Minimus2, and MUMmer compiled in 64-bit mode to be installed and findable in $PATH or tool variables (NUCMER, DELTAFILTER, SHOWCOORDS)

## Evidence
