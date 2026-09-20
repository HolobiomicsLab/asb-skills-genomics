---
name: repeat-length-distribution-comparison-across-runs
description: Use when comparing INC-Seq consensus read sets from different ONT chemistries
  (2D vs 1D2) or aligners and you need to test whether tandem repeat prevalence or
  repeat length distributions differ between them.
when_to_use_negative:
- Input data are raw noisy nanopore reads rather than chopSeq-corrected INC-Seq consensus
  reads, since the prevalence range and distribution shapes were established on consensus
  sequences.
- The goal is homopolymer error characterization or residual deletion analysis rather
  than tandem repeat structure.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: chopSeq
  role: Produces the fragmentation- and orientation-corrected INC-Seq consensus reads
    that serve as the input for tandem repeat scanning
  repo: github.com/umerijaz/nanopore
- name: ONT MinION
  role: Sequencing platform generating the 2D and 1D2 runs whose consensus reads are
    compared for repeat prevalence and length distributions
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giy140
    title: 'NanoAmpli-Seq: a workflow for amplicon sequencing for mixed microbial
      communities on the nanopore sequencing platform'
  source_content_hash: sha256:026a922cfa30cdeca038881bd8b9b8573885fa624ece00320c24ac6f9bbbb4d0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ONT MinION
  - chopSeq
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# repeat-length-distribution-comparison-across-runs

## Summary

Quantify and compare tandem repeat prevalence and repeat length distributions across nanopore INC-Seq consensus runs to test whether these features depend on aligner choice or sequencing chemistry (2D vs 1D2).

## When to use

Use when comparing INC-Seq consensus read sets from different ONT chemistries (2D vs 1D2) or aligners and you need to test whether tandem repeat prevalence or repeat length distributions differ between them.

## When NOT to use

- Input data are raw noisy nanopore reads rather than chopSeq-corrected INC-Seq consensus reads, since the prevalence range and distribution shapes were established on consensus sequences.
- The goal is homopolymer error characterization or residual deletion analysis rather than tandem repeat structure.

## Inputs

- chopSeq-corrected, orientation-corrected INC-Seq consensus reads (FASTA) from deposited 2D and 1D2 ONT MinION runs
- Alternative alignments of the same consensus reads from a second aligner (for aligner-independence testing)

## Outputs

- Per-run tandem repeat prevalence values (fraction of consensus reads containing tandem repeats)
- Per-run tandem repeat length distribution summaries (unimodal vs bimodal shape)
- tandem_repeat_analysis.csv tabulating prevalence and repeat length distribution summaries

## How to apply

Load the chopSeq-corrected, orientation-corrected INC-Seq consensus reads for each deposited 2D and 1D2 ONT MinION run. Scan every read with the etandem tandem repeat detector to identify repeat features and their repeat lengths. Compute the fraction of consensus reads containing tandem repeats per run and check it falls within the reported 60%-75% prevalence range. Tabulate repeat length distributions per run, contrasting the unimodal distribution of 2D reads against the longer, bimodal distribution of 1D2 reads. Re-run the repeat detection after re-aligning with an alternative aligner to verify prevalence and length distributions are independent of aligner choice, then write prevalence values and distribution summaries to a tandem_repeat_analysis.csv output.

## Related tools

- **chopSeq** (Produces the fragmentation- and orientation-corrected INC-Seq consensus reads that serve as the input for tandem repeat scanning) — github.com/umerijaz/nanopore
- **ONT MinION** (Sequencing platform generating the 2D and 1D2 runs whose consensus reads are compared for repeat prevalence and length distributions)

## Examples

```
etandem consensus_reads.fasta -outfile tandem_repeat_analysis.csv -minrepeat 2 -maxrepeat 100
```

## Evaluation signals

- Per-run tandem repeat prevalence falls within the reported 60%-75% range across all 2D and 1D2 runs.
- 2D runs show a unimodal repeat length distribution while 1D2 runs show a bimodal distribution with longer repeats.
- Prevalence and length distributions remain unchanged when the detection is re-run after re-aligning with an alternative aligner.
- tandem_repeat_analysis.csv contains one row per run with both prevalence and repeat length distribution summary values.

## Limitations

- Repeat detection results depend on the input being chopSeq-corrected INC-Seq consensus reads; raw noisy reads would yield different prevalence estimates.
- Reported findings are grounded in the tested mock communities and the specific 2D/1D2 chemistries analyzed in NanoAmpli-Seq.

## Evidence

- [abstract] chopSeq software for fragmentation and read orientation correction of INC-Seq consensus reads: "The developed protocol includes chopSeq software for fragmentation and read orientation correction of INC-Seq consensus reads"
- [other] deposited 2D and 1D2 runs scanned with etandem: [evidence span withheld — anchor status: not_found]
- [other] prevalence and bimodal/unimodal distributions: [evidence span withheld — anchor status: not_found]
- [abstract] accuracy context for 2D and 1D2: "NanoAmpli-Seq accurately estimates the diversity of tested mock communities with average consensus sequence accuracy of 99.5% for 2D and 1D2 sequencing on the nanopore sequencing platform"
