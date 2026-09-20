---
name: per-read-probability-aggregation-stoichiometry
description: Use when nanopore direct RNA sequencing reads have been classified per-read
  by CHEUI-solo Model 1 and transcriptome-wide site-level methylation probabilities
  and stoichiometry estimates are needed for individual experimental conditions.
when_to_use_negative:
- When the goal is differential methylation between two conditions — CHEUI-diff, not
  CHEUI-solo Model 2 aggregation, is the appropriate mode
- When reads have not yet been processed through CHEUI-solo Model 1 to produce per-read
  probability distributions (the aggregation requires these as input)
- 'When a more permissive sensitivity-oriented site call is needed: the article notes
  strict cutoffs reduce CHEUI sensitivity, and many sites found by orthogonal methods
  had high probabilities below the set thresholds'
edam_operation: http://edamontology.org/operation_0431
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3169
tools:
- name: CHEUI
  role: CHEUI-solo Model 2 CNN classifier aggregates Model 1 per-read modification
    probabilities into site-level probabilities and stoichiometry estimates; Model
    1 provides the per-read classifications trained on IVT signals
  repo: https://github.com/comprna/CHEUI-public
- name: nanopore direct RNA sequencing
  role: 'Data source: observed nanopore direct RNA sequencing ionic-current signals
    (FAST5/eventalign) that Model 1 processes to per-read probabilities'
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-47953-7
    title: Prediction of m6A and m5C at single-molecule resolution reveals a transcriptome-wide
      co-occurrence of RNA modifications
  source_content_hash: sha256:64eac2838d72a7ee00051bd416560e0981ae8f4769dd60c9a16030d82ed0c947
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/CHEUI-public
  tools:
  - CHEUI
  - nanopore direct RNA sequencing
  edam_operation: http://edamontology.org/operation_0431
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# per-read-probability-aggregation-stoichiometry

## Summary

Aggregates CHEUI-solo Model 1 per-read m6A/m5C modification probabilities into site-level modification probabilities and stoichiometry estimates using CHEUI-solo Model 2 (a CNN classifier over per-read probability distributions), with a significance cutoff of prediction probability > 0.9999.

## When to use

Use when nanopore direct RNA sequencing reads have been classified per-read by CHEUI-solo Model 1 and transcriptome-wide site-level methylation probabilities and stoichiometry estimates are needed for individual experimental conditions.

## When NOT to use

- When the goal is differential methylation between two conditions — CHEUI-diff, not CHEUI-solo Model 2 aggregation, is the appropriate mode
- When reads have not yet been processed through CHEUI-solo Model 1 to produce per-read probability distributions (the aggregation requires these as input)
- When a more permissive sensitivity-oriented site call is needed: the article notes strict cutoffs reduce CHEUI sensitivity, and many sites found by orthogonal methods had high probabilities below the set thresholds

## Inputs

- CHEUI-solo Model 1 per-read modification probability distributions grouped by transcriptomic site (A sites with coverage >20 reads)
- Nanopore direct RNA FAST5 reads preprocessed via Guppy basecalling, Minimap2 transcriptome alignment, samtools -F 232457 filtering, and Nanopolish eventalign
- Transcriptome reference (GRCh38 / Gencode v38 for the human data)
- In-vitro transcribed (IVT) RNA DRS replicates for FDR/FPR calibration

## Outputs

- Site-level m6A/m5C modification probabilities for each transcriptomic site
- Per-site stoichiometry (fraction modified) estimates
- Significant m6A call set at probability > 0.9999 (e.g., 10,036 sites on 3,905 transcripts, corresponding to 8,776 genomic sites in HEK293 WT)
- Estimated false discovery rate from signal-shuffling controls and FPR from IVT controls (reference m6A FPR 0.0003)

## How to apply

First preprocess FAST5 files (Guppy basecalling, Minimap2 transcriptome alignment, samtools -F 232457 best-match filtering, Nanopolish eventalign signal-to-transcript alignment, per-5mer signal flipping to 5′–3′ orientation), then run CHEUI-solo Model 1 on IVT-trained models to obtain per-read modification probabilities. Sort and group predictions so all probabilities from the same transcriptomic site are pooled (works with single or several replicates), requiring transcriptomic A sites with coverage >20 reads. Feed Model 1 per-read probability distributions into CHEUI-solo Model 2 (a Keras/TensorFlow CNN classifier), which outputs site-level methylation probabilities and stoichiometry (fraction modified) per transcript site. Apply the significance cutoff probability >=0.9999, which was established by shuffling read signals across transcriptomic sites to keep the estimated m6A FDR at 0; this strict cutoff is necessary because of the large number of sites tested. As a control for false positives, run Models 1 and 2 with the same 0.9999 cutoff on in-vitro transcribed (IVT) RNA replicates, where true modification should be absent.

## Related tools

- **CHEUI** (CHEUI-solo Model 2 CNN classifier aggregates Model 1 per-read modification probabilities into site-level probabilities and stoichiometry estimates; Model 1 provides the per-read classifications trained on IVT signals) — https://github.com/comprna/CHEUI-public
- **nanopore direct RNA sequencing** (Data source: observed nanopore direct RNA sequencing ionic-current signals (FAST5/eventalign) that Model 1 processes to per-read probabilities)

## Evaluation signals

- Reproduced HEK293 WT call set at probability > 0.9999 should contain ~10,036 significant m6A transcriptomic sites on ~3,905 transcripts (8,776 genomic sites)
- Approximately 85.12% of significant m6A sites should fall in DRACH motifs
- The m6A false positive rate on in-vitro transcribed WT HeLa replicates should match the reported value of 0.0003
- Per-shuffling FDR estimation (shuffling read signals across transcriptomic sites while keeping site counts fixed) should yield an estimated FDR of 0 for m6A at the 0.9999 cutoff
- Output site table should include transcript ID, site coordinate, coverage (N_valid_cov), and fraction_modified per site

## Limitations

- Very strict cutoffs (probability > 0.9999) reduce CHEUI sensitivity; relaxing them would recover more orthogonal-method sites but at the cost of potential false positives
- Nanopore signal variability limits detection when differences between modified and non-modified reads are comparable to variation among non-modified reads
- Modified sites detected only from nanopore reads require independent validation experiments
- CHEUI's successor SWARM improves accuracy for RNA002 chemistry and adds RNA004/pseudouridine support, so CHEUI results are chemistry-dependent

## Evidence

- [methods] FDR estimation by shuffling read signals across transcriptomic sites: "we shuffled all read signals across all transcriptomic sites, maintaining the same number of transcriptomic sites"
- [discussion] Strict cutoffs trade sensitivity for low FDR: "given the large number of sites tested, we used very strict cut-offs to maintain a low expected false discovery rate (FDR)."
- [readme] CHEUI detects m6A/m5C at single-read and single-site resolution: [evidence span withheld — anchor status: external_doc]
- [intro] Model 1 and Model 2 two-stage design: "CHEUI is based on a two-stage neural network and was trained using read signals generated from in-vitro transcripts (IVTs)"
- [other] Reference site call set for validation: [evidence span withheld — anchor status: not_found]
