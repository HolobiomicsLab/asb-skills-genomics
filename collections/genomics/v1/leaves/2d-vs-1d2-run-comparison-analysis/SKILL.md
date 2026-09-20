---
name: 2d-vs-1d2-run-comparison-analysis
description: Use when comparing nanopore sequencing chemistries (2D vs 1D2) for artifacts
  such as tandem repeats in INC-Seq consensus reads, or when testing whether an artifact
  metric depends on the aligner used during consensus calling.
when_to_use_negative:
- Input reads are raw nanopore reads rather than chopSeq-corrected INC-Seq consensus
  reads — repeat patterns in raw noisy reads are not the target of this comparison.
- The analysis goal is consensus accuracy or homopolymer error profiling (e.g. the
  99.5% accuracy or deletion-in-homopolymer findings) rather than chemistry-dependent
  tandem repeat structure.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: chopSeq
  role: Used upstream to fragment and correct orientation of INC-Seq consensus reads
    before tandem repeat scanning
  repo: github.com/umerijaz/nanopore
- name: ONT MinION
  role: Nanopore sequencing platform whose 2D and 1D2 chemistry runs provide the consensus
    reads being compared
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giy140
    title: 'NanoAmpli-Seq: a workflow for amplicon sequencing for mixed microbial
      communities on the nanopore sequencing platform'
  source_content_hash: sha256:08987a1b26ae1e987388c310011ba176c2775d37df92cbeb37a92e528df5909e
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
yes_rate: 0.6
n_claims: 5
---

# 2d-vs-1d2-run-comparison-analysis

## Summary

A comparative analysis skill that contrasts tandem repeat prevalence and repeat length distributions in chopSeq-corrected INC-Seq consensus reads between ONT MinION 2D and 1D2 sequencing chemistries, and tests dependence on the aligner used during consensus calling. It distinguishes the unimodal 2D repeat-length distribution from the longer, bimodal 1D2 distribution.

## When to use

Use when comparing nanopore sequencing chemistries (2D vs 1D2) for artifacts such as tandem repeats in INC-Seq consensus reads, or when testing whether an artifact metric depends on the aligner used during consensus calling.

## When NOT to use

- Input reads are raw nanopore reads rather than chopSeq-corrected INC-Seq consensus reads — repeat patterns in raw noisy reads are not the target of this comparison.
- The analysis goal is consensus accuracy or homopolymer error profiling (e.g. the 99.5% accuracy or deletion-in-homopolymer findings) rather than chemistry-dependent tandem repeat structure.

## Inputs

- chopSeq-corrected, orientation-corrected INC-Seq consensus reads (FASTA/FASTQ) from 2D ONT MinION runs
- chopSeq-corrected, orientation-corrected INC-Seq consensus reads from 1D2 ONT MinION runs

## Outputs

- tandem_repeat_analysis.csv with per-run tandem repeat prevalence values and repeat length distribution summaries
- Repeat length distribution tables contrasting unimodal (2D) vs bimodal (1D2) patterns

## How to apply

Load the chopSeq-corrected, orientation-corrected INC-Seq consensus reads for both the deposited 2D and 1D2 ONT MinION runs. Scan each read with the etandem tandem repeat detector to identify repeat features and record their repeat lengths. Compute the fraction of consensus reads containing tandem repeats per run and verify it falls within the reported 60%-75% prevalence range, which was found to hold regardless of aligner or chemistry. Tabulate repeat length distributions per run and contrast the unimodal 2D distribution against the longer, bimodal 1D2 distribution. As a control, re-run repeat detection after re-aligning with an alternative aligner to confirm prevalence and length distributions are independent of aligner choice. Write prevalence values and distribution summaries to a structured CSV (tandem_repeat_analysis.csv).

## Related tools

- **chopSeq** (Used upstream to fragment and correct orientation of INC-Seq consensus reads before tandem repeat scanning) — github.com/umerijaz/nanopore
- **ONT MinION** (Nanopore sequencing platform whose 2D and 1D2 chemistry runs provide the consensus reads being compared)

## Examples

```
etandem -min 2 -max 6 consensus_2d.fasta > tandem_repeats_2d.txt; etandem -min 2 -max 6 consensus_1d2.fasta > tandem_repeats_1d2.txt; python tandem_repeat_analysis.py --runs 2d,1d2 --out tandem_repeat_analysis.csv
```

## Evaluation signals

- Per-run tandem repeat prevalence falls within the reported 60%-75% range of INC-Seq consensus reads.
- Prevalence is stable when the aligner used during INC-Seq consensus calling is swapped, confirming aligner independence.
- 2D runs exhibit a unimodal tandem repeat length distribution.
- 1D2 runs exhibit a bimodal repeat length distribution with longer tandem repeats than 2D.
- tandem_repeat_analysis.csv contains prevalence values and distribution summaries for every deposited run analyzed.

## Limitations

- Findings are specific to INC-Seq consensus reads from ONT MinION 2D and 1D2 chemistry runs and may not transfer to other chemistries or platforms.
- Repeat detection results may be sensitive to the etandem detector parameters; aligner-independence was verified but detector-dependence is not established.
- Residual errors in these sequences largely originate from homopolymer deletions, which can interact with tandem repeat structure.

## Evidence

- [results] Tandem repeats were present in 60%-75% of INC-Seq consensus reads regardless of aligner or chemistry, but 1D2 reads had longer tandem repeats with a bimodal length distribution: [evidence span withheld — anchor status: not_found]
- [abstract] chopSeq software for fragmentation and read orientation correction of INC-Seq consensus reads: "The developed protocol includes chopSeq software for fragmentation and read orientation correction of INC-Seq consensus reads"
- [intro] single-molecule sensing technologies on the Oxford Nanopore Technologies (ONT) MinION platform: "single-molecule sensing technologies on the Oxford Nanopore Technologies (ONT) MinION platform, has opened the possibility of obtaining ultra-long reads"
- [readme] Scripts for processing short amplicon reads from Oxford Nanopore: "Scripts for processing short amplicon reads from Oxford Nanopore. For details, please see:"
- [abstract] Nearly all residual errors in NanoAmpli-Seq sequences originate from deletions in homopolymer regions: "Nearly all residual errors in NanoAmpli-Seq sequences originate from deletions in homopolymer regions"
