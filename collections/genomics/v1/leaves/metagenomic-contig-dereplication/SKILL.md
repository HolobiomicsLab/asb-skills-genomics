---
name: metagenomic-contig-dereplication
description: Use when multiple assembled metagenome contig FASTA files (e.g. from
  SOAPdenovo, CLC Bio, Velvet, Newbler) must be pooled and deduplicated prior to OLC-based
  merging, and exact duplicates, reverse complements, or subsequences must be eliminated
  first.
when_to_use_negative:
- When you have raw reads rather than assembled contig Multi-FASTA files — MeGAMerge
  is a post-assembly process and does not assemble reads itself
- When contigs are already deduplicated and you only need to re-run a single assembler
  rather than merge multiple assemblies
- When single-genome (not metagenomic) input is expected to need very different Newbler
  auto-options — use -single_genome=1 only for single-genome inputs, not mixed metagenomes
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0196
tools:
- name: MeGAMerge
  role: Perl wrapper orchestrating dereplication, 2 kb binning, Newbler re-assembly,
    and Minimus2 merging of pooled contigs
  repo: https://github.com/LANL-Bioinformatics/MeGAMerge
- name: Newbler
  role: runAssembly re-assembles the <2 kb small-contig bin (-minLen=150, -cpu=4)
    before merging
- name: Minimus2
  role: AMOS-tool OLC-based merge of Newbler output with contigs >2.0 kb, controlled
    by -minIncludeLen=200, -conserr=0.06, and runtime overlap parameters
  repo: https://github.com/LANL-Bioinformatics/MeGAMerge
- name: AMOS toolbox
  role: Provides Minimus2 (modified to allow ambiguous bases in the assembly)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/srep06480
    title: Improved Assemblies Using a Source-Agnostic Pipeline for MetaGenomic Assembly
      by Merging (MeGAMerge) of Contigs
  source_content_hash: sha256:101bddee8261a4bf4fbd2221a295158d65c9aaeb26bc6367082317d3aec9ebd5
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
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# metagenomic-contig-dereplication

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Dereplicate pooled contig Multi-FASTA files from multiple metagenomic assemblies (removing exact duplicates, reverse complements, and subsequences) with MeGAMerge's custom Perl dereplication step before size-based binning and OLC-based merging, so the merged contig set contains no replicated data.

## When to use

Use when multiple assembled metagenome contig FASTA files (e.g. from SOAPdenovo, CLC Bio, Velvet, Newbler) must be pooled and deduplicated prior to OLC-based merging, and exact duplicates, reverse complements, or subsequences must be eliminated first.

## When NOT to use

- When you have raw reads rather than assembled contig Multi-FASTA files — MeGAMerge is a post-assembly process and does not assemble reads itself
- When contigs are already deduplicated and you only need to re-run a single assembler rather than merge multiple assemblies
- When single-genome (not metagenomic) input is expected to need very different Newbler auto-options — use -single_genome=1 only for single-genome inputs, not mixed metagenomes

## Inputs

- Any number of Multi-FASTA files of assembled metagenomic contigs (any length), from assemblers such as SOAPdenovo, CLC Bio de novo assembler, Velvet, IDBA, Ray, or Newbler
- Path to an output directory
- Directories containing MUMmer and AMOS executables (-bindir) and the Newbler runAssembly executable (-newblerdir)

## Outputs

- A dereplicated, merged contig set written as MergedContigs.fasta in the output directory (default name, overridable with -o=)
- Reduced total number of dereplicated input contigs while maintaining reference genome coverage (assessable by Nucmer alignment to references)

## How to apply

Pool all contig FASTA files from the individual assemblies into a Multi-FASTA set and run MeGAMerge-1.0.pl (a Perl wrapper) on that set. The pipeline first applies its custom dereplication Perl script, which removes exact duplicates, reverse compliments, and subsequences, eliminating replication of data between assemblers. Dereplicated contigs are then binned at a 2.0 kb size cutoff: contigs >2 kb are kept as-is, while smaller contigs are re-assembled with Newbler runAssembly (-minLen=150, -cpu=4). The Newbler output and the >2.0 kb pool are merged with Minimus2 from the AMOS toolbox (-minIncludeLen=200, -conserr=0.06) using user-declarable overlap parameters (minimum overlap length 80, minimum overlap percent identity 98), locating MUMmer and AMOS executables via -bindir. The final merged set is written to the output directory as MergedContigs.fasta (override with -o=), using -force to overwrite previous runs if needed.

## Related tools

- **MeGAMerge** (Perl wrapper orchestrating dereplication, 2 kb binning, Newbler re-assembly, and Minimus2 merging of pooled contigs) — https://github.com/LANL-Bioinformatics/MeGAMerge
- **Newbler** (runAssembly re-assembles the <2 kb small-contig bin (-minLen=150, -cpu=4) before merging)
- **Minimus2** (AMOS-tool OLC-based merge of Newbler output with contigs >2.0 kb, controlled by -minIncludeLen=200, -conserr=0.06, and runtime overlap parameters) — https://github.com/LANL-Bioinformatics/MeGAMerge
- **AMOS toolbox** (Provides Minimus2 (modified to allow ambiguous bases in the assembly))

## Examples

```
MeGAMerge-1.0.pl -overlap=80 -minID=98 -cpu=4 -bindir=/usr/local/amos/bin -o=MergedContigs.fasta merged_output asm1_contigs.fasta asm2_contigs.fasta
```

## Evaluation signals

- No residual exact duplicates, reverse complements, or subsequences remain in the pooled contig set after the dereplication step
- Total number of dereplicated contigs decreases after merging while reference genome coverage (via Nucmer alignment) is maintained
- Largest merged contigs contain more total bases than an equal number of the largest contigs from any single individual input assembly
- Output file MergedContigs.fasta exists in the specified output directory and every input contig is either represented, merged, or intentionally dropped (small <2 kb contigs filtered)
- Predicted gene count (e.g. Prodigal on contigs ≥2 kb) and read-mapping support (BWA mapping, SAMtools-parsed BAM) increase relative to individual input assemblies

## Limitations

- Overlap-based consensus tools' run-times increase exponentially with increasing input dataset size, limiting scalability
- While merging reduces the total number of chimeras input, the merging process itself introduces new chimeras
- Removal of small contigs (<2 kb) impacts total assembly size and N50/N90 metrics, which are themselves not very useful for comparing metagenome assembly performance

## Evidence
