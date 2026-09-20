---
name: deletion-variant-benchmark-evaluation
description: Use when evaluating or reproducing a comparison of deletion SV callers
  on long-read BAM alignments, where per-tool precision/recall/F1 must be computed
  against known truth sets at varying coverage depths.
when_to_use_negative:
- The variant class of interest is insertion or base substitution — the article states
  the study 'currently focuses exclusively on experiments related to deletion variants
  calling'
- Input is short-read Illumina data — DASV's premise relies on the long-read advantages
  of third-generation sequencing data
- No truth set or benchmark calls are available, so precision/recall/F1 cannot be
  computed against a ground truth
edam_operation: http://edamontology.org/operation_3226
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: DASV
  role: Focal deletion SV caller under evaluation; converts CIGAR alignment information
    to 100x100 images, fuses them with encoded sequences via dual attention and a
    ResNet backbone
  repo: https://github.com/deconvolution-w/DASV
- name: cuteSV
  role: Baseline third-generation SV calling tool compared against DASV on the same
    BAM files
- name: SVIM
  role: Baseline third-generation SV calling tool compared against DASV on the same
    BAM files
- name: Sniffles
  role: Baseline third-generation SV calling tool compared against DASV on the same
    BAM files
- name: PBSV
  role: Baseline third-generation SV calling tool compared against DASV on the same
    BAM files
- name: NGMLR
  role: Aligner used to map simulated FASTQ long reads to the reference genome to
    obtain BAM inputs
- name: SURVIVOR
  role: Generates simulation parameter files specifying type, quantity and length
    of simulated deletion variants
- name: PaSS
  role: Simulates PacBio SMRT long reads at 90x/65x/45x/20x coverage (20k bp mean
    read length, 15% error rate)
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae269
    title: Deletion variants calling in third-generation sequencing data based on
      a dual-attention mechanism
  source_content_hash: sha256:12e7f847c5c69e08eb52ef0816e7a8d3a34fa43df5886b5ebc82032b7dfb376a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/deconvolution-w/DASV
  tools:
  - DASV
  - NGMLR
  - PBSV
  - PaSS
  - SURVIVOR
  - SVIM
  - Sniffles
  - cuteSV
  edam_operation: http://edamontology.org/operation_3226
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# deletion-variant-benchmark-evaluation

## Summary

Benchmark DASV against cuteSV, SVIM, Sniffles and PBSV for deletion structural variant calling from third-generation sequencing alignments, scoring precision, recall and F1 against truth sets across simulated (Sim_A/Sim_B at 90x/65x/45x/20x) and real (GIAB HG002 / NIST RM 8392) datasets.

## When to use

Use when evaluating or reproducing a comparison of deletion SV callers on long-read BAM alignments, where per-tool precision/recall/F1 must be computed against known truth sets at varying coverage depths.

## When NOT to use

- The variant class of interest is insertion or base substitution — the article states the study 'currently focuses exclusively on experiments related to deletion variants calling'
- Input is short-read Illumina data — DASV's premise relies on the long-read advantages of third-generation sequencing data
- No truth set or benchmark calls are available, so precision/recall/F1 cannot be computed against a ground truth

## Inputs

- FASTQ files of simulated PacBio SMRT long reads (Sim_A{90X,65X,45X,20X}, Sim_B{90X,65X,45X,20X})
- BAM alignment files from NGMLR against the reference genome
- Reference genome FASTA
- GIAB Ashkenazim Trio (HG002) real sequencing data and aligned files
- NIST Reference Material 8392 benchmark truth set (12,745 variants >50 bp, incl. 5,464 deletions)
- VCF call sets from cuteSV, SVIM, Sniffles, PBSV and DASV

## Outputs

- Per-tool precision, recall and F1 score per coverage depth and deletion length
- simulated_deletion_calling_results.csv with metrics for all five tools
- DASV deletion-variant predictions per 100x100-pixel region for HG002 validation

## How to apply

Simulate or obtain long-read data: generate SURVIVOR simulation parameters for deletions of 50-1000 bp and 1000-5000 bp (homozygous and heterozygous), simulate PacBio SMRT reads with PaSS at 90x/65x/45x/20x coverage (20k bp mean read length, 15% error rate) to yield Sim_A and Sim_B FASTQ sets, then align with NGMLR to BAM. Run DASV with its SAM quality filtering (alignment quality <20 removed, FLAG 4 or >256 removed), CIGAR-based 100x100-pixel image generation (sliding window 100, stride 50) and dual-attention ResNet fusion; run cuteSV, SVIM, Sniffles and PBSV on the identical BAM files. For the real-data benchmark, use GIAB Ashkenazim Trio (HG002) data with the NIST Reference Material 8392 benchmark set containing 5464 deletion callings. Compare each tool's VCF calls to the truth set to compute precision, recall and F1 per coverage depth and deletion length, and write per-tool metrics to a results CSV (simulated_deletion_calling_results.csv).

## Related tools

- **DASV** (Focal deletion SV caller under evaluation; converts CIGAR alignment information to 100x100 images, fuses them with encoded sequences via dual attention and a ResNet backbone) — https://github.com/deconvolution-w/DASV
- **cuteSV** (Baseline third-generation SV calling tool compared against DASV on the same BAM files)
- **SVIM** (Baseline third-generation SV calling tool compared against DASV on the same BAM files)
- **Sniffles** (Baseline third-generation SV calling tool compared against DASV on the same BAM files)
- **PBSV** (Baseline third-generation SV calling tool compared against DASV on the same BAM files)
- **NGMLR** (Aligner used to map simulated FASTQ long reads to the reference genome to obtain BAM inputs)
- **SURVIVOR** (Generates simulation parameter files specifying type, quantity and length of simulated deletion variants)
- **PaSS** (Simulates PacBio SMRT long reads at 90x/65x/45x/20x coverage (20k bp mean read length, 15% error rate))

## Examples

```
ngmlr -r reference.fa -q Sim_A90X.fastq -o Sim_A90X.bam
```

## Evaluation signals

- DASV attains the highest F1 score of all five tools across coverage depths and deletion lengths on simulated data (Supplementary Tables 7a&b, 8a&b)
- DASV achieves recall of at least 82.16% on the low-coverage (20x) simulated condition
- On HG002 validation, feature fusion improves F1 over sequence-only features (Supplementary Table 3)
- All reported precision/recall/F1 values lie in [0,1] and are internally consistent when recomputed from the benchmark comparison (Supplementary Table 4)
- The real-data benchmark resolves against the NIST RM 8392 set of 5,464 deletion callings

## Limitations

- The study covers only deletion calling; insertion and base substitution detection is deferred to future work
- No versions are stated for the compared tools (cuteSV, SVIM, Sniffles, PBSV)
- The 100-sized sliding window conversion into images may cause information loss that is not quantified
- No data availability statement or planned software packaging in the discussion section; use the GitHub repo for code

## Evidence

- [abstract] Compared with four widely used genome structural variation calling tools: cuteSV, SVIM, Sniffles and PBSV: "Compared with four widely used genome structural variation calling tools: cuteSV, SVIM, Sniffles and PBSV"
- [methods] Sim_A and Sim_B dataset construction: "Sim_A90X,65X,45X,20X and Sim_B90X,65X,45X,20X were simulated datasets of homozygous and heterozygous variant groups"
- [methods] NIST benchmark truth set: "available as NIST Reference Material 8392. The benchmark set contains 12 745 independent variants >50 bp, including 7281 insertion callings and 5464 deletion callings"
- [intro] DASV advantage on high-coverage data: "we compared DASV with the advanced third-generation variation detection tools, SVIM, Sniffles, PBSV and cuteSV. The results showed that DASV has notable advantages for data with high coverage depth"
- [readme] README claim of best results: "Our method achieved the best results on all datasets. And we have submitted the manuscript to"
