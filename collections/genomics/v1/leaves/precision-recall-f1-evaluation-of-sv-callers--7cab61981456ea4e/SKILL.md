---
name: precision-recall-f1-evaluation-of-sv-callers
description: Use when evaluating or reproducing deletion SV caller performance on
  long-read data (simulated Sim_A/Sim_B at 90x/65x/45x/20x, or HG002 real data) where
  F1 score against a truth set must be computed and compared across tools.
when_to_use_negative:
- The target variant type is insertion or base substitution — the study explicitly
  focuses exclusively on deletion variant calling
- The reads are short Illumina-style NGS reads rather than third-generation long reads
  (PacBio SMRT/CLR/CCS), for which the compared tools and DASV were designed
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: DASV
  role: Candidate deletion SV caller under evaluation; dual-attention feature fusion
    pipeline run on quality-filtered BAM files
  repo: https://github.com/deconvolution-w/DASV
- name: cuteSV
  role: Baseline third-generation SV caller benchmarked on the same BAM files
- name: SVIM
  role: Baseline third-generation SV caller benchmarked on the same BAM files
- name: Sniffles
  role: Baseline third-generation SV caller benchmarked on the same BAM files
- name: PBSV
  role: Baseline third-generation SV caller benchmarked on the same BAM files
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae269
    title: Deletion variants calling in third-generation sequencing data based on
      a dual-attention mechanism
  source_content_hash: sha256:8526a9ce0adb6fafd6958871d1123259074a8f8b5a7a6da542b1ac63f5a42c2b
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# precision-recall-f1-evaluation-of-sv-callers

## Summary

Benchmark deletion structural variant callers (DASV versus cuteSV, SVIM, Sniffles, PBSV) by comparing each tool's VCF-style calls against a known truth set and computing precision, recall and F1 score per dataset and coverage depth.

## When to use

Use when evaluating or reproducing deletion SV caller performance on long-read data (simulated Sim_A/Sim_B at 90x/65x/45x/20x, or HG002 real data) where F1 score against a truth set must be computed and compared across tools.

## When NOT to use

- The target variant type is insertion or base substitution — the study explicitly focuses exclusively on deletion variant calling
- The reads are short Illumina-style NGS reads rather than third-generation long reads (PacBio SMRT/CLR/CCS), for which the compared tools and DASV were designed

## Inputs

- NGMLR-aligned BAM/SAM files from PacBio SMRT long reads (CLR and CCS HG002 datasets, or simulated Sim_A/Sim_B FASTQ-derived alignments)
- NIST Reference Material 8392 benchmark deletion truth set (5464 deletions >50 bp)
- Simulated truth sets from SURVIVOR parameter files (deletion lengths 50-1000 bp and 1000-5000 bp, homozygous and heterozygous groups)

## Outputs

- Per-tool precision, recall and F1 score at each coverage depth (90x/65x/45x/20x) and deletion length band
- simulated_deletion_calling_results.csv with per-tool metrics mirroring Supplementary Tables 7(a&b) and 8(a&b)
- Real-data comparison table corresponding to Supplementary Table 4

## How to apply

Run each caller — DASV (SAM quality filtering with MAPQ <20 and FLAG 4 or >256 removal, CIGAR-based image generation with 100-pixel sliding window/stride 50, dual-attention multi-scale ResNet) plus the baselines cuteSV, SVIM, Sniffles and PBSV — on identical BAM files produced by aligning reads with NGMLR. For real data, compare calls against the NIST Reference Material 8392 benchmark containing 5464 deletion callings >50 bp from GIAB BioProject PRJNA200694 (HG002 AshkenazimTrio); for simulated data, compare against the SURVIVOR/PaSS truth set of homozygous and heterozygous deletions of 50-1000 bp and 1000-5000 bp. Compute precision, recall and F1 per tool, per coverage depth (90x, 65x, 45x, 20x) and per deletion length band, mirroring Supplementary Tables 4 and 7(a&b)/8(a&b). Judge correctness by verifying DASV attains the highest F1 across tools, with F1 decreasing as coverage decreases and recall of at least 82.16% reported at 20x.

## Related tools

- **DASV** (Candidate deletion SV caller under evaluation; dual-attention feature fusion pipeline run on quality-filtered BAM files) — https://github.com/deconvolution-w/DASV
- **cuteSV** (Baseline third-generation SV caller benchmarked on the same BAM files)
- **SVIM** (Baseline third-generation SV caller benchmarked on the same BAM files)
- **Sniffles** (Baseline third-generation SV caller benchmarked on the same BAM files)
- **PBSV** (Baseline third-generation SV caller benchmarked on the same BAM files)

## Examples

```
git clone https://github.com/deconvolution-w/DASV
```

## Evaluation signals

- DASV attains the highest F1 score of all five tools across coverage depths and deletion lengths
- Recall on low-coverage simulated data is at least 82.16%, as reported for the 20x condition
- F1 values decrease monotonically as coverage depth decreases on real HG002 CLR/CCS data, matching Supplementary Table 4
- Metrics are computed per coverage depth (90x/65x/45x/20x) and deletion length band (50-1000 bp, 1000-5000 bp), matching Supplementary Tables 7(a&b) and 8(a&b) structure
- All five callers are run on identical NGMLR-aligned BAM files so precision/recall/F1 differences are attributable to the callers

## Limitations

- No versions stated for the compared tools (cuteSV, SVIM, Sniffles, PBSV)
- The study currently covers only deletion variant calling; insertion and base substitution detection is future work
- Information loss from the 100-sized sliding window image conversion is not quantified

## Evidence

- [abstract] Compared with four widely used genome structural variation calling tools: cuteSV, SVIM, Sniffles and PBSV: "Compared with four widely used genome structural variation calling tools: cuteSV, SVIM, Sniffles and PBSV"
- [methods] The benchmark set contains 12 745 independent variants >50 bp, including 7281 insertion callings and 5464 deletion callings: "available as NIST Reference Material 8392. The benchmark set contains 12 745 independent variants >50 bp, including 7281 insertion callings and 5464 deletion callings"
- [methods] Sim_A90X,65X,45X,20X and Sim_B90X,65X,45X,20X were simulated datasets: "Sim_A90X,65X,45X,20X and Sim_B90X,65X,45X,20X were simulated datasets of homozygous and heterozygous variant groups"
- [intro] DASV has notable advantages for data with high coverage depth: "we compared DASV with the advanced third-generation variation detection tools, SVIM, Sniffles, PBSV and cuteSV. The results showed that DASV has notable advantages for data with high coverage depth"
- [readme] Our method achieved the best results on all datasets.: "Our method achieved the best results on all datasets."
