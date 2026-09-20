---
name: pacbio-ccs-read-level-5mc-classification
description: Use when calling 5mCpG methylation states at single-molecule (per-read)
  resolution from long (≥10 Kb) PacBio CCS/HiFi reads (subreads.bam or hifi.bam),
  or when benchmarking read-level 5mC classifiers on M.SssI-methylated vs PCR-treated
  control datasets.
when_to_use_negative:
- The goal is detection of hemimethylated CpGs — ccsmeth and similar methods only
  consider symmetric methylation and are incapable of detecting hemimethylated CpGs
- The target is 5mC at non-CpG contexts or other DNA modifications such as 6mA — ccsmeth's
  read-level model is designed for 5mCpG only
- Only genome-wide aggregated site-level methylation frequencies (e.g. bedmethyl)
  are needed — a site-level model or pileup tool is more appropriate than per-read
  classification
edam_topics:
- http://edamontology.org/topic_0657
- http://edamontology.org/topic_0102
tools:
- name: ccsmeth
  role: Primary deep-learning method that detects 5mCpGs from IPD/PW kinetics features
    of PacBio CCS reads at read level; its 5mCpG models serve as the model framework
  repo: https://github.com/PengNi/ccsmeth
- name: HK model
  role: CNN-based holistic kinetic baseline trained on the same 50% training split
    and evaluated head-to-head on the held-out test reads for read-level 5mCpG accuracy
    and AUC
- name: primrose
  role: PacBio's CNN-based 5mCpG detection baseline for comparison (claimed 85% read-level
    accuracy on long PacBio CCS reads)
- name: pbmm2
  role: Aligns PacBio CCS subreads.bam/hifi.bam reads to the reference genome prior
    to methylation calling
  repo: https://github.com/PacificBiosciences/pbmm2
- name: ccsmethphase
  role: Downstream Nextflow pipeline that consumes ccsmeth modbam outputs together
    with PacBio reads (subreads.bam or hifi.bam) and a reference genome for haplotype-aware
    methylation
  repo: https://github.com/PengNi/ccsmethphase
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-39784-9
    title: DNA 5-methylcytosine detection and methylation phasing using PacBio circular
      consensus sequencing
  source_content_hash: sha256:27f3b75c9672f95b905fee62296d809b8b3eedc4f8ba3e7b258d7a75bc38e9f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PengNi/ccsmeth
  tools:
  - HK model
  - ccsmeth
  - ccsmethphase
  - pbmm2
  - primrose
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0657
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# pacbio-ccs-read-level-5mc-classification

## Summary

Detect 5-methylcytosine at CpG sites (5mCpG) per single read using a deep-learning model (ccsmeth) trained on kinetics features (IPDs and PWs) of PacBio circular consensus sequencing (CCS) reads, benchmarked against CNN-based baselines such as the HK model and primrose.

## When to use

Use when calling 5mCpG methylation states at single-molecule (per-read) resolution from long (≥10 Kb) PacBio CCS/HiFi reads (subreads.bam or hifi.bam), or when benchmarking read-level 5mC classifiers on M.SssI-methylated vs PCR-treated control datasets.

## When NOT to use

- The goal is detection of hemimethylated CpGs — ccsmeth and similar methods only consider symmetric methylation and are incapable of detecting hemimethylated CpGs
- The target is 5mC at non-CpG contexts or other DNA modifications such as 6mA — ccsmeth's read-level model is designed for 5mCpG only
- Only genome-wide aggregated site-level methylation frequencies (e.g. bedmethyl) are needed — a site-level model or pileup tool is more appropriate than per-read classification

## Inputs

- Aligned PacBio CCS reads (subreads.bam or hifi.bam) mapped to a reference genome, e.g. via pbmm2
- M.SssI-methyltransferase-treated and PCR-treated paired CCS datasets (e.g. M01&W01, M02&W02, M03&W03) for training/evaluation
- input_sheet.tsv with Group_ID, Sample_ID, Type, and Path columns
- Reference genome FASTA
- ccsmeth 5mCpG pre-trained models (github.com/PengNi/ccsmeth)

## Outputs

- Per-read per-CpG 5mCpG methylation calls (methylated/unmethylated probabilities) at single-molecule resolution
- Read-level classification accuracy and AUC per dataset pair (expected: 0.9232, 0.8788, 0.8765 on M01&W01, M02&W02, M03&W03)
- Benchmark comparison table ccsmeth_read_level_benchmark.csv (ccsmeth vs HK model, AUC margins 5.4%, 4.4%, 3.7%)
- modbam files when converting calls for downstream use

## How to apply

Extract kinetics features — interpulse durations (IPDs) and pulse widths (PWs) — from aligned PacBio CCS reads and feed them to the ccsmeth deep-learning model, which considers symmetric methylation and combines features from both DNA strands around CpG motifs. Train on polymerase-chain-reaction treated and M.SssI-methyltransferase-treated DNA (e.g. dataset pairs M01&W01, M02&W02, M03&W03 organized via an input_sheet.tsv with Group_ID, Sample_ID, Type, and Path columns), using ccsmeth 5mCpG models as the model framework and a 50/50 read-level train/test split. Align reads to the reference genome with pbmm2 before feature extraction. For benchmarking, train the CNN-based HK (holistic kinetic) model on the identical 50% training split and evaluate both models on the held-out test reads, computing per-read accuracy and AUC for each dataset pair. Validate against the reported read-level accuracies of 0.9232, 0.8788, and 0.8765 on M01&W01, M02&W02, and M03&W03, which exceed the HK model by 5.4%, 4.4%, and 3.7% respectively; on long reads overall ccsmeth is expected to reach ~0.90 accuracy and ~0.97 AUC at single-molecule resolution. Write all comparison metrics to a results table (e.g. ccsmeth_read_level_benchmark.csv).

## Related tools

- **ccsmeth** (Primary deep-learning method that detects 5mCpGs from IPD/PW kinetics features of PacBio CCS reads at read level; its 5mCpG models serve as the model framework) — https://github.com/PengNi/ccsmeth
- **HK model** (CNN-based holistic kinetic baseline trained on the same 50% training split and evaluated head-to-head on the held-out test reads for read-level 5mCpG accuracy and AUC)
- **primrose** (PacBio's CNN-based 5mCpG detection baseline for comparison (claimed 85% read-level accuracy on long PacBio CCS reads))
- **pbmm2** (Aligns PacBio CCS subreads.bam/hifi.bam reads to the reference genome prior to methylation calling) — https://github.com/PacificBiosciences/pbmm2
- **ccsmethphase** (Downstream Nextflow pipeline that consumes ccsmeth modbam outputs together with PacBio reads (subreads.bam or hifi.bam) and a reference genome for haplotype-aware methylation) — https://github.com/PengNi/ccsmethphase

## Evaluation signals

- Reproduced read-level accuracies on M01&W01, M02&W02, M03&W03 match 0.9232, 0.8788, 0.8765 within a small tolerance
- ccsmeth AUC exceeds the HK model's by approximately 5.4%, 4.4%, and 3.7% on the three dataset pairs respectively
- On long (≥10 Kb) CCS reads, aggregate read-level accuracy ≈0.90 and AUC ≈0.97 at single-molecule resolution
- Train/test split is strictly 50/50 at the read level within each M.SssI/PCR dataset pair, with both models trained on the identical training half
- Predictions restricted to CpG motifs, with features combined from both DNA strands (symmetric methylation assumption)

## Limitations

- The HK-model baseline requires relatively high CCS subread depth (at least 20× passed subreads for one CCS) for accurate 5mCpG detection, which constrains insert size and read length
- These methods are incapable of detecting hemimethylated CpGs because they only consider symmetric methylation
- Cannot be applied to 5mC detection in non-CpGs or to other DNA modifications such as 6mA
- The strand-specific-methylation model variant has significantly reduced performance due to limited subread depth
- Whether the site-level model design applies to non-CpG 5mCs or other modifications has not been verified

## Evidence

- [abstract] read-level accuracy on long CCS reads: "Using long (≥10 Kb) CCS reads, ccsmeth achieves 0.90 accuracy and 0.97 Area Under the Curve on 5mCpG detection at single-molecule resolution."
- [intro] feature basis of ccsmeth: "we propose ccsmeth, a deep-learning method to detect DNA 5mCpGs by using kinetics features (IPDs and PWs) of PacBio CCS reads"
- [discussion] baseline comparison: "ccsmeth outperforms two existing CNN-based methods on both read-level and site-level 5mC prediction."
- [discussion] symmetric methylation assumption: "all these methods only consider symmetric methylation and combine the features from both DNA strands to predict the methylation states."
- [intro] HK model baseline limitation: "the HK model requires relatively high CCS subread depth (at least 20× passed subreads for one CCS) for accurate 5mCpG detection"
- [intro] primrose baseline accuracy: "primrose25, which has been claimed to have 85% read-level accuracy on 5mCpG detection from long PacBio CCS reads"
- [methods] input file types: "ccsmethphase takes files of PacBio reads (subreads.bam or hifi.bam), and a reference genome as input."
