---
name: consensus-accuracy-versus-coverage-analysis
description: Use when you need to determine the number of Nanopore reads required
  for an accurate long-amplicon consensus (e.g. rDNA barcodes), by plotting consensus
  error rate versus assembly coverage depth.
when_to_use_negative:
- Your goal is demultiplexing or primer/barcode trimming itself rather than coverage–accuracy
  characterization — use MiniBar directly for that step
- The reads are short Illumina reads rather than long Nanopore amplicon reads; the
  majority-vote consensus rationale targets Nanopore's per-read error profile
- You have no independent high-accuracy reference consensus (e.g. Illumina-derived)
  to measure consensus error against
edam_operation: http://edamontology.org/operation_3184
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: MiniBar
  role: Demultiplexes dual-indexed Nanopore reads and, with -F and -T, bins them into
    per-sample files with barcodes and primers trimmed, providing the cleaned fastq
    input for subsampling
  repo: https://github.com/calacademy-research/minibar
- name: Edlib
  role: Python library used to compute edit distances between each Nanopore consensus
    and the Illumina-derived reference consensus
  repo: https://github.com/Martinsos/edlib
- name: Python 2.7
  role: Language in which the subsampling, consensus building, and error aggregation
    script is written (compatible with Python 3)
- name: Illumina
  role: Short-read sequencing platform whose consensus sequences serve as the high-accuracy
    reference for measuring Nanopore consensus error
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giz006
    title: Nanopore sequencing of long ribosomal DNA amplicons enables portable and
      simple biodiversity assessments with high phylogenetic resolution across broad
      taxonomic scale
  source_content_hash: sha256:060b6085d2dfdf753deb434c00aec56e98745f8bb4508491db6bbbe6108a3bb6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/calacademy-research/minibar
  tools:
  - Edlib
  - Illumina
  - MiniBar
  - Python 2.7
  edam_operation: http://edamontology.org/operation_3184
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# consensus-accuracy-versus-coverage-analysis

## Summary

Quantifies how Nanopore consensus sequence accuracy improves with sequencing depth by subsampling a demultiplexed, barcode- and primer-trimmed fastq across coverage levels (10–800 reads), building majority-vote consensus sequences, and scoring them against an Illumina-derived reference consensus with Edlib edit distance.

## When to use

Use when you need to determine the number of Nanopore reads required for an accurate long-amplicon consensus (e.g. rDNA barcodes), by plotting consensus error rate versus assembly coverage depth.

## When NOT to use

- Your goal is demultiplexing or primer/barcode trimming itself rather than coverage–accuracy characterization — use MiniBar directly for that step
- The reads are short Illumina reads rather than long Nanopore amplicon reads; the majority-vote consensus rationale targets Nanopore's per-read error profile
- You have no independent high-accuracy reference consensus (e.g. Illumina-derived) to measure consensus error against

## Inputs

- Demultiplexed, barcode- and primer-trimmed Nanopore fastq for a single specimen (MiniBar -F -P Peperomia_ with -T output)
- Illumina-derived reference consensus sequence for the same amplicon

## Outputs

- consensus_accuracy_vs_coverage.csv: mean (and spread of) consensus percent error versus number of reads, aggregated over 10 subsampling replicates per coverage level

## How to apply

Start from the MiniBar-demultiplexed output for a single specimen, produced with -F binning (e.g. -P Peperomia_ prefix) and the -T option to trim barcode and primer sequence from read ends. In Python, draw 10 independent uniform subsamples without replacement at each of 18 coverage levels spanning 10 to 800 reads. For each subsample, assemble a majority-vote consensus across the multiple read alignment base by base, then compute the Edlib edit distance of each consensus to the Illumina-derived reference consensus and express it as percent error relative to consensus length. Aggregate the 10 replicates per coverage level into a table of mean and spread of consensus error versus read count, written to consensus_accuracy_vs_coverage.csv. The rationale is that random Nanopore errors average out with depth: the paper found error falling from about 0.257% at 10-fold coverage to 0.128% at 20-fold and an optimum of about 0.031% near 300-fold before plateauing, so ~300 reads per specimen suffices.

## Related tools

- **MiniBar** (Demultiplexes dual-indexed Nanopore reads and, with -F and -T, bins them into per-sample files with barcodes and primers trimmed, providing the cleaned fastq input for subsampling) — https://github.com/calacademy-research/minibar
- **Edlib** (Python library used to compute edit distances between each Nanopore consensus and the Illumina-derived reference consensus) — https://github.com/Martinsos/edlib
- **Python 2.7** (Language in which the subsampling, consensus building, and error aggregation script is written (compatible with Python 3))
- **Illumina** (Short-read sequencing platform whose consensus sequences serve as the high-accuracy reference for measuring Nanopore consensus error)

## Examples

```
./minibar.py -T -F -P Peperomia_ Demultiplex.txt my_sequences.fastq
```

## Evaluation signals

- The error-versus-coverage trend reproduces the reported pattern: decreasing from roughly 0.257% at 10 reads to about 0.128% at 20 reads, reaching an optimum of about 0.031% near 300 reads, then plateauing
- consensus_accuracy_vs_coverage.csv exists with 18 coverage levels and 10 replicate error values aggregated per level
- Percent errors are computed as Edlib edit distance divided by consensus length and expressed as a percentage
- Subsampling is done without replacement, with 10 independent random draws per coverage level (spread across replicates should be modest and shrink with depth)

## Limitations

- Consensus error is measured against an Illumina-derived consensus, so accuracy estimates are bounded by Illumina's own errors
- The error curve plateaus near 300-fold coverage; sequencing far beyond ~300 reads per specimen yields no further consensus-accuracy gain
- Analysis requires a specimen with sufficient read depth (up to 800 reads) to support the full subsampling sweep

## Evidence

- [results] Consensus accuracy is already good at low coverage (0.257% average distance to Illumina consensus at 10-fold), improves significantly by 20-fold (0.128%), and is optimal at 300-fold coverage (0.031% distance).: [evidence span withheld — anchor status: not_found]
- [discussion] Other requirements: Edlib library module: "Other requirements: Edlib library module (https://github.com/Martinsos/edlib)"
- [readme] For downstream processing, you'll probably want to trim off the barcode and primer part of the sequence. This is done for you if you supply the -T trim option: "For downstream processing, you'll probably want to trim off the barcode and primer part of the sequence. This is done for you if you supply the -T trim option"
- [methods] Typically you'll want to group your records by identified sample. This can be done by using the -F option: "Typically you'll want to group your records by identified sample. This can be done by using the -F option"
- [intro] The MinION sequencer (Oxford Nanopore Technologies [ONT]) is small in size, lightweight, allows for sequencing of several Gbs of DNA with average read lengths more than 10 kb: "The MinION sequencer (Oxford Nanopore Technologies [ONT]) is small in size, lightweight, allows for sequencing of several Gbs of DNA with average read lengths more than 10 kb"
