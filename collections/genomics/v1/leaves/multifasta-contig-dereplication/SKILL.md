---
name: multifasta-contig-dereplication
description: Use when contig sets from multiple assemblies or related samples are
  being pooled into Multi-FASTA files and must be deduplicated (duplicates, reverse
  complements, subsequences) prior to overlap-layout-consensus merging.
when_to_use_negative:
- Input is a single assembly whose contigs are already unique — dereplication and
  OLC merging add no value and overlap-based tools' run-times increase exponentially
  with input size
- The goal is total assembly size or N50/N90 comparison — the article notes removal
  of small contigs impacts these metrics and they are 'not very useful for comparing
  metagenome assembly performance'
- Only raw reads (not assembled contigs) are available — MeGAMerge is a post-assembly
  process requiring Multi-FASTA contig inputs
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: MeGAMerge
  role: Perl wrapper that pools, dereplicates (custom Perl script), and merges contig
    FASTAs via Newbler and Minimus2; dereplication step is executed within this pipeline
  repo: https://github.com/LANL-Bioinformatics/MeGAMerge
- name: Minimus2
  role: OLC-based merger applied after dereplication/binning to the Newbler output
    plus the >2.0 kb contig pool; modified to allow ambiguous bases
- name: Newbler
  role: Assembles the smaller (<2 kb) dereplicated contigs before the final Minimus2
    merge; parameters -minLen=150 and -minIncludeLen=200 apply
- name: MUMmer
  role: Dependency of AMOS/Minimus2; must be compiled in 64-bit mode (make all CPPFLAGS="-O3
    -DSIXTYFOURBITS") for larger projects
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/srep06480
    title: Improved Assemblies Using a Source-Agnostic Pipeline for MetaGenomic Assembly
      by Merging (MeGAMerge) of Contigs
  source_content_hash: sha256:c0d8c246fd1adf3e72006a7cf116ce76ce28273d203538820519721752e3c2f0
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
  - MUMmer
  - MeGAMerge
  - Minimus2
  - Newbler
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# Multi-FASTA contig dereplication

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Dereplicate pooled Multi-FASTA contig sets (e.g., from multiple metagenome assemblies) using MeGAMerge's custom Perl script, which removes exact duplicates, reverse complements, and subsequences before OLC-based merging.

## When to use

Use when contig sets from multiple assemblies or related samples are being pooled into Multi-FASTA files and must be deduplicated (duplicates, reverse complements, subsequences) prior to overlap-layout-consensus merging.

## When NOT to use

- Input is a single assembly whose contigs are already unique — dereplication and OLC merging add no value and overlap-based tools' run-times increase exponentially with input size
- The goal is total assembly size or N50/N90 comparison — the article notes removal of small contigs impacts these metrics and they are 'not very useful for comparing metagenome assembly performance'
- Only raw reads (not assembled contigs) are available — MeGAMerge is a post-assembly process requiring Multi-FASTA contig inputs

## Inputs

- Two or more Multi-FASTA files of assembled contigs (any number accepted, any contig length)
- Output directory path for the merged result

## Outputs

- Dereplicated pooled contig set (exact duplicates, reverse complements, and subsequences removed)
- MergedContigs.fasta — final OLC-merged contig set (default -o output name)

## How to apply

Assemble or collect contigs independently per source (keeping contigs of at least 150 bases, the MeGAMerge -minLen default), then pool all Multi-FASTA files as inputs to the MeGAMerge pipeline. Run MeGAMerge's custom implemented Perl dereplication step on the pooled contigs to remove exact duplicates, reverse complements, and subsequences — this prevents data replication from inflating the merged assembly. After dereplication, contigs are binned at the 2.0 kb size cutoff: smaller contigs (min 150 bases) go to Newbler (runAssembly, e.g. -cpu=4), while the >2 kb pool goes to Minimus2 with a maximum conservation error of 0.06 (-conserr default). The final merged output is written to MergedContigs.fasta via -o. Rationale: MeGAMerge reduces the total number of (already dereplicated) input contigs while maintaining reference genome coverage, and the largest merged contigs contain more bases than an equal number of largest contigs from any individual input dataset.

## Related tools

- **MeGAMerge** (Perl wrapper that pools, dereplicates (custom Perl script), and merges contig FASTAs via Newbler and Minimus2; dereplication step is executed within this pipeline) — https://github.com/LANL-Bioinformatics/MeGAMerge
- **Minimus2** (OLC-based merger applied after dereplication/binning to the Newbler output plus the >2.0 kb contig pool; modified to allow ambiguous bases)
- **Newbler** (Assembles the smaller (<2 kb) dereplicated contigs before the final Minimus2 merge; parameters -minLen=150 and -minIncludeLen=200 apply)
- **MUMmer** (Dependency of AMOS/Minimus2; must be compiled in 64-bit mode (make all CPPFLAGS="-O3 -DSIXTYFOURBITS") for larger projects)

## Examples

```
MeGAMerge-1.0.pl -cpu=4 -conserr=0.06 -minLen=150 -minIncludeLen=200 -o=MergedContigs.fasta out_dir sample1_contigs.fa sample2_contigs.fa
```

## Evaluation signals

- MergedContigs.fasta exists in the specified output directory (or the user-specified -o filename)
- No exact duplicates, reverse complements, or subsequences remain in the dereplicated contig pool (custom Perl dereplication succeeded)
- The number of contigs in the merged set is reduced relative to the dereplicated input pool, while reference genome coverage (checked via Nucmer alignment) is maintained
- The largest MeGAMerge contigs contain more total bases than an equal number of largest contigs from any individual input dataset
- Dereplication removed duplicates without data replication, confirmed by recomputed contiguity statistics (largest contig length, total bases in contigs >2 kb) exceeding or matching individual assemblies

## Limitations

- Overlap-based consensus tools' run-times increase exponentially with increasing input dataset size, limiting scalability
- Merging introduces new chimeras, though dereplication and the merge process reduce the total number of chimeras input
- Removal of small contigs impacts total assembly size and N50/N90 metrics
- Better clustering, improved overlap consensus assembly, and binning of contigs into overlapping groups were only 'currently being explored', not implemented in this version

## Evidence
