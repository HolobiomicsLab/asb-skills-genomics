---
name: tandem-repeat-feature-detection
description: Use when raw or consensus nanopore reads (FASTA from chopSeq-corrected
  INC-Seq output) need to be scanned for tandem repeats to compare repeat prevalence
  or length distributions across chemistries (2D vs 1D2) or aligner choices.
when_to_use_negative:
- Input is raw noisy nanopore reads that have not been through INC-Seq consensus calling
  and chopSeq orientation correction — repeat calls would be dominated by sequencing
  noise rather than true tandem repeat features
- The goal is homopolymer region error analysis rather than tandem repeat feature
  detection — residual errors in NanoAmpli-Seq sequences originate from homopolymer
  deletions and need a different assessment
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: chopSeq
  role: Produces the fragmented, orientation-corrected INC-Seq consensus reads that
    serve as input to tandem repeat scanning
  repo: github.com/umerijaz/nanopore
- name: ONT MinION
  role: Sequencing platform generating the 2D and 1D2 amplicon reads underlying the
    consensus sequences analyzed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giy140
    title: 'NanoAmpli-Seq: a workflow for amplicon sequencing for mixed microbial
      communities on the nanopore sequencing platform'
  source_content_hash: sha256:8f915016153ec93e3ac085c35457d8a0d79e9fcbf4531eac305581d6004addda
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ONT MinION
  - chopSeq
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# tandem-repeat-feature-detection

## Summary

Detects and characterizes tandem repeat features in chopSeq-corrected INC-Seq consensus reads from Oxford Nanopore 2D and 1D2 amplicon sequencing using the etandem detector, quantifying repeat prevalence and contrasting unimodal (2D) versus bimodal (1D2) repeat length distributions.

## When to use

Use when raw or consensus nanopore reads (FASTA from chopSeq-corrected INC-Seq output) need to be scanned for tandem repeats to compare repeat prevalence or length distributions across chemistries (2D vs 1D2) or aligner choices.

## When NOT to use

- Input is raw noisy nanopore reads that have not been through INC-Seq consensus calling and chopSeq orientation correction — repeat calls would be dominated by sequencing noise rather than true tandem repeat features
- The goal is homopolymer region error analysis rather than tandem repeat feature detection — residual errors in NanoAmpli-Seq sequences originate from homopolymer deletions and need a different assessment

## Inputs

- chopSeq-corrected, orientation-corrected INC-Seq consensus reads (FASTA) from 2D ONT MinION runs
- chopSeq-corrected, orientation-corrected INC-Seq consensus reads (FASTA) from 1D2 ONT MinION runs

## Outputs

- tandem_repeat_analysis.csv with per-run tandem repeat prevalence values and repeat length distribution summaries
- Per-run tabulated tandem repeat length distributions (2D unimodal vs 1D2 bimodal)

## How to apply

Load the chopSeq-corrected, orientation-corrected INC-Seq consensus reads for each deposited ONT MinION run (2D and 1D2). Scan each read with the etandem tandem repeat detector to identify tandem repeat features and record their repeat lengths. Compute the fraction of consensus reads containing tandem repeats per run and verify it falls within the reported 60%-75% prevalence range, which should hold regardless of aligner used during INC-Seq consensus calling. Tabulate repeat length distributions per run and contrast the unimodal 2D distribution with the longer, bimodal 1D2 distribution. Optionally re-run repeat detection on consensus reads generated with an alternative aligner to confirm that prevalence and length distributions are independent of aligner choice; write results to tandem_repeat_analysis.csv.

## Related tools

- **chopSeq** (Produces the fragmented, orientation-corrected INC-Seq consensus reads that serve as input to tandem repeat scanning) — github.com/umerijaz/nanopore
- **ONT MinION** (Sequencing platform generating the 2D and 1D2 amplicon reads underlying the consensus sequences analyzed)

## Evaluation signals

- Per-run tandem repeat prevalence falls within the reported 60%-75% range of INC-Seq consensus reads
- 2D runs show a unimodal tandem repeat length distribution while 1D2 runs show a longer, bimodal distribution
- Prevalence and length distributions remain unchanged when consensus calling is repeated with an alternative aligner
- tandem_repeat_analysis.csv contains one prevalence value plus distribution summary per analyzed run

## Limitations

- Nearly all residual errors in NanoAmpli-Seq consensus sequences originate from deletions in homopolymer regions, which can confound repeat-length estimation
- Findings are specific to nanopore 2D and 1D2 chemistry on tested mock communities; other chemistries or platform revisions may show different distributions

## Evidence

- [abstract] chopSeq software for fragmentation and read orientation correction of INC-Seq consensus reads: "The developed protocol includes chopSeq software for fragmentation and read orientation correction of INC-Seq consensus reads"
- [abstract] NanoAmpli-Seq accuracy: "NanoAmpli-Seq accurately estimates the diversity of tested mock communities with average consensus sequence accuracy of 99.5% for 2D and 1D2 sequencing on the nanopore sequencing platform"
- [abstract] homopolymer residual errors: "Nearly all residual errors in NanoAmpli-Seq sequences originate from deletions in homopolymer regions"
- [readme] repository pointer: "Scripts for processing short amplicon reads from Oxford Nanopore. For details, please see:"
