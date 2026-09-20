---
name: read-level-5mcpg-methylation-calling
description: Use when you have aligned PacBio CCS/HiFi BAMs and need per-read 5mCpG
  methylation probabilities (not just site-level frequencies), e.g. for read-level
  accuracy evaluation against BS-seq ground truth.
when_to_use_negative:
- You need hemimethylated CpG detection — the methods only consider symmetric methylation
  and are incapable of detecting hemimethylated CpGs
- You need 5mC detection in non-CpG contexts or other DNA modifications such as 6mA
  — ccsmeth/primrose cannot be applied for these
- Input reads are nanopore reads rather than PacBio CCS/HiFi — use nanopore-specific
  callers (e.g. deepsignal2, Megalodon) instead
edam_operation: http://edamontology.org/operation_3207
edam_topics:
- http://edamontology.org/topic_3674
tools:
- name: ccsmeth
  role: Primary deep-learning caller that detects 5mCpGs from PacBio CCS reads using
    kinetics features (IPDs and PWs) with a pre-trained read-level model
  repo: https://github.com/PengNi/ccsmeth
- name: primrose
  role: PacBio's CNN-based 5mCpG read-level caller used as the comparison baseline
  repo: https://github.com/PacificBiosciences/pb-CpG-tools
- name: pbmm2
  role: Aligns PacBio hifi reads to the reference genome prior to methylation calling
  repo: https://github.com/PacificBiosciences/pbmm2
- name: ccs
  role: Generates the PacBio HiFi (CCS) reads that are the input to read-level methylation
    calling
  repo: https://github.com/PacificBiosciences/ccs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-39784-9
    title: DNA 5-methylcytosine detection and methylation phasing using PacBio circular
      consensus sequencing
  source_content_hash: sha256:32737d93abeb8e81a1300e366a887aebfd9d29a03a7fcdf572a7f47794ce0725
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PengNi/ccsmeth
  tools:
  - ccs
  - ccsmeth
  - pbmm2
  - primrose
  edam_operation: http://edamontology.org/operation_3207
  edam_topics:
  - http://edamontology.org/topic_3674
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# read-level-5mcpg-methylation-calling

## Summary

Detect 5mCpG methylation states at single-molecule (per-read) resolution from PacBio HiFi/CCS reads using the deep-learning tool ccsmeth, which exploits kinetics features (IPD and pulse width) and a pre-trained read-level model, and benchmark it against PacBio's CNN-based primrose caller.

## When to use

Use when you have aligned PacBio CCS/HiFi BAMs and need per-read 5mCpG methylation probabilities (not just site-level frequencies), e.g. for read-level accuracy evaluation against BS-seq ground truth.

## When NOT to use

- You need hemimethylated CpG detection — the methods only consider symmetric methylation and are incapable of detecting hemimethylated CpGs
- You need 5mC detection in non-CpG contexts or other DNA modifications such as 6mA — ccsmeth/primrose cannot be applied for these
- Input reads are nanopore reads rather than PacBio CCS/HiFi — use nanopore-specific callers (e.g. deepsignal2, Megalodon) instead

## Inputs

- Aligned PacBio CCS/HiFi BAM files (hifi reads, e.g. hg002.chr20_demo.hifi.bam)
- Pre-trained ccsmeth 5mCpG read-level model
- Bisulfite-sequencing (BS-seq) ground-truth methylation labels (e.g. hg002_bsseq_chr20_demo.bed)
- Reference genome for alignment (pbmm2)

## Outputs

- Per-read, per-CpG 5mC methylation probability calls (read-level methyl calls)
- Modified-bam (modbam) files of methylation calls
- Read-level accuracy metrics table (accuracy, sensitivity, specificity, AUC), e.g. read_level_accuracy_metrics.csv

## How to apply

Run ccsmeth in read-level mode on the aligned hifi BAMs, using the kinetics features (IPD and pulse width) and the pre-trained 5mCpG model to assign per-CpG methylation probabilities on each read; convert results to modified-bam (modbam) files if downstream phasing is needed. For benchmarking, run primrose (PacBio's CNN-based 5mCpG caller, used with its companion script in pb-CpG-tools) on the same BAMs, then match predicted read-level 5mCpG calls from both methods against BS-seq ground-truth methylated/unmethylated CpG labels and compute accuracy, sensitivity, specificity, and AUC per test set. On long (>=10 Kb) CCS reads, expect ccsmeth read-level metrics of accuracy 0.8721-0.9062, sensitivity 0.8621-0.8903, specificity 0.8765-0.9220, and AUC 0.9464-0.9682, exceeding primrose on all five test sets (NA12898 chrX, HG002 15/20/24 Kb, SD0651_P1 15 Kb).

## Related tools

- **ccsmeth** (Primary deep-learning caller that detects 5mCpGs from PacBio CCS reads using kinetics features (IPDs and PWs) with a pre-trained read-level model) — https://github.com/PengNi/ccsmeth
- **primrose** (PacBio's CNN-based 5mCpG read-level caller used as the comparison baseline) — https://github.com/PacificBiosciences/pb-CpG-tools
- **pbmm2** (Aligns PacBio hifi reads to the reference genome prior to methylation calling) — https://github.com/PacificBiosciences/pbmm2
- **ccs** (Generates the PacBio HiFi (CCS) reads that are the input to read-level methylation calling) — https://github.com/PacificBiosciences/ccs

## Examples

```
ccsmeth call_read --input hg002.chr20_demo.hifi.bam --ref hg38.fa --model ccsmeth-5mCpG-model --threads 16 --output hg002.chr20_demo.read_level
```

## Evaluation signals

- Reproduced ccsmeth read-level accuracy falls within 0.8721-0.9062, sensitivity within 0.8621-0.8903, specificity within 0.8765-0.9220, and AUC within 0.9464-0.9682 on the five long (>=10 Kb) test sets
- ccsmeth metrics exceed primrose's on all five test sets (NA12898 chrX, HG002 15/20/24 Kb, SD0651_P1 15 Kb)
- Read-level CpG calls match BS-seq ground-truth methylated/unmethylated labels at the expected rate (~0.90 accuracy, ~0.97 AUC)
- Output modbam files carry per-read MM/ML tags covering CpG positions on both strands

## Limitations

- Cannot detect hemimethylated CpGs because only symmetric methylation is considered and features from both DNA strands are combined
- Cannot be applied for 5mC detection in non-CpGs or other DNA modifications such as 6mA
- A strand-specific-methylation model variant has significantly reduced performance due to limited subread depth

## Evidence

- [abstract] ccsmeth achieves 0.90 accuracy and 0.97 AUC on 5mCpG detection at single-molecule resolution: "Using long (≥10 Kb) CCS reads, ccsmeth achieves 0.90 accuracy and 0.97 Area Under the Curve on 5mCpG detection at single-molecule resolution."
- [intro] ccsmeth uses kinetics features (IPDs and PWs) of PacBio CCS reads: "we propose ccsmeth, a deep-learning method to detect DNA 5mCpGs by using kinetics features (IPDs and PWs) of PacBio CCS reads"
- [intro] primrose baseline and its claimed 85% read-level accuracy: "PacBio proposed another CNN-based method primrose25, which has been claimed to have 85% read-level accuracy on 5mCpG detection from long PacBio CCS reads"
- [methods] modbam conversion and hifi.bam input in the methods: "ccsmethphase takes files of PacBio reads (subreads.bam or hifi.bam), and a reference genome as input."
- [intro] pb-CpG-tools as primrose companion: "PacBio provided primrose’s companion script in pb-CpG-tools (https://github.com/PacificBiosciences/pb-CpG-tools)"
- [discussion] Hemimethylation limitation: "Hence, these methods are incapable of detecting hemimethylated CpGs"
