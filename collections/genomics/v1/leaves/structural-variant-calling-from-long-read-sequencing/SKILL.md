---
name: structural-variant-calling-from-long-read-sequencing
description: Use when calling deletion structural variants (>50 bp) from long-read
  alignments (SAM/BAM) of third-generation sequencing data and benchmarking caller
  performance against a truth set.
when_to_use_negative:
- The goal is insertion or base substitution (SNV) variant calling — the paper states
  'This study currently focuses exclusively on experiments related to deletion variants
  calling'
- Input data are short NGS reads (e.g. Illumina ~250–300 bp) rather than third-generation
  long reads
- A versioned, production-ready caller is required — the authors note they 'aim to
  further develop and package this technology into user-friendly software' (future
  work)
edam_operation: http://edamontology.org/operation_3227
tools:
- name: DASV
  role: Dual-attention multi-scale ResNet deletion caller under evaluation; converts
    alignment information into 100x100-pixel images and fuses them with genomic sequence
    data
  repo: https://github.com/deconvolution-w/DASV
- name: cuteSV
  role: Baseline third-generation SV calling tool for comparison on the same CLR/CCS
    HG002 datasets
- name: SVIM
  role: Baseline third-generation SV calling tool for comparison on the same datasets
- name: Sniffles
  role: Baseline third-generation SV calling tool for comparison on the same datasets
- name: PBSV
  role: Baseline third-generation SV calling tool for comparison on the same datasets
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae269
    title: Deletion variants calling in third-generation sequencing data based on
      a dual-attention mechanism
  source_content_hash: sha256:3395809568a07246b6a458e0dacb43dcae451052acea12102deb9ef7c008393d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/deconvolution-w/DASV
  tools:
  - DASV
  - PBSV
  - SVIM
  - Sniffles
  - cuteSV
  edam_operation: http://edamontology.org/operation_3227
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# structural-variant-calling-from-long-read-sequencing

## Summary

Reproduce the DASV deletion-calling workflow on third-generation sequencing (CLR/CCS) data: quality-filter aligned SAM/BAM files, extract deletions from CIGAR fields, encode sequences to 100x100-pixel images, run the dual-attention multi-scale ResNet caller, and benchmark F1/precision/recall against cuteSV, SVIM, Sniffles and PBSV on HG002.

## When to use

Use when calling deletion structural variants (>50 bp) from long-read alignments (SAM/BAM) of third-generation sequencing data and benchmarking caller performance against a truth set.

## When NOT to use

- The goal is insertion or base substitution (SNV) variant calling — the paper states 'This study currently focuses exclusively on experiments related to deletion variants calling'
- Input data are short NGS reads (e.g. Illumina ~250–300 bp) rather than third-generation long reads
- A versioned, production-ready caller is required — the authors note they 'aim to further develop and package this technology into user-friendly software' (future work)

## Inputs

- Aligned long-read SAM/BAM files from HG002 CLR and CCS datasets (GIAB AshkenazimTrio, BioProject PRJNA200694)
- Raw long-read sequencing data from the Ashkenazim Trio
- NIST Reference Material 8392 benchmark set (12,745 variants >50 bp, including 5464 deletions)
- Simulated datasets Sim_A{90X,65X,45X,20X} and Sim_B{90X,65X,45X,20X} (optional)

## Outputs

- Deletion calls (>50 bp) from DASV with coordinates derived from CIGAR-framed regions
- Deletion calls from cuteSV, SVIM, Sniffles and PBSV on the same datasets
- Table of precision, recall and F1 per tool per dataset and coverage depth (Supplementary Table 4 values)

## How to apply

Obtain HG002 raw long-read data and aligned BAM files from the GIAB AshkenazimTrio FTP repository (BioProject PRJNA200694). Quality-control the SAM files by removing fragments with MAPQ <20 and FLAG identifiers of 4 and >256, then frame the range of variation from the CIGAR fields of the controlled file. Encode sample genome bases (A/C/T/G as 2/3/4/5, deletion loci as 9, zero-padding to length 100) and generate 100x100-pixel images of high-frequency deletion regions using a sliding window of 100 and stride 50 (shorter images padded with whitespace, longer ones compressed). Run the dual-attention multi-scale ResNet model (image spatial attention plus sequence/image fusion via GAP/GMP) to call deletions, and run cuteSV, SVIM, Sniffles and PBSV on the same CLR and CCS datasets at each evaluated coverage depth. Compare all calls against the NIST Reference Material 8392 benchmark (5464 deletion callings >50 bp) and compute precision, recall and F1 per tool per coverage depth, recording values corresponding to the paper's Supplementary Table 4.

## Related tools

- **DASV** (Dual-attention multi-scale ResNet deletion caller under evaluation; converts alignment information into 100x100-pixel images and fuses them with genomic sequence data) — https://github.com/deconvolution-w/DASV
- **cuteSV** (Baseline third-generation SV calling tool for comparison on the same CLR/CCS HG002 datasets)
- **SVIM** (Baseline third-generation SV calling tool for comparison on the same datasets)
- **Sniffles** (Baseline third-generation SV calling tool for comparison on the same datasets)
- **PBSV** (Baseline third-generation SV calling tool for comparison on the same datasets)

## Examples

```
git clone https://github.com/deconvolution-w/DASV
```

## Evaluation signals

- DASV should achieve the highest F1 score across all evaluated real-data experiments versus cuteSV, SVIM, Sniffles and PBSV
- F1 should decrease as coverage depth decreases (e.g. across 90X, 65X, 45X, 20X settings)
- All computed deletion calls are >50 bp, consistent with the NIST RM 8392 benchmark definition
- Precision and recall should be balanced for DASV rather than trading one for the other
- Quality control removes all fragments with MAPQ <20 and FLAG 4 or >256 before CIGAR-based deletion framing

## Limitations

- Only deletion variants are covered; insertion and base substitution detection is future work
- No tool versions are stated for cuteSV, SVIM, Sniffles or PBSV, complicating exact reproduction
- Potential information loss during conversion into images due to the 100-sized sliding window (stride 50) is not quantified
- The software is not yet packaged as user-friendly tooling; only the source repository exists

## Evidence

- [abstract] DASV converts gene alignment information into images and integrates them with genomic sequencing data through a dual attention mechanism: "DASV converts gene alignment information into images and integrates them with genomic sequencing data through a dual attention mechanism"
- [methods] Sequenced fragments with an alignment quality number <20 were removed, as were those with a FLAG identifier of 4 and >256: "Sequenced fragments with an alignment quality number <20 were removed, as were those with a FLAG identifier of 4 and >256"
- [methods] the sliding window was set to 100 and the stride length was set to 50. Canvas length and width were set to 100 pixels: "the sliding window was set to 100 and the stride length was set to 50. Canvas length and width were set to 100 pixels"
- [methods] The benchmark set contains 12 745 independent variants >50 bp, including 7281 insertion callings and 5464 deletion callings: "The benchmark set contains 12 745 independent variants >50 bp, including 7281 insertion callings and 5464 deletion callings"
- [abstract] the results demonstrate that DASV consistently achieves a balance between precision and recall, enhancing the F1 score across various datasets: "the results demonstrate that DASV consistently achieves a balance between precision and recall, enhancing the F1 score across various datasets"
