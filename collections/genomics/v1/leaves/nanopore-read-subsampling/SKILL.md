---
name: nanopore-read-subsampling
description: Use when you need to determine how many Nanopore amplicon reads are sufficient
  for an accurate majority-vote consensus, by sweeping subsample sizes and measuring
  consensus error against a higher-accuracy reference consensus.
when_to_use_negative:
- Your goal is sample demultiplexing or binning of raw MinION reads rather than characterizing
  consensus accuracy versus depth — use minibar.py's demultiplexing options instead
- You have no independent higher-accuracy reference (e.g. Illumina-derived consensus)
  against which to compute edit distance, so subsampled consensus error cannot be
  calibrated
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: MiniBar
  role: Demultiplexes dual-indexed MinION reads, bins them into per-sample files (-F,
    -P prefix) and trims barcodes and primers from sequence ends (-T) to produce the
    input FASTQ
  repo: https://github.com/calacademy-research/minibar
- name: Python 2.7
  role: Hosts the subsampling, majority-vote consensus building, and aggregation of
    error rates across coverage levels
- name: Edlib
  role: Computes the edit distance of each subsampled consensus to the Illumina-derived
    reference consensus
  repo: https://github.com/Martinsos/edlib
- name: Illumina
  role: Short-read platform whose consensus provides the reference for measuring Nanopore
    consensus error
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giz006
    title: Nanopore sequencing of long ribosomal DNA amplicons enables portable and
      simple biodiversity assessments with high phylogenetic resolution across broad
      taxonomic scale
  source_content_hash: sha256:bd6cbb853bf6d6f34d503e275a319dfb60a0611272b6d74b93937d0e664952d5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/calacademy-research/minibar
  tools:
  - Edlib
  - Illumina
  - MiniBar
  - Python 2.7
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# nanopore-read-subsampling

## Summary

Randomly subsamples demultiplexed Nanopore long-read FASTQ reads across a range of assembly coverages (10–800 reads) to quantify how consensus sequence error depends on read depth, identifying the coverage at which majority-vote consensus accuracy plateaus.

## When to use

Use when you need to determine how many Nanopore amplicon reads are sufficient for an accurate majority-vote consensus, by sweeping subsample sizes and measuring consensus error against a higher-accuracy reference consensus.

## When NOT to use

- Your goal is sample demultiplexing or binning of raw MinION reads rather than characterizing consensus accuracy versus depth — use minibar.py's demultiplexing options instead
- You have no independent higher-accuracy reference (e.g. Illumina-derived consensus) against which to compute edit distance, so subsampled consensus error cannot be calibrated

## Inputs

- Demultiplexed, barcode- and primer-trimmed FASTQ for a single specimen (MiniBar output with -F -P Peperomia_ and -T)
- Illumina-derived reference consensus sequence for the same amplicon

## Outputs

- consensus_accuracy_vs_coverage.csv with mean and spread of percent consensus error per coverage level (10 independent replicates x 18 coverage levels from 10 to 800 reads)

## How to apply

Start from a single specimen's barcode- and primer-trimmed FASTQ produced by a MiniBar run using the -F binning and -P prefix options together with the -T trim option (e.g. prefix 'Peperomia_'). In Python, draw 10 independent random subsamples without replacement at each of 18 coverage levels spanning 10 to 800 sequences. For each subsample, build a majority-vote consensus from the multiple read alignment at each base position. Compute the edit distance of every consensus to an Illumina-derived reference consensus with the Edlib library, converting distances to percent error relative to consensus length. Aggregate the 10 replicate error rates per coverage level into a table of mean (and spread of) consensus error versus number of reads, written to consensus_accuracy_vs_coverage.csv.

## Related tools

- **MiniBar** (Demultiplexes dual-indexed MinION reads, bins them into per-sample files (-F, -P prefix) and trims barcodes and primers from sequence ends (-T) to produce the input FASTQ) — https://github.com/calacademy-research/minibar
- **Python 2.7** (Hosts the subsampling, majority-vote consensus building, and aggregation of error rates across coverage levels)
- **Edlib** (Computes the edit distance of each subsampled consensus to the Illumina-derived reference consensus) — https://github.com/Martinsos/edlib
- **Illumina** (Short-read platform whose consensus provides the reference for measuring Nanopore consensus error)

## Examples

```
minibar.py -T -F -P Peperomia_ IndexCombinationPeperomonia.txt my_sequences.fastq
```

## Evaluation signals

- Error rate must decrease with depth, from roughly 0.257% at 10 reads to about 0.031% near 300 reads before plateauing
- Subsampling must be without replacement, with 10 independent replicates at each of 18 coverage levels spanning 10 to 800 sequences
- Each consensus error is an edit distance (Edlib) normalized to percent of consensus length relative to the Illumina-derived consensus
- Output CSV must contain mean and spread of error rate per coverage level (consensus_accuracy_vs_coverage.csv)

## Limitations

- Optimal accuracy (~0.031%) requires very high coverage (about 300-fold), which increases sequencing cost; accuracy plateaus beyond ~300 reads
- Low-coverage consensus (10 reads) already incurs about 0.257% error, which may be inadequate for fine-scale taxonomic resolution
- Error estimation depends on the quality of the Illumina-derived reference consensus; biases in that reference propagate to the error curve

## Evidence

- [methods] Consensus accuracy is already good at low coverage: [evidence span withheld — anchor status: not_found]
- [methods] Coverage sweep workflow: [evidence span withheld — anchor status: not_found]
- [results] Validation trend: [evidence span withheld — anchor status: not_found]
- [readme] MiniBar trimming option: "-T trims barcode and primer from each end of the sequence, then outputs record"
- [readme] Edlib purpose: [evidence span withheld — anchor status: not_found]
