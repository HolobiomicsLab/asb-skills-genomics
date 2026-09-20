---
name: per-locus-fisher-exact-hypothesis-testing
description: Use when ONT nanopolish call-methylation TSVs exist for tumor and matched
  control and the goal is per-locus hypomethylation P-values at LINE1 3' transduction
  source elements, combined across positions.
when_to_use_negative:
- The goal is detecting or classifying somatic SVs or mobile element insertions themselves
  — use the nanomonsv Canonical SV or Single breakend SV modules instead; this skill
  only tests methylation at already-inferred source loci.
- Only basecalled reads/alignments are available without the original fast5 signal
  files — 'nanopolish index' and 'call-methylation' cannot run from BAM/FASTQ alone.
- Need an aggregate methylation comparison across the whole genome or broad regions
  rather than per-locus tests at specific LINE1 source elements.
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3053
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: nanopolish
  role: Indexes fast5 raw signal data ('nanopolish index', v0.11.1) and produces the
    per-read, per-position methylation log-likelihood ratio TSVs via 'call-methylation'
    from which methylation frequencies are computed
  repo: https://github.com/jts/nanopolish
- name: harmonicmeanp
  role: R package (version 3.0) used to combine per-locus Fisher's exact P-values
    for each LINE1 source element into asymptotically exact combined P-values
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkad526
    title: Precise characterization of somatic complex structural variations from
      tumor/control paired long-read sequencing data with nanomonsv
  source_content_hash: sha256:8bf2776942d96b39a7dc51dfb0c6cc6a74104f0b248c9c6508576f10b238ea0a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jts/nanopolish
  tools:
  - harmonicmeanp
  - nanopolish
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3053
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 3
---

# per-locus-fisher-exact-hypothesis-testing

## Summary

Per-locus one-sided Fisher's exact tests on nanopolish-derived per-position methylation frequencies, with P-values combined per LINE1 3' transduction source element using the harmonicmeanp package, to test tumor-vs-matched-control hypomethylation from ONT raw signal-level methylation calls.

## When to use

Use when ONT nanopolish call-methylation TSVs exist for tumor and matched control and the goal is per-locus hypomethylation P-values at LINE1 3' transduction source elements, combined across positions.

## When NOT to use

- The goal is detecting or classifying somatic SVs or mobile element insertions themselves — use the nanomonsv Canonical SV or Single breakend SV modules instead; this skill only tests methylation at already-inferred source loci.
- Only basecalled reads/alignments are available without the original fast5 signal files — 'nanopolish index' and 'call-methylation' cannot run from BAM/FASTQ alone.
- Need an aggregate methylation comparison across the whole genome or broad regions rather than per-locus tests at specific LINE1 source elements.

## Inputs

- Original ONT fast5 signal files for tumor and matched control samples (required by 'nanopolish index')
- Per-read, per-position methylation log-likelihood ratio TSVs from 'nanopolish call-methylation' for both tumor and matched control
- Genomic coordinates of the inferred LINE1 3' transduction source element loci (the 23 reference source elements)

## Outputs

- Per-position methylation frequencies for tumor and matched control (script from the nanopolish website)
- methylation_pvalues.tsv containing per-locus one-sided Fisher's exact test P-values and harmonicmeanp combined P-values for each LINE1 source element

## How to apply

Run 'nanopolish index' (nanopolish 0.11.1) on the original fast5 files of tumor and matched control to associate read IDs with signal-level data, then execute 'nanopolish call-methylation' on both datasets to obtain per-read, per-position methylation log-likelihood ratio TSVs. Compute methylation frequency at each genomic position with the script provided on the nanopolish software website (methylated calls ÷ total calls per position). At each inferred LINE1 source element locus, tabulate methylation frequencies from tumor and matched control and compute a one-sided Fisher's exact test P-value with the alternative that tumor methylation is lower than matched control. Combine per-locus P-values for each LINE1 source element into asymptotically exact combined P-values with the harmonicmeanp R package version 3.0. Write all per-locus Fisher P-values and harmonicmeanp combined P-values to methylation_pvalues.tsv.

## Related tools

- **nanopolish** (Indexes fast5 raw signal data ('nanopolish index', v0.11.1) and produces the per-read, per-position methylation log-likelihood ratio TSVs via 'call-methylation' from which methylation frequencies are computed) — https://github.com/jts/nanopolish
- **harmonicmeanp** (R package (version 3.0) used to combine per-locus Fisher's exact P-values for each LINE1 source element into asymptotically exact combined P-values)

## Examples

```
nanopolish index -d fast5 tumor.fastq; nanopolish call-methylation -r tumor.fastq -b tumor.bam -g genome.fa > tumor_methylation.tsv
```

## Evaluation signals

- methylation_pvalues.tsv contains, for each of the 23 reference LINE1 source elements, both a per-locus Fisher's exact P-value and a harmonicmeanp combined P-value.
- Tumor methylation ratios are lower than matched control at every locus and all 23 combined P-values are significant, reproducing Figure 6A / Supplementary Figure S15.
- Per-position methylation frequencies recompute correctly from the call-methylation TSV (calls favoring methylation ÷ total calls at that position).
- Fisher tests are one-sided in the direction tumor < control; a two-sided or reversed-direction test would fail the reported validation.
- 'nanopolish index' was run on the original fast5 files (v0.11.1) for both tumor and matched control before 'call-methylation', confirming signal-level data were used.

## Limitations

- Requires the original fast5 signal files; methylation cannot be re-called from alignments alone.
- Nanopolish presently does not support R10.4 flowcells (README note on R10 support).
- The test depends on source element loci already inferred by the mobile element insertion classification workflow; it cannot itself discover the insertions.
- The reference-genome-based framework may be unreliable for events in poorly characterized repetitive regions and long repeat expansion events (article discussion).

## Evidence

- [other] Execute 'nanopolish call-methylation' on both tumor and control datasets to produce the TSV file of per-read, per-position methylation log-likelihood ratios.: [evidence span withheld — anchor status: not_found]
- [other] At each inferred LINE1 source element locus, calculate a one-sided Fisher's exact test P-value comparing methylation frequency between tumor and matched control.: [evidence span withheld — anchor status: not_found]
- [other] all 23 reference LINE1 source elements showed significantly lower methylation ratios in tumors than in matched controls.: [evidence span withheld — anchor status: not_found]
- [abstract] we have developed a workflow to classify mobile element insertions while elucidating their in-depth properties, such as 5' truncations, internal inversions, as well as source sites for 3': "we have developed a workflow to classify mobile element insertions while elucidating their in-depth properties, such as 5' truncations, internal inversions, as well as source sites for 3'"
- [readme] Software package for signal-level analysis of Oxford Nanopore sequencing data.: [evidence span withheld — anchor status: external_doc]
