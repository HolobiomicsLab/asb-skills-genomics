---
name: isoform-transcript-error-correction
description: Use when long PacBio Iso-Seq FLNC reads from targeted sequencing of highly
  similar gene copies must be error-corrected de novo into end-to-end transcript sequences
  at nucleotide-level precision, without a reference genome.
when_to_use_negative:
- Reads do not span the target region end-to-end — IsoCon assumes full-length reads
  (original use case is targeted gene-family Iso-Seq; other uses require reads spanning
  the region of interest)
- The goal is to definitively assign transcripts to individual gene copies or determine
  gene family size — the paper states transcriptome-only sequencing 'does not allow
  us to conclusively assign each transcript to a gene copy'
- Short-read (Illumina) data without long PacBio reads — Illumina-based correction
  (e.g., proovread) is described as unable to correct most errors in this setting
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0769
tools:
- name: IsoCon
  role: De novo error correction and statistical filtering of PacBio CCS/FLNC reads
    into transcript candidates; pipeline and stat_filter subcommands
  repo: https://github.com/ksahlin/IsoCon
- name: ICE
  role: Baseline PacBio Iso-Seq clustering tool whose transcripts are imported into
    IsoCon's stat_filter-style evaluation for comparison
- name: parasail
  role: Pairwise alignments from each read to the cluster center during the correction
    phase; also used for nucleotide-level Ensembl match counting
  repo: https://github.com/jeffdaily/parasail-python
- name: edlib
  role: Edit-distance sequence alignment used in the evaluation/comparison scripts
  repo: https://github.com/Martinsos/edlib
- name: IsoCon_Eval
  role: Snakemake pipeline and evaluation scripts for benchmarking IsoCon vs ICE on
    simulated and experimental data
  repo: https://github.com/ksahlin/IsoCon_Eval
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-018-06910-x
    title: Deciphering highly similar multigene family transcripts from Iso-Seq data
      with IsoCon
  source_content_hash: sha256:dacf512f1d1d1359f82ca9b7820be8523ae0bb68fb9a942b6223940e580dc5d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ksahlin/IsoCon
  tools:
  - ICE
  - IsoCon
  - IsoCon_Eval
  - edlib
  - parasail
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# isoform-transcript-error-correction

## Summary

De novo error correction and redundancy removal of PacBio circular consensus sequence (CCS) reads from targeted Iso-Seq data with IsoCon, yielding nucleotide-level precise transcript candidates from highly similar multigene family copies where ICE and Illumina-based correction fail.

## When to use

Use when long PacBio Iso-Seq FLNC reads from targeted sequencing of highly similar gene copies must be error-corrected de novo into end-to-end transcript sequences at nucleotide-level precision, without a reference genome.

## When NOT to use

- Reads do not span the target region end-to-end — IsoCon assumes full-length reads (original use case is targeted gene-family Iso-Seq; other uses require reads spanning the region of interest)
- The goal is to definitively assign transcripts to individual gene copies or determine gene family size — the paper states transcriptome-only sequencing 'does not allow us to conclusively assign each transcript to a gene copy'
- Short-read (Illumina) data without long PacBio reads — Illumina-based correction (e.g., proovread) is described as unable to correct most errors in this setting

## Inputs

- FLNC PacBio Iso-Seq reads as FASTQ (preferred, quality values used for statistical analysis) or FASTA
- Optional CCS BAM file passed via --ccs when using FASTA input
- External candidate transcript sets (e.g., ICE output) for stat_filter-style evaluation

## Outputs

- Predicted transcript candidates in /path/to/output/final_candidates.fa
- Reads that could not be corrected or clustered (unclustered reads output)

## How to apply

First pre-process reads with the Iso-Seq bioinformatic pipeline to remove primers, barcodes, and chimeras, then run IsoCon on the FLNC reads with 'IsoCon pipeline -fl_reads <reads.fastq> -outfolder </path/to/output>' (fastq is preferred over fasta because IsoCon uses quality values for statistical analysis). IsoCon iteratively clusters reads via nearest-neighbor graph path traversal, error-corrects half of the low-support variant positions per iteration relative to a cluster center using parasail pairwise alignments, and derives candidate transcripts whose output is final_candidates.fa. It then applies statistical filtering (stat_filter): candidate pairs are assigned reads by greedy closest-neighbor matching and tested pairwise with a one-sided Chernoff-type bound P(T >= t), removing candidates with p-value above threshold alpha (default 0.01), at most tau per iteration; candidates differing by more than 10 variant positions relative to other candidates are not statistically evaluated, and the --min_test_ratio X parameter (default 5) omits testing candidate c against t when c has X times more support. For benchmarking, external candidate sets (e.g., ICE transcripts) can be imported via IsoCon's stat_filter-style evaluation since they need not originate from IsoCon's error correction step. Correct application is judged by reproducibility on the small test dataset and by comparing shared transcripts and Ensembl exact matches against alternatives.

## Related tools

- **IsoCon** (De novo error correction and statistical filtering of PacBio CCS/FLNC reads into transcript candidates; pipeline and stat_filter subcommands) — https://github.com/ksahlin/IsoCon
- **ICE** (Baseline PacBio Iso-Seq clustering tool whose transcripts are imported into IsoCon's stat_filter-style evaluation for comparison)
- **parasail** (Pairwise alignments from each read to the cluster center during the correction phase; also used for nucleotide-level Ensembl match counting) — https://github.com/jeffdaily/parasail-python
- **edlib** (Edit-distance sequence alignment used in the evaluation/comparison scripts) — https://github.com/Martinsos/edlib
- **IsoCon_Eval** (Snakemake pipeline and evaluation scripts for benchmarking IsoCon vs ICE on simulated and experimental data) — https://github.com/ksahlin/IsoCon_Eval

## Examples

```
IsoCon pipeline -fl_reads <reads.fastq> -outfolder </path/to/output>
```

## Evaluation signals

- IsoCon runs successfully on the small test dataset at https://github.com/ksahlin/IsoCon/tree/master/test/data and produces final_candidates.fa
- Benchmark invariant: ~121 transcripts shared between the two testes samples (~32%) for IsoCon vs <2% for ICE and original reads, and 11% for Illumina-corrected reads
- Benchmark invariant: 21 exact nucleotide-level matches to the 61 unique Ensembl coding sequences of the nine ampliconic families for IsoCon vs only 8 for ICE
- Statistical filtering respects alpha = 0.01 default, removes at most tau highest-p-value candidates per iteration, and skips candidates with >10 variant positions or when min_test_ratio (default 5) applies
- Simulated-data evaluation via IsoCon_Eval snakemake pipeline shows substantially higher precision and recall than ICE across read counts (20–12500), abundance models, and runs

## Limitations

- IsoCon has no functionality to distinguish alleles from distinct gene copies; further downstream analysis is needed for allele-vs-copy assignment
- Transcriptome-only sequencing does not provide a definitive answer to the size of each gene family or conclusive gene-copy assignment
- Novel gene copies are difficult to distinguish from splice variants or RNA-editing
- RT-PCR error sources cannot be excluded; replicate cDNA libraries and further validation are still needed
- Very low-abundance transcripts might not be captured since deep PacBio sequencing can be cost-prohibitive
- IsoCon assumes reads have been pre-processed with the Iso-Seq pipeline (primers, barcodes, chimeras removed)

## Evidence

- [abstract] We present IsoCon, a high-precision method using long PacBio Iso-Seq reads to tackle this challenge.: "We present IsoCon, a high-precision method using long PacBio Iso-Seq reads to tackle this challenge."
- [intro] IsoCon uses a cautiously iterative process to correct obvious errors, without overcorrecting rare variants.: "IsoCon uses a cautiously iterative process to correct obvious errors, without overcorrecting rare variants."
- [readme] IsoCon pipeline usage: "IsoCon pipeline -fl_reads <reads.fastq> -outfolder </path/to/output>"
- [discussion] Candidate filtering threshold: "Then, we identify the candidates with pi greater than a significance threshold α. This α is a parameter to our algorithm, set by default to 0.01."
- [discussion] Variant position threshold: "Candidate transcripts that have more than a threshold of variant positions (default of 10) relative to all other candidate transcripts are not statistically evaluated"
- [results] Benchmark outcome: [evidence span withheld — anchor status: not_found]
- [discussion] Preprocessing requirement: "IsoCon assumes the reads have been pre-processed with the Iso-Seq bioinformatic pipeline to remove primers, barcodes, and reads that are chimeras"
