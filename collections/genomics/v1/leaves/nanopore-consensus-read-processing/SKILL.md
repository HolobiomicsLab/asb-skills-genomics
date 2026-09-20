---
name: nanopore-consensus-read-processing
description: Use when handling INC-Seq consensus reads from ONT MinION amplicon sequencing
  (2D or 1D2 chemistry) that require fragmentation and orientation correction with
  chopSeq before downstream analysis.
when_to_use_negative:
- Input is raw, noisy nanopore reads rather than INC-Seq consensus reads — those require
  the upstream clustering/consensus-calling steps (e.g. nanoClust) of NanoAmpli-Seq
  first.
- Reads are from short-read platforms such as Illumina MiSeq or Ion Torrent PGM, whose
  short read lengths are the limitation this nanopore workflow addresses.
- The goal is reference-based mapping of noisy reads; the paper's rationale is to
  avoid reference dependence because incorrect mapping is likely due to high raw nanopore
  read error rates.
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0196
tools:
- name: chopSeq
  role: Fragmentation and read orientation correction of INC-Seq consensus reads
- name: nanoClust
  role: Read partitioning-based de novo clustering and within-cluster consensus calling
    that produces the full-length 16S rRNA gene consensus reads this skill post-processes
  repo: github.com/umerijaz/nanopore
- name: etandem
  role: Tandem repeat detection applied to chopSeq-corrected consensus reads to obtain
    repeat lengths per run
- name: ONT MinION
  role: Nanopore instrument generating the 2D and 1D2 sequencing runs whose consensus
    reads are processed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giy140
    title: 'NanoAmpli-Seq: a workflow for amplicon sequencing for mixed microbial
      communities on the nanopore sequencing platform'
  source_content_hash: sha256:a2b1548d2aabf7f760c70e456d868e109e5215f0131c5c63fcf597da6fd6661e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ONT MinION
  - chopSeq
  - etandem
  - nanoClust
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# nanopore-consensus-read-processing

## Summary

Post-processing of INC-Seq consensus reads produced on the ONT MinION platform, using chopSeq to fragment and orientation-correct consensus reads before downstream analyses such as tandem repeat characterization of 2D and 1D2 nanopore runs.

## When to use

Use when handling INC-Seq consensus reads from ONT MinION amplicon sequencing (2D or 1D2 chemistry) that require fragmentation and orientation correction with chopSeq before downstream analysis.

## When NOT to use

- Input is raw, noisy nanopore reads rather than INC-Seq consensus reads — those require the upstream clustering/consensus-calling steps (e.g. nanoClust) of NanoAmpli-Seq first.
- Reads are from short-read platforms such as Illumina MiSeq or Ion Torrent PGM, whose short read lengths are the limitation this nanopore workflow addresses.
- The goal is reference-based mapping of noisy reads; the paper's rationale is to avoid reference dependence because incorrect mapping is likely due to high raw nanopore read error rates.

## Inputs

- INC-Seq consensus reads from ONT MinION runs (2D and 1D2 chemistry)
- deposited nanopore amplicon FASTQ read sets

## Outputs

- chopSeq-corrected, orientation-corrected INC-Seq consensus reads
- per-run tandem repeat prevalence values (fraction of consensus reads containing tandem repeats)
- tandem repeat length distribution summaries (e.g. written to tandem_repeat_analysis.csv)

## How to apply

Load the INC-Seq consensus reads generated on the ONT MinION platform (2D or 1D2 chemistry) and run chopSeq, the software developed in the NanoAmpli-Seq workflow, to fragment the reads and correct their read orientation. Use the chopSeq-corrected, orientation-corrected consensus reads as the substrate for downstream feature detection, e.g. scanning each read with the etandem tandem repeat detector to identify tandem repeat features and their repeat lengths. When comparing chemistries, tabulate repeat length distributions per run and contrast the unimodal 2D distribution with the longer, bimodal 1D2 distribution; verify that the fraction of consensus reads containing tandem repeats falls in the reported 60%-75% prevalence range. To test robustness to aligner choice, re-run the repeat detection after re-aligning with an alternative aligner and confirm that prevalence and length distributions are independent of the aligner used during INC-Seq consensus calling.

## Related tools

- **chopSeq** (Fragmentation and read orientation correction of INC-Seq consensus reads)
- **nanoClust** (Read partitioning-based de novo clustering and within-cluster consensus calling that produces the full-length 16S rRNA gene consensus reads this skill post-processes) — github.com/umerijaz/nanopore
- **etandem** (Tandem repeat detection applied to chopSeq-corrected consensus reads to obtain repeat lengths per run)
- **ONT MinION** (Nanopore instrument generating the 2D and 1D2 sequencing runs whose consensus reads are processed)

## Evaluation signals

- Tandem repeat prevalence per run falls in the reported 60%-75% range of consensus reads containing tandem repeats.
- 2D runs yield a unimodal tandem repeat length distribution; 1D2 runs yield a longer, bimodal distribution.
- Repeat prevalence and length distributions remain unchanged when the INC-Seq consensus calling aligner is swapped for an alternative aligner.
- Downstream consensus sequences retain the reported average accuracy of 99.5% for 2D and 1D2 sequencing on tested mock communities.
- All reads are in a corrected orientation after the chopSeq step, with no residual reverse-orientation consensus reads.

## Limitations

- Nearly all residual errors in the processed consensus sequences originate from deletions in homopolymer regions; homopolymer-aware base calling or error correction would be needed to reach short-read-comparable accuracy.
- Consensus accuracy and read processing were validated on tested mock communities; behavior on real mixed microbial communities may differ.
- 1D2 and 2D chemistries produce qualitatively different tandem repeat length distributions, so results are not directly comparable across chemistries without per-run tabulation.

## Evidence

- [abstract] The developed protocol includes chopSeq software for fragmentation and read orientation correction of INC-Seq consensus reads: "The developed protocol includes chopSeq software for fragmentation and read orientation correction of INC-Seq consensus reads"
- [abstract] while nanoClust algorithm was designed for read partitioning-based de novo clustering and within cluster consensus calling: "while nanoClust algorithm was designed for read partitioning-based de novo clustering and within cluster consensus calling"
- [readme] Scripts for processing short amplicon reads from Oxford Nanopore.: "Scripts for processing short amplicon reads from Oxford Nanopore."
- [intro] incorrect mapping to a reference database is likely due to high error rates of raw nanopore reads: "incorrect mapping to a reference database is likely due to high error rates of raw nanopore reads"
