---
name: tumor-control-differential-methylation-analysis
description: Use when ONT raw signal-level methylation data from paired tumor/control
  samples must be compared locus-by-locus, e.g. to test whether LINE1 promoter regions
  are hypomethylated at somatically inferred LINE1 3' transduction source elements.
when_to_use_negative:
- The goal is calling somatic structural variations or mobile element insertions themselves
  from tumor/control long-read data — that is nanomonsv's role; this skill assumes
  the LINE1 source elements are already inferred
- Input methylation data is from bisulfite sequencing or methylation arrays rather
  than ONT raw signal (fast5) — nanopolish call-methylation requires original fast5
  signal-level data
- Testing loci where both breakpoints/background regions are in poorly characterized
  highly repetitive reference regions is unreliable with reference-genome-based frameworks
edam_topics:
- http://edamontology.org/topic_3295
tools:
- name: nanopolish
  role: nanopolish index (v0.11.1) builds the fast5 index; nanopolish call-methylation
    produces the per-read, per-position methylation log-likelihood ratio TSV used
    to compute methylation frequencies
  repo: https://github.com/jts/nanopolish
- name: harmonicmeanp
  role: R package (version 3.0) that combines per-locus one-sided Fisher's exact P-values
    into asymptotically exact combined P-values per LINE1 source element
- name: nanomonsv
  role: 'upstream: infers the somatic LINE1 3'' transductions and their source sites
    that define the loci tested by this methylation skill'
  repo: https://github.com/friend1ws/nanomonsv
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkad526
    title: Precise characterization of somatic complex structural variations from
      tumor/control paired long-read sequencing data with nanomonsv
  source_content_hash: sha256:048f71cd651226c247079cb07c3da73d6b428ae08383ecd7185461cdcb90e11a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jts/nanopolish
  tools:
  - harmonicmeanp
  - nanomonsv
  - nanopolish
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# tumor-control-differential-methylation-analysis

## Summary

Quantifies differential methylation between tumor and matched control from Oxford Nanopore raw signal-level methylation calls (nanopolish), testing specific genomic loci such as somatically inferred LINE1 3' transduction source elements with per-locus Fisher's exact tests and harmonicmeanp combined P-values.

## When to use

Use when ONT raw signal-level methylation data from paired tumor/control samples must be compared locus-by-locus, e.g. to test whether LINE1 promoter regions are hypomethylated at somatically inferred LINE1 3' transduction source elements.

## When NOT to use

- The goal is calling somatic structural variations or mobile element insertions themselves from tumor/control long-read data — that is nanomonsv's role; this skill assumes the LINE1 source elements are already inferred
- Input methylation data is from bisulfite sequencing or methylation arrays rather than ONT raw signal (fast5) — nanopolish call-methylation requires original fast5 signal-level data
- Testing loci where both breakpoints/background regions are in poorly characterized highly repetitive reference regions is unreliable with reference-genome-based frameworks

## Inputs

- ONT fast5 files (original raw signal data) for tumor and matched control
- ONT read sequences (fastq/fast) for tumor and matched control
- Reference genome for call-methylation
- BED/list of genomic positions for the somatically inferred LINE1 3' transduction source element loci

## Outputs

- Per-read, per-position methylation log-likelihood ratio TSV from nanopolish call-methylation (tumor and control)
- Per-position methylation frequency table (tumor and control)
- methylation_pvalues.tsv containing per-locus one-sided Fisher's exact P-values and harmonicmeanp combined asymptotically exact P-values

## How to apply

Run 'nanopolish index' (nanopolish 0.11.1) on the original fast5 files of tumor and matched control to associate read IDs with signal-level data, then run 'nanopolish call-methylation' on both datasets to produce a TSV of per-read, per-position methylation log-likelihood ratios. Compute methylation frequency at each genomic position using the script provided on the nanopolish software website. At each inferred LINE1 source element locus, calculate a one-sided Fisher's exact test P-value comparing methylation frequency between tumor and matched control (one-sided because the hypothesis is tumor hypomethylation). Combine per-locus P-values for each element into asymptotically exact P-values with the harmonicmeanp package version 3.0, and write all per-locus Fisher P-values and harmonicmeanp combined P-values to methylation_pvalues.tsv. Validation requires that methylation ratios in tumor are significantly lower than in matched control for all 23 reference LINE1 source elements, reproducing Figure 6A and Supplementary Figure S15.

## Related tools

- **nanopolish** (nanopolish index (v0.11.1) builds the fast5 index; nanopolish call-methylation produces the per-read, per-position methylation log-likelihood ratio TSV used to compute methylation frequencies) — https://github.com/jts/nanopolish
- **harmonicmeanp** (R package (version 3.0) that combines per-locus one-sided Fisher's exact P-values into asymptotically exact combined P-values per LINE1 source element)
- **nanomonsv** (upstream: infers the somatic LINE1 3' transductions and their source sites that define the loci tested by this methylation skill) — https://github.com/friend1ws/nanomonsv

## Examples

```
nanopolish index -d fast5/ reads.fastq.gz; nanopolish call-methylation -r reads.fasta -g genome.fa reads.bam > methylation.tsv
```

## Evaluation signals

- All 23 reference LINE1 source elements show significantly lower tumor vs matched-control methylation ratios (reproduces Figure 6A, Supplementary Figure S15)
- nanopolish call-methylation output TSV exists for both tumor and control and contains per-read, per-position log-likelihood ratios
- Per-locus P-values in methylation_pvalues.tsv are one-sided with direction of effect consistent with tumor hypomethylation
- harmonicmeanp combined P-values are asymptotically exact and finite for every source element
- nanopolish index was built against the original fast5 files (not rebasecalled or surrogate signal data) for both samples

## Limitations

- Nanopolish does not support R10.4 flowcells (per README note on R10 support), restricting applicability to earlier pore chemistries
- Requires original fast5 signal files, which are large and often not retained in archives
- Reference-genome-based frameworks may not be appropriate for long repeat expansion events or regions where the reference is unreliable; subclonal LINE1 events at modest sequencing depth are hard to detect

## Evidence

- [other] nanopolish index step: [evidence span withheld — anchor status: not_found]
- [other] harmonicmeanp combination step: [evidence span withheld — anchor status: not_found]
- [readme] nanopolish signal-level analysis scope: [evidence span withheld — anchor status: external_doc]
