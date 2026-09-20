---
name: long-read-corrector-benchmarking
description: Use when evaluating how a long-read corrector's performance changes with
  input error rate or ploidy, e.g., simulated PacBio CLR triploid E. coli reads at
  15% vs. 5% error.
when_to_use_negative:
- You need to evaluate computational resource usage — the paper notes VeChat 'does
  not win the competition in terms of computational resources', with longer runtimes
  and higher peak memory on large datasets
- You are optimizing preprocessing/error-correction/assembly pipeline combinations
  for polyploid, cancer, or metagenomic settings — the article states this exploration
  is beyond the study's scope and requires further efforts
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: VeChat
  role: Haplotype-aware long-read error corrector under benchmark; corrects PacBio
    CLR reads using variation graphs (--platform pb)
  repo: https://github.com/HaploKit/vechat
- name: PBSIM2
  role: Simulates PacBio CLR reads from the triploid E. coli reference at controlled
    error rates (15% and 5%)
- name: Minimap2
  role: Aligns corrected reads back to the triploid reference to compute corrected-read
    error rates; also used in VeChat cycle 1 for minimizer-based all-vs-all overlaps
  repo: https://github.com/lh3/minimap2
- name: Racon
  role: State-of-the-art long-read corrector/consensus tool providing comparison points
    for the error-correction benchmark
  repo: https://github.com/lbcb-sci/racon
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-34381-8
    title: 'VeChat: correcting errors in long reads using variation graphs'
  source_content_hash: sha256:5644d5ea72fd7bcc0a7172f9cfb19f284b4a54b4b193059ebbdb17f867a88aa8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HaploKit/vechat
  tools:
  - Minimap2
  - PBSIM2
  - Racon
  - VeChat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# long-read-corrector-benchmarking

## Summary

Benchmarks long-read error correction (e.g., VeChat on simulated PacBio CLR reads from diploid/triploid/tetraploid genomes) by correcting raw read sets under varying sequencing error rates, realigning corrected reads with minimap2, and computing corrected-read error rates to quantify performance.

## When to use

Use when evaluating how a long-read corrector's performance changes with input error rate or ploidy, e.g., simulated PacBio CLR triploid E. coli reads at 15% vs. 5% error.

## When NOT to use

- You need to evaluate computational resource usage — the paper notes VeChat 'does not win the competition in terms of computational resources', with longer runtimes and higher peak memory on large datasets
- You are optimizing preprocessing/error-correction/assembly pipeline combinations for polyploid, cancer, or metagenomic settings — the article states this exploration is beyond the study's scope and requires further efforts

## Inputs

- Simulated PacBio CLR long reads (FASTQ, e.g., reads.fq.gz) from a triploid E. coli reference (three E. coli strains) at 30x coverage per haplotype
- Triploid E. coli reference genome (FASTA) for read simulation (PBSIM2) and read alignment (minimap2)
- Read sets simulated at multiple sequencing error rates (e.g., 15% and 5%)

## Outputs

- Corrected reads (FASTA, e.g., reads.corrected.fa) per error-rate condition
- minimap2 alignments of corrected reads against the triploid reference (SAM/PAF)
- Comparison table of corrected-read error rates per condition (e.g., vechat_triploid_error_rates.csv)

## How to apply

Simulate long reads from a multi-strain (e.g., triploid E. coli) reference with PBSIM2 at two error rates (15% and 5%), at fixed coverage (30x per haplotype). Correct each read set with the corrector (VeChat: `vechat reads.fq.gz --platform pb -o reads.corrected.fa`, multi-threaded with -t). Align each corrected FASTA back to the triploid reference with minimap2 (PacBio CLR preset, `-ax map-pb`) and compute the overall corrected-read error rate from the alignments. Tabulate corrected error rates per input-error condition side by side; on the VeChat triploid benchmark, corrected error drops from ~0.091% (15% input error) to ~0.009% (5% input error).

## Related tools

- **VeChat** (Haplotype-aware long-read error corrector under benchmark; corrects PacBio CLR reads using variation graphs (--platform pb)) — https://github.com/HaploKit/vechat
- **PBSIM2** (Simulates PacBio CLR reads from the triploid E. coli reference at controlled error rates (15% and 5%))
- **Minimap2** (Aligns corrected reads back to the triploid reference to compute corrected-read error rates; also used in VeChat cycle 1 for minimizer-based all-vs-all overlaps) — https://github.com/lh3/minimap2
- **Racon** (State-of-the-art long-read corrector/consensus tool providing comparison points for the error-correction benchmark) — https://github.com/lbcb-sci/racon

## Examples

```
vechat reads.fq.gz --platform pb -o reads.corrected.fa; minimap2 -ax map-pb triploid_ref.fa reads.corrected.fa > corrected.sam
```

## Evaluation signals

- Corrected-read error rate at 5% input error reproduces the reported 0.009% (with ~0.091% at 15% input error) on the simulated triploid PacBio CLR dataset
- Error-rate reduction factor on triploid genomes falls within the reported ~9–26x range versus state-of-the-art correctors on PacBio CLR reads
- Corrected error rate monotonically improves as the simulated input error rate decreases (15% → 5%)
- Both corrected FASTA outputs realign to the triploid reference with minimap2 map-pb alignments covering the simulated read set
- Results table (e.g., vechat_triploid_error_rates.csv) contains side-by-side rows for the 15% and 5% input-error conditions

## Limitations

- VeChat experiences longer runtimes and higher peak memory usage than competitors, particularly on large datasets
- VeChat uses off-the-shelf approaches (e.g., edit-distance and sequence-to-graph alignments) without exploiting all their features, causing unnecessary overheads that more efficient routines could avoid
- Optimal combinations of preprocessing, error correction, and assembly for polyploid, cancer, or metagenomic settings remain unexplored and require further work

## Evidence

- [results] On simulated PacBio CLR reads, VeChat achieves approximately 14–30, 9–26 and 4–11 times lower error rates on diploid, triploid and tetraploid genomes, respectively: "VeChat achieves approximately 14–30, 9–26 and 4–11 times lower error rates on diploid, triploid and tetraploid genomes, respectively."
- [results] Cycle 1 overlap computation with minimap2: "step 1 computes minimizer based all-vs-all overlaps, for which we employ Minimap2"
- [methods] VeChat example test invocation: "One can test the program using the small sequencing read file `example/reads.fq.gz`"
- [readme] Minimap2 PacBio CLR alignment preset: [evidence span withheld — anchor status: external_doc]
- [abstract] Headline error-reduction claim: "long reads corrected by VeChat contain 4 to 15 (Pacific Biosciences) and 1 to 10 times (Oxford Nanopore Technologies) less errors than when being corrected by state of the art approaches"
