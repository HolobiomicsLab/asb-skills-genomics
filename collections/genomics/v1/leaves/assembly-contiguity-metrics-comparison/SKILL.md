---
name: assembly-contiguity-metrics-comparison
description: Decision support for choosing between cross-sample merged and individual
  assemblies when evaluating contig sets from multiple assemblers or related samples
  by contiguity metrics.
when_to_use_negative:
- Input is raw reads with no prior assembly — MeGAMerge operates on assembled contigs,
  not reads
- The assemblies come from unrelated samples or data types that MeGAMerge's overlap-layout-consensus
  merging cannot align (no shared sequence)
- The goal is to compare total assembly size or N50/N90 only — the paper notes these
  metrics are 'not very useful for comparing metagenome assembly performance' and
  are affected by removal of small contigs
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: MeGAMerge
  role: Post-assembly wrapper that dereplicates, bins contigs by a 2.0 kb cutoff,
    runs Newbler on smaller contigs and Minimus2 on the >2 kb pool, and emits the
    merged contig set used for the metric comparison
  repo: https://github.com/LANL-Bioinformatics/MeGAMerge
- name: Newbler
  role: Assembles the pooled smaller contigs (<2 kb) within the MeGAMerge pipeline
    before the final Minimus2 merge
- name: Minimus2
  role: OLC-based merger (AMOS toolbox) that merges the Newbler output with the original
    >2.0 kb contigs into MergedContigs.fasta
- name: BWA
  role: Maps reads back to the final MeGAMerged contigs for additional validation
    statistics
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/srep06480
    title: Improved Assemblies Using a Source-Agnostic Pipeline for MetaGenomic Assembly
      by Merging (MeGAMerge) of Contigs
  source_content_hash: sha256:839699796a674ee4c302ed245ed2593c6e094096966614fe67cbe8b11c269e32
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
  - MeGAMerge
  - Minimus2
  - Newbler
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# assembly-contiguity-metrics-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compare contiguity statistics (largest contig length, total bases in contigs >2 kb, number of predicted genes) of a MeGAMerge cross-sample merge against the best within-sample merges to quantify whether merging improves the assembly.

## When to use

Decision support for choosing between cross-sample merged and individual assemblies when evaluating contig sets from multiple assemblers or related samples by contiguity metrics.

## When NOT to use

- Input is raw reads with no prior assembly — MeGAMerge operates on assembled contigs, not reads
- The assemblies come from unrelated samples or data types that MeGAMerge's overlap-layout-consensus merging cannot align (no shared sequence)
- The goal is to compare total assembly size or N50/N90 only — the paper notes these metrics are 'not very useful for comparing metagenome assembly performance' and are affected by removal of small contigs

## Inputs

- Multiple FASTA files of assembled contigs (Multi-FASTA format) from independent assemblies of related samples
- MeGAMerge output MergedContigs.fasta (merged contig set)
- Within-sample assembly contig sets for baseline comparison

## Outputs

- MergedContigs.fasta — improved merged contig set
- assembly_comparison_metrics.csv — per-merge contiguity metrics (largest contig length, total bases in contigs >2 kb) for cross-sample vs within-sample merges

## How to apply

Retrieve the input contig sets (e.g., HMP SRA reads assembled independently per sample, keeping contigs ≥150 bases per the MeGAMerge -minLen default and excluding contigs <200 bases from Minimus via -minIncludeLen default 200). Pool and dereplicate the contig sets with MeGAMerge's custom Perl dereplication step, removing exact duplicates, reverse complements, and subsequences. Run MeGAMerge with Newbler (-cpu=4) on the smaller contigs and Minimus2 (-conserr 0.06) on the >2 kb pool, writing MergedContigs.fasta via -o. Then recompute contiguity statistics — largest contig length, total bases in contigs >2 kb — for the cross-sample merge and each within-sample merge, and record the comparison (e.g., in assembly_comparison_metrics.csv). Correct application is judged by the merged set at least matching, ideally exceeding, the best within-sample merge, consistent with the finding that the largest MeGAMerge contigs contain more bases than an equal number of largest contigs from any individual input data set.

## Related tools

- **MeGAMerge** (Post-assembly wrapper that dereplicates, bins contigs by a 2.0 kb cutoff, runs Newbler on smaller contigs and Minimus2 on the >2 kb pool, and emits the merged contig set used for the metric comparison) — https://github.com/LANL-Bioinformatics/MeGAMerge
- **Newbler** (Assembles the pooled smaller contigs (<2 kb) within the MeGAMerge pipeline before the final Minimus2 merge)
- **Minimus2** (OLC-based merger (AMOS toolbox) that merges the Newbler output with the original >2.0 kb contigs into MergedContigs.fasta)
- **BWA** (Maps reads back to the final MeGAMerged contigs for additional validation statistics)

## Examples

```
MeGAMerge-1.0.pl -cpu=4 -conserr=0.06 -o MergedContigs.fasta outdir sampleA_contigs.fasta sampleB_contigs.fasta
```

## Evaluation signals

- MergedContigs.fasta and assembly_comparison_metrics.csv both exist after the run
- The cross-sample merge's largest-contig length is ≥ the best within-sample merge's largest contig
- Total bases in contigs >2 kb for the merged set is ≥ (ideally >) the best within-sample merge
- Merged contigs show no replication of data — the largest MeGAMerge contigs contain more bases than an equal number of the largest contigs from any individual input dataset
- Dereplication step has removed exact duplicates, reverse complements, and subsequences from the pooled contig set before merging

## Limitations

- Overlap-based consensus tools' run-times increase exponentially with increasing input dataset size, limiting scalability
- The merging process introduces new chimeras, even though MeGAMerge reduces the total number of chimeras input into the process
- Removal of small contigs (<2 kb) impacts total assembly size and N50/N90 metrics, which are consequently not very useful for comparing metagenome assembly performance
- Improvements such as better clustering, improved overlap consensus based assembly, and binning of contigs into overlapping groups were only 'currently being explored', not implemented

## Evidence

- [discussion] largest contigs claim: "It is always the case that the largest MeGAMerge contigs contain more bases than an equal number of largest contigs from any individual input data set"
- [discussion] dereplication step: "Dereplication of contigs is performed using a custom implemented Perl script, to remove exact duplicates, reverse compliments, and subsequences"
- [discussion] Minimus2 merge stage: "the sequences generated by Newbler and the original contigs greater than 2.0 kb in length are MeGAMerged using Minimus2, a tool from the AMOS toolbox"
- [readme] CLI options: "MeGAMerge-1.0.pl [options] output_directory <list of fastas>"
- [methods] minLen parameter: "-minLen=NN             Minimum length to include in newbler assemblies (default 150)"
