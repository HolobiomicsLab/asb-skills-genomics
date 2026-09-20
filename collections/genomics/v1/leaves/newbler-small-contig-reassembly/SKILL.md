---
name: newbler-small-contig-reassembly
description: Use when running the MeGAMerge post-assembly merge pipeline and you have
  a binned pool of sub-2 kb contigs from multiple assemblies that should be re-assembled
  by Newbler before Minimus2 merging.
when_to_use_negative:
- Input is a single high-quality assembly with few or no sub-2 kb contigs — the reassembly
  pass adds little and Newbler's auto-options may exclude contigs unnecessarily
- Input is raw sequencing reads rather than already-assembled contig FASTA files;
  MeGAMerge expects Multi-FASTA files of assembled contigs of any length
- Input contigs have not been dereplicated first — the pipeline requires removal of
  exact duplicates, reverse complements, and subsequences before binning
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: Newbler
  role: runAssembly is invoked on the small (<2 kb) contig bin with -minLen=150 and
    -cpu=4 to reassemble sub-2 kb contigs before Minimus2 merging
- name: MeGAMerge
  role: Perl wrapper that performs dereplication, 2 kb binning, invokes Newbler on
    the small-contig bin, and combines its output with the >2.0 kb pool
  repo: https://github.com/LANL-Bioinformatics/MeGAMerge
- name: Minimus2
  role: Consumes the combined Newbler output plus large contigs to perform the final
    OLC-based merge
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/srep06480
    title: Improved Assemblies Using a Source-Agnostic Pipeline for MetaGenomic Assembly
      by Merging (MeGAMerge) of Contigs
  source_content_hash: sha256:255ec10542e0496a2807f812c5aeefdd87fb638d2e3d968359cbc7d98cf32c4b
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
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# newbler-small-contig-reassembly

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Re-assembles the pool of small (<2 kb) contigs produced by individual assemblies with Newbler's runAssembly, so that short contigs can be extended before the Minimus2 merge step of MeGAMerge.

## When to use

Use when running the MeGAMerge post-assembly merge pipeline and you have a binned pool of sub-2 kb contigs from multiple assemblies that should be re-assembled by Newbler before Minimus2 merging.

## When NOT to use

- Input is a single high-quality assembly with few or no sub-2 kb contigs — the reassembly pass adds little and Newbler's auto-options may exclude contigs unnecessarily
- Input is raw sequencing reads rather than already-assembled contig FASTA files; MeGAMerge expects Multi-FASTA files of assembled contigs of any length
- Input contigs have not been dereplicated first — the pipeline requires removal of exact duplicates, reverse complements, and subsequences before binning

## Inputs

- Dereplicated bin of small contigs (<2 kb) in Multi-FASTA format, pooled from multiple individual assemblies
- Path to the Newbler runAssembly executable (via -newblerdir)

## Outputs

- Newbler assembly output contigs from the small-contig bin, to be combined with the >2.0 kb contig pool and passed to Minimus2
- Contributions to the final merged contig set (default -o=MergedContigs.fasta in the output directory)

## How to apply

After MeGAMerge dereplicates the pooled input contigs (removing exact duplicates, reverse complements, and subsequences), contigs are binned at a 2.0 kb size cutoff; contigs smaller than 2.0 kb are routed to this skill. Run Newbler's runAssembly on the small-contig bin with -minLen=150 (minimum length to include in newbler assemblies) and -cpu=4 (default CPU count), then combine the Newbler output with the pool of contigs greater than 2.0 kb before handing the combined set to Minimus2 (-minIncludeLen=200, -conserr=0.06, minimum overlap 80 bp at 98% identity). The rationale is to ignore poorly covered, difficult-to-assemble, error-prone small contigs from individual assemblies while giving short fragments a chance to be extended through a fresh assembly pass. For single-genome inputs, set -single_genome=1 so Newbler runs once with auto-options reducing excluded contigs, which overrides -minLen and -minIncludeLen. Executable locations are controlled via -newblerdir (runAssembly) and -bindir (MUMmer/AMOS executables).

## Related tools

- **Newbler** (runAssembly is invoked on the small (<2 kb) contig bin with -minLen=150 and -cpu=4 to reassemble sub-2 kb contigs before Minimus2 merging)
- **MeGAMerge** (Perl wrapper that performs dereplication, 2 kb binning, invokes Newbler on the small-contig bin, and combines its output with the >2.0 kb pool) — https://github.com/LANL-Bioinformatics/MeGAMerge
- **Minimus2** (Consumes the combined Newbler output plus large contigs to perform the final OLC-based merge)

## Examples

```
MeGAMerge-1.0.pl -cpu=4 -minLen=150 -newblerdir=/usr/local/newbler -bindir=/usr/local/amos/bin -o=MergedContigs.fasta out_dir contigs_A.fasta contigs_B.fasta
```

## Evaluation signals

- The Newbler run used the small-contig bin only (contigs below the 2.0 kb cutoff) and its output was combined with the pool of contigs >2.0 kb
- Newbler options match the pipeline defaults: -minLen=150 and -cpu=4, unless -single_genome=1 was set (which overrides -minLen and -minIncludeLen)
- The final merged output (default MergedContigs.fasta) contains fewer total contigs than the dereplicated input pool while maintaining reference genome coverage
- MeGAMerged contigs are longer than contigs from any individual input assembly (largest MeGAMerge contigs contain more bases than an equal number of largest contigs from any single dataset)

## Limitations

- Overlap-based consensus tools' run-times increase exponentially with increasing input dataset size, limiting scalability on very large inputs
- Very small contigs (<2 kb) are removed from the dataset, which impacts total assembly size and N50/N90 metrics
- For multi-genome inputs Newbler's auto-options may exclude many contigs; -single_genome=1 mitigates this but applies only to single-genome inputs

## Evidence
