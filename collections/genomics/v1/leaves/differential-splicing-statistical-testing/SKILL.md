---
name: differential-splicing-statistical-testing
description: Use when you have LRTS (e.g. PacBio) bam files for two or more sample
  groups and need to detect differential splicing events (exon skipping, 5'/3' alternative
  splice sites, intron retention, mutually exclusive exons) or differential TSS/PAS
  usage at a stated FDR.
when_to_use_negative:
- Input is short-read RNA-seq data — IsoTools is designed for Long Read Transcriptome
  Sequencing (LRTS) bam input
- You only need transcriptome reconstruction or novel-transcript classification without
  a group comparison (no case/control contrast exists)
- Event support is extremely sparse and low coverage makes the two-proportions z-test
  unstable without the overdispersion-aware beta-binomial treatment or per-event filtering
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: IsoTools
  role: Python framework providing segment-graph event extraction, the two-proportions
    z-test and beta-binomial differential testing, and rMATS/MISO export of events
  repo: github.com/MatthiasLienhard/isotools
- name: rMATS
  role: 'Short-read differential splicing tool used as an external cross-check: events
    are exported from IsoTools and statistics compared via Spearman correlation (rho
    = 0.68 in the VPA-vs-CTL case)'
- name: Python
  role: Language environment for running the IsoTools module (installed via pip) and
    any downstream scripting of the differential-events table
provenance:
  source_task_ids:
  - task_007
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad364
    title: 'IsoTools: a flexible workflow for long-read transcriptome sequencing analysis'
  source_content_hash: sha256:2e37c3ed7183792780f558246b0dcb8fd184f78b11d548794b12244195bb4b31
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IsoTools
  - Python
  - rMATS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# differential-splicing-statistical-testing

## Summary

Statistical detection of differential alternative splicing and transcript start/polyadenylation site usage between sample groups from long-read transcriptome sequencing data, using IsoTools' segment-graph event definition combined with a two-proportions z-test or a beta-binomial model that captures within-group PSI overdispersion.

## When to use

Use when you have LRTS (e.g. PacBio) bam files for two or more sample groups and need to detect differential splicing events (exon skipping, 5'/3' alternative splice sites, intron retention, mutually exclusive exons) or differential TSS/PAS usage at a stated FDR.

## When NOT to use

- Input is short-read RNA-seq data — IsoTools is designed for Long Read Transcriptome Sequencing (LRTS) bam input
- You only need transcriptome reconstruction or novel-transcript classification without a group comparison (no case/control contrast exists)
- Event support is extremely sparse and low coverage makes the two-proportions z-test unstable without the overdispersion-aware beta-binomial treatment or per-event filtering

## Inputs

- Aligned long-read (LRTS) bam files of full-length transcripts, one per sample
- Reference annotation in gff3/gtf format
- Sample group assignments (e.g. 'CTL' vs 'VPA') and target FDR threshold

## Outputs

- Differential-events table with per-event p-values and FDR-corrected significance, categorized by event type (ES, 5AS, 3AS, IR, ME)
- Counts of differential splicing events per gene/event category
- Differential TSS and PAS usage lists
- Optionally, rMATS/MISO-format event export for cross-tool comparison

## How to apply

Build an IsoTools Transcriptome object from the gff3/gtf reference annotation and import each sample's aligned full-length-read bam via add_sample_from_bam, then add QC metrics and save as pickle. Extract alternative splicing events from the segment graphs and TSS/PAS events with IsoTools' built-in event extraction, then run the differential test between defined groups (e.g. 'control' vs 'treatment'): the default two-proportions z-test compares supporting-read proportions per event, while the beta-binomial option models within-group PSI variability with a Beta(a,b) prior fit numerically (L-BFGS) to account for overdispersion. Correct p-values to the target FDR (e.g. 1% via Benjamini-Hochberg) and summarize counts per event category. Optionally export events for rMATS/MISO and cross-check effect sizes by Spearman correlation. In the VPA-vs-CTL hepatocyte case this yielded 777 DSEs in 538 genes (26 ME, 253 ES, 90 5AS, 59 3AS, 349 IR), 956 differential TSSs and 290 PASs at FDR 1%, with IsoTools-vs-rMATS Spearman rho = 0.68.

## Related tools

- **IsoTools** (Python framework providing segment-graph event extraction, the two-proportions z-test and beta-binomial differential testing, and rMATS/MISO export of events) — github.com/MatthiasLienhard/isotools
- **rMATS** (Short-read differential splicing tool used as an external cross-check: events are exported from IsoTools and statistics compared via Spearman correlation (rho = 0.68 in the VPA-vs-CTL case))
- **Python** (Language environment for running the IsoTools module (installed via pip) and any downstream scripting of the differential-events table)

## Examples

```
from isotools import Transcriptome; isoseq = Transcriptome.from_reference('tests/data/example.gff.gz'); isoseq.add_sample_from_bam('example_1_CTL.bam', sample_name='CTL', group='CTL', platform='SequelII')
```

## Evaluation signals

- Per-category event counts match expectations from the reference run: 777 DSEs in 538 genes (26 ME, 253 ES, 90 5AS, 59 3AS, 349 IR) at FDR 1%
- Differential TSS count ≈ 956 and differential PAS count ≈ 290 for the VPA-vs-CTL hepatocyte data
- Spearman correlation between IsoTools and rMATS statistics is approximately 0.68
- For beta-binomial mode: null-group p-values pass a KS-uniformity check (KS p > 0.05) and, with overdispersion forced to zero, p-values match the binomial LRT within tolerance (< 1e-3)
- The differential-events table schema carries per-event group proportions, p-values, and FDR-corrected significance consistent with the requested 1% FDR

## Limitations

- Agreement with short-read rMATS quantification is only moderate (Spearman rho = 0.68), so cross-tool differences should be expected
- The two-proportions z-test does not model within-group overdispersion; the beta-binomial alternative must be used when PSI variability within groups is non-negligible
- Results depend on prior transcriptome reconstruction quality — bam import, QC metrics (add_qc_metrics) and any add_filter steps (e.g. FRAGMENTS tags) precede the test and affect input event support

## Evidence

- [abstract] a graph-based method for identifying alternative splicing events and a statistical approach based on the beta-binomial distribution for detecting differential events: "a graph-based method for identifying alternative splicing events and a statistical approach based on the beta-binomial distribution for detecting differential events"
- [intro] Detection of differential alternative splicing between samples and groups of samples.: "Detection of differential alternative splicing between samples and groups of samples."
- [discussion] added option to export alternative splicing events for MISO and rMATS: "added option to export alternative splicing events for MISO and rMATS"
- [readme] IsoTools is a python module for Long Read Transcriptome Sequencing (LRTS) analysis.: "IsoTools is a python module for Long Read Transcriptome Sequencing (LRTS) analysis."
