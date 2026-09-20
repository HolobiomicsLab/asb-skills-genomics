---
name: multiple-testing-correction
description: Use when a tncRNA or gene count matrix (control vs stress conditions,
  ≥3 biological replicates each) is tested for differential expression and raw per-feature
  p-values must be FDR-controlled before calling up/down-regulation.
when_to_use_negative:
- Input is raw small RNA-seq FASTQ that has not yet been adapter-trimmed (Trim Galore)
  and quantified into a tncRNA count matrix — use tncRNA identification/quantification
  first
- Comparison has fewer than three biological replicates per condition — the source
  workflow explicitly retains only such sample sets before DE testing
- Input is already a table of adjusted p-values or DE results — no re-correction is
  needed; proceed directly to significance filtering
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: DESeq2
  role: Runs differential expression analysis on the tncRNA count matrix using the
    Wald test with Benjamini–Hochberg-corrected p-values (default) and per-tncRNA
    log2 fold-change estimates
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:78acd7316902e56a9437ecbca9144f9cf37187d7fb364ad42f2d5404160e8c8e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/skbinfo/tncRNA-Toolkit
  tools:
  - DESeq2
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# multiple-testing-correction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies Benjamini–Hochberg multiple-testing correction to differential expression testing of tncRNAs (via DESeq2's Wald test), so that stress-responsive tncRNA calls (adjusted p < 0.05) control the false discovery rate across thousands of simultaneous tests.

## When to use

Use when a tncRNA or gene count matrix (control vs stress conditions, ≥3 biological replicates each) is tested for differential expression and raw per-feature p-values must be FDR-controlled before calling up/down-regulation.

## When NOT to use

- Input is raw small RNA-seq FASTQ that has not yet been adapter-trimmed (Trim Galore) and quantified into a tncRNA count matrix — use tncRNA identification/quantification first
- Comparison has fewer than three biological replicates per condition — the source workflow explicitly retains only such sample sets before DE testing
- Input is already a table of adjusted p-values or DE results — no re-correction is needed; proceed directly to significance filtering

## Inputs

- tncRNA count matrix (rows = tncRNAs, columns = samples) derived from small RNA-seq quantification (e.g. tncRNA Toolkit output)
- sample metadata table assigning samples to control/treatment conditions, stress type, duration, and tissue, with ≥3 biological replicates per condition

## Outputs

- Table of differentially expressed tncRNAs with log2FC and BH-adjusted significance values (differentially_expressed_tncRNAs.csv)
- Up-/down-regulated labels per tncRNA (log2FC > 1 up; log2FC < −1 down)

## How to apply

Load the tncRNA count matrix and sample metadata, retaining only sample sets with at least three biological replicates per control and treatment condition. Run DESeq2 differential expression analysis using the Wald test, leaving pAdjustMethod at the DESeq2 default of Benjamini and Hochberg, which controls FDR across all tested tncRNAs. Filter results to adjusted p value < 0.05, then label tncRNAs with log2FC > 1 as up-regulated and log2FC < −1 as down-regulated. The rationale is that cataloging differentially expressed tncRNAs under abiotic/biotic stress involves thousands of simultaneous tests, so uncorrected p-values would inflate false positives; BH correction is applied before any significance-based filtering or downstream target analysis.

## Related tools

- **DESeq2** (Runs differential expression analysis on the tncRNA count matrix using the Wald test with Benjamini–Hochberg-corrected p-values (default) and per-tncRNA log2 fold-change estimates)

## Examples

```
R: res <- DESeq2::results(dds, pAdjustMethod="BH", test="Wald"); de <- subset(res, padj<0.05 & (log2FoldChange>1 | log2FoldChange< -1))
```

## Evaluation signals

- Only tncRNAs meeting the significance criterion (adjusted p value < 0.05 under the BH-corrected Wald test) appear in the output table
- Every output row carries both a log2FC value and an adjusted (BH) p-value, not a raw p-value
- Directional labeling is consistent with the thresholds: up-regulated rows have log2FC > 1 and down-regulated rows have log2FC < −1
- Input QC passed: each control and treatment condition retained ≥3 biological replicates before testing
- Number of significant calls is plausible relative to the article's scale (e.g. up to ~1748 DE tncRNAs in maize tassel after 48 h heat stress), not near-total or near-zero

## Limitations

- Differential expression of tncRNAs and their putative targets requires further experimental validation; the BH-corrected calls alone do not establish mechanistic stress-response roles
- Conventional RNA-seq suffers from tRNA modification biases, which can distort tncRNA counts feeding the DE test; tRNA-seq would be needed to overcome this
- Sample sets lacking ≥3 biological replicates per condition are excluded, so stress/tissue combinations with sparse metadata cannot be tested

## Evidence

- [methods] Run DESeq2 differential expression analysis on the count matrix using the Wald test, with p-values corrected for multiple testing via the Benjamini and Hochberg method (DESeq2 default).: [evidence span withheld — anchor status: not_found]
- [methods] only tncRNAs meeting the significance criterion (p value < 0.05 under the BH-corrected Wald test) appear in the output table: [evidence span withheld — anchor status: not_found]
