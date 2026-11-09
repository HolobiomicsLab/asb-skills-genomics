---
name: deep-learning-image-sequence-fusion-for-deletion-detection
description: Use when calling deletion structural variants (>50 bp) from long-read
  (ONT/PacBio CLR or CCS) human genome alignments, especially when maximizing F1 balance
  between precision and recall against a GIAB/NIST benchmark.
when_to_use_negative:
- The goal is insertion or base substitution calling — the study focuses exclusively
  on deletion variants calling, with insertions/substitutions stated as future work
- Input data are Illumina short reads (250–300 bp) rather than third-generation long
  reads, for which the image/sequence fusion pipeline was designed
- Only low-coverage data are available and F1 must remain high — the paper reports
  F1 decreasing as coverage depth decreases
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3474
tools:
- name: DASV
  role: 'Primary pipeline: converts alignment information into images and fuses them
    with encoded sequences via a dual-attention multi-scale ResNet to call deletions'
  repo: https://github.com/deconvolution-w/DASV
- name: cuteSV
  role: Baseline deletion caller used for precision/recall/F1 comparison on the same
    HG002 datasets
- name: SVIM
  role: Baseline third-generation SV caller used for comparison
- name: Sniffles
  role: Baseline third-generation SV caller used for comparison
- name: PBSV
  role: Baseline third-generation SV caller used for comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae269
    title: Deletion variants calling in third-generation sequencing data based on
      a dual-attention mechanism
  source_content_hash: sha256:f9bf1d5d84eb1e10ef22306a96b175d1c7fb7734e9fe345cf4d075decdbf0eab
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
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# deep-learning-image-sequence-fusion-for-deletion-detection

## Summary

DASV, a deep-learning deletion SV caller for third-generation sequencing data that converts read-alignment information into 100x100-pixel images and fuses them with encoded genomic sequences via a dual-attention mechanism on a multi-scale ResNet backbone, outperforming cuteSV, SVIM, Sniffles and PBSV in F1 score.

## When to use

Use when calling deletion structural variants (>50 bp) from long-read (ONT/PacBio CLR or CCS) human genome alignments, especially when maximizing F1 balance between precision and recall against a GIAB/NIST benchmark.

## When NOT to use

- The goal is insertion or base substitution calling — the study focuses exclusively on deletion variants calling, with insertions/substitutions stated as future work
- Input data are Illumina short reads (250–300 bp) rather than third-generation long reads, for which the image/sequence fusion pipeline was designed
- Only low-coverage data are available and F1 must remain high — the paper reports F1 decreasing as coverage depth decreases

## Inputs

- Aligned long-read BAM/SAM files (e.g. HG002 CLR and CCS from GIAB AshkenazimTrio, PRJNA200694)
- Reference/sample genome base sequences from the pre-processed BAM file
- Truth-set VCF/benchmark (NIST Reference Material 8392) for evaluation

## Outputs

- Deletion SV calls (>50 bp) from the dual-attention multi-scale ResNet model
- 100x100-pixel alignment images of high-frequency deletion regions
- Precision, recall and F1 comparison table versus cuteSV, SVIM, Sniffles and PBSV

## How to apply

Start from aligned BAM/SAM files (e.g. HG002 from GIAB AshkenazimTrio, BioProject PRJNA200694) and quality-filter by removing fragments with MAPQ <20 and FLAG identifiers 4 and >256. Extract deletion features from the CIGAR field, count deletions per region and select high-frequency regions based on average sequencing depth (sliding window 100, stride 50). Encode sample-genome base sequences in the framed variation range (A/C/T/G as 2/3/4/5, deletion loci as 9, zero-padding to length 100) and render each region as a 100x100-pixel image, padding smaller images with whitespace and compressing larger ones. Run the dual-attention multi-scale ResNet model (spatial attention on the image, sequence/image fusion via GAP and GMP downsampling) to call deletions. Benchmark calls against the NIST Reference Material 8392 truth set (5464 deletion callings >50 bp) computing precision, recall and F1, expecting F1 to exceed cuteSV, SVIM, Sniffles and PBSV and to decrease with lower coverage depth.

## Related tools

- **DASV** (Primary pipeline: converts alignment information into images and fuses them with encoded sequences via a dual-attention multi-scale ResNet to call deletions) — https://github.com/deconvolution-w/DASV
- **cuteSV** (Baseline deletion caller used for precision/recall/F1 comparison on the same HG002 datasets)
- **SVIM** (Baseline third-generation SV caller used for comparison)
- **Sniffles** (Baseline third-generation SV caller used for comparison)
- **PBSV** (Baseline third-generation SV caller used for comparison)

## Evaluation signals

- DASV should achieve the highest F1 score across all evaluated real-data (CLR and CCS HG002) experiments versus cuteSV, SVIM, Sniffles and PBSV
- F1 values should decrease monotonically as coverage depth decreases (90X, 65X, 45X, 20X), and results should match Supplementary Table 4
- Precision and recall should remain balanced rather than strongly skewed, per the claimed 'balance between precision and recall'
- All called deletions should exceed the 50 bp threshold used in the NIST RM 8392 benchmark (5464 deletions >50 bp)
- Generated alignment images must be exactly 100x100 pixels (smaller padded with whitespace, larger compressed)

## Limitations

- The method covers only deletion variant calling; insertion and base substitution detection are future work
- Potential information loss arises from the 100-sized sliding window during image conversion, which is not quantified
- The F1 advantage is most notable on high-coverage data; performance degrades as coverage decreases
- Tool versions for the compared callers (cuteSV, SVIM, Sniffles, PBSV) are not stated, complicating exact reproduction

## Evidence

- [abstract] DASV converts gene alignment information into images and integrates them with genomic sequencing data through a dual attention mechanism: "DASV converts gene alignment information into images and integrates them with genomic sequencing data through a dual attention mechanism"
- [methods] Sequenced fragments with an alignment quality number <20 were removed, as were those with a FLAG identifier of 4 and >256: "Sequenced fragments with an alignment quality number <20 were removed, as were those with a FLAG identifier of 4 and >256"
- [methods] the sliding window was set to 100 and the stride length was set to 50. Canvas length and width were set to 100 pixels: "the sliding window was set to 100 and the stride length was set to 50. Canvas length and width were set to 100 pixels"
- [abstract] the results demonstrate that DASV consistently achieves a balance between precision and recall, enhancing the F1 score across various datasets: "the results demonstrate that DASV consistently achieves a balance between precision and recall, enhancing the F1 score across various datasets"
- [readme] Our method achieved the best results on all datasets.: "Our method achieved the best results on all datasets."
