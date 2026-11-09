---
name: ccs-read-5mc-detection
description: Use when you have PacBio CCS/HiFi reads (subreads.bam or hifi.bam) and
  need per-read 5mCpG methylation probabilities, or site-level CpG methylation frequencies
  validated against BS-seq/nanopore benchmarks.
when_to_use_negative:
- The input is Oxford Nanopore reads rather than PacBio CCS/HiFi reads — ccsmeth's
  models rely on PacBio-specific IPD/PW kinetics features, so nanopore-based callers
  (e.g., deepsignal2) are the appropriate choice.
- The goal is detecting hemimethylated CpGs, non-CpG 5mCs, or other modifications
  such as 6mA — the article states these methods only consider symmetric CpG methylation
  and cannot detect these states.
- Only very short CCS inserts are available — performance is reduced when CCS reads
  are shorter than ~10 Kb (the strand-specific model is significantly reduced due
  to limited subread depth).
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0622
tools:
- name: ccsmeth
  role: 'Primary tool: extracts per-read CpG IPD/PW kinetics features (count mode)
    and runs the read-level deep-learning model to call 5mCpG probabilities'
  repo: https://github.com/PengNi/ccsmeth
- name: ccsmethphase
  role: Nextflow pipeline used to aggregate read-level calls into site-level methylation-frequency
    bedmethyl files (mods_freq step)
  repo: https://github.com/PengNi/ccsmethphase
- name: pbmm2
  role: Aligns ccsmeth modbam files to the reference genome before site-level aggregation
  repo: https://github.com/PacificBiosciences/pbmm2
- name: deepsignal2
  role: Prior deep-learning method for nanopore methylation detection whose DNN structure
    relates to the ccsmeth model lineage; used for comparison
  repo: https://github.com/PengNi/deepsignal2
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-39784-9
    title: DNA 5-methylcytosine detection and methylation phasing using PacBio circular
      consensus sequencing
  source_content_hash: sha256:2d2b760e2c544338050c107a8d172858cd52cba98bfc6e1b43afa2490359f979
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PengNi/ccsmeth
  tools:
  - ccsmeth
  - ccsmethphase
  - deepsignal2
  - pbmm2
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# ccs-read-5mc-detection

## Summary

Detect 5-methylcytosine at single-molecule (per-read per-CpG) resolution using ccsmeth deep-learning models on PacBio circular consensus sequencing (HiFi) reads, exploiting IPD and PW kinetics features, then aggregate calls to site-level methylation frequencies with ccsmethphase for benchmarking against BS-seq and nanopore methylation.

## When to use

Use when you have PacBio CCS/HiFi reads (subreads.bam or hifi.bam) and need per-read 5mCpG methylation probabilities, or site-level CpG methylation frequencies validated against BS-seq/nanopore benchmarks.

## When NOT to use

- The input is Oxford Nanopore reads rather than PacBio CCS/HiFi reads — ccsmeth's models rely on PacBio-specific IPD/PW kinetics features, so nanopore-based callers (e.g., deepsignal2) are the appropriate choice.
- The goal is detecting hemimethylated CpGs, non-CpG 5mCs, or other modifications such as 6mA — the article states these methods only consider symmetric CpG methylation and cannot detect these states.
- Only very short CCS inserts are available — performance is reduced when CCS reads are shorter than ~10 Kb (the strand-specific model is significantly reduced due to limited subread depth).

## Inputs

- PacBio CCS reads (subreads.bam or hifi.bam), e.g., HG002 15/20/24 Kb libraries subsampled to 10x or total CCS reads (~71.0x), SD0651_P1 15 Kb, CHM13
- Reference genome FASTA (e.g., hg38)
- Pretrained ccsmeth 5mCpG models (from github.com/PengNi/ccsmeth)

## Outputs

- Per-read per-CpG 5mCpG methylation probability calls
- Modified-bam (modbam) files with 5mCpG probabilities
- Site-level methylation-frequency bedmethyl files (via ccsmethphase mods_freq)
- validation metrics such as Pearson correlations with BS-seq/nanopore (e.g., ccsmeth_site_level_correlations.csv) or read-level accuracy metrics (accuracy_metrics.tsv)

## How to apply

Run ccsmeth in count mode to extract per-read CpG kinetics features (IPD and PW) from the CCS reads, then apply the ccsmeth read-level deep-learning model (trained on PCR-treated unmethylated and M.SssI-methylated control DNA, using long >=10 Kb CCS reads) to obtain 5mCpG probabilities per read per CpG. Convert the output to modified-bam (modbam) files, align them to the reference genome (e.g., hg38) with pbmm2, and aggregate read-level calls into site-level methylation-frequency bedmethyl files using the ccsmethphase mods_freq step. Optionally filter ambiguous calls using the delta-p confidence margin ∆p = |Pr − P'r| with P'r = 1 − Pr: discarding calls with ∆p below 0.33 improves read-level accuracy by 3.5–4.2% at the cost of dropping 8.9–12.8% of calls. Validate by intersecting CpG methylation frequencies with matched BS-seq (HG002, SD0651_P1) or nanopore (CHM13) calls and computing Pearson correlations — expect >0.90 even at 10x coverage, e.g., 0.9198/0.9083/0.9087 for 10x HG002 15/20/24 Kb vs BS-seq and 0.9463/0.9271/0.9410 for total-coverage HG002.

## Related tools

- **ccsmeth** (Primary tool: extracts per-read CpG IPD/PW kinetics features (count mode) and runs the read-level deep-learning model to call 5mCpG probabilities) — https://github.com/PengNi/ccsmeth
- **ccsmethphase** (Nextflow pipeline used to aggregate read-level calls into site-level methylation-frequency bedmethyl files (mods_freq step)) — https://github.com/PengNi/ccsmethphase
- **pbmm2** (Aligns ccsmeth modbam files to the reference genome before site-level aggregation) — https://github.com/PacificBiosciences/pbmm2
- **deepsignal2** (Prior deep-learning method for nanopore methylation detection whose DNN structure relates to the ccsmeth model lineage; used for comparison) — https://github.com/PengNi/deepsignal2

## Evaluation signals

- Read-level accuracy on long (>=10 Kb) CCS reads reaches ~0.90 accuracy and 0.97 AUC at single-molecule resolution on control DNA with known methylation states.
- Site-level Pearson correlation with BS-seq exceeds 0.90 even with only 10x reads (0.9198/0.9083/0.9087 for HG002 15/20/24 Kb); total-coverage HG002 gives 0.9463/0.9271/0.9410 vs BS-seq.
- After delta-p filtering at ∆p = 0.33, read-level accuracy improves by 3.5–4.2 percentage points while 8.9–12.8% of calls are discarded.
- Output modbam files align cleanly with pbmm2 and mods_freq produces bedmethyl files with per-CpG methylation frequencies on matched CpG sites.
- ccsmeth outperforms existing CNN-based methods (HK model, primrose) on both read-level and site-level 5mC prediction, especially under low coverage.

## Limitations

- Methods considering only symmetric methylation cannot detect hemimethylated CpGs.
- The approach cannot be applied to 5mC detection in non-CpGs or other DNA modifications such as 6mA; applicability of the site-level model design to non-CpG 5mCs is unverified.
- The strand-specific-methylation model's performance is significantly reduced due to limited subread depth.
- Prior-work context: existing SMRT-based methods require relatively high CCS subread depth (HK model needs at least 20x passed subreads per CCS), motivating long-insert CCS libraries.

## Evidence

- [abstract] Read-level accuracy and AUC benchmarks on long CCS reads: "Using long (≥10 Kb) CCS reads, ccsmeth achieves 0.90 accuracy and 0.97 Area Under the Curve on 5mCpG detection at single-molecule resolution."
- [abstract] Site-level correlation benchmark at low coverage: "ccsmeth achieves >0.90 correlations with bisulfite sequencing and nanopore sequencing using only 10× reads"
- [discussion] Expected accuracy for the strand-specific model variant: "The strand-specific-methylation model achieves 0.85 accuracy in the HG002 15 Kb dataset at read level"
- [methods] Acceptable input file types for the pipeline: "ccsmethphase takes files of PacBio reads (subreads.bam or hifi.bam), and a reference genome as input."
- [abstract] Deep-learning basis of the method on CCS reads: "we present ccsmeth, a deep-learning method to detect DNA 5mCpGs using CCS reads"
