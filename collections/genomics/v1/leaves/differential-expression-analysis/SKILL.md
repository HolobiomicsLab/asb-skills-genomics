---
name: differential-expression-analysis
description: Use when a tncRNA (or other small RNA) count matrix with biological replicates
  for control and stress-treated samples (abiotic or biotic) needs significance testing
  to catalog stress-responsive tncRNAs.
when_to_use_negative:
- Input reads have not yet been detected/classified as tncRNAs — run the tncRNA Toolkit
  detection pipeline (tRNAscan-SE, artificial-genome mapping with bowtie) to build
  the count matrix first
- Conditions lack at least three biological replicates for control and treatment,
  as required by the study design
- The goal is tncRNA discovery/annotation from raw small RNA FASTQ rather than comparison
  of pre-quantified expression between conditions
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: DESeq2
  role: Performs differential expression testing on the tncRNA count matrix using
    the Wald test with Benjamini-Hochberg p-value correction
- name: tncRNA Toolkit
  role: Upstream pipeline that detects, classifies, and quantifies tncRNAs (tRF-5s,
    tRF-3s, tRF-1, tRNA halves) from quality-filtered small RNA FASTQ to yield the
    count matrix input
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: Trim Galore
  role: Adapter and quality trimming of single-end small RNA reads before tncRNA detection,
    ensuring reliable input counts
  repo: https://github.com/FelixKrueger/TrimGalore
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:d74394a634e2dcc07dd2eafe1028c45b42ac9e31165e21bcb82c5a8f17fa00b4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/skbinfo/tncRNA-Toolkit
  repo_url: https://github.com/skbinfo/tncRNA-Toolkit
  tools:
  - DESeq2
  - Trim Galore
  - tncRNA Toolkit
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# differential-expression-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Statistical identification of tRNA-derived non-coding RNAs (tncRNAs) that are differentially expressed between control and stress conditions in plants, using DESeq2's negative-binomial Wald test on tncRNA count matrices with Benjamini-Hochberg multiple-testing correction.

## When to use

Use when a tncRNA (or other small RNA) count matrix with biological replicates for control and stress-treated samples (abiotic or biotic) needs significance testing to catalog stress-responsive tncRNAs.

## When NOT to use

- Input reads have not yet been detected/classified as tncRNAs — run the tncRNA Toolkit detection pipeline (tRNAscan-SE, artificial-genome mapping with bowtie) to build the count matrix first
- Conditions lack at least three biological replicates for control and treatment, as required by the study design
- The goal is tncRNA discovery/annotation from raw small RNA FASTQ rather than comparison of pre-quantified expression between conditions

## Inputs

- tncRNA count matrix (tRF-5, tRF-3, tRF-1, tRNA halves) quantified from small RNA-seq reads
- sample metadata mapping samples to control/treatment conditions, stress type, duration, and tissue

## Outputs

- differentially_expressed_tncRNAs.csv containing tncRNA identifiers with log2FC and BH-adjusted significance values, filtered to adjusted p < 0.05

## How to apply

Load the tncRNA count matrix (produced by the tncRNA Toolkit pipeline) together with sample metadata, retaining only contrasts where each control and treatment condition has at least three biological replicates. Run DESeq2's differential expression workflow with the Wald test, and correct p-values for multiple testing via Benjamini and Hochberg (the DESeq2 default). Filter results to significant tncRNAs with adjusted p value < 0.05, labeling tncRNAs with log2FC > 1 as up-regulated and log2FC < -1 as down-regulated. Write the table of differentially expressed tncRNAs with log2FC and significance values to differentially_expressed_tncRNAs.csv. This mirrors the paper's strategy of cataloging tncRNAs differentially expressed under various abiotic and biotic stresses to probe their role in stress resistance.

## Related tools

- **DESeq2** (Performs differential expression testing on the tncRNA count matrix using the Wald test with Benjamini-Hochberg p-value correction)
- **tncRNA Toolkit** (Upstream pipeline that detects, classifies, and quantifies tncRNAs (tRF-5s, tRF-3s, tRF-1, tRNA halves) from quality-filtered small RNA FASTQ to yield the count matrix input) — https://github.com/skbinfo/tncRNA-Toolkit
- **Trim Galore** (Adapter and quality trimming of single-end small RNA reads before tncRNA detection, ensuring reliable input counts) — https://github.com/FelixKrueger/TrimGalore

## Examples

```
dds <- DESeqDataSetFromMatrix(countData=counts, colData=meta, design=~condition); res <- results(DESeq(dds), pAdjustMethod="BH"); de <- subset(res, padj<0.05 & abs(log2FoldChange)>1)
```

## Evaluation signals

- Only tncRNAs meeting the significance criterion (adjusted p value < 0.05 under the BH-corrected Wald test) appear in the output table
- Rows are directionally labeled: log2FC > 1 up-regulated, log2FC < -1 down-regulated
- Every contrast retained in the analysis has at least three biological replicates for both control and treatment conditions
- Output file differentially_expressed_tncRNAs.csv contains log2FC and significance columns for each tncRNA

## Limitations

- DE tncRNAs and their putative targets require experimental validation; in-silico differential expression alone does not confirm function
- Conventional RNA-seq suffers from tRNA modification biases; tRNA-seq would be needed to fully overcome them when quantifying tncRNA expression
- No single standardized nomenclature exists for tncRNA subtypes, complicating cross-study comparison of DE results

## Evidence
