---
name: multi-assembly-contig-merge-configuration
description: Use when you have multiple Multi-FASTA contig files from different assemblers
  (e.g. SOAPdenovo, CLC Bio, Velvet, Newbler) and need to merge them into a single
  improved non-redundant metagenomic contig set.
when_to_use_negative:
- You need a single de novo assembly from raw reads rather than a post-assembly merge
  of existing contig sets
- Runtime constraints forbid OLC-based merging — the article notes overlap-based consensus
  tools' run-times increase exponentially with increasing input dataset size
- Input contigs are already dereplicated at the subsequence level and already merged,
  e.g. a single final contig set with no per-assembler FASTA files to pool
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: MeGAMerge
  role: Perl wrapper that orchestrates dereplication, 2.0 kb binning, Newbler assembly,
    and Minimus2 merging of input contig FASTAs
  repo: https://github.com/LANL-Bioinformatics/MeGAMerge
- name: Newbler
  role: runAssembly re-assembles the small-contig bin (<2.0 kb) with -minLen=150 and
    configurable -cpu
- name: Minimus2
  role: OLC-based merge of the Newbler output with contigs >2.0 kb using -minIncludeLen=200
    and -conserr=0.06
- name: AMOS toolbox
  role: Source of the Minimus2 executable; modified to allow ambiguous bases in the
    assembly
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/srep06480
    title: Improved Assemblies Using a Source-Agnostic Pipeline for MetaGenomic Assembly
      by Merging (MeGAMerge) of Contigs
  source_content_hash: sha256:779bf9c519df58967316aa0d80360da706995b113862b06be2a6d63c5e7f6966
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
yes_rate: 0.7
n_claims: 10
---

# multi-assembly-contig-merge-configuration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Configure and run MeGAMerge, a Perl wrapper that pools contig FASTA files from multiple individual assemblies, dereplicates and size-bins them, then re-assembles with Newbler and merges with Minimus2 (OLC-based) into an improved merged contig set, with overlap parameters declarable at runtime.

## When to use

Use when you have multiple Multi-FASTA contig files from different assemblers (e.g. SOAPdenovo, CLC Bio, Velvet, Newbler) and need to merge them into a single improved non-redundant metagenomic contig set.

## When NOT to use

- You need a single de novo assembly from raw reads rather than a post-assembly merge of existing contig sets
- Runtime constraints forbid OLC-based merging — the article notes overlap-based consensus tools' run-times increase exponentially with increasing input dataset size
- Input contigs are already dereplicated at the subsequence level and already merged, e.g. a single final contig set with no per-assembler FASTA files to pool

## Inputs

- Any number of Multi-FASTA files containing assembled contigs of any length
- Newbler (runAssembly) installation, in $PATH or via -newblerdir
- Minimus2 from the AMOS toolbox plus MUMmer (nucmer, delta-filter, show-coords), in $PATH or via -bindir

## Outputs

- Merged contig set written to output_directory as MergedContigs.fasta (customizable via -o)

## How to apply

Run MeGAMerge-1.0.pl [options] output_directory <list of fastas>, requiring Newbler's runAssembly and Minimus2 (AMOS) plus MUMmer executables in $PATH or via -bindir/-newblerdir. The tool first dereplicates pooled contigs with a custom Perl script (removing exact duplicates, reverse complements, and subsequences), then bins contigs at a 2.0 kb cutoff: small contigs go to Newbler (runAssembly with -minLen=150, -cpu=4 default) while contigs >2.0 kb pass through. Newbler output is combined with the >2.0 kb pool and merged with Minimus2 using -minIncludeLen=200 and -conserr=0.06, with a minimum overlap length of 80 bp (-overlap) at 98% minimum identity (-minID). Use -single_genome=1 for single-genome inputs to run one Newbler pass with fewer excluded contigs (overrides -minLen and -minIncludeLen), and -force to overwrite a previous output directory; the final merged set is written as MergedContigs.fasta in the output directory.

## Related tools

- **MeGAMerge** (Perl wrapper that orchestrates dereplication, 2.0 kb binning, Newbler assembly, and Minimus2 merging of input contig FASTAs) — https://github.com/LANL-Bioinformatics/MeGAMerge
- **Newbler** (runAssembly re-assembles the small-contig bin (<2.0 kb) with -minLen=150 and configurable -cpu)
- **Minimus2** (OLC-based merge of the Newbler output with contigs >2.0 kb using -minIncludeLen=200 and -conserr=0.06)
- **AMOS toolbox** (Source of the Minimus2 executable; modified to allow ambiguous bases in the assembly)

## Examples

```
perl MeGAMerge-1.0.pl -minID=98 -overlap=80 -minIncludeLen=200 -conserr=0.06 -bindir=/usr/local/amos/bin -o=MergedContigs.fasta -force outdir asm1.fasta asm2.fasta asm3.fasta
```

## Evaluation signals

- MergedContigs.fasta is written to the output directory and contains contigs without replication of data across input assemblies
- The largest merged contigs contain more bases than an equal number of the largest contigs from any individual input dataset
- Dereplication removed exact duplicates, reverse complements, and subsequences before merging; total input contig count drops while reference coverage (Nucmer alignment) is maintained
- Newbler stage used the expected parameters (-minLen=150, -cpu, or -single_genome=1 overrides) and Minimus2 stage used -minIncludeLen=200 and -conserr=0.06 with the user-specified -overlap/-minID
- Downstream validation: read mapping with BWA (BAM parsed with SAMtools), gene prediction (contigs <2 kb removed before Prodigal), and similarity searches support the merged contigs

## Limitations

- Overlap-based consensus tools' run-times increase exponentially with increasing input dataset size
- The merging process does introduce new chimeras, even though MeGAMerge reduces the total number of chimeras input and removes unmappable contigs
- Removing small contigs (<2 kb) impacts total assembly size and N50/N90 metrics, which the authors note are not very useful for comparing metagenome assembly performance
- MUMmer must be compiled in 64-bit mode for larger genome projects, and AMOS must be configured to locate nucmer, delta-filter, and show-coords

## Evidence
