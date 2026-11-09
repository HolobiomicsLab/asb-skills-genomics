---
name: multi-assembler-contig-generation
description: Use when you have a metagenomic shotgun read set (e.g. Illumina FASTQ,
  trimmed) and want to maximize contig diversity and quality by assembling with several
  assemblers/Kmers (51–71) before post-assembly merging.
when_to_use_negative:
- You only have a single assembler run and no additional assemblies to merge — the
  OLC-based merge has little to combine and overlap-based consensus tools' run-times
  increase exponentially with input dataset size
- Input is already assembled contigs with no underlying reads available for independent
  re-assembly (in that case run MeGAMerge directly on the existing Multi-FASTA files
  instead)
- Total assembly size and N50/N90 are the primary metrics of interest — removing contigs
  < 2.0 kb explicitly trades these metrics for contig quality
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3174
tools:
- name: MeGAMerge
  role: Perl wrapper orchestrating pooling, dereplication, binning, Newbler re-assembly
    and Minimus2 merging of contigs
  repo: https://github.com/LANL-Bioinformatics/MeGAMerge
- name: SOAPdenovo
  role: Individual de novo Illumina assembler contributing contigs to the pool
- name: Velvet
  role: Individual de novo Illumina assembler contributing contigs to the pool
- name: IDBA
  role: Additional assembler tested for several samples, contributing contigs to the
    pool
- name: Ray
  role: Additional assembler tested for several samples, contributing contigs to the
    pool
- name: CLC Bio de novo assembler
  role: Commercial alternative assembler for individual Illumina assemblies
- name: Newbler
  role: Re-assembles smaller contigs (< 2.0 kb) before merging, run via runAssembly
- name: Minimus2
  role: OLC-based merger (modified to include ambiguous bases) of Newbler output and
    > 2.0 kb contigs, from the AMOS toolbox
- name: AMOS toolbox
  role: Provides Minimus2 used for the final merge; requires MUMmer
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/srep06480
    title: Improved Assemblies Using a Source-Agnostic Pipeline for MetaGenomic Assembly
      by Merging (MeGAMerge) of Contigs
  source_content_hash: sha256:31791e98763a923adfe34ce01fb50c70ad31ebdbe178b8c07e38974856390628
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
  - MetaSim
  - Minimus2
  - Newbler
  - Ray
  - SOAPdenovo
  - Velvet
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# multi-assembler-contig-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generate a diverse pool of de novo contigs from the same metagenomic read set by running multiple assemblers (SOAPdenovo, Velvet, IDBA, Ray, and optionally CLC Bio) independently, then pooling and dereplicating the results so they can be merged into an improved consensus set via the MeGAMerge OLC-based pipeline.

## When to use

Use when you have a metagenomic shotgun read set (e.g. Illumina FASTQ, trimmed) and want to maximize contig diversity and quality by assembling with several assemblers/Kmers (51–71) before post-assembly merging.

## When NOT to use

- You only have a single assembler run and no additional assemblies to merge — the OLC-based merge has little to combine and overlap-based consensus tools' run-times increase exponentially with input dataset size
- Input is already assembled contigs with no underlying reads available for independent re-assembly (in that case run MeGAMerge directly on the existing Multi-FASTA files instead)
- Total assembly size and N50/N90 are the primary metrics of interest — removing contigs < 2.0 kb explicitly trades these metrics for contig quality

## Inputs

- Trimmed metagenomic shotgun reads (e.g. Illumina FASTQ from a HiSeq 2000 lane)
- Multiple assembler configurations: SOAPdenovo, Velvet, IDBA, Ray (and optionally CLC Bio de novo assembler) across Kmers 51–71
- Newbler runAssembly executable and Minimus2 from the AMOS toolbox with MUMmer (nucmer, delta-filter, show-coords) in $PATH or via -bindir

## Outputs

- Pooled, dereplicated Multi-FASTA contig set from all individual assemblies
- Newbler-assembled output for contigs < 2.0 kb combined with the >2.0 kb contig pool
- Final merged contig set (default MergedContigs.fasta) written to the output_directory

## How to apply

Trim reads with a sliding-window protocol at minimum quality score 2, then assemble the trimmed reads de novo and independently with SOAPdenovo, Velvet, IDBA, and Ray across Kmers 51–71 (CLC Bio de novo assembler is the commercial alternative). Pool all generated contigs in Multi-FASTA format and dereplicate with the custom Perl script that removes exact duplicates, reverse complements, and subsequences. Bin the dereplicated contigs at a 2.0 kb cutoff: very small contigs are removed, smaller contigs are reassembled with Newbler (-minLen=150, -minIncludeLen=200, -conserr=0.06), and the Newbler output plus the >2.0 kb pool are merged with Minimus2 (from the AMOS toolbox, requiring MUMmer compiled in 64-bit mode), producing the final contig set with the -o option (default MergedContigs.fasta). The rationale is that MeGAMerge is source-agnostic and accepts any number of FASTA files, so more independent assemblies yield a richer OLC graph and larger merged contigs with more predicted genes without replication of data.

## Related tools

- **MeGAMerge** (Perl wrapper orchestrating pooling, dereplication, binning, Newbler re-assembly and Minimus2 merging of contigs) — https://github.com/LANL-Bioinformatics/MeGAMerge
- **SOAPdenovo** (Individual de novo Illumina assembler contributing contigs to the pool)
- **Velvet** (Individual de novo Illumina assembler contributing contigs to the pool)
- **IDBA** (Additional assembler tested for several samples, contributing contigs to the pool)
- **Ray** (Additional assembler tested for several samples, contributing contigs to the pool)
- **CLC Bio de novo assembler** (Commercial alternative assembler for individual Illumina assemblies)
- **Newbler** (Re-assembles smaller contigs (< 2.0 kb) before merging, run via runAssembly)
- **Minimus2** (OLC-based merger (modified to include ambiguous bases) of Newbler output and > 2.0 kb contigs, from the AMOS toolbox)
- **AMOS toolbox** (Provides Minimus2 used for the final merge; requires MUMmer)

## Examples

```
MeGAMerge-1.0.pl -overlap=80 -minID=98 -conserr=0.06 -minLen=150 -minIncludeLen=200 -cpu=4 -o MergedContigs.fasta output_dir soapdenovo.fa velvet.fa idba.fa ray.fa
```

## Evaluation signals

- Number of contigs: merged set should be substantially smaller than the pooled dereplicated input (e.g. synthetic 1,000-genome benchmark: 11.7M dereplicated contigs reduced to under 3.7M)
- Unmapped contigs decrease after merging (318 input vs 114 output in the 1,000-genome benchmark, quantified by BWA read mapping parsed with SAMtools)
- Reference genome coverage is maintained (80.26% input vs 80.25% output, measured by Nucmer alignment to references)
- Largest MeGAMerge contigs contain more bases than an equal number of largest contigs from any individual input assembly
- Chimera count falls overall (10,087 to 9,225 with 6,940 carried over), flagging contigs that did not contiguously align to a single reference genome

## Limitations

- Overlap-based consensus tools' run-times increase exponentially with increasing input dataset size, limiting scalability
- The merging process introduces new chimeras even though the total number of chimeras input is reduced
- Removal of contigs < 2.0 kb impacts total assembly size and N50/N90 metrics, which are themselves not very useful for comparing metagenome assembly performance
- Requires Newbler (runAssembly) and Minimus2/AMOS with MUMmer compiled in 64-bit mode; AMOS must be configured with NUCMER, DELTAFILTER, and SHOWCOORDS paths

## Evidence
