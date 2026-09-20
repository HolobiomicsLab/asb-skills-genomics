---
name: two-proportions-z-test-hypothesis-testing
description: Use when you have a reconstructed IsoTools Transcriptome (from PacBio/SequelII
  LRTS bam files plus gff3/gtf reference annotation) split into control/treatment
  groups and need counts of differential splicing events or TSS/PAS usage shifts.
when_to_use_negative:
- Input is short-read RNA-seq only (no long-read bam files) — IsoTools' event extraction
  and z-test require full-length transcript alignments; use short-read tools like
  rMATS directly
- No reference annotation in gff3/gtf format is available and none can be supplied,
  since Transcriptome.from_reference is required before event definition
- The goal is transcript reconstruction or QC rather than differential event testing
  between sample groups
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3512
tools:
- name: IsoTools
  role: Python framework providing segment-graph-based alternative splicing event
    definition and the two-proportions z-test differential splicing analysis with
    FDR correction; also exports events for rMATS
  repo: https://github.com/MatthiasLienhard/isotools
- name: rMATS
  role: Short-read differential splicing tool used as independent validation; IsoTools
    exports alternative splicing events for rMATS and Spearman correlation of statistics
    is computed
- name: Python
  role: Language in which the IsoTools module is implemented and the differential
    analysis is scripted
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad364
    title: 'IsoTools: a flexible workflow for long-read transcriptome sequencing analysis'
  source_content_hash: sha256:e23d11935dbe3746dc542cd59460d894d169325fd497531a6c1429c825732f3d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MatthiasLienhard/isotools
  tools:
  - IsoTools
  - Python
  - rMATS
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# two-proportions-z-test-hypothesis-testing

## Summary

Detect differential alternative splicing, TSS and PAS usage between sample groups from long-read transcriptome sequencing (LRTS) using IsoTools' two-proportions z-test with FDR correction, validated against short-read rMATS quantification.

## When to use

Use when you have a reconstructed IsoTools Transcriptome (from PacBio/SequelII LRTS bam files plus gff3/gtf reference annotation) split into control/treatment groups and need counts of differential splicing events or TSS/PAS usage shifts.

## When NOT to use

- Input is short-read RNA-seq only (no long-read bam files) — IsoTools' event extraction and z-test require full-length transcript alignments; use short-read tools like rMATS directly
- No reference annotation in gff3/gtf format is available and none can be supplied, since Transcriptome.from_reference is required before event definition
- The goal is transcript reconstruction or QC rather than differential event testing between sample groups

## Inputs

- IsoTools Transcriptome pickle built from LRTS bam files (e.g. PacBio SequelII) and a gff3/gtf reference annotation
- Sample group labels (e.g. 'control' vs 'treatment', CTL vs VPA hepatocytes)
- FDR threshold (1%) for multiple-testing correction

## Outputs

- Differential-events table of significant events at FDR 1% with per-category counts (e.g. 777 DSEs in 538 genes: 26 ME, 253 ES, 90 5AS, 59 3AS, 349 IR)
- Differential TSS usage counts (956) and PAS usage counts (290) for VPA vs CTL
- rMATS-format export of alternative splicing events and IsoTools-vs-rMATS Spearman correlation (rho = 0.68)

## How to apply

Load the reconstructed transcriptome pickle (built via Transcriptome.from_reference on a gff3 annotation and add_sample_from_bam for each LRTS bam, then add_qc_metrics and save). Extract alternative splicing events from the segment graphs (exon skipping, 5'/3' alternative splice sites, intron retention, mutually exclusive exons) plus alternative TSS/PAS events using IsoTools' built-in event extraction. Run the differential event test between 'control' and 'treatment' groups with the two-proportions z-test as implemented by IsoTools' differential splicing analysis, correcting p-values at 1% FDR. For the VPA vs CTL hepatocyte case this yields 777 DSEs in 538 genes (26 ME, 253 ES, 90 5AS, 59 3AS, 349 IR), 956 differential TSSs and 290 differential PASs. Optionally export events with IsoTools' rMATS export option and compute the Spearman rank correlation against rMATS statistics (expected rho = 0.68) as cross-platform validation.

## Related tools

- **IsoTools** (Python framework providing segment-graph-based alternative splicing event definition and the two-proportions z-test differential splicing analysis with FDR correction; also exports events for rMATS) — https://github.com/MatthiasLienhard/isotools
- **rMATS** (Short-read differential splicing tool used as independent validation; IsoTools exports alternative splicing events for rMATS and Spearman correlation of statistics is computed)
- **Python** (Language in which the IsoTools module is implemented and the differential analysis is scripted)

## Examples

```
from isotools import Transcriptome; isoseq = Transcriptome.from_reference('tests/data/example.gff.gz'); isoseq.add_sample_from_bam('example_1_CTL.bam', sample_name='CTL', group='control', platform='SequelII'); isoseq.add_qc_metrics('example.fa'); isoseq.save('example_isotools.pkl')
```

## Evaluation signals

- Per-category DSE counts match the reported figures: 777 DSEs in 538 genes (26 ME, 253 ES, 90 5AS, 59 3AS, 349 IR) at FDR 1%
- Differential TSS count equals 956 and differential PAS count equals 290 for VPA vs CTL hepatocytes
- IsoTools-vs-rMATS Spearman rank correlation of event statistics is approximately 0.68
- All reported events pass the 1% FDR correction applied to the two-proportions z-test p-values
- Events are derivable from segment graphs and cover the expected categories (exon skipping, 5'/3' alternative splice sites, intron retention, mutually exclusive exons)

## Limitations

- Validation against rMATS shows only moderate agreement (Spearman rho = 0.68), so LRTS-based event calls and short-read quantification will not fully coincide
- Results are specific to the analysed design (VPA-treated vs control primary human hepatocytes); counts will differ for other datasets and cannot be used as fixed expectations
- The differential test depends on prior transcriptome reconstruction and QC; errors in bam import or annotation propagate into event counts

## Evidence

- [abstract] z-test differential events: "we integrate a graph-based method for identifying alternative splicing events and a statistical approach based on the beta-binomial distribution for detecting differential events"
- [abstract] VPA application: "we applied IsoTools to PacBio LRTS data of human hepatocytes treated with the histone deacetylase inhibitor valproic acid"
- [intro] event definition scope: "Definition of alternative splicing events based on segment graphs. Detection of differential alternative splicing between samples and groups of samples."
- [readme] import workflow: [evidence span withheld — anchor status: external_doc]
- [discussion] rMATS export: "added option to export alternative splicing events for MISO and rMATS"
