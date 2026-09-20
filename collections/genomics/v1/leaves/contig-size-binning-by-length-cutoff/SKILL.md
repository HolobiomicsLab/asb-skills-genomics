---
name: contig-size-binning-by-length-cutoff
description: Use when multiple individual-assembly contig FASTA files have been pooled
  and dereplicated, and you must split them into a large-contig pool and a small-contig
  bin (2.0 kb cutoff) before overlap-layout-consensus merging.
when_to_use_negative:
- Input is raw reads rather than assembled contig Multi-FASTA files — MeGAMerge operates
  post-assembly on FASTA contigs.
- You need to merge contigs without any length-based exclusion of small contigs, e.g.
  when poorly covered short contigs are themselves of interest; binning removes very
  small contigs from the dataset before merging.
- Input dataset is extremely large, since overlap-based consensus tools' run-times
  increase exponentially with increasing input dataset size.
edam_operation: http://edamontology.org/operation_3129
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: MeGAMerge
  role: Perl wrapper that pools, dereplicates, and bins contigs at the 2.0 kb size
    cutoff and orchestrates the Newbler and Minimus2 merge steps
  repo: https://github.com/LANL-Bioinformatics/MeGAMerge
- name: Newbler
  role: Re-assembles the small-contig bin via runAssembly (-minLen=150, -cpu=4) before
    output is combined with the >2.0 kb pool
- name: Minimus2
  role: OLC-based merge of the Newbler output plus contigs greater than 2.0 kb, using
    -minIncludeLen=200 and -conserr=0.06
- name: AMOS toolbox
  role: Provides Minimus2 (modified to allow ambiguous bases); MUMmer and AMOS executables
    located via -bindir
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/srep06480
    title: Improved Assemblies Using a Source-Agnostic Pipeline for MetaGenomic Assembly
      by Merging (MeGAMerge) of Contigs
  source_content_hash: sha256:90e11d148fa1c6b66376d5a6faca6ecde9b805914adc9dd47d63b6b9654de149
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
  edam_operation: http://edamontology.org/operation_3129
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

# contig-size-binning-by-length-cutoff

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A post-assembly filtering/binning step from the MeGAMerge pipeline that splits pooled, dereplicated contigs into two pools using a 2.0 kb size cutoff, so small contigs can be re-assembled with Newbler and large contigs bypass to Minimus2 merging.

## When to use

Use when multiple individual-assembly contig FASTA files have been pooled and dereplicated, and you must split them into a large-contig pool and a small-contig bin (2.0 kb cutoff) before overlap-layout-consensus merging.

## When NOT to use

- Input is raw reads rather than assembled contig Multi-FASTA files — MeGAMerge operates post-assembly on FASTA contigs.
- You need to merge contigs without any length-based exclusion of small contigs, e.g. when poorly covered short contigs are themselves of interest; binning removes very small contigs from the dataset before merging.
- Input dataset is extremely large, since overlap-based consensus tools' run-times increase exponentially with increasing input dataset size.

## Inputs

- Pooled, dereplicated contig sequences in Multi-FASTA format from multiple individual assemblies (e.g. SOAPdenovo, CLC Bio, Velvet, Newbler)

## Outputs

- Large-contig pool (contigs > 2.0 kb) passed directly to Minimus2
- Small-contig bin (< 2.0 kb) fed to Newbler runAssembly
- Final merged contig set (default MergedContigs.fasta) written to the output directory

## How to apply

After pooling contigs from multiple assemblies and dereplicating them with the custom MeGAMerge Perl step (removing exact duplicates, reverse complements, and subsequences), bin each contig by length against the 2.0 kb cutoff. Contigs longer than 2.0 kb bypass Newbler and go directly into the Minimus2 merging pool, while the small-contig bin is re-assembled with Newbler runAssembly using -minLen=150 and -cpu=4; Newbler output is then combined with the >2.0 kb pool. The rationale is to remove very small contigs from direct merging so poorly covered, difficult-to-assemble, or error-prone data from individual assemblies are excluded, while still giving short contigs a chance to be incorporated via Newbler before the Minimus2 merge (run with -minIncludeLen=200, -conserr=0.06, minimum overlap 80 bp at 98% identity). The final merged contig set is written as MergedContigs.fasta in the output directory.

## Related tools

- **MeGAMerge** (Perl wrapper that pools, dereplicates, and bins contigs at the 2.0 kb size cutoff and orchestrates the Newbler and Minimus2 merge steps) — https://github.com/LANL-Bioinformatics/MeGAMerge
- **Newbler** (Re-assembles the small-contig bin via runAssembly (-minLen=150, -cpu=4) before output is combined with the >2.0 kb pool)
- **Minimus2** (OLC-based merge of the Newbler output plus contigs greater than 2.0 kb, using -minIncludeLen=200 and -conserr=0.06)
- **AMOS toolbox** (Provides Minimus2 (modified to allow ambiguous bases); MUMmer and AMOS executables located via -bindir)

## Examples

```
MeGAMerge-1.0.pl -minLen=150 -cpu=4 -minIncludeLen=200 -conserr=0.06 -bindir=/usr/local/amos/bin -o=MergedContigs.fasta merged_output contigs_A.fasta contigs_B.fasta
```

## Evaluation signals

- All contigs in the Newbler input bin are below the 2.0 kb cutoff and all contigs passed directly to Minimus2 are above it — verify by length-checking the FASTA files per bin.
- Contig count after dereplication-plus-binning is reduced relative to the pooled input, consistent with removal of duplicates, reverse complements, subsequences, and very small contigs.
- The small-contig bin was assembled with Newbler using -minLen=150 and its output combined with the >2.0 kb pool before Minimus2 (check MeGAMerge run logs with -d for debug output).
- Final merged contig set (MergedContigs.fasta) exists in the output directory and reflects successful merging, e.g. larger N50 contigs than any individual input assembly.
- When the source is a single genome, -single_genome=1 was used (one Newbler run, fewer excluded contigs); for metagenomes, default binning/merging parameters were left in place.

## Limitations

- Very small contigs (<2 kb) are removed from the dataset before merging, which impacts total assembly size and N50/N90 metrics (metrics that are themselves not very useful for comparing metagenome assembly performance).
- Overlap-based consensus tools' run-times increase exponentially with increasing input dataset size, so the binning-plus-merging pipeline does not scale trivially.
- While the merging process introduces new chimeras, MeGAMerge reduces the total number of chimeras input into the process — chimeras are reduced overall but not eliminated.

## Evidence
