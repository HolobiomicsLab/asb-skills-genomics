---
name: pacbio-ccs-kinetics-feature-inference
description: Use when you have aligned PacBio HiFi/CCS BAMs and need per-read, single-molecule
  CpG methylation probabilities using native kinetics (IPD/PW) features, or when benchmarking
  ccsmeth vs primrose read-level 5mC calling.
when_to_use_negative:
- Detection of non-CpG 5mC or other DNA modifications such as 6mA — the article states
  ccsmeth 'cannot be applied for 5mC detection in non-CpGs or the detection of other
  DNA modifications (such as 6mA)'
- Detection of hemimethylated CpGs — the models only consider symmetric methylation
- Site-level (pooled) methylation frequencies rather than single-molecule read-level
  calls — use the site-level ccsmeth model or aligned_bam_to_cpg_scores instead
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0654
tools:
- name: ccsmeth
  role: Primary deep-learning tool; runs read-level 5mCpG inference on aligned CCS
    BAMs using IPD and PW kinetics features with a pre-trained 5mCpG model
  repo: https://github.com/PengNi/ccsmeth
- name: primrose
  role: PacBio CNN-based comparison baseline for read-level 5mCpG calling on the same
    aligned CCS BAMs
- name: pbmm2
  role: Aligns PacBio HiFi reads to the reference genome to produce the sorted aligned
    BAM input for methylation calling
  repo: https://github.com/PacificBiosciences/pbmm2
- name: pb-CpG-tools
  role: Companion tooling for primrose outputs; provides aligned_bam_to_cpg_scores
    for aggregating site methylation probabilities from mapped HiFi reads
  repo: https://github.com/PacificBiosciences/pb-CpG-tools
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-39784-9
    title: DNA 5-methylcytosine detection and methylation phasing using PacBio circular
      consensus sequencing
  source_content_hash: sha256:6eb6d32ea6a2e699c5ac491fe6c2f5f7df15978f04786144a97ea66ff91c9a81
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PengNi/ccsmeth
  tools:
  - ccsmeth
  - pb-CpG-tools
  - pbmm2
  - primrose
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# pacbio-ccs-kinetics-feature-inference

## Summary

Deep-learning read-level 5mCpG detection on PacBio circular consensus sequencing (HiFi) reads using kinetic features (IPDs and pulse widths) via ccsmeth's pre-trained model, benchmarked against primrose on long (≥10 Kb) reads.

## When to use

Use when you have aligned PacBio HiFi/CCS BAMs and need per-read, single-molecule CpG methylation probabilities using native kinetics (IPD/PW) features, or when benchmarking ccsmeth vs primrose read-level 5mC calling.

## When NOT to use

- Detection of non-CpG 5mC or other DNA modifications such as 6mA — the article states ccsmeth 'cannot be applied for 5mC detection in non-CpGs or the detection of other DNA modifications (such as 6mA)'
- Detection of hemimethylated CpGs — the models only consider symmetric methylation
- Site-level (pooled) methylation frequencies rather than single-molecule read-level calls — use the site-level ccsmeth model or aligned_bam_to_cpg_scores instead

## Inputs

- Aligned PacBio hifi BAM (e.g., hg002.chr20_demo.hifi.bam, or NA12898 chrX / HG002 15/20/24 Kb / SD0651_P1 15 Kb test sets)
- Pre-trained ccsmeth 5mCpG read-level model
- Reference genome for alignment context
- BS-seq ground-truth CpG labels (e.g., hg002_bsseq_chr20_demo.bed) for benchmarking

## Outputs

- Per-read, per-CpG methylation probabilities from ccsmeth
- Per-read 5mCpG predictions from primrose (baseline)
- read_level_accuracy_metrics.csv with accuracy, sensitivity, specificity, and AUC per test set per method

## How to apply

Collect the aligned hifi BAM(s) (e.g., NA12898 chrX 10 Kb insert, HG002 15/20/24 Kb, SD0651_P1 15 Kb) and download the pre-trained ccsmeth 5mCpG read-level model from github.com/PengNi/ccsmeth. Run ccsmeth in read-level mode on each BAM so the deep-learning model consumes kinetics features (IPDs and PWs) and assigns a per-CpG methylation probability on every read; the model assumes symmetric methylation and combines features from both DNA strands. As a comparison baseline, run primrose (PacBio's CNN-based caller, with companion scripts in pb-CpG-tools) on the same BAMs. Match both methods' per-CpG calls against BS-seq ground-truth methylated/unmethylated labels and compute accuracy, sensitivity, specificity, and ROC AUC per test set per method, assembling them into read_level_accuracy_metrics.csv.

## Related tools

- **ccsmeth** (Primary deep-learning tool; runs read-level 5mCpG inference on aligned CCS BAMs using IPD and PW kinetics features with a pre-trained 5mCpG model) — https://github.com/PengNi/ccsmeth
- **primrose** (PacBio CNN-based comparison baseline for read-level 5mCpG calling on the same aligned CCS BAMs)
- **pbmm2** (Aligns PacBio HiFi reads to the reference genome to produce the sorted aligned BAM input for methylation calling) — https://github.com/PacificBiosciences/pbmm2
- **pb-CpG-tools** (Companion tooling for primrose outputs; provides aligned_bam_to_cpg_scores for aggregating site methylation probabilities from mapped HiFi reads) — https://github.com/PacificBiosciences/pb-CpG-tools

## Evaluation signals

- Reproduced ccsmeth read-level metrics fall within reported ranges: accuracy 0.8721–0.9062, sensitivity 0.8621–0.8903, specificity 0.8765–0.9220, AUC 0.9464–0.9682
- ccsmeth's accuracy, sensitivity, specificity, and AUC exceed primrose's on all five long-read test sets
- Headline agreement with the abstract's ~0.90 accuracy and 0.97 AUC on long (≥10 Kb) CCS reads
- Every predicted CpG in the output can be matched to a BS-seq ground-truth methylated/unmethylated label before metric computation
- read_level_accuracy_metrics.csv contains one row per test set per method with all four metrics populated

## Limitations

- Performance claims are grounded on long (≥10 Kb) CCS reads; shorter inserts may not reproduce the reported 0.90 accuracy / 0.97 AUC
- Models only consider symmetric methylation and combine features from both DNA strands, so hemimethylated CpGs cannot be detected
- Cannot detect non-CpG 5mC or other modifications such as 6mA; whether the site-level model design extends to non-CpG 5mCs is unverified
- primrose itself is claimed to have only ~85% read-level accuracy on long CCS reads, so the baseline gap should be interpreted accordingly

## Evidence

- [intro] ccsmeth detects 5mCpG from kinetics features: "we propose ccsmeth, a deep-learning method to detect DNA 5mCpGs by using kinetics features (IPDs and PWs) of PacBio CCS reads"
- [abstract] Headline performance on long CCS reads: "Using long (≥10 Kb) CCS reads, ccsmeth achieves 0.90 accuracy and 0.97 Area Under the Curve on 5mCpG detection at single-molecule resolution."
- [intro] primrose baseline claim: "PacBio proposed another CNN-based method primrose25, which has been claimed to have 85% read-level accuracy on 5mCpG detection from long PacBio CCS reads"
- [methods] Input file types for inference: "ccsmethphase takes files of PacBio reads (subreads.bam or hifi.bam), and a reference genome as input."
- [discussion] Symmetric-methylation assumption: "all these methods only consider symmetric methylation and combine the features from both DNA strands to predict the methylation states."
- [readme] ccsmeth tool purpose (README): [evidence span withheld — anchor status: external_doc]
