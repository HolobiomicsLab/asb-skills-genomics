---
name: sv-call-benchmarking-against-nist-truth-set
description: Use when evaluating deletion SV calling performance on third-generation
  sequencing data with a known truth set, or when reproducing precision/recall/F1
  comparisons across multiple callers on HG002 GIAB samples.
when_to_use_negative:
- The goal is insertion or base substitution calling — the article states 'This study
  currently focuses exclusively on experiments related to deletion variants calling'.
- No truth set (e.g. NIST RM 8392) is available, so precision/recall/F1 cannot be
  computed against a benchmark.
- Input data are short-read (Illumina-style NGS) rather than third-generation long
  reads, which the evaluated callers were designed for.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: DASV
  role: 'Method under evaluation: converts SAM alignment information into images,
    applies a dual-attention multi-scale ResNet to call deletion variants'
  repo: https://github.com/deconvolution-w/DASV
- name: cuteSV
  role: Baseline deletion caller compared against DASV on the same HG002 datasets
- name: SVIM
  role: Baseline deletion caller compared against DASV on the same HG002 datasets
- name: Sniffles
  role: Baseline deletion caller compared against DASV on the same HG002 datasets
- name: PBSV
  role: Baseline deletion caller compared against DASV on the same HG002 datasets
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae269
    title: Deletion variants calling in third-generation sequencing data based on
      a dual-attention mechanism
  source_content_hash: sha256:b73ca949048f86447934f78ddb96606adac8a047698589dfffbe8c5872dd1080
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
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# SV call benchmarking against NIST truth set

## Summary

Benchmark structural-variant deletion callers (DASV, cuteSV, SVIM, Sniffles, PBSV) on real GIAB HG002 long-read data by comparing each tool's deletion calls against the NIST Reference Material 8392 benchmark and computing precision, recall and F1 across coverage depths and CLR/CCS platforms.

## When to use

Use when evaluating deletion SV calling performance on third-generation sequencing data with a known truth set, or when reproducing precision/recall/F1 comparisons across multiple callers on HG002 GIAB samples.

## When NOT to use

- The goal is insertion or base substitution calling — the article states 'This study currently focuses exclusively on experiments related to deletion variants calling'.
- No truth set (e.g. NIST RM 8392) is available, so precision/recall/F1 cannot be computed against a benchmark.
- Input data are short-read (Illumina-style NGS) rather than third-generation long reads, which the evaluated callers were designed for.

## Inputs

- HG002 raw long-read sequencing data and aligned BAM files from GIAB AshkenazimTrio (BioProject PRJNA200694)
- CLR and CCS HG002 datasets at multiple coverage depths
- NIST Reference Material 8392 benchmark VCF (12745 variants >50 bp, including 5464 deletions)

## Outputs

- Per-tool deletion call sets (DASV, cuteSV, SVIM, Sniffles, PBSV) per dataset and coverage depth
- Precision, recall and F1 score table per tool per CLR/CCS dataset and depth (corresponding to Supplementary Table 4)

## How to apply

Download HG002 raw long-read data and aligned BAM files from the GIAB AshkenazimTrio FTP repository (BioProject PRJNA200694). Run DASV by quality-filtering SAM files (removing fragments with MAPQ <20 and FLAG identifiers 4 or >256), extracting deletion features from CIGAR fields, encoding bases (A/C/T/G as 2/3/4/5, deletion loci as 9, zero-padding to length 100), generating 100x100-pixel images of high-frequency regions (sliding window 100, stride 50), and running the dual-attention multi-scale ResNet model. Run cuteSV, SVIM, Sniffles and PBSV on the same CLR and CCS datasets at the evaluated coverage depths. Compare each tool's deletion calls against the NIST Reference Material 8392 benchmark, which contains 5464 deletion callings >50 bp, then compute precision, recall and F1 per tool per platform and depth. Correct application is confirmed when DASV achieves the highest F1 across all evaluated real-data experiments, with F1 decreasing as coverage depth decreases.

## Related tools

- **DASV** (Method under evaluation: converts SAM alignment information into images, applies a dual-attention multi-scale ResNet to call deletion variants) — https://github.com/deconvolution-w/DASV
- **cuteSV** (Baseline deletion caller compared against DASV on the same HG002 datasets)
- **SVIM** (Baseline deletion caller compared against DASV on the same HG002 datasets)
- **Sniffles** (Baseline deletion caller compared against DASV on the same HG002 datasets)
- **PBSV** (Baseline deletion caller compared against DASV on the same HG002 datasets)

## Evaluation signals

- DASV achieves the highest F1 score across all evaluated real-data experiments versus cuteSV, SVIM, Sniffles and PBSV
- F1 score decreases monotonically as coverage depth decreases across evaluated depths
- Each tool's calls are validated against the NIST Reference Material 8392 set of 5464 deletions >50 bp
- Recomputed precision and recall show DASV consistently balances precision and recall rather than trading one for the other
- Recorded values match the paper's Supplementary Table 4

## Limitations

- No versions are stated for the compared tools (cuteSV, SVIM, Sniffles, PBSV), complicating exact reproduction
- The study covers only deletion variant calling; insertion and base substitution detection is left as future work
- Potential information loss during conversion into images, primarily due to the use of 100-sized sliding windows, is not quantified

## Evidence

- [abstract] the results demonstrate that DASV consistently achieves a balance between precision and recall, enhancing the F1 score across various datasets: "the results demonstrate that DASV consistently achieves a balance between precision and recall, enhancing the F1 score across various datasets"
- [methods] Deletion features were obtained based on the CIGAR field of the quality-controlled SAM file: "Deletion features were obtained based on the CIGAR field of the quality-controlled SAM file"
- [methods] Sequenced fragments with an alignment quality number <20 were removed, as were those with a FLAG identifier of 4 and >256: "Sequenced fragments with an alignment quality number <20 were removed, as were those with a FLAG identifier of 4 and >256"
- [methods] available as NIST Reference Material 8392. The benchmark set contains 12 745 independent variants >50 bp, including 7281 insertion callings and 5464 deletion callings: "The benchmark set contains 12 745 independent variants >50 bp, including 7281 insertion callings and 5464 deletion callings"
- [discussion] This study currently focuses exclusively on experiments related to deletion variants calling: "This study currently focuses exclusively on experiments related to deletion variants calling"
