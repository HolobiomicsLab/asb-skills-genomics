---
name: variation-graph-based-error-correction
description: Use when correcting noisy long reads from mixed samples (metagenomes,
  diploid/triploid/tetraploid or polyploid genomes) where consensus-template correctors
  mask low-frequency haplotype variants, and lower corrected error rates are the priority.
when_to_use_negative:
- 'Minimizing compute is the goal: on large datasets VeChat has longer runtimes and
  higher peak memory usage than competitors (e.g. Daccord achieved the lowest error
  rate on the human gut microbiome dataset while VeChat only matched read accuracy).'
- Input reads are already corrected or the goal is polishing an assembled contig/consensus
  — Racon is the consensus module for correcting raw contigs, and VeChat targets raw
  long-read self-correction.
- The study scope is haplotype-aware correction; exploring optimal preprocessing/error-correction/assembly
  pipeline combinations for polyploid, cancer or metagenomic settings was explicitly
  left as future work.
edam_operation: http://edamontology.org/operation_3184
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: VeChat
  role: 'Haplotype-aware self-correction tool: builds ad-hoc variation graphs from
    raw reads, prunes error-induced nodes/edges, derives haplotype-aware consensus
    sequences'
  repo: https://github.com/HaploKit/vechat
- name: minimap2
  role: Computes minimizer-based all-vs-all read overlaps (cycle 1) and base-level
    alignments for haplotype-aware overlap filtration (cycle 2); also used to align
    corrected reads to references for error-rate quantification
  repo: https://github.com/lh3/minimap2
- name: Racon
  role: Comparison baseline / state-of-the-art consensus-based corrector benchmarked
    against VeChat
  repo: https://github.com/lbcb-sci/racon
- name: yacrd
  role: Chimeric-read detection preprocessing using all-against-all read mapping coverage
    pile-ups
  repo: https://github.com/natir/yacrd
- name: fpa
  role: Filtering of all-against-all read mapping outputs (internal match, containment,
    dovetails, overlap length) prior to correction
  repo: https://github.com/natir/fpa
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-34381-8
    title: 'VeChat: correcting errors in long reads using variation graphs'
  source_content_hash: sha256:3a48f612be1cd9779ddbecb2c119ba33b8c672aa70ef5f89115da851d3b0fff8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HaploKit/vechat
  tools:
  - Minimap2
  - PBSIM2
  - Racon
  - VeChat
  - fpa
  - minimap2
  - yacrd
  edam_operation: http://edamontology.org/operation_3184
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# variation-graph-based-error-correction

## Summary

Haplotype-aware self-correction of noisy long reads (PacBio CLR or ONT) using variation graphs instead of consensus templates, so that true low-frequency haplotype variants are not masked as errors. Implemented by VeChat in a two-cycle workflow built on Minimap2 overlaps and POA graph construction.

## When to use

Use when correcting noisy long reads from mixed samples (metagenomes, diploid/triploid/tetraploid or polyploid genomes) where consensus-template correctors mask low-frequency haplotype variants, and lower corrected error rates are the priority.

## When NOT to use

- Minimizing compute is the goal: on large datasets VeChat has longer runtimes and higher peak memory usage than competitors (e.g. Daccord achieved the lowest error rate on the human gut microbiome dataset while VeChat only matched read accuracy).
- Input reads are already corrected or the goal is polishing an assembled contig/consensus — Racon is the consensus module for correcting raw contigs, and VeChat targets raw long-read self-correction.
- The study scope is haplotype-aware correction; exploring optimal preprocessing/error-correction/assembly pipeline combinations for polyploid, cancer or metagenomic settings was explicitly left as future work.

## Inputs

- Raw noisy long reads in FASTA/FASTQ (optionally gzip-compressed, e.g. reads.fq.gz) from PacBio CLR or Oxford Nanopore sequencers
- Simulated read sets (e.g. PBSIM2 PacBio CLR reads from triploid E. coli strains) at varying coverage (10x-50x per haplotype) or input error rates (5%-15%)
- Small test file example/reads.fq.gz bundled with the VeChat repository

## Outputs

- Corrected full-length reads in FASTA (e.g. reads.corrected.fa)
- Corrected-error-rate summaries (e.g. corrected-read error rate per condition, tabulated in CSV such as vechat_triploid_error_rates.csv) after aligning corrected reads to reference strains with minimap2

## How to apply

Run VeChat in two cycles: cycle 1 computes minimizer-based all-vs-all overlaps with Minimap2, builds a read alignment pile, divides it into windows, constructs a variation graph via partial order alignment (POA), and iteratively prunes spurious nodes/edges — using a frequent itemset model over read coverage, sequencing errors and character co-occurrence — repeating pruning and re-alignment until convergence; the optimal alignment path of each target subread gives corrected subreads which are concatenated into a pre-corrected full-length read. Cycle 2 recomputes all-vs-all overlaps plus base-level alignments enabling haplotype-aware read overlap filtration, then derives haplotype-aware consensus sequences from the variation graphs with a dynamic programming algorithm (without re-pruning), and concatenates them per target read to produce the final output. Set the platform flag to match the data (--platform pb for PacBio, e.g. `vechat reads.fq.gz --platform pb -o reads.corrected.fa`), optionally multi-thread with -t, and keep default graph-pruning parameters (-d 0.2 minimum confidence, -s 0.2 minimum support) so the graph retains low-frequency haplotype variants; --min-confidence and --min-support control which edges survive pruning. Rationale: graph-based reference systems capture variants of lower frequencies, unlike consensus templates which mistake them for errors.

## Related tools

- **VeChat** (Haplotype-aware self-correction tool: builds ad-hoc variation graphs from raw reads, prunes error-induced nodes/edges, derives haplotype-aware consensus sequences) — https://github.com/HaploKit/vechat
- **minimap2** (Computes minimizer-based all-vs-all read overlaps (cycle 1) and base-level alignments for haplotype-aware overlap filtration (cycle 2); also used to align corrected reads to references for error-rate quantification) — https://github.com/lh3/minimap2
- **Racon** (Comparison baseline / state-of-the-art consensus-based corrector benchmarked against VeChat) — https://github.com/lbcb-sci/racon
- **yacrd** (Chimeric-read detection preprocessing using all-against-all read mapping coverage pile-ups) — https://github.com/natir/yacrd
- **fpa** (Filtering of all-against-all read mapping outputs (internal match, containment, dovetails, overlap length) prior to correction) — https://github.com/natir/fpa

## Examples

```
vechat reads.fq.gz --platform pb -o reads.corrected.fa
```

## Evaluation signals

- Corrected-read error rate should drop markedly versus raw input; on simulated PacBio CLR triploid E. coli reads the corrected error rate fell from 0.311% at 10x to 0.017% at 50x coverage per haplotype, and from 0.091% at 15% to 0.009% at 5% input error rate.
- Corrected error rates on triploid PacBio CLR data should be roughly 9-26 times lower than state-of-the-art correction (14-30x diploid, 4-11x tetraploid).
- Reported QV/switch-error metrics: approximately 1.5 times lower error rate and 1.2 times lower switch error than CONSENT on HG002.
- Default pruning parameters (-d 0.2, -s 0.2) must be retained (or consciously varied) — pruning too aggressively masks low-frequency haplotype variants, which is the failure mode VeChat is designed to avoid.
- Sanity check the install by running VeChat on the bundled example/reads.fq.gz and confirming a corrected FASTA is produced.

## Limitations

- VeChat does not win in computational resources: on large datasets it experiences longer runtimes and higher peak memory usage.
- It uses off-the-shelf approaches (e.g. edit-distance-based and sequence-to-graph alignments) without exploiting all their features, causing unnecessary overheads that could be avoided.
- Exploration of optimal preprocessing/error-correction/assembly pipeline combinations for polyploid, cancer, or metagenomic settings is beyond the scope of the study and requires further efforts.

## Evidence

- [results] two-cycle variation graph workflow: "step 1 computes minimizer based all-vs-all overlaps, for which we employ Minimap229"
- [results] graph pruning mechanism: "we make use of a frequent itemset model that involves read coverage, sequencing errors and co-occurrence of characters in reads"
- [abstract] rationale for graphs over consensus: "Unlike consensus sequence templates, graph-based reference systems are not affected by such biases, so do not mistakenly mask true variants as errors"
- [methods] platform-specific invocation: "One can test the program using the small sequencing read file `example/reads.fq.gz`"
- [methods] pruning thresholds: "minimum confidence for keeping edges in the graph (default: 0.2)"
- [discussion] performance limitation: "VeChat does not win the competition in terms of computational resources. In particular on large datasets, VeChat experiences longer runtimes and higher peak memory usage"
