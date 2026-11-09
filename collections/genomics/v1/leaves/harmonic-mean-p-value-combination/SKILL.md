---
name: harmonic-mean-p-value-combination
description: Use when per-genomic-position methylation calls from ONT nanopolish data
  must be aggregated into a single locus-level P-value for tumor-vs-control hypomethylation
  at somatically inferred LINE1 3' transduction source elements.
when_to_use_negative:
- When only a single genomic position (not a multi-locus LINE1 source element) is
  tested — a one-sided Fisher's exact test on that position suffices and no P-value
  combination is needed
- When the question is methylation difference in either direction (two-sided), since
  the workflow's tests are explicitly one-sided for tumor hypomethylation
- When the input is not ONT signal-level methylation data (e.g. bisulfite or array
  methylation), as the nanopolish index/call-methylation front end depends on original
  fast5 files
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3393
tools:
- name: nanopolish
  role: Indexes original fast5 files ('nanopolish index', v0.11.1) and produces per-read,
    per-position methylation log-likelihood ratio TSVs via 'nanopolish call-methylation',
    the substrate for the Fisher tests
  repo: github.com/jts/nanopolish
- name: harmonicmeanp
  role: R package (version 3.0) that combines per-locus Fisher's exact test P-values
    for each LINE1 source element into asymptotically exact combined P-values
- name: nanomonsv
  role: Mobile element insertion workflow classifies insertions and identifies the
    source sites for 3' transductions that define the LINE1 loci tested here
  repo: https://github.com/friend1ws/nanomonsv
- name: Oxford Nanopore Technologies (ONT)
  role: Long-read sequencing platform whose raw fast5 signal data feed the nanopolish
    methylation calls
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkad526
    title: Precise characterization of somatic complex structural variations from
      tumor/control paired long-read sequencing data with nanomonsv
  source_content_hash: sha256:8db5418691b39cc4edd4378596508c7d40e4e5eca55066a3c8dea7bab5f3a05b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Oxford Nanopore Technologies (ONT)
  - harmonicmeanp
  - nanomonsv
  - nanopolish
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3393
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# harmonic-mean-p-value-combination

## Summary

Combines per-locus significance P-values (one-sided Fisher's exact tests on nanopolish methylation frequencies) at a LINE1 source element into a single asymptotically exact combined P-value using the harmonicmeanp R package (v3.0), to test whether tumor methylation is lower than matched control across the whole locus.

## When to use

Use when per-genomic-position methylation calls from ONT nanopolish data must be aggregated into a single locus-level P-value for tumor-vs-control hypomethylation at somatically inferred LINE1 3' transduction source elements.

## When NOT to use

- When only a single genomic position (not a multi-locus LINE1 source element) is tested — a one-sided Fisher's exact test on that position suffices and no P-value combination is needed
- When the question is methylation difference in either direction (two-sided), since the workflow's tests are explicitly one-sided for tumor hypomethylation
- When the input is not ONT signal-level methylation data (e.g. bisulfite or array methylation), as the nanopolish index/call-methylation front end depends on original fast5 files

## Inputs

- nanopolish per-read, per-position methylation log-likelihood ratio TSVs from tumor ONT fast5 data
- nanopolish per-read, per-position methylation log-likelihood ratio TSVs from matched control ONT fast5 data
- Per-position methylation frequency table computed from the nanopolish TSVs
- Genomic coordinates of somatically inferred LINE1 3' transduction source element loci (23 reference source elements)

## Outputs

- methylation_pvalues.tsv containing per-locus one-sided Fisher's exact test P-values and harmonicmeanp combined P-values
- Locus-level hypomethylation significance calls for the 23 reference LINE1 source elements (Figure 6A, Supplementary Figure S15)

## How to apply

First run 'nanopolish index' (nanopolish 0.11.1) on the original fast5 files of tumor and matched control to link read IDs to signal-level data, then 'nanopolish call-methylation' on both to obtain per-read, per-position methylation log-likelihood ratio TSVs, and compute per-position methylation frequency with the script provided on the nanopolish website. At each inferred LINE1 source element locus, calculate a one-sided Fisher's exact test P-value comparing methylation frequency between tumor and matched control — one-sided because the research question is specifically hypomethylation in tumor. Combine the per-locus P-values for each LINE1 source element into asymptotically exact P-values with the harmonicmeanp package version 3.0, then write all per-locus Fisher P-values and harmonicmeanp combined P-values to methylation_pvalues.tsv. Correct application reproduces the paper's reported result: all 23 reference LINE1 source elements show significantly lower methylation ratios in tumor than matched control (Figure 6A, Supplementary Figure S15).

## Related tools

- **nanopolish** (Indexes original fast5 files ('nanopolish index', v0.11.1) and produces per-read, per-position methylation log-likelihood ratio TSVs via 'nanopolish call-methylation', the substrate for the Fisher tests) — github.com/jts/nanopolish
- **harmonicmeanp** (R package (version 3.0) that combines per-locus Fisher's exact test P-values for each LINE1 source element into asymptotically exact combined P-values)
- **nanomonsv** (Mobile element insertion workflow classifies insertions and identifies the source sites for 3' transductions that define the LINE1 loci tested here) — https://github.com/friend1ws/nanomonsv
- **Oxford Nanopore Technologies (ONT)** (Long-read sequencing platform whose raw fast5 signal data feed the nanopolish methylation calls)

## Examples

```
nanopolish index -d fast5/ tumor_reads.fastq && nanopolish call-methylation -r tumor_reads.fastq -g hg38.fa > tumor_methylation.tsv
```

## Evaluation signals

- All 23 reference LINE1 source elements show combined harmonicmeanp P-values indicating significantly lower methylation ratios in tumor than matched control
- methylation_pvalues.tsv contains both the per-locus one-sided Fisher's exact test P-values and the harmonicmeanp combined P-values for every source element
- Combined P-values are asymptotically exact harmonic-mean P-values from the harmonicmeanp package version 3.0 (not naive Bonferroni or Fisher aggregations)
- Result replicates the paper's Figure 6A and Supplementary Figure S15 pattern of tumor hypomethylation at 3' transduction source sites
- Per-position methylation frequencies recomputed from the nanopolish TSVs are consistent between tumor and control datasets processed with the same pipeline

## Limitations

- Requires original ONT fast5 signal files for both tumor and matched control — nanopolish index cannot be run without them
- Nanopolish does not support newer flowcell chemistries: 'Presently nanopolish does not support R10.4 flowcells as variant and' (methylation support is similarly constrained)
- Only one breakpoint of LINE1-mediated rearrangements is anchored in well-characterized sequence; loci where both breakpoints fall in highly repetitive regions are not resolvable by the upstream nanomonsv Single breakend module

## Evidence

- [results] all 23 reference LINE1 source elements showed significantly lower methylation ratios in tumors than in matched controls: [evidence span withheld — anchor status: not_found]
- [methods] asymptotically exact combined P-values with harmonicmeanp 3.0: [evidence span withheld — anchor status: not_found]
- [methods] one-sided Fisher's exact test at each locus: [evidence span withheld — anchor status: not_found]
- [abstract] nanomonsv workflow identifies 3' transduction source sites tested here: "we have developed a workflow to classify mobile element insertions while elucidating their in-depth properties, such as 5' truncations, internal inversions, as well as source sites for 3'"
- [readme] nanopolish flowcell support limitation: [evidence span withheld — anchor status: external_doc]
