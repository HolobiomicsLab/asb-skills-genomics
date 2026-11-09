---
name: fdr-multiple-testing-correction
description: Use when calling differential alternative splicing events (exon skipping,
  5'/3' alt splice sites, intron retention, mutually exclusive exons) or differential
  TSS/PAS usage between sample groups from long-read transcriptome data and you must
  control the false discovery rate across thousands of.
when_to_use_negative:
- You are testing a single pre-specified event rather than a genome-wide catalog —
  no multiple-testing correction is needed and an FDR cutoff would be unnecessarily
  conservative
- Input is short-read RNA-seq without a reconstructed long-read transcriptome — use
  short-read tools like rMATS directly instead of IsoTools' segment-graph events
- Raw per-event p-values or event definitions have not yet been computed — FDR correction
  operates on an existing p-value vector
edam_operation: http://edamontology.org/operation_3675
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3512
tools:
- name: IsoTools
  role: Defines alternative splicing and TSS/PAS events from segment graphs, runs
    the differential event test between sample groups, and applies FDR correction
    (1%) to the resulting p-values; also exports events for rMATS cross-validation
  repo: https://github.com/MatthiasLienhard/isotools
- name: rMATS
  role: Independent short-read re-quantification of the exported alternative splicing
    events; its statistics are correlated with IsoTools' (Spearman rho = 0.68) to
    validate the corrected differential calls
- name: Python
  role: Language in which IsoTools is implemented and in which the differential testing
    and FDR-correction workflow is executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad364
    title: 'IsoTools: a flexible workflow for long-read transcriptome sequencing analysis'
  source_content_hash: sha256:2602ae3846ddcd621019e05255fdc2b363f7ac6825dc1cc2f80ebcae3fc404ed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MatthiasLienhard/isotools
  tools:
  - IsoTools
  - Python
  - rMATS
  edam_operation: http://edamontology.org/operation_3675
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# fdr-multiple-testing-correction

## Summary

Apply false discovery rate (FDR) correction to p-values from IsoTools' differential splicing and TSS/PAS usage tests (VPA vs control primary human hepatocytes), controlling at 1% FDR so that only robustly differential alternative splicing events are called.

## When to use

Use when calling differential alternative splicing events (exon skipping, 5'/3' alt splice sites, intron retention, mutually exclusive exons) or differential TSS/PAS usage between sample groups from long-read transcriptome data and you must control the false discovery rate across thousands of simultaneously tested events.

## When NOT to use

- You are testing a single pre-specified event rather than a genome-wide catalog — no multiple-testing correction is needed and an FDR cutoff would be unnecessarily conservative
- Input is short-read RNA-seq without a reconstructed long-read transcriptome — use short-read tools like rMATS directly instead of IsoTools' segment-graph events
- Raw per-event p-values or event definitions have not yet been computed — FDR correction operates on an existing p-value vector

## Inputs

- IsoTools Transcriptome object (saved .pkl built from LRTS bam files plus gff3/gtf reference annotation)
- Sample group labels (e.g. 'control' vs 'treatment' / CTL vs VPA)
- Alternative splicing event and TSS/PAS event definitions derived from segment graphs
- Raw per-event p-values from the differential test

## Outputs

- Differential-events table of events passing the 1% FDR threshold with adjusted p-values
- Per-category event counts (expected: 777 DSEs in 538 genes; 26 ME, 253 ES, 90 5AS, 59 3AS, 349 IR)
- Counts of differential TSSs (956) and differential PASs (290)

## How to apply

After defining alternative splicing events from IsoTools segment graphs (ME, ES, 5AS, 3AS, IR) and alternative TSS/PAS events, compute per-event p-values between the 'control' and 'treatment' sample groups using IsoTools' built-in differential test (a statistical test on transcript proportions; the VPA-vs-CTL workflow used a two-proportions z-test, and IsoTools also implements a beta-binomial-based approach). Because thousands of events are tested genome-wide, correct the raw p-values for multiple testing and retain events significant at 1% FDR. Summarize survivors into a differential-events table and count events by category; for VPA vs CTL hepatocytes this yields 777 DSEs in 538 genes (26 ME, 253 ES, 90 5AS, 59 3AS, 349 IR), 956 differential TSSs, and 290 differential PASs. Validate by exporting events via IsoTools' rMATS export and checking that IsoTools and rMATS statistics agree (Spearman rho ≈ 0.68).

## Related tools

- **IsoTools** (Defines alternative splicing and TSS/PAS events from segment graphs, runs the differential event test between sample groups, and applies FDR correction (1%) to the resulting p-values; also exports events for rMATS cross-validation) — https://github.com/MatthiasLienhard/isotools
- **rMATS** (Independent short-read re-quantification of the exported alternative splicing events; its statistics are correlated with IsoTools' (Spearman rho = 0.68) to validate the corrected differential calls)
- **Python** (Language in which IsoTools is implemented and in which the differential testing and FDR-correction workflow is executed)

## Examples

```
from isotools import Transcriptome; isoseq = Transcriptome.load('example_1_isotools.pkl'); ds = isoseq.diff_splicing(group1='CTL', group2='VPA'); sig = ds[ds['p_adj'] <= 0.01]
```

## Evaluation signals

- Per-category counts of significant events match the reported figures: 777 DSEs in 538 genes at FDR 1% (26 ME, 253 ES, 90 5AS, 59 3AS, 349 IR)
- Differential TSS count equals 956 and differential PAS count equals 290 for the VPA vs CTL hepatocyte comparison
- Spearman rank correlation between IsoTools and rMATS event statistics is approximately 0.68
- Every event in the output table has an adjusted p-value ≤ 0.01 (1% FDR) and carries a gene assignment (538 genes total)
- Event extraction and testing were run on the reconstructed transcriptome saved via add_qc_metrics and save, not on raw bam files

## Limitations

- IsoTools' differential event detection is statistical inference on reconstructed transcript proportions; agreement with short-read rMATS is only moderate (Spearman rho = 0.68), so corrected calls should not be treated as ground truth
- The headline VPA-vs-CTL counts (777 DSEs, 956 TSSs, 290 PASs) are dataset-specific; other datasets will yield different numbers
- Differential event tests require adequate read support per group; low-coverage events will lack power or produce unreliable proportions

## Evidence

- [abstract] Graph-based event definition grounds the FDR workflow: "we integrate a graph-based method for identifying alternative splicing events and a statistical approach based on the beta-binomial distribution for detecting differential events"
- [readme] Differential detection between groups is the IsoTools feature the correction is applied to: "Detection of differential alternative splicing between samples and groups of samples."
- [intro] Event types underlying the multiple-tested p-value vector: "Definition of alternative splicing events based on segment graphs."
