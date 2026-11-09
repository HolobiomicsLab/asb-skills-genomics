---
name: az-single-nucleotide-classification
description: Use when PacBio CCS BAM reads with kinetic tags (fi, ri, fp, rp) must
  be classified at the single-nucleotide level as Z or A — e.g., site-specific analysis
  of hybrid A/Z (HgaI) or dZ-DNA/canonical samples.
when_to_use_negative:
- Goal is whole-read dZ-DNA molecule detection rather than per-site Z/A calling —
  use the k21-full-ZA model with -k 21 and the SVM read classifier instead
- Input reads lack PacBio kinetic tags (fi, ri, fp, rp) or were not sequenced with
  --keep-kinetics; Z-Calling cannot compute PW/IPD features from them
- Reads are shorter than 500 bp or contain fewer than 100 A+Z bases — they are excluded
  due to limited A/Z bases
edam_operation: http://edamontology.org/operation_3229
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3511
tools:
- name: Z-Calling
  role: Provides the k11-mixed-AZ MLP base-calling module (z-calling-base), BAM filtering,
    z-bam2txt conversion, and zfreq aggregation used for single-nucleotide A/Z classification
  repo: https://github.com/xiaochuanle/Z-Calling
- name: PacBio Circular Consensus Sequencing (CCS)
  role: Generates the HiFi BAM inputs with preserved pulse width (PW) and inter-pulse
    duration (IPD) kinetic signals via --keep-kinetics
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42003-026-09849-8
    title: 'Z-Calling: a tool for A/Z (2,6-diaminopurine) base calling and dZ-DNA
      detection using PacBio HiFi reads'
  source_content_hash: sha256:4648a72c3b86f5ef6ce9fb01c8b1911e03ee1b6558b62202da93072aeab72b0e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xiaochuanle/Z-Calling
  repo_url: https://github.com/xiaochuanle/Z-Calling
  tools:
  - PacBio Circular Consensus Sequencing (CCS)
  - Z-Calling
  edam_operation: http://edamontology.org/operation_3229
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3511
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# az-single-nucleotide-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Single-nucleotide A/Z (2,6-diaminopurine) base calling from PacBio HiFi/CCS reads using the Z-Calling k11-mixed-AZ MLP model, which discriminates Z from A bases per position by exploiting Z-induced pulse-width (PW) and inter-pulse-duration (IPD) kinetic perturbations.

## When to use

Use when PacBio CCS BAM reads with kinetic tags (fi, ri, fp, rp) must be classified at the single-nucleotide level as Z or A — e.g., site-specific analysis of hybrid A/Z (HgaI) or dZ-DNA/canonical samples.

## When NOT to use

- Goal is whole-read dZ-DNA molecule detection rather than per-site Z/A calling — use the k21-full-ZA model with -k 21 and the SVM read classifier instead
- Input reads lack PacBio kinetic tags (fi, ri, fp, rp) or were not sequenced with --keep-kinetics; Z-Calling cannot compute PW/IPD features from them
- Reads are shorter than 500 bp or contain fewer than 100 A+Z bases — they are excluded due to limited A/Z bases

## Inputs

- PacBio CCS BAM file with kinetic tags (fi, ri, fp, rp), e.g. Sequel II or Revio HiFi reads from dZ-DNA/canonical or hybrid HgaI A/Z samples
- Pre-trained scripted TorchScript checkpoint model/k11-mixed-AZ/scripted_m11.pth (used with -k 11)
- Optional reference FASTA used for alignment (required for downstream zfreq aggregation)

## Outputs

- Output BAM annotated with ZP tags recording Z probability for each A/T base (modifications also stored in MM and ML tags)
- Tab-delimited site table from z-bam2txt (QueryName, ReadID, Strand, QueryPos, Base, RefName, RefPos, Probability for mapped reads)
- zfreq table of Z/(A+Z) ratio per reference chromosome and coordinate

## How to apply

First filter the raw CCS BAM with Z-Calling's filter_bam.py, requiring kinetic tags (fi, ri, fp, rp), a Median_PW/Median_IPD ratio of at least 0.3 and below the 99th percentile, and at least 3 passes on both forward (fn>=3) and reverse (rn>=3) strands; restrict analysis to 11-mer context k-mers containing a single A/Z target base to avoid interference among adjacent A/Zs. Then run the k11-mixed-AZ MLP (model/k11-mixed-AZ/scripted_m11.pth with -k 11), which was trained on combined E. coli dZ-DNA/canonical and fruit fly HgaI hybrid data, on CPU via PyTorch to emit a ZP tag per A/T base. For table output, convert the ZP-tagged BAM with z-bam2txt and, for mapped reads, aggregate Z/(A+Z) ratios per reference coordinate with zfreq. Prefer this mixed-strategy model over the k11-full-ZA baseline (trained only on dZ-DNA/canonical data) when hybrid A/Z content is expected, since the baseline drops to AUC ~0.61 on HgaI data.

## Related tools

- **Z-Calling** (Provides the k11-mixed-AZ MLP base-calling module (z-calling-base), BAM filtering, z-bam2txt conversion, and zfreq aggregation used for single-nucleotide A/Z classification) — https://github.com/xiaochuanle/Z-Calling
- **PacBio Circular Consensus Sequencing (CCS)** (Generates the HiFi BAM inputs with preserved pulse width (PW) and inter-pulse duration (IPD) kinetic signals via --keep-kinetics)

## Examples

```
build/z-calling-base -k 11 -t 16 FilteredMappingBAM.bam model/k11-mixed-AZ/scripted_m11.pth Output_BAM
```

## Evaluation signals

- On the four HGAI (fruit fly HgaI) test datasets, AUC should fall in 0.942-0.949 and F1 in 0.868-0.878
- On Revio/Sequel II dZ-DNA/canonical datasets, AUC should fall in 0.951-0.952 and F1 in 0.864-0.894
- The k11-mixed-AZ model should clearly exceed the k11-full-ZA baseline AUC of 0.609-0.622 on HgaI datasets
- Filtered reads must satisfy fn>=3 and rn>=3 passes and a Median_PW/Median_IPD ratio of >=0.3 and below the 99th percentile; only k-mers with a single A/Z are analyzed
- Output BAM must carry ZP tags per A/T base; torch version check '2.2.2+cpu' confirms the CPU PyTorch environment

## Limitations

- Z-induced kinetic shifts are subtle (0.5-1.5-fold) and overlap substantially with canonical Adenine variance due to inherent PacBio kinetic measurement noise, limiting hybrid sample quantification
- Only context k-mers containing one A/Z target base are analyzed, so adjacent A/Z interference is avoided but densely modified loci are not directly resolvable
- Datasets must meet the quality threshold of at least 60,000 reads with forward/reverse pass numbers no less than 3

## Evidence

- [results] k11-mixed-AZ MLP model trained on combined E. coli dZ-DNA/canonical and fruit fly HgaI data: [evidence span withheld — anchor status: not_found]
- [results] mixed model retained robust performance on dZ-DNA/canonical datasets: [evidence span withheld — anchor status: not_found]
- [discussion] single A/Z k-mer restriction to avoid adjacent interference: "To avoid interference among adjacent A/Zs, only context k-mers containing one A/Z (target base) were analyzed"
- [readme] kinetic tag requirements for input reads: "**Required Tags:** Kinetic signals (fi, ri, fp, rp) must be present."
- [readme] mixed model available for site-specific analysis: "**Description:** Adds a ZP tag to reads recording the Z probability for each A/T base. Best for site-specific analysis."
- [discussion] CCS generation preserving kinetics: "Circular Consensus Sequencing (CCS) BAM files were generated using the ‘--keep-kinetics’ option to preserve pulse width (PW) and inter-pulse duration (IPD) signals"
