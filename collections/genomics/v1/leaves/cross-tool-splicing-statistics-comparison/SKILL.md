---
name: cross-tool-splicing-statistics-comparison
description: Use when IsoTools (long-read LRTS) differential splicing results need
  independent validation against a short-read tool such as rMATS, and a quantitative
  agreement metric (Spearman correlation) is required.
when_to_use_negative:
- The transcriptome has not been reconstructed and quality-controlled in IsoTools
  yet (no saved pickle from bam import) — run the import/QC workflow first
- The goal is de novo event discovery, not cross-tool statistical agreement — use
  IsoTools' segment-graph event extraction alone
- Only short-read RNA-seq data is available and no LRTS bam files exist — IsoTools'
  differential event framework and this comparison are designed for long-read transcriptome
  sequencing data
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3308
tools:
- name: IsoTools
  role: Python LRTS framework used to reconstruct the transcriptome, define alternative
    splicing events from segment graphs, run the differential event test at 1% FDR,
    and export events for rMATS (and MISO)
  repo: https://github.com/MatthiasLienhard/isotools
- name: rMATS
  role: Short-read splicing quantification tool used as the independent comparison
    target; IsoTools statistics are correlated against rMATS statistics (Spearman
    rho = 0.68 in the VPA case study)
- name: Python
  role: Language in which the IsoTools analysis and correlation computation are performed
    (isotools installed via pip)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad364
    title: 'IsoTools: a flexible workflow for long-read transcriptome sequencing analysis'
  source_content_hash: sha256:c8e8cf128e2951aee2ef37f47ce98fba0b0690ab3ca046bc661261f4e786c64c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MatthiasLienhard/isotools
  tools:
  - IsoTools
  - Python
  - rMATS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# Cross-tool splicing statistics comparison

## Summary

Validates long-read differential splicing results by exporting IsoTools alternative splicing events for rMATS and comparing the two tools' statistics with a Spearman rank correlation, as demonstrated on PacBio LRTS data of valproic-acid-treated vs control human hepatocytes.

## When to use

Use when IsoTools (long-read LRTS) differential splicing results need independent validation against a short-read tool such as rMATS, and a quantitative agreement metric (Spearman correlation) is required.

## When NOT to use

- The transcriptome has not been reconstructed and quality-controlled in IsoTools yet (no saved pickle from bam import) — run the import/QC workflow first
- The goal is de novo event discovery, not cross-tool statistical agreement — use IsoTools' segment-graph event extraction alone
- Only short-read RNA-seq data is available and no LRTS bam files exist — IsoTools' differential event framework and this comparison are designed for long-read transcriptome sequencing data

## Inputs

- IsoTools Transcriptome pickle built from LRTS bam files and gff3/gtf reference annotation (e.g. tests/data/example_1_isotools.pkl)
- Sample-group labels (e.g. 'CTL' vs 'VPA') used for the differential two-proportions z-test at 1% FDR
- rMATS statistics on the same samples (from the exported alternative splicing events)

## Outputs

- Differential-events table summarizing significant events per category (expected: 777 DSEs in 538 genes: 26 ME, 253 ES, 90 5AS, 59 3AS, 349 IR)
- Counts of differential TSSs (956) and PASs (290)
- Spearman rank correlation (rho) between IsoTools and rMATS statistics (expected ~0.68)

## How to apply

Build the IsoTools Transcriptome from LRTS bam files plus gff3/gtf reference annotation (add_sample_from_bam, add_qc_metrics, save as pickle), then extract alternative splicing events (exon skipping, 5'/3' alternative splice sites, intron retention, mutually exclusive exons) from the segment graphs and run the IsoTools differential splicing test between 'control' and 'treatment' groups at 1% FDR. Export the alternative splicing events using IsoTools' rMATS export option (IsoTools also supports MISO export). Run rMATS on the same samples to obtain short-read splicing statistics for the matched events. Compute the Spearman rank correlation between IsoTools and rMATS statistics over shared events; in the VPA vs CTL hepatocyte case study this yielded rho = 0.68 alongside 777 DSEs in 538 genes (26 ME, 253 ES, 90 5AS, 59 3AS, 349 IR), 956 differential TSSs and 290 differential PASs.

## Related tools

- **IsoTools** (Python LRTS framework used to reconstruct the transcriptome, define alternative splicing events from segment graphs, run the differential event test at 1% FDR, and export events for rMATS (and MISO)) — https://github.com/MatthiasLienhard/isotools
- **rMATS** (Short-read splicing quantification tool used as the independent comparison target; IsoTools statistics are correlated against rMATS statistics (Spearman rho = 0.68 in the VPA case study))
- **Python** (Language in which the IsoTools analysis and correlation computation are performed (isotools installed via pip))

## Examples

```
from isotools import Transcriptome; isoseq = Transcriptome.from_reference('tests/data/example.gff.gz'); isoseq.add_sample_from_bam('tests/data/example_1_CTL.bam', sample_name='CTL', group='CTL', platform='SequelII'); isoseq.add_qc_metrics('tests/data/example.fa'); isoseq.save('example_isotools.pkl')
```

## Evaluation signals

- Per-category differential splicing counts match the reported figures: 777 DSEs in 538 genes broken down as 26 ME, 253 ES, 90 5AS, 59 3AS, 349 IR
- Differential TSS and PAS counts reproduce 956 and 290 respectively at 1% FDR
- Spearman rank correlation between IsoTools and rMATS statistics is approximately 0.68
- All events tested at FDR of 1% (p-values corrected to a 1% false discovery rate)
- Exported event file is accepted by rMATS (IsoTools rMATS export option used successfully)

## Limitations

- Agreement with short-read tools is moderate (Spearman rho = 0.68), so IsoTools and rMATS statistics will not be identical; discrepancies reflect read-type differences
- The statistical test in IsoTools is based on the beta-binomial/two-proportions framework and corrects to a fixed FDR level (1% in the case study), so changing the FDR changes event counts
- The comparison requires matched samples processed by both tools; IsoTools' differential analysis applies to samples organized into defined groups

## Evidence

- [abstract] we integrate a graph-based method for identifying alternative splicing events and a statistical approach based on the beta-binomial distribution for detecting differential events: "we integrate a graph-based method for identifying alternative splicing events and a statistical approach based on the beta-binomial distribution for detecting differential events"
- [discussion] added option to export alternative splicing events for MISO and rMATS: "added option to export alternative splicing events for MISO and rMATS"
- [intro] Definition of alternative splicing events based on segment graphs. Detection of differential alternative splicing between samples and groups of samples.: "Definition of alternative splicing events based on segment graphs.
* Detection of differential alternative splicing between samples and groups of samples."
- [readme] isoseq.add_sample_from_bam(f'../tests/data/example_1_{sa}.bam', sample_name=sa, group=sa, platform='SequelII'): "for sa in ('CTL', 'VPA'):
    isoseq.add_sample_from_bam(f'../tests/data/example_1_{sa}.bam', sample_name=sa, group=sa, platform='SequelII')"
- [methods] IsoTools is a python module for Long Read Transcriptome Sequencing (LRTS) analysis: "IsoTools is a python module for Long Read Transcriptome Sequencing (LRTS) analysis."
