---
name: long-read-genomics-benchmarking
description: Use when two long-read methylation/modification callers must be compared
  on the same aligned HiFi BAMs, with per-CpG read-level predictions scored against
  BS-seq labels to produce accuracy/sensitivity/specificity/AUC tables.
when_to_use_negative:
- The modification of interest is not 5mCpG — the models consider symmetric CpG methylation
  only and cannot detect hemimethylated CpGs, non-CpG 5mCs, or other modifications
  such as 6mA
- Site-level (aggregate) methylation benchmarking is the goal — this skill targets
  single-molecule read-level evaluation; site-level comparisons use different models
  and correlation-to-BS-seq metrics
- Input reads are nanopore or short-read — the ccsmeth/primrose comparison is specific
  to PacBio CCS kinetics (IPD/PW) and HiFi reads
edam_operation: http://edamontology.org/operation_3207
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0622
tools:
- name: ccsmeth
  role: deep-learning 5mCpG detector run in read-level mode using IPD/pulse-width
    kinetics features of PacBio CCS reads
  repo: https://github.com/PengNi/ccsmeth
- name: primrose
  role: PacBio CNN-based 5mCpG caller used as the read-level comparison baseline on
    the same aligned CCS BAMs
- name: pbmm2
  role: aligner used to produce the sorted HiFi BAM alignments that the benchmarked
    callers consume
  repo: https://github.com/PacificBiosciences/pbmm2
- name: pb-CpG-tools
  role: PacBio companion scripts (aligned_bam_to_cpg_scores) for generating site methylation
    probabilities from mapped HiFi reads
  repo: https://github.com/PacificBiosciences/pb-CpG-tools
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-39784-9
    title: DNA 5-methylcytosine detection and methylation phasing using PacBio circular
      consensus sequencing
  source_content_hash: sha256:25dacdce0bb6636d478774065cb22747735ebd4a42e4a34b5507f34a89257f54
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PengNi/ccsmeth
  tools:
  - ccsmeth
  - pb-CpG-tools
  - pbmm2
  - primrose
  edam_operation: http://edamontology.org/operation_3207
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# long-read-genomics-benchmarking

## Summary

Benchmarking skill for head-to-head evaluation of long-read methylation callers (ccsmeth vs primrose) on PacBio HiFi/CCS reads, scoring read-level 5mCpG predictions against bisulfite-sequencing ground truth with accuracy, sensitivity, specificity, and ROC AUC.

## When to use

Use when two long-read methylation/modification callers must be compared on the same aligned HiFi BAMs, with per-CpG read-level predictions scored against BS-seq labels to produce accuracy/sensitivity/specificity/AUC tables.

## When NOT to use

- The modification of interest is not 5mCpG — the models consider symmetric CpG methylation only and cannot detect hemimethylated CpGs, non-CpG 5mCs, or other modifications such as 6mA
- Site-level (aggregate) methylation benchmarking is the goal — this skill targets single-molecule read-level evaluation; site-level comparisons use different models and correlation-to-BS-seq metrics
- Input reads are nanopore or short-read — the ccsmeth/primrose comparison is specific to PacBio CCS kinetics (IPD/PW) and HiFi reads

## Inputs

- aligned PacBio HiFi/CCS BAM files (>=10 Kb insert, e.g., NA12898 chrX, HG002 15/20/24 Kb, SD0651_P1 15 Kb)
- bisulfite-sequencing (BS-seq) ground-truth methylated/unmethylated CpG labels (e.g., hg002_bsseq_chr20_demo.bed)
- pre-trained ccsmeth 5mCpG read-level model
- primrose (PacBio CNN-based 5mCpG caller) for baseline comparison

## Outputs

- per-read per-CpG methylation probability calls from each method
- accuracy, sensitivity, specificity, and AUC per method per test set
- comparison table such as read_level_accuracy_metrics.csv (one row per test set per method)

## How to apply

Assemble matched test sets of aligned PacBio CCS BAMs plus bisulfite-sequencing ground truth — in the source study these were five long (>=10 Kb) CCS sets: NA12898 chrX 10 Kb insert, HG002 15/20/24 Kb, and SD0651_P1 15 Kb. Run each caller in read-level mode on the identical BAMs (ccsmeth using kinetics features IPD and pulse width with its pre-trained 5mCpG model; primrose as PacBio's CNN baseline), yielding per-CpG methylation probabilities per read. Match predicted methylated/unmethylated CpG calls from both methods to the BS-seq labels per read and compute accuracy, sensitivity, specificity, and ROC AUC per method per dataset. Assemble one row per test-set-per-method (e.g., read_level_accuracy_metrics.csv) so ranges can be checked against reported values. Validation: reproduced ccsmeth read-level metrics should fall within accuracy 0.8721-0.9062, sensitivity 0.8621-0.8903, specificity 0.8765-0.9220, AUC 0.9464-0.9682, consistent with the reported ~0.90 accuracy and 0.97 AUC, and exceed primrose's ~85% read-level accuracy on all five datasets.

## Related tools

- **ccsmeth** (deep-learning 5mCpG detector run in read-level mode using IPD/pulse-width kinetics features of PacBio CCS reads) — https://github.com/PengNi/ccsmeth
- **primrose** (PacBio CNN-based 5mCpG caller used as the read-level comparison baseline on the same aligned CCS BAMs)
- **pbmm2** (aligner used to produce the sorted HiFi BAM alignments that the benchmarked callers consume) — https://github.com/PacificBiosciences/pbmm2
- **pb-CpG-tools** (PacBio companion scripts (aligned_bam_to_cpg_scores) for generating site methylation probabilities from mapped HiFi reads) — https://github.com/PacificBiosciences/pb-CpG-tools

## Evaluation signals

- Reproduced ccsmeth read-level accuracy falls within 0.8721-0.9062 across all five long-read test sets
- Reproduced AUC falls within 0.9464-0.9682, consistent with the reported 0.90 accuracy / 0.97 AUC headline
- ccsmeth accuracy, sensitivity, specificity, and AUC are higher than primrose on all five datasets
- The output table has exactly one row per test set per method (10 rows for 5 test sets x 2 methods)
- Per-read CpG calls match BS-seq labels after coordinate-matched pairing against the methylated/unmethylated ground truth

## Limitations

- Methods consider only symmetric methylation and combine features from both DNA strands, so they cannot detect hemimethylated CpGs
- Cannot be applied to 5mC detection in non-CpG contexts or other DNA modifications such as 6mA
- The strand-specific-methylation model shows significantly reduced performance due to limited subread depth
- primrose's companion tooling (pb-CpG-tools) is only available for 64-bit Linux platforms

## Evidence

- [abstract] ccsmeth achieves 0.90 accuracy and 0.97 AUC on long reads: "Using long (≥10 Kb) CCS reads, ccsmeth achieves 0.90 accuracy and 0.97 Area Under the Curve on 5mCpG detection at single-molecule resolution."
- [intro] primrose claimed 85% read-level accuracy baseline: "PacBio proposed another CNN-based method primrose25, which has been claimed to have 85% read-level accuracy on 5mCpG detection from long PacBio CCS reads"
- [discussion] ccsmeth outperforms two CNN-based methods: "ccsmeth outperforms two existing CNN-based methods on both read-level and site-level 5mC prediction."
- [intro] test datasets sequenced with PacBio CCS: "we sequenced amplified and M.SssI-treated DNA of human sample NA12898 using PacBio CCS with 10 Kb insert size"
- [methods] ccsmeth models from GitHub: "Check ccsmeth (https://github.com/PengNi/ccsmeth) to get ccsmeth 5mCpG models."
