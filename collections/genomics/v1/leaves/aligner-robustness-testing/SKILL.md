---
name: aligner-robustness-testing
description: Use when a feature detected in INC-Seq consensus reads (e.g., tandem
  repeats flagged by etandem) might be an artifact of the consensus-calling aligner
  or of 2D vs 1D2 ONT MinION chemistry, and you need to separate aligner effects from
  chemistry effects.
when_to_use_negative:
- Input is raw noisy nanopore reads rather than INC-Seq consensus reads — repeat content
  on raw reads does not answer whether consensus-calling introduced the artifact
- The question is general consensus accuracy of NanoAmpli-Seq (99.5% on mock communities),
  not sensitivity of a specific feature to the aligner or chemistry
- Reads have not been fragmentation- and orientation-corrected with chopSeq, so repeat
  scans would be confounded by reverse-orientation duplicates
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0080
tools:
- name: etandem
  role: tandem repeat detector used to scan each consensus read and report repeat
    features and repeat lengths
- name: chopSeq
  role: produces the fragmentation- and orientation-corrected INC-Seq consensus reads
    that are the input to the repeat scan
  repo: github.com/umerijaz/nanopore
- name: ONT MinION
  role: sequencing platform whose 2D and 1D2 chemistries provide the paired runs contrasted
    in the robustness test
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giy140
    title: 'NanoAmpli-Seq: a workflow for amplicon sequencing for mixed microbial
      communities on the nanopore sequencing platform'
  source_content_hash: sha256:1ac6b2d3ec7264a5346e8e67fd54253284d46716ef6fa2b7e6adc34a3e2514a7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ONT MinION
  - chopSeq
  - etandem
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# aligner-robustness-testing

## Summary

Tests whether a downstream observation on INC-Seq consensus reads (e.g., tandem repeat prevalence and repeat length distributions) depends on the aligner used during INC-Seq consensus calling or on nanopore chemistry (2D vs 1D2), rather than being a property of the biological data.

## When to use

Use when a feature detected in INC-Seq consensus reads (e.g., tandem repeats flagged by etandem) might be an artifact of the consensus-calling aligner or of 2D vs 1D2 ONT MinION chemistry, and you need to separate aligner effects from chemistry effects.

## When NOT to use

- Input is raw noisy nanopore reads rather than INC-Seq consensus reads — repeat content on raw reads does not answer whether consensus-calling introduced the artifact
- The question is general consensus accuracy of NanoAmpli-Seq (99.5% on mock communities), not sensitivity of a specific feature to the aligner or chemistry
- Reads have not been fragmentation- and orientation-corrected with chopSeq, so repeat scans would be confounded by reverse-orientation duplicates

## Inputs

- chopSeq-corrected, orientation-corrected INC-Seq consensus reads (FASTA/FASTQ) for deposited 2D and 1D2 ONT MinION runs
- raw noisy nanopore reads from the same runs for re-alignment with an alternative aligner
- etandem repeat-detection output (repeat features and repeat lengths per read)

## Outputs

- Per-run tandem repeat prevalence values (fraction of consensus reads containing tandem repeats)
- Per-run tandem repeat length distributions and modality summary (unimodal vs bimodal)
- Comparison table of prevalence and length distributions across aligners and chemistries, e.g. tandem_repeat_analysis.csv

## How to apply

Start from the chopSeq-corrected, orientation-corrected INC-Seq consensus reads for both the deposited 2D and 1D2 ONT MinION runs. Scan each read with the etandem tandem repeat detector to extract tandem repeat features and their repeat lengths, then compute the fraction of consensus reads containing tandem repeats per run. Tabulate repeat length distributions per run and contrast modality (the reported result was 60%-75% prevalence with unimodal lengths for 2D versus longer, bimodal lengths for 1D2). Re-run the identical etandem scan after re-aligning the underlying noisy reads with an alternative aligner during INC-Seq consensus calling; if prevalence and length distributions are unchanged, the observation is independent of aligner choice and attributable to chemistry (1D2 vs 2D).

## Related tools

- **etandem** (tandem repeat detector used to scan each consensus read and report repeat features and repeat lengths)
- **chopSeq** (produces the fragmentation- and orientation-corrected INC-Seq consensus reads that are the input to the repeat scan) — github.com/umerijaz/nanopore
- **ONT MinION** (sequencing platform whose 2D and 1D2 chemistries provide the paired runs contrasted in the robustness test)

## Evaluation signals

- Per-run tandem repeat prevalence falls within the reported 60%-75% range across aligners and chemistries
- 2D runs yield a unimodal tandem repeat length distribution; 1D2 runs yield longer repeats with a bimodal distribution
- Prevalence and length-distribution summaries are stable when the INC-Seq consensus calling is repeated with an alternative aligner
- All reads scanned are chopSeq-corrected and orientation-corrected (no reverse-complement duplicates inflating repeat counts)
- Output CSV contains one prevalence value and one length-distribution summary per run, keyed by run and aligner

## Limitations

- Robustness conclusions are limited to the deposited 2D and 1D2 ONT MinION runs and tested mock communities; other chemistries (e.g., later 1D base callers) are not covered
- Residual consensus errors are dominated by homopolymer deletions, so indel-sensitive repeat detection may reflect homopolymer error structure rather than true tandem repeat content
- Finding an aligner-independent result does not rule out other sources of artifact (e.g., base caller version, PCR chimera effects)

## Evidence

- [abstract] chopSeq-corrected, orientation-corrected INC-Seq consensus reads are the analysis input: "The developed protocol includes chopSeq software for fragmentation and read orientation correction of INC-Seq consensus reads"
- [abstract] 2D and 1D2 chemistry are the comparison conditions with reported 99.5% consensus accuracy: "NanoAmpli-Seq accurately estimates the diversity of tested mock communities with average consensus sequence accuracy of 99.5% for 2D and 1D2 sequencing"
- [readme] repository provides scripts for processing the Oxford Nanopore amplicon reads used here: "Scripts for processing short amplicon reads from Oxford Nanopore."
- [abstract] residual errors are homopolymer deletions, a caveat for repeat-length interpretation: "Nearly all residual errors in NanoAmpli-Seq sequences originate from deletions in homopolymer regions"
