---
name: amplicon-consensus-sequence-assembly
description: Use when you have demultiplexed, barcode- and primer-trimmed Nanopore
  amplicon FASTQ for one specimen and need to quantify how consensus accuracy scales
  with read depth or pick an optimal assembly coverage.
when_to_use_negative:
- Input reads have not yet been demultiplexed or have barcodes/primers still attached
  — run MiniBar (-F, -P, -T) first
- The goal is de novo genome-scale assembly or variant calling rather than majority-vote
  consensus of a single amplicon
- Analysis of bulk-community (mock or environmental) samples where the article notes
  long rDNA amplicon sequencing is not yet reliable due to undetermined taxonomic
  bias
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0091
tools:
- name: MiniBar
  role: Demultiplexes dual-indexed MinION reads and trims barcodes/primers (-T) to
    produce per-specimen FASTQ inputs for consensus assembly
  repo: https://github.com/calacademy-research/minibar
- name: Edlib
  role: Python library used to compute the edit distance between each Nanopore consensus
    and the Illumina-derived reference consensus
  repo: https://github.com/Martinsos/edlib
- name: Python 2.7
  role: Language for subsampling reads, calling majority-vote consensus bases, and
    aggregating error rates
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giz006
    title: Nanopore sequencing of long ribosomal DNA amplicons enables portable and
      simple biodiversity assessments with high phylogenetic resolution across broad
      taxonomic scale
  source_content_hash: sha256:cfa94407d3474ec02a97298fefbf491211ee87d6563a4e84033f6970300688d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/calacademy-research/minibar
  tools:
  - Edlib
  - MiniBar
  - Python 2.7
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# amplicon-consensus-sequence-assembly

## Summary

Builds majority-vote consensus sequences from Nanopore long-rDNA amplicon reads and quantifies consensus accuracy versus assembly coverage. It replaces single-read genotyping with depth-supported consensus calling, benchmarked by edit distance to an Illumina-derived reference consensus.

## When to use

Use when you have demultiplexed, barcode- and primer-trimmed Nanopore amplicon FASTQ for one specimen and need to quantify how consensus accuracy scales with read depth or pick an optimal assembly coverage.

## When NOT to use

- Input reads have not yet been demultiplexed or have barcodes/primers still attached — run MiniBar (-F, -P, -T) first
- The goal is de novo genome-scale assembly or variant calling rather than majority-vote consensus of a single amplicon
- Analysis of bulk-community (mock or environmental) samples where the article notes long rDNA amplicon sequencing is not yet reliable due to undetermined taxonomic bias

## Inputs

- Demultiplexed, barcode- and primer-trimmed FASTQ for a single specimen (MiniBar output with -F -P Peperomia_ and -T)
- Illumina-derived reference consensus sequence for the same amplicon
- Coverage-level specification (e.g. 18 levels from 10 to 800 reads, 10 replicates each)

## Outputs

- Majority-vote consensus sequence(s) per subsample (consensus FASTA/sequence per replicate)
- consensus_accuracy_vs_coverage.csv: mean and spread of consensus error (%) versus number of reads per coverage level

## How to apply

First demultiplex and trim with MiniBar (run minibar.py with -F -P <prefix> and the -T trim option) so each specimen is a clean FASTQ of barcode/primer-free rDNA reads. In Python 2.7 (or 3), randomly draw 10 independent uniform subsamples without replacement at each of 18 coverage levels spanning 10 to 800 reads. For each subsample, build a majority-vote consensus from the multiple read alignment at every base position. Compute each consensus's edit distance to the Illumina-derived reference consensus with the Edlib library, convert to percent error relative to consensus length, and aggregate the 10 replicate error rates per coverage level into consensus_accuracy_vs_coverage.csv. Accuracy drops from ~0.257% error at 10-fold to ~0.128% at 20-fold and is optimal (~0.031%) near 300-fold, plateauing beyond that.

## Related tools

- **MiniBar** (Demultiplexes dual-indexed MinION reads and trims barcodes/primers (-T) to produce per-specimen FASTQ inputs for consensus assembly) — https://github.com/calacademy-research/minibar
- **Edlib** (Python library used to compute the edit distance between each Nanopore consensus and the Illumina-derived reference consensus) — https://github.com/Martinsos/edlib
- **Python 2.7** (Language for subsampling reads, calling majority-vote consensus bases, and aggregating error rates)

## Examples

```
./minibar.py -T -F -P Peperomia_ IndexCombinationPeperomonia.txt PeperomiaTestSet.fasta >PeperomiaTestSet_SampleIDs.fa
```

## Evaluation signals

- Consensus error must decrease with depth: ~0.257% at 10 reads, ~0.128% at 20 reads, ~0.031% near 300 reads, then plateau
- Each coverage level must contain exactly 10 independent uniform subsamples drawn without replacement before aggregation
- Edit distances must be reported as percent error relative to consensus length, not raw counts
- Output CSV must contain columns for read number, mean consensus error, and spread across the 10 replicates
- MiniBar run summary should show most reads identified as HH (both barcode+primer hits) before consensus assembly

## Limitations

- Coverage above ~300 reads yields no meaningful accuracy gain, so very high depth wastes sequencing and compute
- Accuracy is benchmarked against an Illumina-derived consensus, so systematic errors shared by both platforms are invisible
- Further research into the sources of the observed bias is required before long rDNA amplicon sequencing can be utilized as a reliable resource for the analysis of bulk samples

## Evidence

- [intro] We explore the error profile of Nanopore consensus sequences and introduce MiniBar, a new software to demultiplex dual indexed Nanopore amplicon sequences: "We explore the error profile of Nanopore consensus sequences and introduce MiniBar, a new software to demultiplex dual indexed Nanopore amplicon sequences"
- [discussion] Other requirements: Edlib library module (https://github.com/Martinsos/edlib): "Other requirements: Edlib library module (https://github.com/Martinsos/edlib)"
- [discussion] Programming language: Python 2.7 (but can be run in Python 3): "Programming language: Python 2.7 (but can be run in Python 3)"
- [readme] -T trims barcode and primer from each end of the sequence, then outputs record: "-T trims barcode and primer from each end of the sequence, then outputs record"
